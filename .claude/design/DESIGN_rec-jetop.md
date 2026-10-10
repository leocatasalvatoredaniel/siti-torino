# Design · rec.jetop.com (strumenti REC e modulo di candidatura)

Sistema grafico unico per le pagine di rec.jetop.com ridisegnate il 10/10/2026 con hallmark
(redesign multi-pagina), impeccable, design-taste-frontend, redesign-existing-projects e
ui-ux-pro-max. Ogni modifica futura a queste pagine parte da qui: se serve qualcosa che il
sistema non prevede, si aggiorna prima questo file.

Il file sta in siti-torino e non nel repo JEToP per scelta di Daniel: nel repo JEToP va solo
il codice del sito.

## Pagine e confini

| Pagina | Percorso | Famiglia | Stato |
|---|---|---|---|
| Pagina pubblica | `/` (`index.html`, `landing.js`, `meta-pixel.js`, `assets/landing/`) | riferimento del brand | **non si tocca** (design Figma 1:1, richiesta di Daniel) |
| Modulo di candidatura | `/candidati/` + `/app.js` | pubblica | ridisegnata |
| Banchetti | `/banchetti/` | strumenti | ridisegnata |
| Statistiche | `/rec-dashboard/` | strumenti | ridisegnata |
| Colloqui | `/rec-dashboard/colloqui/` | strumenti | ridisegnata |

La pagina pubblica è la fonte del DNA: fondo indaco con le onde a linee sottili, iris, Red Hat
Display, parole scritte a mano in Schoolbell color iris, foto con bordo iris.

## Genere

- Modulo di candidatura: **atmospheric** con l'accento del brand (iris, non caldo): la tela
  indaco con le onde della pagina pubblica fa da ambiente, il modulo ci sta sopra.
- Strumenti: modalità **Operate** (impeccable): leggibilità, coerenza e velocità prima
  dell'espressione; il brand vive nei dettagli (indaco, iris, numeri in Red Hat Mono).

## Famiglie di macrostruttura

- Pubblica (modulo): **Narrative Workflow**. Cinque tappe davvero ordinate (dati, email, studi,
  aree, presentazione), numerate. Su computer una colonna laterale fissa mostra le tappe e
  quali sono complete; su telefono un indicatore compatto. Niente riquadro unico attorno al
  modulo: le tappe stanno sul fondo, separate da filetti.
- Strumenti: **Bento Grid** di moduli di ampiezza diversa sul fondo indaco, senza schede
  dentro schede. Varianti:
  - Statistiche: testa **Stat-Led** (un numero guida grande e gli altri valori in una riga
    di dati, niente otto schede uguali), poi i grafici in moduli 2/1 colonne.
  - Colloqui e banchetti: il calendario o la griglia dei turni è il modulo dominante;
    liste, filtri e azioni gli stanno attorno.

## Tema (personalizzato, ancorato al brand)

Palette OKLCH. Neutri tinti verso l'indaco (tono 283-292), un solo accento (iris), colori di
stato solo per gli stati.

- `--color-paper` oklch(17.5% 0.05 283): fondo degli strumenti (indaco notte)
- `--color-paper-2` oklch(21.5% 0.058 284): superfici (moduli, intestazione)
- `--color-paper-3` oklch(26% 0.066 285): campi, elementi in rilievo, hover
- `--color-paper-4` oklch(30% 0.07 286): menu e finestre sopra le superfici
- `--color-ground` oklch(25.5% 0.102 279): fondo del brand (#1B1852), solo nel modulo
- `--color-rule` oklch(32% 0.055 286): filetti e bordi decorativi
- `--color-rule-2` oklch(52% 0.06 288): bordi dei controlli (≥3:1 sulle superfici)
- `--color-ink` oklch(95% 0.012 290): testo
- `--color-ink-2` oklch(83% 0.028 292): testo secondario
- `--color-muted` oklch(72% 0.035 293): etichette e note (≥4.5:1 su tutte le superfici)
- `--color-accent` oklch(74% 0.115 298): iris per link, attivo, valori in evidenza
- `--color-accent-brand` oklch(63% 0.11 300): iris del brand (#9379C2) per le parole a mano
- `--color-accent-fill` oklch(51.2% 0.175 289): iris pieno (#684DC2) dei pulsanti principali
- `--color-accent-ink` oklch(98% 0.008 290): testo sopra l'iris pieno (5.8:1)
- `--color-accent-wash` oklch(26.5% 0.085 290): fondo tenue di selezione e chip attivi
- `--color-focus` oklch(80% 0.12 298): anello di focus
- `--color-ok` / `--color-warn` / `--color-err` con i rispettivi `-wash` per i fondi tenui

I colori delle aree (M&C verde, D&V arancio, IT viola, S&P blu, T&DA giallo) restano: sono una
codifica dei dati che le persone hanno già imparato, non decorazione.

## Tipografia

- Display e testo: **Red Hat Display** (self-hosted, `/fonts/RedHatDisplay-var.woff2`).
  Titoli 700-800 con tracking -0.02em; testo 400, interlinea 1.5.
- Dati: **Red Hat Mono** per numeri, orari, date, conteggi e codici (`tabular-nums`).
- Mano: **Schoolbell** solo nel modulo di candidatura, al massimo due punti per pagina
  (la parola «JEToP» del titolo e il messaggio finale), sempre in `--color-accent-brand`.
- Mai corsivi nei titoli. Niente maiuscoletto con tracking largo sopra i titoli.
- Scala 1.25 da 16px: 12.8 · 16 · 20 · 25 · 31 · 39; titolo pagina `clamp(1.75rem, 3vw + 1rem, 2.75rem)`.

## Spazi, forme, profondità

- Scala 4pt con nomi: `--space-3xs` 2px … `--space-3xl` 96px; mai px sparsi.
- Forme: controlli interattivi a pillola (pulsanti, chip, selettori), campi 10px, moduli 14px.
- Profondità con la luminosità (superficie più alta = più chiara), non con ombre colorate.
  Ombre solo sotto finestre e toast, scure e strette.
- Z-index a sei livelli con nome (`--z-raised` … `--z-tooltip`).

## Movimento

- Easing `--ease-out` cubic-bezier(0.16, 1, 0.3, 1), `--ease-in`, `--ease-in-out`.
- Durate 120 / 220 / 420 ms. Solo transform e opacity.
- Un solo ingresso orchestrato nel modulo (le tappe, sfalsate, ≤500ms). Negli strumenti nessun
  ingresso: finestre in scala 0.96→1, scheda candidato che scorre da destra.
- `prefers-reduced-motion`: tutto diventa dissolvenza ≤150ms; i grafici non si animano.

## Microinterazioni

- Successo silenzioso dove il risultato si vede; toast solo per esiti non visibili o errori.
- Conferma esplicita solo per azioni irreversibili verso i candidati (email, rifiuto, ritiro).
- Hover dentro `@media (hover: hover)`; pressione `translateY(1px)`.
- Focus sempre visibile e istantaneo: `outline 2px var(--color-focus)` con offset 2px.
- Tocchi da almeno 44px su telefono e schermi touch.

## Icone

- **Phosphor** peso regular (MIT), viewBox 256, riempimento `currentColor`, 1.15em.
- Nessuna emoji nell'interfaccia. Unica eccezione: il 🎉 dentro il testo dell'email di
  accettazione, che è contenuto dell'email e non interfaccia.

## Intestazione e piè di pagina

- Strumenti: intestazione compatta (logo, «REC», selettore Statistiche / Colloqui / Banchetti
  con la pagina attiva evidenziata, stato aggiornamento, Esci). Su telefono il selettore va
  su una seconda riga a tutta larghezza. Ai soci senza ruolo REC i banchetti non mostrano il
  selettore.
- Modulo: intestazione allineata ai bordi (logo + «Candidature» a sinistra, jetop.com a destra),
  senza barra né filetto.
- Piè di pagina: una sola riga (Ft2), senza lineette lunghe.

## Testi

- Niente lineette lunghe (`—`, `–`) nei testi visibili: due punti, virgola o punto.
- Il punto mediano al massimo uno per riga nelle righe di metadati.
- Niente punti esclamativi nei messaggi di successo. Frasi in italiano piano, verbi concreti.
- Campi, ordine dei campi, nomi dei campi, testo del consenso privacy e indirizzi delle pagine
  non cambiano.

## Cosa le pagine condividono

Il foglio `assets/jetop-ui.css` (caricato con `?v=AAAAMMGG` perché nginx non manda
`Cache-Control` per i `.css`): font, token, base (selezione, scrollbar, focus, riduzione
movimento, icone), intestazione degli strumenti, pulsanti, campi, badge, finestre, toast.
Ogni pagina tiene nel proprio `<style>` solo ciò che è suo, sempre tramite i token.

## Dettagli fissati durante la costruzione

- Intestazione degli strumenti: fissa in alto su computer; su telefono (≤760px) va su due righe e
  scorre via con la pagina, per non occupare il 14% dello schermo.
- Livelli: intestazione `--z-sticky` (200), scheda candidato `--z-modal` (400), finestre
  `--z-modal + 10`, toast `--z-toast` (500). Le finestre devono stare sopra l'intestazione.
- Chip del calendario: niente barretta laterale; il colore dell'area tinge bordo e fondo con
  `color-mix(in oklab, …)`. Non usare `in oklch`: mescolando il giallo con l'indaco la tinta
  gira attraverso il rosso e le chip T&DA diventano rosa.
- Statistiche: i colori dei grafici si leggono dai token a runtime (`token()` in `app.js`);
  Chart.js 4.4 accetta stringhe `oklch()`.
- Menu a tendina: freccia Phosphor `caret-down` in data URI (unico colore scritto in esadecimale,
  `#B49BEA` = `--color-accent`, perché `var()` non funziona dentro un data URI).
- Spaziature: padding, margin e gap solo su multipli di 4px.
- Il modulo riserva sempre lo spazio dei messaggi (`.status` con `min-height`), cosi' un errore
  che compare non sposta la pagina.

## Cosa le pagine possono variare

La macrostruttura dentro la famiglia (Stat-Led per le statistiche, griglia dominante per
colloqui e banchetti) e la disposizione dei moduli. Mai il tema, i font o la voce dei pulsanti.
