# Tallyo — Identità visiva (v1)

Scelta: **palette Laguna** (tema scuro, predefinito) con tema chiaro derivato, **font Manrope**, stile **glassmorphism**.
I valori ufficiali sono in [`design/tokens.json`](../design/tokens.json): l'app li importa da lì, non li riscrive a mano.

Anteprima interattiva usata per la scelta: https://claude.ai/artifact/51X4N3h6VzB7ih59H7vqFN

---

## 1. Concetto

*Vetro calmo sopra un'aurora che cambia con i tuoi soldi.*
Le informazioni stanno su lastre di vetro smerigliato; dietro, una luce tipo aurora (acqua di laguna al tramonto) dà profondità e racconta lo stato del budget. Tre parole: **calmo, preciso, italiano**.

## 2. Colori

### Tema scuro (predefinito)

| Ruolo | Valore | Uso |
|---|---|---|
| Fondo | `#061A20` | Sfondo dell'app, blu notte petrolio |
| Aurora 1 | `#13A89E` | Luce principale dello sfondo |
| Aurora 2 | `#1D4E89` | Profondità |
| Aurora 3 | `#C9803F` | Riflesso caldo, piccolo e tenue |
| Accento | `#5EE6D0` | Azioni, selezioni, barra del ciclo, tab attiva |
| Lyo | `#5EE6D0` → `#F2A65A` | Solo la sfera di Lyo |
| Testo | `#EAF5F3` (secondario al 66%) | |
| Comodo | `#4ADE9B` | Stato "sei in linea" |
| Attento | `#F5C451` | Stato "budget stretto" |
| In rosso | `#FF9E95` | Stato "sotto zero" |

### Tema chiaro

Stessa identità di Laguna (verde acqua + riflesso ambra) su base chiara. Fondo `#E6ECEF`, aurora menta/cielo/pesca, vetro bianco al 55%, testo `#0F2329`, accento `#0A766D`. Valori completi in `design/tokens.json`.

### Regole

- I **colori di stato** dicono come stai andando e basta: mai usati come decorazione. Sono sempre accompagnati da un'etichetta o un'icona (chi non distingue i colori deve capire lo stesso).
- Il colore di Lyo (sfumatura acqua → ambra) appare **solo** dove parla Lyo.
- Contrasti verificati: testo e colori di stato superano 4,5:1 sul vetro in entrambi i temi (alcuni valori del tema chiaro sono stati scuriti rispetto all'anteprima per questo motivo).

## 3. Tipografia

**Manrope** per tutto (titoli, testo, numeri), con **cifre tabulari** attive ovunque compaiano importi: gli importi restano allineati in colonna e il numero del giorno non "balla" quando cambia.
Fallback: SF Pro (iOS), Roboto (Android).

| Stile | Dimensione / peso | Uso |
|---|---|---|
| Numero del giorno | 56 / 600, spaziatura −4% | Safe-to-spend in home |
| Titolo | 28 / 700 | Titoli di schermata |
| Intestazione | 19 / 700 | Saluto, titoli di carte, numeri secondari |
| Importo | 17 / 600 | Importi in lista |
| Testo | 15 / 400 | Testo corrente, messaggi di Lyo |
| Didascalia | 12 / 500 | Date, note |
| Etichetta | 11 / 700, maiuscolo, +10% | Intestazioni di sezione |

Il simbolo € segue il numero, più piccolo e nel colore secondario (`27,00 €`). Formato italiano: virgola decimale, punto per le migliaia.

## 4. Vetro

1. **Tre livelli, non di più:** sfondo aurora → carte di vetro → elementi fluttuanti (tab bar, barra di Lyo). Le righe dentro una carta sono piatte, separate da linee sottili.
2. **Ricetta:** blur 24–30, saturazione 160%, riempimento bianco 8% (scuro) / 55% (chiaro), bordo 1 px, luce sottile sul bordo alto, ombra morbida.
3. **Il testo sta sempre sul vetro**, mai direttamente sull'aurora.
4. **L'aurora racconta lo stato:** più fredda quando sei comodo, leggermente più calda quando il budget si stringe. Transizione lenta, mai lampeggiante.
5. **Animazione:** l'unico elemento che si muove da solo è la sfera di Lyo (respiro lento mentre pensa/parla). Rispettare l'impostazione di sistema "riduci movimento".

## 5. Forme e spazi

- Raggi: carta principale 26, carte 20, icone 9, pillole e barre completamente arrotondate.
- Spaziatura su scala 4/8/12/16/24/32. Margine laterale schermo 16.
- Icone a tratto (1,8 px), angoli arrotondati, nessuna emoji nell'interfaccia.

## 6. Implementazione (Expo / React Native)

- **iOS 26+:** usare il materiale nativo "Liquid Glass" tramite il modulo Expo dedicato (da verificare in fase di setup), con fallback a `expo-blur` sulle versioni precedenti.
- **Android:** `expo-blur` solo su carte grandi e barre, mai nelle liste lunghe; sui dispositivi lenti riempimento semitrasparente senza blur.
- **Aurora:** gradiente radiale disegnato con Skia (`@shopify/react-native-skia`), animato a bassa frequenza.
- **Font:** Manrope caricato con `expo-font`; `fontVariant: ['tabular-nums']` su tutti gli importi.
- **Temi:** il tema segue l'impostazione di sistema, con scelta manuale nelle impostazioni.
