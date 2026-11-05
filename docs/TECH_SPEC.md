# Tallyo — Specifica tecnica (v0.1)

Riferimento prodotto: [`PRODUCT_VISION.md`](PRODUCT_VISION.md). Questo documento copre il **core (Fase 0)**: app iOS + Android, banca collegata, contanti, ricorrenze, profili dipendente/P.IVA, safe-to-spend.

---

## 1. Stack

| Livello | Scelta | Perché |
|---|---|---|
| App mobile | **Expo (React Native) + TypeScript** | Un solo codice per iOS e Android; build e pubblicazione sugli store con EAS; widget e moduli nativi possibili quando servono |
| Logica di calcolo | **Pacchetto TypeScript condiviso** (`packages/core`) | Lo stesso motore gira nell'app (calcolo offline immediato) e nel backend (job notturni, Lyo). Puro, senza dipendenze, testabile al 100% |
| Backend | **Supabase** (Postgres + Auth + Edge Functions + pg_cron), region **UE (Francoforte)** | Postgres con Row Level Security per l'isolamento tra utenti; edge functions in TypeScript; costo ~0 in fase personale |
| Open banking | Provider PSD2 (vedi §2) chiamato **solo dal backend** | Le chiavi del provider non devono mai stare nell'app |
| AI | API Claude dal backend: modello piccolo (Haiku) per parsing/categorie, modello medio (Sonnet) per Lyo | Costi bassi sulle operazioni frequenti, qualità alta sulla conversazione |
| Voce | Speech-to-text di sistema (iOS/Android), on-device, italiano | Gratis, privato, veloce |

Struttura repository (monorepo `pnpm`):

```
tallyo/
├─ apps/mobile/            # Expo app
├─ packages/core/          # motore: safe-to-spend, ricorrenze, tasca, tasse
├─ supabase/
│  ├─ migrations/          # schema SQL + RLS
│  └─ functions/           # bank-*, ingest-*, lyo-*
└─ docs/
```

> Alternativa valutata: Flutter. Scartata perché obbligherebbe a scrivere il motore in Dart (app) e TypeScript (backend), oppure a duplicarlo.

---

## 2. Collegamento bancario

### 2.1 Provider

| Provider | Note |
|---|---|
| **Enable Banking** | Copertura ampia in Italia; prevede una modalità per collegare **i propri conti** senza licenza AISP: ideale per la Fase 0. **Candidato principale.** |
| Tink (Visa) | Molto solido, orientato a contratti business; ottimo per il lancio pubblico |
| Salt Edge / Yapily | Alternative valide per la fase di lancio |
| GoCardless Bank Account Data | Chiuso ai nuovi clienti nel 2025 (da riverificare) |

**Primo task tecnico:** verificare che HYPE sia presente nella lista banche (ASPSP) del provider e quali dati espone (saldo, movimenti, descrizioni, data valuta, codici transazione). Se HYPE non fosse coperto: import CSV/estratto conto come ripiego temporaneo, mantenendo la stessa pipeline di ingestione.

Il connettore è dietro un'interfaccia (`BankProvider`) per poter cambiare provider senza toccare il resto.

### 2.2 Flusso di collegamento

```
App                      Edge Function                Provider / Banca
 │ "Collega HYPE"  ──►  bank-auth-start  ──────────►  crea sessione di autorizzazione
 │ ◄── URL banca ──────────────┘
 │ apre browser → login HYPE + SCA (app HYPE)
 │ ◄── redirect tallyo://bank-callback?code=…
 │ ───────────────►  bank-auth-callback ─────────►  scambia code → session/consent
 │                         └─ salva bank_connection (consent_expires_at)
 │                         └─ primo sync (storico 90–365 gg secondo banca)
```

### 2.3 Sincronizzazione

- **Sync automatico** con pg_cron, massimo **4 volte al giorno per conto** (limite PSD2 per gli accessi senza l'utente presente).
- **Sync all'apertura dell'app** (utente presente → non conta nel limite), con debounce.
- **Deduplicazione:** chiave = `provider_transaction_id`, oppure hash di (conto, data, importo, descrizione normalizzata) se l'id manca; gestione dei movimenti `pending` → `booked`.
- **Scadenza consenso:** il consenso va riconfermato periodicamente (tipicamente 90–180 giorni, dipende da banca e provider). Notifica 7 giorni prima + banner in home; il safe-to-spend mostra "dati aggiornati al …" se il sync è vecchio.

---

## 3. Modello dati (essenziale)

Tutti gli importi in **centesimi interi** (`bigint`), valuta `EUR` di default. Ogni tabella ha `user_id` con **RLS** `user_id = auth.uid()`.

| Tabella | Campi chiave |
|---|---|
| `profiles` | `income_mode` (`salary` \| `irregular`), `safety_buffer_cents`, `tax_profile_id` |
| `accounts` | `type` (`bank` \| `cash` \| `meal_voucher`), `name`, `bank_connection_id`, `current_balance_cents`, `balance_updated_at` |
| `bank_connections` | `provider`, `aspsp_name`, `provider_session_id` (cifrato), `consent_expires_at`, `status` |
| `transactions` | `account_id`, `booked_at`, `amount_cents` (±), `raw_description`, `merchant_id`, `category_id`, `kind` (`expense` \| `income` \| `transfer` \| `atm_withdrawal` \| `cash_adjustment`), `source` (`bank` \| `voice` \| `manual` \| `receipt`), `status`, `recurrence_id`, `is_business_income` |
| `merchants` | `display_name`, `patterns[]`, `default_category_id`, `logo_url` |
| `categories` | `name`, `parent_id`, `is_essential` |
| `recurrences` | `label`, `merchant_id`, `amount_cents`, `amount_variability` (`fixed` \| `variable`), `frequency` (`weekly` \| `monthly` \| `bimonthly` \| `quarterly` \| `yearly`), `anchor_day`, `next_due_on`, `status` (`suggested` \| `confirmed` \| `ignored`), `confidence` |
| `income_cycles` | `expected_on`, `expected_amount_cents`, `source_recurrence_id` |
| `tax_profiles` | `regime` (`forfettario`), `coefficiente_redditivita`, `aliquota_imposta`, `inps_gestione`, `aliquota_inps` |
| `set_asides` | `kind` (`tax` \| `sinking_fund` \| `goal`), `label`, `target_cents`, `due_on`, `accrued_cents` |
| `cash_reconciliations` | `estimated_cents`, `declared_cents`, `delta_cents`, `created_at` |

---

## 4. Motore (`packages/core`)

Funzioni pure: ricevono dati, restituiscono risultato **più spiegazione**. Nessuna chiamata AI dentro il motore.

### 4.1 Rilevamento ricorrenze

1. Raggruppa le uscite per commerciante normalizzato.
2. Per ogni gruppo con ≥ 3 occorrenze calcola gli intervalli tra le date; riconosci la frequenza se la mediana cade in una finestra (7±1, 14±2, 30±3, 61±5, 91±7, 365±15 giorni).
3. Importo: `fixed` se la deviazione è < 5%, altrimenti `variable` (previsione = media ponderata delle ultime occorrenze, con stagionalità per le utenze quando c'è almeno un anno di storico).
4. `confidence` da regolarità di intervalli e importi → le ricorrenze sopra soglia diventano `suggested`, l'utente conferma con un tap.
5. Stessa logica sulle entrate → individua lo **stipendio** (importo alto, ricorrenza mensile, stesso ordinante).

### 4.2 Safe-to-spend — profilo `salary`

```
orizzonte      = da oggi al prossimo accredito previsto (escluso)
liquidità      = Σ saldi conti bancari + Tasca contanti stimata
impegni        = Σ ricorrenze confermate in scadenza nell'orizzonte
               + quote accantonamenti (fondi annuali, obiettivi) maturate nell'orizzonte
               + cuscinetto di sicurezza
disponibile    = liquidità − impegni
oggi           = max(0, disponibile / giorni_rimanenti)
```

Output: `{ today_cents, horizon_cents, horizon_end, lines: [...] }`, dove `lines` è la spiegazione riga per riga mostrata al tap.

Casi limite: stipendio in ritardo (se l'accredito non arriva entro +3 giorni, l'orizzonte si estende e l'utente viene avvisato); `disponibile < 0` → stato "in rosso" con quanto manca, non un numero negativo al giorno.

### 4.3 Safe-to-spend — profilo `irregular` (freelance / P.IVA)

Problema: senza uno stipendio non c'è un "prossimo accredito" su cui ragionare. Soluzione: **"stipendio virtuale"**.

- Le entrate di lavoro vanno in una **riserva** (concetto contabile dentro Tallyo, non serve un conto separato).
- Ogni mese la riserva "paga" all'utente uno stipendio virtuale fisso; Tallyo propone come valore iniziale la mediana delle entrate nette mensili degli ultimi 6–12 mesi × 0,9.
- Il safe-to-spend usa la formula §4.2 con orizzonte = fine del mese virtuale e liquidità = stipendio virtuale del mese − spese già fatte.
- Indicatore secondario **"autonomia"**: quanti mesi di stipendio virtuale copre la riserva attuale (es. "Riserva: 3,4 mesi").

### 4.4 Accantonamento tasse — regime forfettario

Per ogni entrata marcata `is_business_income` (suggerita automaticamente, confermabile):

```
imponibile = incasso × coefficiente_redditività      (es. 78% per molti professionisti)
inps       = imponibile × aliquota_inps              (es. Gestione Separata)
imposta    = (imponibile − inps) × aliquota_imposta  (5% primi 5 anni, poi 15%)
accantonamento = inps + imposta
```

Esempio: incasso 1.000 €, coefficiente 78%, INPS 26,07%, imposta 15% → imponibile 780 €, INPS 203,35 €, imposta 86,50 € → **accantonare ~290 € (29%)**.

- L'accantonamento viene escluso dalla liquidità spendibile e mostrato come "Fondo tasse".
- Le scadenze F24 (saldo + primo acconto a giugno, secondo acconto a novembre) diventano ricorrenze previste.
- Artigiani/commercianti (INPS con contributi fissi minimi + percentuale) come variante del profilo.
- **Parametri configurabili e da verificare ogni anno**; in app la cifra è sempre presentata come stima, non consulenza fiscale.

### 4.5 Tasca contanti

- **Prelievo ATM** riconosciuto da codice transazione/descrizione (`PRELIEVO`, `ATM`, `BANCOMAT`, …) → `kind = atm_withdrawal`, la Tasca sale.
- **Spese in contanti** (voce/manuale/scontrino) → la Tasca scende.
- **Burn rate**: media mobile esponenziale delle uscite contanti giornaliere (registrate + non tracciate da riconciliazione). Tra due riconciliazioni la Tasca stimata = ultimo valore noto + prelievi − spese registrate − burn rate × giorni per la parte non registrata.
- **Riconciliazione**: notifica quando la stima è bassa o dopo 5–7 giorni → l'utente indica quanti contanti ha → la differenza diventa una transazione `cash_adjustment` ripartita sulle categorie abituali.

---

## 5. Livello AI (backend)

| Funzione | Input → Output | Modello |
|---|---|---|
| Parsing input vocale/testo | "16 euro di benzina in contanti" → `{amount_cents: 1600, category: "carburante", account: "cash", date: oggi}` (output strutturato) | piccolo |
| Normalizzazione commercianti | `POS 1234 ESSELUNGA MI VIA…` → `Esselunga` (cache in `merchants`: si chiama l'AI solo per descrizioni mai viste) | piccolo |
| Categorizzazione | regole + cache → AI solo per i casi nuovi; le correzioni dell'utente diventano regole | piccolo |
| OCR scontrini (Fase 3) | foto → righe, totale, codice fiscale, detraibilità | vision |
| **Lyo** | domanda → **tool call** al motore/DB (`query_transactions`, `get_safe_to_spend`, `list_recurrences`…) → risposta con link alle transazioni usate | medio |

Regole:
- L'AI **non calcola mai** importi: interpreta e spiega; i numeri arrivano dal motore o da query SQL.
- **Minimizzazione dati**: ai modelli vanno solo descrizione, importo, data e categoria; mai IBAN, nomi di persone o identificativi del conto.
- Prompt e modelli versionati; risposte di parsing validate con uno schema prima di essere salvate.

---

## 6. Sicurezza e privacy

- Hosting e database in UE; RLS su tutte le tabelle; token del provider bancario cifrati (pgsodium/Vault).
- Login con Apple / Google / email + **Face ID / impronta** per aprire l'app.
- Cache locale cifrata (SecureStore per chiavi, database locale cifrato per i dati offline).
- "Elimina tutto" in un tap: revoca il consenso bancario e cancella i dati.
- Nessun analytics di terze parti sui dati finanziari.

---

## 7. Piano di lavoro Fase 0

| # | Blocco | Risultato verificabile |
|---|---|---|
| 1 | Verifica provider + HYPE | Account sandbox del provider, HYPE presente in lista, campi disponibili documentati |
| 2 | Setup monorepo | Expo app che parte su iOS e Android, Supabase locale, CI con lint + test |
| 3 | `packages/core` | Ricorrenze, safe-to-spend (salary + irregular), tasse forfettario, Tasca: tutti con test su dati di esempio |
| 4 | Schema DB + Auth | Migrazioni, RLS, login Apple/Google/email |
| 5 | Collegamento banca | Flusso §2.2 funzionante con il tuo conto HYPE reale; sync + dedup |
| 6 | Schermata **Oggi** | Safe-to-spend del giorno, spiegazione, prossime uscite, Tasca |
| 7 | Movimenti + input rapido | Lista unificata, inserimento manuale e vocale |
| 8 | Onboarding | Scelta profilo (dipendente / P.IVA), conferma stipendio e ricorrenze suggerite |
| 9 | Uso reale (2–4 settimane) | Diario degli errori del safe-to-spend → tarature del motore |

---

## 8. Test

- `packages/core`: test unitari con scenari realistici (mese tipo dipendente, freelance con 2 fatture/mese, stipendio in ritardo, bolletta variabile, prelievo + riconciliazione).
- Edge functions: test con provider in sandbox.
- App: test dei componenti chiave + prova manuale su dispositivo iOS e Android a ogni rilascio.
