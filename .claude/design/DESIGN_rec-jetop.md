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
| Fantasocio | `/rec-dashboard/colloqui/#fantasocio` (modalità a tutto schermo della pagina colloqui) | strumenti, registro giocoso | nuova (23/10) |

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
- Liste lunghe negli strumenti: ogni sezione ha nel titolo il numero di elementi e un pulsante
  «Nascondi»/«Mostra» (ricordato nel browser); una lista aperta oltre le prime righe ha
  «Mostra meno» sia in cima sia in fondo, e richiudendo si torna all'inizio della lista.
- Statistiche: sotto la testata, il modulo «Candidature» è una riga di stato (pallino verde o rosso, titolo, dettaglio con le date in Red Hat Mono) con i comandi a destra. Le azioni immediate chiedono conferma in linea (domanda + «Sì, …» + «Annulla»), non con una finestra. Le date si modificano in un modulo che si apre sotto, con due campi `datetime-local` a `color-scheme: dark`.
- Niente liste doppie: un sottoinsieme della lista principale (per esempio «in attesa di slot»)
  è una vista rapida sopra la lista, a pillola con il numero (`aria-pressed`), non una sezione
  separata. Le viste vanno a capo su telefono, così sono tutte visibili.
- Il modulo riserva sempre lo spazio dei messaggi (`.status` con `min-height`), cosi' un errore
  che compare non sposta la pagina.

## Modalità Fantasocio (23/10)

Unica pagina degli strumenti con un registro giocoso, chiesto da Daniel («più giocosa ma sempre con una ux/ui
stupenda»). La metafora è l'asta del fantacalcio: il candidato chiamato è una figurina, le aree sono squadre con la
loro rosa. Tema, font, icone e voce dei pulsanti restano quelli del sistema; il gioco sta nella composizione, nei
colori delle aree e in un solo momento di movimento (il timbro).

- **Uso:** proiettata durante la riunione, una persona decide da tastiera. Con il fantasocio aperto la radice va al
  112.5% da 1600×860 e al 131.25% da 2200×1200, così crescono anche le finestre che si aprono sopra.
- **Struttura (l'asta):** listone a sinistra (220-280px), scena al centro, rose a destra (240-320px). Sotto i 1500px il
  listone diventa un pannello; sotto i 760px anche le rose. La barra della decisione è `sticky` in fondo alla scena.
- **Figurina:** modulo con raggio 20px; in alto una fascia alta 44px con i colori della 1ª e della 2ª area
  (`linear-gradient(100deg, a1 0 58%, a2 58% 100%)`, stacco netto, nessuna sfumatura); medaglione con le iniziali
  sovrapposto alla fascia (anello del colore della 1ª area); numero «n. X di N» sulla fascia, colore `--color-paper`.
  Dentro, due colonne da 760px di scena in su (container query `scena`): fatti a sinistra, motivazione e note a
  destra separate da un filetto di 1px e con scorrimento proprio.
- **Squadre (voci del tecnico, dal 23/10 sera):** sette pillole alte almeno 52px su una griglia di 5 colonne ×
  2 righe, una colonna per area nell'ordine M&C, D&V, IT, S&P, T&DA:
  - M&C, IT e S&P (`INTERE`) occupano da sole le due righe (`--col`, `--riga`) e sotto il codice mostrano il nome
    per esteso;
  - D&V e T&DA hanno le due sotto-aree una sopra l'altra;
  - i tasti vanno da 1 a 7 e ogni pillola ha il colore dell'area madre (`colTec`);
  - spenta ha il bordo tinto dall'area; accesa ha fondo `color-mix(in oklab, area 32%, paper-3)` e bordo pieno
    dell'area;
  - sotto il nome compare «1ª/2ª scelta» o «convocato» (con lucchetto);
  - il nome è a .9375rem (.875rem con scena sotto i 1100px);
  - sul telefono le colonne sono 3 a posizionamento automatico, senza pallino né icona.
- **Area da scegliere:** chi è al tecnico senza voce valida ha il glifo mezzo pieno (`.fa-gl.completare`) e la chip
  del timbro in stile avviso (`i.da`, `--color-warn`): «area da scegliere», oppure «D&V: sotto-area?» se ha D&V o
  T&DA interi. Ha anche una rosa «Area da scegliere», con il titolo nel colore di avviso.
- **Finestra «Aree del tecnico» (fuori dal fantasocio):** caselle su due colonne (`.pk-tecs9`), una riga per area;
  M&C, IT e S&P occupano la riga intera (`.solo`). Sotto i 560px c'è una colonna sola.
- **Timbro della decisione:** riquadro con bordo 3px `currentColor`, ruotato di -5°: iris «Al tecnico» con le aree,
  muted «Non passa», verde `--color-ok` «Convocato». Cade (scala 1.8 → 0.94 → 1, 420ms) solo quando la decisione
  cambia; con movimento ridotto compare in dissolvenza.
- **Movimento:** eccezione documentata al «negli strumenti nessun ingresso»: cambiando candidato la figurina entra
  da destra o da sinistra (32px, 220ms); un nome appena entrato in una rosa fa un anello del colore dell'area.
- **Tasti visibili:** `kbd` in Red Hat Mono con bordo `--color-rule-2`; nascosti su schermi touch e sotto i 760px.
- **Successo silenzioso:** la decisione si vede sul timbro e nelle rose, niente toast; gli errori restano toast.
- **Livelli:** il fantasocio sta a `--z-sticky` + 100: sopra l'intestazione, sotto scheda candidato, finestre e avvisi,
  che si aprono sopra.

Regola emersa costruendolo: **non animare né far transitare colori tra tinte lontane** (per esempio il colore di
un'area verso l'indaco): Chromium interpola passando per rosso e giallo. Si animano solo forma e opacità; i cambi di
colore sono istantanei.

## Convoca al tecnico (23/10 sera)

Due passi nella stessa finestra (`.modal.picker`), dal pulsante «Convoca al tecnico» accanto a «Fantasocio».

- **Passo 1, le aree (`.ca-aree`):** cinque righe-pulsante `.ca-scelta`, alte almeno 56px, con quadratino del colore
  dell'area, nome per esteso e, sotto, i gruppi Telegram («un gruppo Telegram per Talent e per Data Analysis»).
  A destra ci sono i numeri («3 al tecnico da convocare · 1 già convocato») e la freccia. Il bordo è tinto
  dall'area; al passaggio del mouse diventa pieno e il fondo prende il 10% del colore.
- **Passo 2, la lista:**
  - titolo «Tecnico di …» con il quadratino, poi i link Telegram (`.ca-link`, colonne da 240px), la ricerca
    (`.ca-cerca`) e i gruppi (`.ca-gruppo`, titolo `h5` piccolo e muted con il numero a destra);
  - ogni candidato è una riga `.ca-riga` con un filetto sopra: nome in grassetto, scelte e stato in piccolo;
  - per M&C, IT e S&P la riga intera è la casella; per D&V e T&DA ci sono due pillole `.ca-voce`, una per
    sotto-area, accese in iris quando spuntate;
  - «non l'aveva scelta» è una chip in stile avviso (`.ca-fuori`);
  - chi è già convocato resta visibile con opacità .6.


## Cosa le pagine possono variare

La macrostruttura dentro la famiglia (Stat-Led per le statistiche, griglia dominante per
colloqui e banchetti) e la disposizione dei moduli. Mai il tema, i font o la voce dei pulsanti.
