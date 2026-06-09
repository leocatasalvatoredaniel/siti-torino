# Report sessione JEToP REC Automation — 2026-06-09

Sessione operativa su n8n live (`jetop-n8n-phue.onrender.com`, v2.21.7). Tutti i deploy
sono stati eseguiti con protocollo: GET fresco → backup timestamped → modifica minima →
validazione (JSON, sintassi Code node, riferimenti `$()`, flag Sheets) → PUT con
`settings` preservate → verifica nel response del PUT → test live.

Nessun segreto è riportato in questo documento. I backup pre/post modifica dei
workflow (JSON completi) sono rimasti nel container di sessione e non sono stati
committati perché contengono il token del bot negli URL dei nodi HTTP.

## Stato finale

| Workflow | ID | Stato | Note |
|---|---|---|---|
| Workflow REC JEToP (principale) | `HirNCuA0RjYvVPp1` | **inactive** (come richiesto) | 105 nodi, preflight documentato sotto |
| Workflow Telegram JEToP | `m7WJCsjBYyPYRmUT` | active | 231 nodi (+1: `Postgres - Cancel Pending Annulla`) |
| JEToP Error Handler | `PhG1qNhFlOhzhMME` | active | notifica nel gruppo HR corretto |

Verifica finale su tutti e tre: zero `parse_mode: Markdown`, zero occorrenze del vecchio
chat_id `-5196229967`, zero espressioni con `}}` adiacenti, `executeOnce/alwaysOutputData`
integri sui nodi Sheets, `settings.errorWorkflow` presente sui due workflow grandi.

## T00 — Preflight

- `/healthz` 200. Supabase non raggiungibile via TCP dalla sandbox: le query sono state
  eseguite tramite un workflow helper temporaneo (webhook + nodo Postgres con credential
  esistente, whitelist di query predefinite), **eliminato a fine sessione**.
- DB al preflight: 2 `slot_assignments` (2 slot distinti, 0 doppi), 1 record con tutte le
  `protection_id_*` null, 1 `pending_reschedule` scaduto da >24h, 0 `telegram_pending`,
  0 `blocked_slots`, `automation_deliveries` vuota. `rec_scheduling_windows` corretti
  (marzo 01→31/03/2026, ottobre 07→28/10/2026), `scheduling_paused=false`, TTL 24h.

## T01 + T03 — Fix `/stato` (P01, P03) ✅ testato

- `Postgres - Stato Extra`: aggiunti slot distinti, slot con 2 colloqui, conteggio+dettaglio
  protezioni null, conteggio `blocked_slots`.
- `Code - Formatta Stato`: riscritto in HTML con escaping su tutti i valori dinamici, body
  pre-serializzato; nuove voci: «Colloqui tracciati Supabase», «Slot orari distinti», «Slot
  con 2 colloqui», «Spostamenti non confermati», «Record con protezioni Sheets mancanti»
  con elenco candidato — data (DD-MM-YYYY) — ora e nota «repair separato, non automatico»;
  mantenuto il blocco disallineamento Notion/Supabase; aggiunto conteggio «Convocato» da
  `Stato Approvazione`.
- `HTTP - Invia Stato`: `jsonBody = {{ $json.body }}`, niente più Markdown.
- Test: comando simulato via webhook, output verificato nell'execution (conteggi corretti,
  protezione mancante elencata con nome candidato, resa HTML pulita).

## T02 — Pulizia `pending_reschedule` su annulla (P02) ✅ testato

- Nuovi callback `sposta_cancel:<pageId>` e `assegna_cancel:<pageId>`:
  - `Code - Build Opzioni Sposta`: bottoni «❌ Annulla» e «✖️ Annulla sessione» → `sposta_cancel`.
  - `Code - Confirm Assegna`: bottone «Annulla» → `assegna_cancel` (prima usava `ritira_annulla:x`).
  - Whitelist azioni in `IF - Is Reset Action` estesa; nuova rule 49 nello
    `Switch - Routing Callback` → nuovo nodo `Postgres - Cancel Pending Annulla`
    (DELETE mirato per page_id) → `Postgres - Annulla Sessione` (pulizia `telegram_pending`)
    → `HTTP - Cancella Messaggio Sessione`. Il fallback dello switch è scalato a indice 50.
  - I path `sposta_confirm`/`assegna_exec` non sono stati toccati (il pending serve fino a conferma).
- **Bug aggiuntivi trovati e corretti nello stesso ramo:**
  - Il cleanup TTL (`Postgres - Cleanup Pending Reschedule`) era collegato all'output [1]
    di `Trigger Timeout Telegram`, che è uno scheduleTrigger a singolo output: **non era mai
    partito**. Ricollegato all'output [0]. Verificato live: al run orario successivo il
    pending scaduto da 26h è stato rimosso.
  - `IF - Ci Sono Scaduti` usava `$input.all().length > 0` con `alwaysOutputData` a monte
    (errore E04): **il run orario falliva ogni ora** (`Notion - Reset Scaduto` su page_id
    undefined) e l'alert finiva nel gruppo morto. Corretto con filtro su `page_id`.
    Run orario delle 22:00 verificato success.
  - `HTTP - Notifica Pending Reschedule Cleanup` aveva un'espressione rotta dall'origine
    (sequenza `}}` adiacente che chiude prematuramente il template n8n) più emoji corrotte:
    riscritta (HTML, emoji 🧹, brace spaziate) e validata con invio reale controllato.
- Test E2E: riga pending di test inserita → callback `sposta_cancel` simulato con message_id
  reale → riga eliminata, `telegram_pending` pulito, messaggio chiuso, execution success.

## T04 — Telegram HTML + Error Handler (P06, E01) ✅ testato

- Workflow Telegram convertito integralmente a HTML con escaping:
  `HTTP - Conferma Reset`, `HTTP - Conferma Reset Sheets`, `HTTP - Comando Non Riconosciuto`,
  `HTTP - Notifica Timeout`, `HTTP - Notifica Riprendi`, `HTTP - Invia Agenda`,
  `Code - Formatta Agenda`, `Code - Format Candidato`, `Code - Format Slot Detail`,
  `Code - Build Opzioni Sposta`, `Code - Confirm Assegna` (anche data `DD/MM/YYYY` → `DD-MM-YYYY`),
  `Code - Build Orari Assegna` (emoji corrotte ??), `Code - Show Resp Prima Assegna`,
  `Code - Show Resp Seconda`, `Code - Show Talent`, `Code - Build Cal Week 0/Nav`
  (asterischi letterali → grassetto HTML).
- Error handler (P06): `chat_id` → `-1003726144586`, emoji 🚨, HTML con escaping, timestamp
  Europe/Rome `DD-MM-YYYY HH:MM`, body pre-serializzato nel Code node. Corretto anche un bug
  pre-esistente per cui il messaggio d'errore usciva come `[object Object]` (l'errore vive in
  `execution.error`, non al top level).
- Workflow principale (inattivo): vecchio gruppo sostituito in `Postgres - INSERT Telegram
  Pending`, `Telegram - Chiedi Disponibilità`, `HTTP - Notifica Soglia Candidati` (riscritto:
  aveva anche un template malformato e `??`); subject di `Gmail - Riepilogo Non Schedulabili`
  (`??` → ⚠️).
- Test live (update simulati, risposte verificate nelle execution): comando sconosciuto,
  `/riprendi_scheduling`, `/reset` → schermata conferma → `reset_cancel`, scheda candidato
  (`cerca_candidate`), dettaglio slot (`verifica_slot`, date DD-MM-YYYY), `/calendario` +
  navigazione settimane (settimana 1 parziale 07–09/10 conforme a E10). Error handler testato
  end-to-end con errore controllato: alert arrivato nel gruppo HR e leggibile; validato anche
  su un errore reale (run orario delle 21:00, vedi sopra). `Code - Formatta Agenda` e
  `HTTP - Notifica Timeout` validati solo sintatticamente (girano su trigger schedulati).
- I messaggi di test inviati nel gruppo durante la sessione sono stati cancellati.

## T05 — Preflight workflow principale (P04) — resta INATTIVO

Checklist GO/NO-GO al momento della verifica:
- Candidati: 2 (entrambi di test), `Colloquio Schedulato` + `Da Approvare` → **0 candidati
  `Approvato`** → all'attivazione il ramo convocazioni (Mod3, poll 5 min) non invierebbe nulla.
- `automation_deliveries` vuota: nessun invio bloccato in `sending`/`error`, nessun rischio
  di doppio invio pendente.
- Calendar ID: manca su «Gigi d'Alessio» (candidato di test, slot 07-10-2026 09:00).
- Trigger all'attivazione: cron 9:00 gated dal periodo REC (giugno fuori finestra → no-op);
  riepilogo 10:00 con guardia caso-vuoto presente; Gmail Mod4 filtra `is:unread` con subject
  specifici («Convocazione colloquio conoscitivo», «Spostamento colloquio JEToP», «Colloquio
  JEToP») — **prima di attivare**: controllare che in hr@jetop.com non ci siano mail non
  lette con quei subject.
- Decisione utente: workflow lasciato inattivo.

## T06 — Idempotenza convocazioni ✅ verificato

- Vincolo `UNIQUE(page_id, delivery_type, slot_key)` presente a DB; schema completo.
- Lock conforme: INSERT `sending` con ON CONFLICT, riprocesso solo per `error`/`pending`/
  `sending`>30min; `Marca Inviata` a fine catena. Nessuna correzione necessaria.
- Rischio residuo accettato (design): se la catena fallisce tra `Google Calendar - Crea
  Colloquio` e `Marca Convocazione Inviata`, il retry dopo 30 minuti può creare un secondo
  evento Calendar (l'email resta protetta dal lock solo se il fallimento avviene prima di
  `Gmail - Invia Convocazione`). Finestra piccola; eventuale hardening: delivery separate
  per email ed evento.

## T07 — Notion date PATCH (P05, E06) ✅ testato

- `Notion - Aggiorna Slot` (flusso reschedule email, unico update di `Data Colloquio`)
  sostituito in place con HTTP Request `PATCH /v1/pages/{pageId}` (credential `notionApi`
  predefinita, header `Notion-Version: 2022-06-28`), stesse properties di prima:
  Data Colloquio (date), Ora, Responsabile 1°/2° Area, Membro Talent, Luogo.
  Nome/connessioni invariati; il nodo Calendar a valle legge da `Code - Parse Scelta`
  (nessuna dipendenza dall'output).
- `Notion - Crea Proposta` resta nodo Notion (creazione pagina, non critica).
- Test: PATCH no-op controllato sul candidato di test (stesso valore `2026-10-07`) via
  workflow temporaneo con identica configurazione credential → 200, data verificata nel
  response. Workflow di test eliminato.

## T08 — Secondo colloquio / output [1] — NESSUNA MODIFICA (confermato da Daniel)

- `IF - Secondo Colloquio` out[0] (secondo colloquio): `INSERT telegram_pending` (ora con
  chat corretto) → `Telegram - Chiedi Disponibilità` → approvazione nel WF Telegram
  (`yes` → luogo → `confirm` → `Notion - Approva Candidato` imposta `Approvato` + Luogo →
  `Postgres - Salva Slot Secondo`) → la convocazione parte da Mod3.
- out[1] (primo colloquio): non è un buco — `Notion - Crea Proposta` imposta già
  `Stato Approvazione='Approvato'` e Mod3 (poll 5 min sui candidati `Approvato`) esegue
  lock → Calendar → email. Collegare out[1] a `Formatta Data` romperebbe il riferimento
  `$('Aggregate - Comprimi Lookup')` (esiste solo nell'esecuzione Mod3 — E05) e duplicherebbe
  il percorso. Lasciato non connesso per design, come confermato.

## Rischi residui / follow-up consigliati

1. **Token bot hardcodato** negli URL di decine di nodi HTTP in tutti e tre i workflow:
   migrare a credential n8n/variabile, fuori scope di questa sessione.
2. **Protezioni Sheets mancanti**: 1 record (slot 07-10-2026 10:00) con `protection_id_*`
   null — ora segnalato da `/stato`; repair da eseguire come intervento separato.
3. **«◀️ Indietro» nella schermata opzioni sposta** non cancella il pending (callback di
   navigazione condivisi senza page_id): coperto da sovrascrittura ON CONFLICT al riuso e
   dal TTL ora funzionante.
4. **Attivazione del principale**: prima di attivare, ripetere il controllo candidati
   `Approvato`/delivery e verificare le mail non lette in hr@jetop.com con i subject del
   trigger Mod4; il Calendar ID mancante di «Gigi d'Alessio» va sistemato o il candidato
   rimosso (è un dato di test).
5. `Code - Formatta Agenda` e `HTTP - Notifica Timeout` convertiti ma non eseguibili a
   comando: osservare il primo run schedulato (agenda mattutina / timeout approvazione).
