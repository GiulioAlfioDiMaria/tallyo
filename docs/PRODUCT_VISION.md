# Tallyo — Product Vision (v0.1, bozza di lavoro)

> **Tallyo sa quanto puoi spendere oggi — contanti compresi.**

Documento di lavoro: analisi della bozza iniziale, panorama competitivo, proposte di differenziazione e definizione del prodotto (MVP → v2). Le parti marcate **[DA DECIDERE]** sono scelte aperte.

---

## 1. Lettura della bozza: l'intento reale

La bozza descrive tre cose diverse che vanno tenute distinte:

| Livello | Cosa c'è nella bozza | Ruolo nel prodotto |
|---|---|---|
| **Problema** | Il saldo bancario mente: non conta i costi fissi in arrivo e non vede i contanti | È il *perché* dell'app |
| **Promessa** | "Safe-to-Spend": un numero affidabile di quanto posso spendere | È il *cosa* — il cuore del prodotto |
| **Mezzi** | Open banking, OCR, voce, AI di categorizzazione, copilot | Sono il *come* — abilitano la promessa, non sono il prodotto |

L'intento: **eliminare l'ansia di "ce la faccio fino allo stipendio?"** per chi usa una neobank (HYPE) ma vive ancora in un'economia parzialmente in contanti (Italia).

Osservazioni critiche sulla bozza:

1. **Troppi pilastri di pari peso.** Ingestione, categorizzazione, forecast e coaching sono presentati allo stesso livello. Sul mercato, categorizzazione e grafici sono commodity. Il valore distintivo è *Safe-to-Spend + contanti*: tutto il resto deve servire quello.
2. **Il ciclo "fine mese" è sbagliato per molti.** Lo stipendio arriva il 27, il 10, a fine mese, oppure è irregolare (freelance, studenti). Il ciclo va ancorato **all'accredito delle entrate**, non al calendario.
3. **I contanti sono trattati come "un'altra fonte di input"**, ma sono il vero problema irrisolto: nessuno registra ogni caffè. Serve un modello diverso (vedi §4.1).
4. **L'AI non deve fare i conti.** I numeri (saldo, safe-to-spend, proiezioni) devono essere calcolati da un motore deterministico. L'LLM serve per *capire* (parsing, categorie, linguaggio naturale) e *spiegare*, mai per sommare. Questo va reso esplicito nell'architettura: è anche un argomento di fiducia verso l'utente.
5. **Dipendenza da HYPE.** Ottimo come primo caso d'uso personale, ma come prodotto deve essere multi-banca (Revolut, BBVA, Satispay, Fineco, Intesa…) fin dal design dei dati.

---

## 2. Panorama competitivo

### 2.1 Mappa

| Competitor | Mercato | Punti forti | Cosa manca rispetto a Tallyo |
|---|---|---|---|
| **App delle banche/neobank** (HYPE, Revolut, Intesa "Spese e budget", BBVA) | IT/EU | Gratis, dati nativi, categorie automatiche | Vedono solo il proprio conto; zero contanti; niente safe-to-spend reale |
| **Wallet by BudgetBakers** | EU (CZ), molto usata in IT | Sync bancario EU, multi-conto, budget | UX da "contabile", AI limitata, contanti solo manuali |
| **Spendee / Money Lover / Toshl / 1Money** | Globale | Inserimento manuale rapido, wallet condivisi | Principalmente manuali, nessuna vera previsione |
| **YNAB** | US/globale | Metodologia forte (envelope budgeting), community | Costoso, curva d'apprendimento ripida, poco adatto all'IT |
| **Copilot Money / Monarch** | US | UX eccellente, AI categorization, ricorrenze | Solo US (Plaid), nessun focus contanti |
| **PocketGuard** | US | "In My Pocket" = concetto simile a safe-to-spend | Solo US, UX datata |
| **Cleo** | US/UK | Chat AI con personalità, Gen-Z | Orientata a micro-credito; non disponibile in IT |
| **Emma / Snoop / Rocket Money** | UK/US | Rilevamento abbonamenti, cancellazione | Non in IT; nessun contante |
| **Fintonic** | ES (ritirata dall'IT) | Era la PFM di riferimento in Italia | Spazio lasciato libero |

### 2.2 Il buco di mercato

Incrociando le due dimensioni rilevanti:

```
                    Previsione / Safe-to-spend
                    bassa                 alta
                 +---------------------+---------------------+
 Contanti   alta | app manuali         |                     |
 gestiti         | (Spendee, Toshl)    |     >>> TALLYO <<<  |
                 +---------------------+---------------------+
            bassa| app bancarie,       | Copilot, PocketGuard|
                 | Wallet              | (solo US)           |
                 +---------------------+---------------------+
```

- In Italia nessuno occupa il quadrante "previsione affidabile + contanti".
- L'Italia è tra i paesi dell'Eurozona con uso di contante più alto: il problema è locale e concreto.
- Il contesto fiscale italiano (730, spese detraibili, bollo, RC auto, TARI, bollette bimestrali) non è coperto da nessuna app straniera. È un fossato naturale.

---

## 3. Posizionamento

**Per chi:** 20–40enni italiani con almeno una neobank/carta digitale, che usano ancora contanti, con entrate che arrivano a date precise (stipendio, borsa di studio) o irregolari (freelance, P.IVA forfettaria).

**Promessa:** *"Ogni mattina sai quanto puoi spendere oggi, senza fare conti e senza registrare ogni caffè."*

**Contro chi:** contro il saldo grezzo dell'app bancaria e contro il foglio Excel.

**Tre aggettivi del brand:** *calmo, preciso, italiano.* (Non "gamificato", non "giudicante".)

---

## 4. Proposte di differenziazione

Ordinate per impatto/fattibilità. Le prime tre sono il cuore del prodotto.

### 4.1 ⭐ Modello "Tasca" per i contanti (invece di registrare ogni spesa)

Il problema dei contanti non è l'input, è che nessuno lo fa con costanza. Proposta:

1. **Il prelievo ATM crea la Tasca.** Tallyo vede dal conto un prelievo da 100 € → la Tasca contanti sale a 100 €. I contanti *entrano* nel sistema automaticamente.
2. **Le spese in contanti la svuotano**, registrate con voce/foto/tap (opzionale!).
3. **Riconciliazione gentile:** ogni tanto (es. ogni 5–7 giorni o quando la Tasca stimata è bassa) una notifica: *"Dovresti avere circa 38 € in tasca. Quanti ne hai?"* → l'utente risponde "20" → i 18 € mancanti diventano **"spese contanti non tracciate"**, ripartite secondo le abitudini apprese (es. caffè/bar).
4. **Burn rate dei contanti:** Tallyo impara che bruci ~6 €/giorno in contanti e lo include nel safe-to-spend anche quando non registri nulla.

Risultato: dati sui contanti *accurati in aggregato* con sforzo quasi zero. Questa è la feature che nessun competitor ha.

### 4.2 ⭐ Safe-to-Spend ancorato allo stipendio, giornaliero

- Ciclo = **dall'ultimo accredito al prossimo** (rilevato automaticamente, modificabile).
- Formula (deterministica, spiegabile):

  ```
  Disponibile = saldi conti + Tasca contanti
              − uscite fisse previste fino al prossimo accredito
              − accantonamenti (fondi per spese annuali, obiettivi)
              − cuscinetto di sicurezza (configurabile)
  Safe-to-Spend oggi = Disponibile / giorni rimanenti al prossimo accredito
  ```

- Il numero principale della home è **"Oggi puoi spendere 27 €"**, con il dettaglio "fino al 27 novembre: 540 €".
- Effetto *rollover*: se oggi spendi 10 €, domani il giornaliero sale. Gratificante senza gamification forzata.
- **Tap sul numero = spiegazione riga per riga** (trasparenza totale, nessuna black box).

### 4.3 ⭐ Fondi per le spese "invisibili" annuali (sinking funds)

Il vero motivo per cui gli italiani vanno in affanno non è Spotify, è **RC auto, bollo, TARI, conguaglio bollette, regali di Natale, vacanze**. Tallyo:

- le rileva dallo storico (pagamenti annuali/semestrali) o le propone da una lista italiana pre-compilata;
- calcola l'accantonamento mensile (RC auto 480 €/anno a marzo → 40 €/mese);
- lo **sottrae già dal safe-to-spend**, così a marzo i soldi "ci sono già".

Questo trasforma il "previsionale" della bozza da *mese* a *anno*: differenziante e molto utile.

### 4.4 Modulo "Fisco Italia": scontrini detraibili per il 730

- L'OCR riconosce lo **scontrino parlante** (farmacia, visite mediche, ottico) con codice fiscale.
- Tallyo li etichetta "detraibile 19%" e mostra in tempo reale: *"Spese sanitarie detraibili 2026: 642 € → circa 102 € di rimborso stimato"*.
- A fine anno: export PDF/ZIP pronto per il CAF o per il controllo del 730 precompilato.
- Estensioni: spese veterinarie, abbonamenti trasporto pubblico, palestra per figli, interessi mutuo.

Nessuna app straniera lo farà mai: è un fossato.

### 4.5 "Posso permettermelo?" (pre-acquisto)

Domanda prima della spesa, non dopo: *"Posso comprare scarpe da 120 €?"* → *"Sì, ma il safe-to-spend giornaliero scende da 27 € a 19 € fino al 27. In alternativa aspetta l'accredito: mancano 9 giorni."* Accessibile da widget, voce, Siri/Google Assistant, chat.

### 4.6 Ricorrenze intelligenti (dalla bozza, migliorata)

- Rilevamento di ricorrenze fisse e **variabili** (bollette: stima con media/stagionalità, intervallo di confidenza).
- **Allarmi:** aumento prezzo abbonamento ("Netflix è passato da 13,99 a 15,99"), abbonamenti duplicati (Apple Music + Spotify), prove gratuite in scadenza, abbonamenti inutilizzati da mesi.
- Guide di disdetta per i servizi italiani più comuni (palestre, pay-TV, telefonia).

### 4.7 Copilot conversazionale "con le prove"

- Domande in linguaggio naturale ("Quanto mi è costata la macchina quest'anno?").
- L'LLM traduce la domanda in una query sul DB → il motore calcola → l'LLM spiega. **Ogni risposta mostra le transazioni usate** (cliccabili). Niente numeri inventati.
- Tono: consulente calmo, mai colpevolizzante. Massimo 1–2 insight proattivi a settimana (no spam di notifiche).
- **Weekly check-in** (domenica sera, 60 secondi): riepilogo settimana, conferma 2–3 transazioni dubbie, riconciliazione Tasca, previsione settimana prossima.

### 4.8 Altre idee (v2+)

- **Garanzie e resi:** dallo scontrino, promemoria scadenza garanzia (2 anni) e finestra di reso.
- **Spese condivise:** conto di coppia/coinquilini, quota parte nel safe-to-spend, integrazione o alternativa a Splitwise.
- **Modalità P.IVA forfettaria:** accantonamento automatico % per tasse e INPS sulle entrate → safe-to-spend al netto delle tasse future. Nicchia enorme e mal servita.
- **Satispay e buoni pasto** come "conti" di prima classe (i buoni pasto sono denaro con vincoli d'uso).
- **Widget / Live Activity / Apple Watch:** safe-to-spend sempre visibile; registrazione vocale dal polso.

---

## 5. Cose da ridimensionare o scartare dalla bozza

| Idea della bozza | Valutazione |
|---|---|
| Intercettazione notifiche push HYPE | Fattibile solo su **Android** (NotificationListenerService). Su iOS non è possibile. Utile come fallback/realtime su Android, non come pilastro. |
| GoCardless (ex Nordigen) come connettore | **Verificare la disponibilità attuale**: GoCardless ha chiuso Bank Account Data ai nuovi clienti nel 2025. Alternative EU: Enable Banking, Tink, Salt Edge, Yapily. Per uso personale sui propri conti Enable Banking offre un accesso gratuito. |
| Parsing email di HYPE | Fragile; da usare solo se HYPE invia email per ogni transazione (da verificare). |
| Whisper per la voce | Ok lato server, ma su mobile lo speech-to-text di sistema (iOS/Android) supporta l'italiano, è gratuito e on-device: preferibile per privacy e latenza. |
| Report sintetici "da contabile" | Commodity. Tenerli minimi; il valore è nella home e nel copilot. |
| Gamification / badge | Sconsigliata: il tono "calmo" è parte del posizionamento. |

---

## 6. Definizione del prodotto

### 6.1 Principi

1. **Un numero prima di tutto:** la home mostra il safe-to-spend di oggi. Tutto il resto è a un tap.
2. **Zero obblighi di input:** l'app funziona anche se non registri mai un contante (grazie al modello Tasca + burn rate). Registrare migliora la precisione, non è un requisito.
3. **AI per capire, matematica per contare.** Ogni numero è spiegabile riga per riga.
4. **Privacy by design:** dati in UE, crittografia, nessuna vendita di dati, cancellazione totale in un tap. Elaborazione on-device dove possibile.
5. **Italiano nativo:** lingua, fisco, abitudini (bollette bimestrali, tredicesima, quattordicesima, buoni pasto).

### 6.2 Schermate principali

1. **Oggi (Home):** safe-to-spend giornaliero grande; barra "fino al prossimo accredito"; prossime 3 uscite previste; Tasca contanti; pulsante unico "+" (voce / foto / manuale).
2. **Calendario:** vista giorni fino al prossimo accredito (e mesi successivi) con uscite previste, accantonamenti e saldo proiettato giorno per giorno.
3. **Movimenti:** lista unificata conti + contanti, con commercianti normalizzati e categorie; filtri e ricerca naturale.
4. **Piani:** ricorrenze, fondi annuali, obiettivi, budget per categoria (opzionali).
5. **Copilot:** chat + insight settimanali + check-in domenicale.

### 6.3 Roadmap

**Fase 0 — Prototipo personale (4–6 settimane)**
Obiettivo: usarla tu, ogni giorno, con HYPE e i tuoi contanti.
- Import movimenti HYPE (open banking via Enable Banking per i propri conti, oppure import CSV/estratto conto come ripiego).
- Tasca contanti con prelievi automatici + inserimento vocale/manuale.
- Rilevamento ricorrenze base + conferma utente.
- Safe-to-spend giornaliero ancorato allo stipendio con spiegazione.

**Fase 1 — MVP pubblico (beta chiusa, 50–200 utenti)**
- Multi-banca, onboarding, categorizzazione AI, OCR scontrini.
- Fondi annuali (sinking funds) con lista italiana pre-compilata.
- Riconciliazione Tasca e burn rate.
- Weekly check-in.
- Metriche chiave: % utenti che aprono la home ≥ 4 giorni/settimana; errore medio del safe-to-spend vs. realtà; retention a 30 giorni.

**Fase 2 — Differenzianti**
- Modulo Fisco 730, "Posso permettermelo?", copilot conversazionale con prove, alert abbonamenti, widget.

**Fase 3 — Espansione**
- Modalità P.IVA forfettaria, spese condivise/coppia, garanzie, Satispay/buoni pasto.

### 6.4 Architettura (proposta)

```
 Mobile app (React Native/Expo oppure Flutter)
   ├─ STT on-device (italiano)          ├─ camera → upload scontrino
   └─ cache locale cifrata
                │
                ▼
 Backend (es. Supabase: Postgres + Auth + Edge Functions, region UE)
   ├─ Connettore open banking (Enable Banking / Tink / Salt Edge)
   ├─ Ingestion pipeline → normalizzazione → dedup
   ├─ LLM layer (parsing testo/voce → JSON, vision OCR scontrini,
   │             normalizzazione commercianti, categorie, NL→query)
   ├─ Motore deterministico: ricorrenze, forecast, safe-to-spend,
   │             fondi annuali, Tasca contanti
   └─ Insight engine (regole + LLM solo per la formulazione)
```

Modello dati di base: `accounts` (banca, contanti, buoni pasto), `transactions`, `merchants`, `categories`, `recurrences`, `sinking_funds`, `income_cycles`, `cash_reconciliations`, `receipts`, `insights`.

### 6.5 Regolamentazione e rischi

- **PSD2/AISP:** per servire utenti terzi con dati bancari serve una licenza AISP o operare tramite un provider licenziato (modello "agent"/white-label). Per il prototipo personale non è un problema.
- **Rinnovo consenso:** le connessioni open banking richiedono riconferma periodica (tipicamente ogni 90/180 giorni a seconda di banca e provider) → progettare un flusso di riconnessione indolore.
- **GDPR:** dati finanziari sensibili → hosting UE, DPA con i fornitori LLM, minimizzazione dei dati inviati ai modelli (es. niente IBAN/nominativi nel prompt).
- **Consulenza finanziaria:** il copilot deve restare su budgeting e spesa; evitare consigli su investimenti (attività regolamentata).
- **Dipendenza da provider esterni:** astrarre il connettore bancario dietro un'interfaccia per poterlo sostituire.

### 6.6 Modello di business [DA DECIDERE]

- **Freemium:** gratis 1 conto + contanti + safe-to-spend; **Plus** (~3,99 €/mese o ~29,99 €/anno) multi-conto, OCR illimitato, copilot, Fisco 730, fondi annuali.
- I costi variabili principali (connessioni open banking per utente, chiamate LLM/OCR) vanno misurati in Fase 1 per validare il prezzo.
- Mai monetizzare vendendo dati o prodotti finanziari: incoerente col posizionamento.

---

## 7. Naming

**Tallyo** è una buona scelta: corto, internazionale, significato coerente ("tally" = fare il conto), suona amichevole. Da verificare:

- Marchio su **EUIPO/UIBM** (classi 9 e 36) e assenza di conflitti con "Tally" (es. Tally Solutions, software contabile molto diffuso).
- Disponibilità domini (`tallyo.app`, `tallyo.it`, `gettallyo.com`) e handle social.
- Pronuncia italiana (*tàllio* vs *tallìo*): decidere e comunicarla.

Tra le alternative della bozza, **Saldo** è forte in italiano ma generico e difficilmente registrabile; **Ratio** e **Clarus** sono già molto usati. Consiglio: tenere **Tallyo**.

Possibili claim:
- *"Quanto puoi spendere oggi. Davvero."*
- *"I tuoi conti, contanti compresi."*
- *"Fai pace con fine mese."*

---

## 8. Decisioni aperte

1. **Target iniziale:** dipendenti con stipendio fisso, oppure anche freelance/P.IVA fin da subito?
2. **Piattaforma:** iOS + Android insieme (React Native/Flutter) o una sola piattaforma prima?
3. **Ambizione:** strumento personale che può diventare prodotto, oppure startup fin dall'inizio (con impatto su licenza AISP, budget, tempi)?
4. **Connettore bancario per il prototipo:** open banking subito (Enable Banking) o import CSV per partire più veloce?
5. **Modulo da prototipare per primo dopo il core:** Fisco 730 o Fondi annuali?
