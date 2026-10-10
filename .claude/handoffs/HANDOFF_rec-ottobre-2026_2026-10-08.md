# REC Ottobre 2026: automazioni n8n, dashboard colloqui e gestione candidati (stato al 08/10 sera)

**Date:** 2026-10-08
**Status:** IN PROGRESS (REC in corso; nessun lavoro di codice a metà)
**Bead(s):** none
**Epic:** REC Ottobre 2026 JEToP (automazione recruitment)
**Chain:** `standalone-e011a4a8` seq `1`
**Parent:** `none — first in chain`
**Prior chain:** none — first in chain

> **Nota privacy.** Questo repo (siti-torino) è pubblico. Per scelta dell'utente qui non ci sono
> dati personali: i candidati sono indicati con iniziali o con un indizio, mai con email o telefoni.
> Non ci sono credenziali né ID di credenziali n8n. Nomi completi e contatti si leggono su Notion
> o nella dashboard. Non aggiungere dati personali a questo file.

---

## Reference Documents

- Questo file è l'unico riferimento: il repo siti-torino non ha CLAUDE.md né bibbia di progetto.
- Repo JEToP privato `JEToP/recruitment-form-jetop` (form, banchetti, dashboard REC). L'utente ha chiesto di **non mettere lì note o file di lavoro**: le note vanno in siti-torino.
- `JEToP/recruitment-form-jetop/docs/n8n-auth-google.md`: documento già esistente sul login Google del sito.

## The Goal

L'utente è Daniel Leocata, area Talent di JEToP. Gestisce il **REC Ottobre 2026**, il recruitment dei nuovi soci, interamente automatizzato su n8n (https://n8n.jetop.com):
- form di candidatura;
- esclusioni automatiche;
- scheduling dei colloqui sulle disponibilità dei responsabili (foglio Google);
- convocazioni via email;
- gestione delle risposte dei candidati (conferme, spostamenti);
- solleciti e annullamenti;
- bot Telegram per il team;
- dashboard web su rec.jetop.com.

Il mio ruolo è **monitorare, correggere ed estendere** queste automazioni mentre il REC è in corso, e gestire i casi dei singoli candidati.

Obiettivo finale: il REC deve girare da solo, con gli umani che intervengono solo dove serve una decisione. L'utente ha chiesto anche questo: "voglio un calendario nella dashboard, dove posso vedere e gestire tutti i colloqui, praticamente tutte le cose che posso fare da telegram". La dashboard è completata in 3 fasi, tutte in produzione.

## Where We Are

- **REC attivo.** La finestra di scheduling parte dal 28/09/2026: righe 4..295 dei fogli disponibilità (settimane 1-4). Scheduling automatico alle 9:00, lunedì-sabato, con preavviso di 2 giorni.
- **Calendario finale deciso da Daniel il 10/10:** conoscitivi fino a giovedì 22/10 compreso, tutto il giorno (la sera c'è il fantasocio, ma Daniel ha chiesto di lasciare il giorno intero disponibile); venerdì 23/10 invio delle email; 26-29/10 colloqui tecnici, organizzati a parte (niente settimana 5 nei fogli, il bot non fissa i tecnici).
  - `app_settings.rec_scheduling_windows`: finestra «ottobre» con `end` portato dal 18/10 al **22/10** (solo `Trigger Schedulato` delle 9:00 dipende dalla finestra). Per tornare indietro si rimette `"end":"2026-10-18"`.
  - `blocked_slots`: 23/10 tutto il giorno («Invio email, niente colloqui») e 24/10 tutto il giorno («Fine conoscitivi, niente colloqui»), 24 righe (i blocchi del 22 sera sono stati tolti), si tolgono dalla dashboard (Sblocca) o con un `delete` su quelle date.
  - Fogli disponibilità (M&C, D&V, IT, S&P, T&DA): **tutte** le colonne «Colonna di …» ora vanno da riga 5 a riga 271 (22/10 19:00), anche per i responsabili uscenti che prima si fermavano alla riga 222. Celle 224-271 bianche, 272-295 (23 e 24/10) grigie (0.88) e protette solo da «Struttura» (hr@). Nota sulla cella A223 con il calendario. Prima della modifica: uscenti fino a 222, nuovi e T&DA fino a 295.
- **Tutti i 14 workflow REC sono attivi.** La tabella completa con i trigger è in Evidence & Data. C'è un backup spento (`vhL5QItyosHVLwH6`) da non riattivare.
- **Dashboard calendario colloqui:** fasi 1, 2 e 3 completate e in produzione su `rec.jetop.com/rec-dashboard/colloqui/`. Ultimo merge su main: `39e3449`, che include la fase 3 (`7e7bc24` su dev).
- **Backend dashboard:** workflow n8n `REC — Dashboard Colloqui API` (`iD9mAViOGLgKM2M9`, 72 nodi), con i webhook `rec-colloqui-lista` (GET) e `rec-colloqui-azione` (POST).
- **Annullamento automatico dei colloqui non confermati: attivo.** Workflow `AsjcIF8Sab5ZpXFH`, cron `*/10 8-21 * * *`.
  - Primo giro reale il 08/10 alle 19:30 e alle 19:40: due candidati annullati (J.E.V. e C.A.), email inviate, esecuzioni 63575 e 63583, ultimo nodo `HTTP - Avviso Gruppo`.
- **Solleciti limitati a 3** (`MAX_SOLLECITI = 3` in `Code - Chi Sollecitare`). Nessun sollecito se il colloquio è a meno di 24h, perché in quel caso subentra l'annullamento. Il testo del sollecito ora avvisa dell'annullamento.
- **Mod3 (convocazioni ogni 5 minuti):** corretto il dedupe. Prima, riportare un candidato a uno slot già convocato in passato bloccava la nuova convocazione (caso L.). La correzione è nel lock `Postgres - Lock Convocazione`.
- **Error handler `PhG1qNhFlOhzhMME`:** gli errori DNS (`ENOTFOUND`, `getaddrinfo`, `DNS server`, `server is offline`) contano come transitori e si segnalano solo dopo 3 ripetizioni. Prima arrivava spam di avvisi su Telegram.
- **Sub risposte candidati (`i0ZbyVmKy5ae7XuM`):** corretta la regex che taglia la parte citata delle email. Prima "on" dentro "non" troncava il messaggio (casi Gr., Ca., Ro.).
- **Approvazione dei nuovi colloqui: automatica.** L'utente: "togli la cosa che dobbiamo approvarle noi". Lo scheduling mette direttamente `Slot Confermato` e Mod3 convoca.
- **Le disponibilità di Daniel del 16/10 sono state rimosse**: non è disponibile dalla sera del 15/10 al 20/10. I colloqui di quel giorno sono stati spostati dove possibile; gli altri candidati hanno ricevuto un'email che annuncia lo spostamento.
- **L. riconvocato al 13/10 alle 19:00**, scelto dall'utente tra le opzioni. Commissione: un responsabile S&P, uno IT e Daniel come Talent.
- **M.P.:** il Membro Talent è stato cambiato con una persona della sotto-area Talent (prima era una persona di Data Analysis).
- **Te. (12/10 13:00) e D.L. (14/10 11:00):** spostati, email inviate in automatico.
- **4 candidati in "Da Ricontrollare"** (iniziali C., G., D., A.): email inviata. Lo scheduling delle 9:00 legge anche questo stato, quindi dovrebbe riassegnarli quando ci sono slot.
- **Due candidati con vincoli, ancora senza slot compatibile.** Entrambi hanno ricevuto la risposta "stiamo cercando":
  - Gri. (risponde "non posso il lunedì");
  - Ro. (martedì mattina; lunedì, mercoledì e venerdì pomeriggio; giovedì dalle 16).
- **Lista d'attesa: 14 candidati senza slot.** Il collo di bottiglia sono le **disponibilità dei Talent**. Sul gruppo Telegram è stato pubblicato un messaggio che chiede ai Talent di spuntare ore, con l'elenco delle ore più utili.
- **Skill di handoff installate.** Le skill `handoff` e `handoffplan` (repo REMvisual/claude-handoff, licenza MIT) sono in `~/.claude/skills/` del container, che è temporaneo, e in `siti-torino/.claude/skills/`, che resta per le sessioni future su questo repo.
- **Tre canali temporanei di test rimasti attivi su n8n**, creati da sessioni precedenti e mai cancellati:
  - `__tmp_g` `y9BO62IWZzbqHYUe` (creato il 04/10);
  - `__tmp_g` `ZZm9X9wIqTs09Whf` (22/09);
  - `__tmp_sql` `hqWHW9U5HbXUSdW9` (28/09).

  Il classificatore ha negato la cancellazione via API, quindi vanno eliminati a mano dalla UI di n8n. Sono innocui (webhook protetti da un segreto casuale ormai perso) ma esposti.
- **Gli script di supporto nel container sono persi** a fine sessione (scratchpad temporaneo). Vanno ricreati, vedi Quick Start. Non sono nel repo per scelta: contengono ID di credenziali e il repo è pubblico.

## What We Tried (Chronological)

Sessione lunga, dal 22/09 all'08/10, con 5 compattazioni di contesto. Le voci sono in ordine di tempo.

1. **Date nel foglio disponibilità (22/09).**
   - Problema: cambiare la data in `⚙️ Config!C4` non aggiornava i fogli. Le etichette data in colonna A erano testo fisso, con un calendario sbagliato (giorni della settimana del 2024).
   - Correzione: rigenerate a mano le 18 etichette (28/09 → 17/10) in tutti i 5 fogli.
   - L'anno era scritto fisso (`2026-`) in `Algoritmo Scheduling`, `Code - Trova Slot Alternativi` e `State - Out` (workflow `VlgUU8NTZLZeCrNW`). Ora è calcolato con `annoDa(gg, mm)`: dalla finestra REC in `app_settings.rec_scheduling_windows` per lo scheduling, dall'anno corrente con tolleranza di 180 giorni altrove.
2. **Campo nome dei banchetti.** Accettava email e password dall'autocompletamento.
   - Aggiunti `sembraUnNome()` (3-60 caratteri, niente `@`, niente cifre, solo lettere con spazi, apostrofi, trattini e punti) e `iniziali()` per la capitalizzazione.
   - Pulizia dati: 1 email convertita nel nome reale, 4 nomi normalizzati.
3. **Bug "un solo item" in molti nodi n8n.**
   - In "runOnceForAllItems", `$json` è solo il primo item: i nodi sono stati riscritti con `$input.all()`, `itemMatching(i)` oppure la modalità runOnceForEachItem.
   - Postgres con `queryBatching: 'independently'` per mantenere l'accoppiamento degli item. Ha risolto anche gli errori "Multiple matches found".
4. **Lettura risposte email: da Gmail Trigger a polling ogni 2 minuti.**
   - Il Gmail Trigger Mod4 perdeva email; ora è disabilitato.
   - Al suo posto `Trigger 2min - Risposte`, poi `Gmail - Cerca Risposte` con la query su oggetto e le etichette `REC/Gestita` (Label_9) / `REC/Errore` (Label_10), poi il sub-workflow eseguito un'email alla volta.
5. **Scelta tra le opzioni di spostamento (caso M.O., esecuzione 40209).**
   - `Postgres - Check Pending` usava l'id del messaggio Gmail invece dell'id della pagina Notion, così la candidata riceveva di nuovo le stesse opzioni.
   - Corretto. Sistemato tutto il percorso di scelta: chiavi `rp`/`rs` con il suffisso "(Area)", partecipanti del calendario, verifica che lo slot sia ancora libero.
6. **Gemini.**
   - Il piano gratuito ha 20 richieste al giorno ed era esaurito. Passati al modello Gemma `gemma-4-26b-a4b-it`; il parser estrae l'ultimo oggetto JSON dalla risposta.
   - Aggiunto il riconoscimento della scelta via regex ("2", "la seconda", "opzione 2"), senza passare dal modello.
   - Niente risposte automatiche Gemini ai messaggi generici (scelta dell'utente): vanno su Telegram come "Messaggio da gestire" e l'email viene rimessa come non letta.
7. **Sotto-aree** (T&DA Talent/Data, D&V Design/Videomaking, M&C Marketing/Communication, S&P Sales/Partnership).
   - La tabella sta in `⚙️ Config!B13:D` (Nome | Area | Sotto-area).
   - Il Membro Talent deve essere solo della sotto-area Talent. Per i responsabili d'area si preferisce la sotto-area del candidato, con ripiego sull'area intera.
   - Un "Head" vale per tutte le sotto-aree della sua area.
   - Workflow `HQxuUrIPubqppVC5`: raggruppa le colonne per sotto-area con `moveDimension` e colora le intestazioni. Le celle unite fanno fallire gli spostamenti, quindi prima si separano, poi si sposta, poi si riuniscono. Testato su una copia del foglio.
8. **Conferma presenza e solleciti.**
   - Nuove proprietà Notion: `Conferma Presenza` (In Attesa / Sollecitato / Ha Risposto / Confermata), `Data Convocazione`, `Ultimo Avviso`, `Solleciti Inviati`.
   - Sollecito ogni 24h dall'ultimo avviso, interrotto quando il candidato conferma o risponde.
   - **Poi (08/10):** massimo 3 solleciti, e annullamento automatico se non conferma entro 24h dal colloquio.
9. **Preavviso.**
   - All'inizio 24h; poi l'utente ha chiesto 2 giorni (`PREAVVISO_GIORNI = 2`, `ANTICIPO_GIORNI = 2` in Controlla Periodo).
   - Per gli spostamenti chiesti dal candidato (email e Sposta da Telegram) il preavviso minimo è **3 ore**, non 2 giorni.
10. **Carico distribuito.** Un responsabile S&P veniva scelto sempre anche quando altri erano liberi. Ora l'algoritmo prova tutte le combinazioni rs/talent e bilancia il carico. Stesso criterio nello spostamento.
11. **Approvazione su Notion → automatica.**
    - Prima c'era `Da Approvare` su Notion. Un caso (C.An.) non ha ricevuto la convocazione perché nessuno approvava: 25 colloqui erano fermi.
    - L'utente ha chiesto di togliere l'approvazione: ora è automatica.
    - Il secondo colloquio nello stesso slot va in `In Attesa Conferma Team`: Telegram chiede il luogo (Sala riunioni o Ufficio), senza limite di tempo per scegliere.
12. **Bot Telegram uniformato.**
    - Ogni messaggio modifica quello del callback, con "✖️ Annulla sessione" e i pulsanti indietro.
    - Le graffe `}}` nei body JSON statici rompono le espressioni n8n: ora sono scritte come `{`/`}`.
    - Nuove funzioni: Aggiungi responsabile, Cambia responsabile, Sposta (3 opzioni), Ritira con liberazione delle celle, Agenda con la commissione.
13. **Email dei candidati.**
    - Tolta la firma "sent automatically with n8n" da 12 nodi Gmail (`appendAttribution=false`).
    - Gli alias del mittente venivano scartati: aggiunto Risolvi Mittente più un avviso per i mittenti sconosciuti.
    - Le email di esclusione restavano bloccate su "sending" quando Gemini era sovraccarico: ora `onError` continua con un testo di riserva.
14. **Form candidature.**
    - Valori tradotti dal browser ("Masterly", "Yes"): mappati di nuovo in italiano con `normalizza()`. Lato HTML, valori delle opzioni fissi (PR #24/#25 in siti-torino).
    - Università "Altro": stato `Da Verificare` più avviso su Telegram. UniTO scritta in "Altro" viene esclusa in automatico (regex anche con "á").
15. **Banchetti: ore massime configurabili.**
    - Era un limite fisso di 6h (`user_bookings < 4` turni da 1h30). Ora c'è la chiave `max_hours` in `banchetto_config` (0 = nessun limite) e un campo nel pannello admin.
    - Commit `95c5f02` su dev, merge `5b65786` su main.
16. **Fogli: settimana 4.** `copyPaste` duplicava gli intervalli delle formattazioni condizionali, rendendo grigie celle libere. Rimossi gli intervalli della settimana 4 da 23 regole. `salute.py` controlla tutti gli intervalli.
17. **Dashboard calendario.**
    - Fase 1: lista, effettuato, ritira, luogo.
    - Fase 2: disponibilità, prenota, blocca, sblocca.
    - Fase 3: commissione, tecnico, fantasocio, accetta, rifiuta.
    - Test locali con Playwright e API simulata, poi push su dev, verifica su staging tramite fetch da n8n, poi merge su main.
18. **Bug della dashboard trovati in test.**
    - Una risposta vuota di n8n veniva presa per successo: ora qualsiasi `j.ok !== true` è un errore, e le letture hanno retryOnFail.
    - Università vuota nella lista: n8n accorcia le chiavi (`property_universit`), risolto con un fallback sul prefisso.
    - Testo "null" nel picker: risolto con `filter(Boolean)`.
    - La select della commissione restava su "— scegli —": risolto con `prima && ...`.
19. **Tentativi negati dal classificatore (non ripetere uguali):**
    - token admin temporaneo;
    - webhook "ponte" generico che esegue callback del bot;
    - cancellazione di branch remoti;
    - l'08/10 anche la cancellazione dei canali `__tmp` rimasti e la copia nel repo pubblico degli script con gli ID delle credenziali.
20. **Spam di avvisi DNS (08/10):** errori di risoluzione DNS dell'host n8n. È un problema dell'infrastruttura (IT), non dei workflow; l'error handler ora li tratta come transitori.

## Key Decisions

- **Approvazione automatica dei colloqui.** Alternativa scartata: approvazione su Notion, che bloccava le convocazioni.
- **Annullamento automatico dei non confermati:** stato `Ritirato` più email, scelta consigliata e accettata.
  - Alternative scartate: solo avviso al team, oppure stato "Da Riprocessare" senza email.
  - Testo dell'email: **senza invito a riscrivere** (scelta dell'utente).
  - Attivo subito, anche sui casi già in corso.
- **Massimo 3 solleciti**, e nessun sollecito sotto le 24h dal colloquio, per non sovrapporsi all'annullamento.
- **Preavviso:** 2 giorni per lo scheduling automatico; 3 ore minime per gli spostamenti chiesti dal candidato. L'utente: "quando loro chiedono di spostarlo non ci dev'essere il limite di 2 giorni".
- **Niente date promesse senza disponibilità reale.** L'utente: "non possiamo sempre dire di si". Le opzioni di spostamento escono solo da slot con persone libere.
- **Dashboard con API dedicata e autenticata** (Bearer più ruolo rec/admin).
  - Alternativa scartata: un "ponte" generico verso il bot Telegram, negato dal classificatore e meno sicuro.
  - Esclusi dalla dashboard: `/reset`, `/reset_sheets`, pausa e ripresa dello scheduling.
- **Sotto-aree in una tabella su Config**, non nei nomi delle colonne: le coordinate salvate in `slot_assignments` restano valide.
  - I colori e l'ordine si sincronizzano ogni ora (`:05`, escluse le 9 per non interferire con lo scheduling).
- **Gemma al posto di Gemini** per la quota giornaliera, più una regex diretta per le scelte.
- **Note di lavoro e handoff in siti-torino, non nel repo JEToP** (richiesta dell'utente dell'08/10). In siti-torino, che è pubblico, niente dati personali né ID di credenziali.
- **Massimo 2 colloqui per slot.** Il secondo richiede la Sala riunioni o l'Ufficio; il primo è nella Sala Ex Allievi.

## Evidence & Data

### Workflow n8n (stato live 08/10 sera)

| ID | Nome | Trigger | Note |
|---|---|---|---|
| `HirNCuA0RjYvVPp1` | Workflow REC JEToP (principale) | `0 9 * * 1-6` scheduling; ogni 5 min Mod3; `0 10 * * 1-6` riepilogo Mod2b; ogni 2 min risposte; `15 9-20 * * *` solleciti; Notion Trigger Escluso; webhook `form-cand-status` | 89 nodi. Gmail Trigger Mod4 disabilitato |
| `i0ZbyVmKy5ae7XuM` | REC — Risposte candidati (una email alla volta) | chiamato dal principale | 67 nodi |
| `m7WJCsjBYyPYRmUT` | Workflow Telegram JEToP | Telegram Trigger; timeout ogni ora; mattutino `0 8 * * *` | 351 nodi |
| `iD9mAViOGLgKM2M9` | REC — Dashboard Colloqui API | webhook `rec-colloqui-lista`, `rec-colloqui-azione` (POST) | 72 nodi |
| `AsjcIF8Sab5ZpXFH` | REC — Annullamento colloqui non confermati | `*/10 8-21 * * *` | 18 nodi, nuovo l'08/10 |
| `OQHWrx3WCHIUMyTc` | REC — Promemoria dopo i colloqui | `35 9 * * 1-6` | 4 nodi |
| `HQxuUrIPubqppVC5` | REC — Sotto-aree fogli (ordine e colori) | `5 7,8,10-21 * * *` | 10 nodi |
| `jiCiWpiLd2wt8wva` | REC — Pulizia responsabili aggiunti | `20,50 * * * *` | 15 nodi |
| `zPRbwd7oiCUd647v` | REC — Dashboard Stats API | webhook `rec-stats-0c3ca2d1f337` | 10 nodi, senza errorWorkflow |
| `FLAIlj9UWzZjOYbx` | Form Candidature Web | webhook `form-cand-send-code`, `form-cand-verify`, `form-cand-submit` | 37 nodi |
| `VlgUU8NTZLZeCrNW` | REC — Disponibilità Colloqui API | webhook `colloqui-login`, `-verify`, `-state`, `-save` | pagina web disponibilità, da usare nel prossimo REC (oggi si usa ancora il foglio Excel) |
| `QMElucsw0TaxcxoO` | REC — Banchetti Volantinaggio API | webhook `banchetti-*`, `auth-google` | 53 nodi |
| `K6R9pe8ahsr3zgTH` | REC — Reminder Banchetti | ore 9 | |
| `PhG1qNhFlOhzhMME` | JEToP Error Handler | Error Trigger | errorWorkflow di quasi tutti |
| `vhL5QItyosHVLwH6` | Workflow REC JEToP - BACKUP | (spento) | **non riattivare** |
| `y9BO62IWZzbqHYUe`, `ZZm9X9wIqTs09Whf`, `hqWHW9U5HbXUSdW9` | `__tmp_g` / `__tmp_sql` | webhook temporanei | **da eliminare a mano** |

### Risorse dati

| Risorsa | ID / dove | Dettagli |
|---|---|---|
| Foglio disponibilità responsabili | `1WJIBqGCH4ZBBBT8p_UWvOcxblhDwNedd0IO5evQd72Y` | gid: M&C `666945027`, D&V `1496408726`, IT `1064096795`, S&P `1321516727`, T&DA - Talent `161019212`, ⚙️ Config `1261763214` |
| Tabella sotto-aree | `'⚙️ Config'!B13:D120` | Nome / Area / Sotto-area. Protetta: solo Talent |
| Data inizio REC | `⚙️ Config!C4` | 28/09/2026, 4 settimane |
| Struttura fogli area | — | Riga intestazione = quella con colonna B = "Ora". Nomi da colonna C (indice colonna 0-based = indice array). Date senza anno ("Lun\n19/10"). Settimane 1-4 su righe 4..295 |
| Foglio soci | `1U04TgdFQKunz6_qNio9p7Fv1eMFyIk2wvWQOHWeAvRQ`, tab "Lista soci attivi" (gid `1288456596`) | colonne "Nome Completo", "Mail della JE" (F). hr@ può solo leggere. Mail sbagliate corrette nel codice con la mappa `_MAIL_GIUSTE` |
| Notion Candidati REC | DB `3655c5721def80fe93d2f422e8e35ba2` | proprietà sotto |
| Notion Config REC | `056f3c16251444aea91c0384e9646022` | |
| Gmail hr@jetop.com | etichette `REC` = Label_8, `REC/Gestita` = Label_9, `REC/Errore` = Label_10 | |
| Gruppo Telegram team | chat_id `-1003726144586` | ogni azione da dashboard scrive "🖥️ Dalla dashboard · email" |
| Postgres (credenziale "Postgres account" in n8n) | tabelle sotto | |

### Notion: stati usati

| Proprietà | Valori |
|---|---|
| Stato | Candidatura Ricevuta, Da Ricontrollare, Da Verificare, Colloquio Schedulato, Colloquio Effettuato, Ritirato, Escluso (e stati post-colloquio: tecnico, fantasocio, accettato o rifiutato) |
| Stato Approvazione | Da Approvare (non più usato), In Attesa Conferma Team (secondo colloquio nello slot), Slot Confermato, Convocazione Inviata, Da Riprocessare |
| Conferma Presenza | In Attesa, Sollecitato, Ha Risposto, Confermata |
| Risposte (dal 10/10) | **Ultima Risposta**: testo delle risposte del candidato, le più recenti in alto, ognuna con «dd/MM HH:mm · …», massimo 1.900 caratteri. **Data Ultima Risposta**: data e ora. Le scrive `Notion - Segna Risposta` nel sub; per chi era già in «Ha Risposto» il 10/10 le ho recuperate dalla casella hr@ |
| Altre | Data Convocazione, Ultimo Avviso, Solleciti Inviati, Responsabili Aggiuntivi, Luogo, ID Evento, Prima/Seconda Area, Prima/Seconda Sotto-Area, responsabili (formato "Nome (Area)"), Membro Talent |

Verificare con l'API i nomi esatti delle proprietà prima di scriverci: alcuni sono riportati a memoria.

### Postgres: tabelle

| Tabella | Chiave e colonne | Uso |
|---|---|---|
| `slot_assignments` | PK `page_id`; per ogni ruolo (`rp`, `rs`, `talent`): `sheet_name_*`, `gid_*`, `row_*`, `col_*`, `protection_id_*` (NOT NULL); `data_colloquio`, `ora_colloquio` | celle bloccate per ogni colloquio |
| `slot_extra` | unique (`page_id`, `gid`, `col`); `row_num`, `protection_id` | responsabili aggiunti |
| `blocked_slots` | unique (`data`, `ora`); `motivo` | slot bloccati da Telegram o dashboard |
| `pending_reschedule` | `page_id`, `options` (JSON con chiavi `data`, `ora`, `formatted_date`, `rp`, `foglio_rp`, `gid_rp`, `row_rp`, `col_rp`, `rs…`, `talent…`) | proposte di spostamento aperte |
| `telegram_pending` | `page_id`, `slot_data` JSON | solo secondi colloqui in attesa del luogo |
| `automation_deliveries` | unique (`page_id`, `delivery_type`, `slot_key`); `status` (`sending`/`sent`) | dedupe email. Tipi: `convocazione`, `email_esclusione`, `onboarding`, `esclusione_tecnico`, `annullamento`. `slot_key` = `YYYY-MM-DD\|HH:MM` |
| `app_settings` | chiave `rec_scheduling_windows` | finestre di scheduling |
| `banchetto_auth`, `banchetto_members`, `banchetto_config`, `banchetto_bookings` | | login Google condiviso e banchetti |
| `n8n_error_throttle`, `n8n_error_occurrences` | | soglie dell'error handler |

`banchetto_config`: `max_people=4`, `min_hours=3`, `max_hours` (nuovo; 0 = nessun limite, predefinito 6), `open=true`, `admins` (indirizzi di ruolo areait@ e hr@), `members_only=false`. Ogni turno dura 1h30.

### Commit (repo JEToP/recruitment-form-jetop)

| Commit | Ramo | Contenuto |
|---|---|---|
| `95c5f02` | dev | banchetti: ore massime configurabili |
| `5b65786` | main | merge banchetti |
| `3d366db` | dev | dashboard colloqui fase 1 |
| `550d00f` | main | merge fase 1 |
| `4ba2972` | dev | dashboard colloqui fase 2 |
| `391de83` | main | merge fase 2 |
| `7e7bc24` | dev | dashboard colloqui fase 3 |
| `39e3449` | main | merge fase 3 |
| `30bbd11` / `cc8ffad` | dev / main | statistiche ogni 5 minuti, solo con la scheda visibile |
| `d01abb4` / `575f8c9` | dev / main | «Annulla e rimetti in attesa» |
| `c45890a` / `70edeb7` | dev / main | testi di «Tieni occupato» |
| `bb310c7` / `8a50e2f` | dev / main | risposte dei candidati in «Da gestire» |
| `7b4591a` / `8a0209e` | dev / main | «Cerca candidati» sotto il calendario |
| `208c3c9` / `6859fc1` | dev / main | filtri della ricerca in una finestra |
| `6b878d5` | dev | correzione: «Sposta colloquio» restava su «Leggo i fogli disponibilità…» quando non c'erano proposte (variabile `sposta` non definita in `pickerScelta`, introdotta con `d01abb4`) |
| `0767d8f` / `60d7082` | dev / main | rifinitura delle due dashboard |
| `9a1a2f8` / `1e8d618` | dev / main | nuovo sistema grafico (hallmark) per modulo di candidatura, banchetti, statistiche e colloqui; la pagina pubblica resta invariata |
| `c07c196` / `b8e7545` | dev / main | colloqui: liste richiudibili in ogni punto («Nascondi» per sezione, «Mostra meno» in cima e in fondo, anche nella ricerca); mostrato a Daniel prima del merge |
| `a804190` / `021a7a9` | dev / main | colloqui: «Dopo il colloquio» e «In attesa di uno slot» diventano viste rapide della lista «Candidati» (richiesta di Daniel: «se c'è già la lista dei candidati con i filtri, non ha senso tenere altre schede separate»); mostrato prima del merge (dev e main attuali) |

In siti-torino sono state unite le PR #21-#25 (banchetti con redirect e capienza, dashboard senza candidature, limite sulla motivazione, menu a tendina immuni alla traduzione).

### Numeri raccolti

| Misura | Valore | Quando |
|---|---|---|
| Celle bloccate attese = blocchi = celle grigie | 33 = 33 = 33, 0 orfane | fine settembre |
| Celle dei fogli coerenti | 202 | inizio ottobre (salute.py) |
| Convocazioni inviate dopo lo sblocco dell'approvazione | 25 | inizio ottobre |
| Workflow senza errori al controllo di salute | 11 su 11 | inizio ottobre |
| Candidati in attesa di slot | 14 | 08/10 |
| Annullamenti automatici al primo giro | 2 (esecuzioni 63575 e 63583) | 08/10 19:30 e 19:40 |
| Loop infinito Telegram (noop) | 465 iterazioni, poi rimosso | fine settembre |
| Quota Gemini gratuita | 20 richieste/giorno, poi passaggio a Gemma | fine settembre |
| Durata colloquio | 40 minuti | |
| Capienza slot | 2 colloqui (il secondo in Sala riunioni o Ufficio) | |

### Azioni della dashboard (`rec-colloqui-azione`, POST con `{action, pageId, ...}`)

| Azione | Cosa fa |
|---|---|
| `effettuato` | Notion Stato → Colloquio Effettuato |
| `ritira` | libera le celle (regole di formattazione condizionale e protezioni), cancella l'evento del calendario, righe del DB, Notion Ritirato |
| `luogo` | luogo del secondo colloquio (Sala riunioni / Ufficio), con la stessa logica della conferma su Telegram |
| `disponibilita` | slot liberi con le persone disponibili (per il picker) |
| `prenota` | assegna o sposta uno slot (`forza`, `luogo` per il secondo nello slot, minimo 3h di preavviso) |
| `blocca` / `sblocca` | `blocked_slots`, anche `tuttoGiorno` |
| `commissione`, `cambia`, `aggiungi` | mostra, cambia o aggiunge responsabili (celle, DB, calendario, Notion) |
| `tecnico`, `escludi_fanta`, `ripesca` | esiti dopo il colloquio |
| `accetta`, `rifiuta` | email Gmail con lock su `automation_deliveries` |
| `presenza` (dal 10/10) | segna «Conferma Presenza» = Confermata, dopo aver gestito la risposta di un candidato in «Ha Risposto» |
| `rimetti` (dal 10/10) | «Annulla e rimetti in attesa», per quando non c'è un altro slot. Libera celle ed extra, cancella l'evento e le righe nel DB, rimette Notion come lo script dell'8/10 (`Da Ricontrollare`, data, ora, commissione, luogo, evento e conferma vuoti). Manda l'email di rinvio con lock `automation_deliveries` di tipo `rinvio` per lo slot. Con `blocca: true` (casella «Tieni occupato questo orario») inserisce anche l'orario in `blocked_slots`: l'orario resta occupato e non riceve altri colloqui. I testi distinguono «commissione liberata» da «orario occupato» o «orario disponibile». Script `fase4_dash.py` (86 nodi), provato a secco e su un candidato finto |

Ogni azione scrive sul gruppo Telegram chi l'ha fatta. Auth: Bearer token del login Google del sito (localStorage `jetop_banchetti_auth`), verificato su `banchetto_auth`; ruolo `rec` o `admin` da `banchetto_members` oppure dagli admin in config. CORS `allowedOrigins`: `https://rec.jetop.com`, staging, `http://localhost:3999`.

### Comandi del bot Telegram

`/start`, `/stato`, `/verifica`, `/reset`, `/reset_sheets`, `/pausa_scheduling`, `/riprendi_scheduling`, `/blocca_slot`, `/libera_slot`, `/fantasocio`, `/esiti`, `/rifiuta`, più l'agenda e i pulsanti inline: Sposta, Assegna, Ritira, Aggiungi responsabile, Cambia responsabile, Colloquio fatto, Ripesca, scelta del luogo dei secondi colloqui.

### Regex del taglio delle citazioni (sub, `Code - Parse Email` e `HTTP - Avvisa Messaggio Da Gestire`)

```
/(^|\n)[ \t>]*(Il giorno |Il (lun|mar|mer|gio|ven|sab|dom)\w* |On )[^\n]{0,200}(\n[^\n]{0,200})?(ha scritto|wrote)\s*:|(^|\n)[^\n]*(ha scritto|wrote)\s*:\s*(\n|$)|(^|\n)[^\n]*(-----Original Message-----|Da: .*\n.*Inviato:)/i
```

### Aggiunta al lock delle convocazioni (principale, `Postgres - Lock Convocazione`)

```
OR (automation_deliveries.status='sent' AND '{{ String($json.calendar_event_id || '').replace(/[^A-Za-z0-9_-]/g, '') }}' = '')
```

Serve a riconvocare quando lo slot è uguale a uno già convocato in passato ma l'evento non esiste più.

## Code Analysis

**Ciclo di vita di un candidato**

1. **Form** (`FLAIlj9UWzZjOYbx`).
   - Verifica del codice email, poi la candidatura su Notion: `Candidatura Ricevuta`, oppure `Da Verificare` se l'università è "Altro".
   - Se è escluso (requisiti), il nodo `Notion - Trigger Escluso` del principale manda l'email di esclusione (Gemini/Gemma con testo di riserva) e la registra in `automation_deliveries` come `email_esclusione`.
2. **Scheduling alle 9:00** (`Trigger Schedulato`).
   - `Controlla Periodo` (`ANTICIPO_GIORNI = 2`) legge i 5 fogli, la Config delle sotto-aree e l'occupazione dal DB (1 per ogni assegnazione, 99 se bloccato, 0 per gli extra).
   - `Algoritmo Scheduling`:
     - candidati `Candidatura Ricevuta` e `Da Ricontrollare`, ordinati per Data Candidatura;
     - `PREAVVISO_GIORNI = 2`;
     - prova tutte le combinazioni rs/talent, con 4 passate di preferenza di sotto-area e bilanciamento del carico.
   - Blocca le celle (formattazione grigia più protezione), salva `slot_assignments` e mette Notion a `Slot Confermato`, oppure `In Attesa Conferma Team` se è il secondo colloquio nello slot.
3. **Mod3, ogni 5 minuti.**
   - Prende `Slot Confermato`, crea l'evento sul calendario (partecipanti dalla mail JE del foglio soci) e invia la convocazione.
   - Oggetto "Convocazione colloquio conoscitivo"; "Spostamento colloquio JEToP: nuova data" se c'era uno slot precedente.
   - Mette `Convocazione Inviata`, Conferma `In Attesa`, Solleciti 0, Data Convocazione e Ultimo Avviso.
4. **Riepilogo alle 10:00 (Mod2b).** Email e Telegram con i candidati non schedulabili e il motivo, per area e sotto-area ("D&V · Design 1 su 2").
5. **Risposte, ogni 2 minuti.** Ricerca Gmail, poi il sub-workflow:
   - `Code - Parse Email` (taglio delle citazioni, testo da HTML);
   - `Notion - Trova Candidato` (email del mittente e stato Colloquio Schedulato; gli alias passano da Risolvi Mittente);
   - `Postgres - Check Pending` (id Notion). Se c'è una proposta aperta, va sul ramo scelta: regex, poi Gemma.
   - Altrimenti `Gemini - Classifica Intent`:
     - `conferma` → Presenza Confermata;
     - `modifica` → `Code - Trova Slot Alternativi`: 3 opzioni con persone libere e distinte, sotto-area Talent, minimo 3h; vanno in `pending_reschedule` e l'email "Opzioni disponibili" mostra "(1, 2 o 3)" adattato al numero;
     - `altro` → email rimessa non letta e avviso Telegram.
   - Scelta valida: controllo che lo slot sia ancora libero, libera le celle vecchie, prenota le nuove, aggiorna Notion, calendario (PATCH dei partecipanti) e DB, poi email di conferma.
6. **Solleciti alle :15, dalle 9 alle 20.** `Code - Chi Sollecitare`:
   - stato `In Attesa`/`Sollecitato`;
   - 24h dall'Ultimo Avviso;
   - `Solleciti Inviati < 3`;
   - inizio del colloquio meno ora ≥ 24h.
7. **Annullamento ogni 10 minuti, dalle 8 alle 21** (`AsjcIF8Sab5ZpXFH`). Un candidato per giro. Condizioni:
   - Stato `Colloquio Schedulato` e Approvazione `Convocazione Inviata`;
   - Conferma `In Attesa`/`Sollecitato`;
   - 3h ≤ (inizio − ora) ≤ 24h;
   - Data Convocazione ≥ 24h fa;
   - nessun `annullamento` già in `automation_deliveries`.

   Azioni, in ordine: lock, libera le celle, cancella l'evento, cancella le righe del DB, Notion (`Ritirato`, `Da Riprocessare`, Luogo vuoto, ID Evento vuoto), email "Il tuo colloquio con JEToP del dd/mm è stato annullato", segna come inviata, avviso su Telegram.
8. **Promemoria dopo i colloqui** alle 9:35: ricorda al team gli esiti da registrare.

**Pattern di blocco delle celle**
- **Prenotare:** regola di formattazione condizionale con formula `=TRUE` e sfondo grigio (0.6, 0.6, 0.6) sulla cella, più `addProtectedRange` (editor hr@jetop.com, descrizione `Slot assegnato - non modificare`).
- **Liberare:** si legge `?fields=sheets(properties.sheetId,protectedRanges(protectedRangeId),conditionalFormats(ranges))`, poi si cancellano le regole che toccano la cella (indice decrescente, `startRowIndex = r-1`, `startColumnIndex = c`) e le protezioni.

SQL per raccogliere le celle di un candidato:

```sql
SELECT COALESCE((SELECT json_agg(x) FROM (
  SELECT gid_rp AS gid, row_rp AS r, col_rp AS c, protection_id_rp AS p FROM slot_assignments WHERE page_id='X'
  UNION ALL SELECT gid_rs,row_rs,col_rs,protection_id_rs FROM slot_assignments WHERE page_id='X'
  UNION ALL SELECT gid_talent,row_talent,col_talent,protection_id_talent FROM slot_assignments WHERE page_id='X'
  UNION ALL SELECT gid,row_num,col,protection_id FROM slot_extra WHERE page_id='X') x),'[]'::json) AS celle
```

**Costanti e dettagli**
- Ora legale: cambio il 2026-10-25, calcolato a mano in `Parse Scelta`. Dopo quella data, controllare gli orari degli eventi.
- Il Membro Talent deve stare nella sotto-area Talent (o essere un Head di T&DA).
- Le tre persone della commissione devono essere distinte.
- Telegram Sposta propone solo inizi tra le 8 e le 16 (limite rimasto).
- Reset settimanale da Telegram: salta la scheda Config e cancella solo le protezioni `Slot assegnato - non modificare`.

**Insidie n8n imparate (costose da riscoprire)**
- In un Code node "runOnceForAllItems", `$json` è il primo item: usare `$input.all()`.
- In "runOnceForEachItem" si restituisce un oggetto, oppure `null` per scartare l'item.
- Dentro `={{ }}`, una sequenza `}}` chiude l'espressione: separare le graffe (`} }`) o usare `}`.
- Il `pageId` del nodo Notion va passato come `{__rl:true, value, mode:'id'}`, non come stringa.
- Notion: `matchType: 'allFilters'`.
- Le query Notion filtrate vanno fatte con un POST con body.
- I nodi Postgres con 0 righe non emettono item: usare `alwaysOutputData` dove serve.
- Ordine di esecuzione v1: i rami partono dall'alto verso il basso secondo la posizione sul canvas.
- I sub-workflow vanno pubblicati prima dei workflow che li chiamano. Execute Workflow v1.1 `mode:'each'`.
- PUT via API: mandare solo `name`, `nodes`, `connections` e le `settings` ammesse. **Mantenere `id` e `webhookId` dei nodi**, altrimenti i webhook si registrano di nuovo.
- Nodi Gmail: `appendAttribution: false` (niente firma n8n).
- Chiamare i webhook di produzione per prova genera falsi avvisi su Telegram: per controllare che n8n risponda usare `/healthz`.
- Fogli: `moveDimension` fallisce con celle unite, quindi prima si separano, si sposta e si riuniscono. `copyPaste` duplica le formattazioni condizionali.
- Nella risposta della lista, n8n accorcia chiavi lunghe come `property_universit…`: leggere con un fallback sul prefisso.

**Query Gmail delle risposte** (`Gmail - Cerca Risposte`: getAll, simple:false, limit 20):

```
(subject:"Convocazione colloquio conoscitivo" OR subject:"Spostamento colloquio JEToP" OR subject:"Colloquio JEToP") -from:jetop.com -in:sent -label:REC-Gestita -label:REC-Errore newer_than:7d
```

Per far rielaborare un'email basta togliere l'etichetta `REC/Gestita`: il giro dei 2 minuti la riprende.

**Tecniche di test usate (sicure)**
- **Copia di prova:** si duplica il workflow come temporaneo, con un webhook al posto del trigger e le scritture (Gmail, Notion, Sheets, Postgres) sostituite da nodi Set/Code finti. Si lancia, si leggono i risultati, si cancella.
- **Esecuzione immediata di un workflow schedulato:** si imposta temporaneamente il cron al minuto successivo e poi si ripristina (modello `run_now.py`). Evitare le 9:00 e le 10:00.
- **Dashboard in locale:** `python3 -m http.server 3999` dalla radice del repo, con Playwright che simula le risposte dei webhook n8n tramite route.
  - Usare `playwright-core@1.56` installato nello scratchpad con `executablePath` di Chromium in `/opt/pw-browsers/` (verificare il percorso esatto con `ls`). Non lanciare `playwright install`.
  - Per fermare il server **non** usare `pkill -f "http.server 3999"`: termina anche la shell. Usare il PID.
- **Staging:** dal container non si raggiunge direttamente. Lo si legge con un workflow temporaneo n8n che fa un GET (modello `check_staging_col.py`). L'URL dello staging è tra gli `allowedOrigins` del webhook di `zPRbwd7oiCUd647v`.
- **Controllare che n8n risponda:** `https://n8n.jetop.com/healthz`. Non chiamare i webhook di produzione con payload di prova.

**Altre regole del foglio disponibilità**
- Dal 10/10 la settimana 4 la spuntano tutti i responsabili, ma solo fino al 22/10 compreso (prima solo la nuova lista). Le protezioni per colonna si chiamano "Colonna di <Nome>".
- **Nuove persone (10/10 sera):** «Daniele Munafo» in IT (colonna G, scritto come nel foglio soci perché l'invito del calendario trova l'email confrontando le parole del nome) e «Caterina Mana» in T&DA (Talent, colonna K, poi spostata dalla sincronizzazione delle sotto-aree nel gruppo Talent). Protezioni `Colonna di …` fino alla riga 271, con editor la persona e hr@. Gabriele Corazzari era nella richiesta iniziale ma Daniel l'ha tolto del tutto.
- **Config, colonna E «Dirige colloqui come Talent»:** con «Sì» una persona di un'altra area può essere Membro Talent usando la sua colonna di area (una sola colonna, quindi mai due colloqui alla stessa ora; la cella bloccata è nel suo foglio). Oggi «Sì» per Lorenzo Amadi e Vincenzo Gulotta (S&P), Christian Lo Vetere e Diego Campanale (IT; righe aggiunte in Config con sotto-area vuota). Regola scelta da Daniel: **prima i Talent**. Lo scheduling cerca prima uno slot con un Talent del foglio T&DA in tutte le combinazioni di sotto-area, e usa gli extra solo se non ne trova; le proposte di spostamento (email, Telegram, dashboard) mettono prima le opzioni con un Talent e completano con gli extra. **Per spegnere la funzione basta svuotare la colonna E**: il codice torna a comportarsi come prima.
- **Workflow toccati il 10/10 sera** (backup in scratchpad come `bk_talent_*.json`, perso a fine sessione):
  - letture della Config portate da `B13:D120` a `B13:E120` in 10 nodi HTTP;
  - `HirNCuA0RjYvVPp1`: `Algoritmo Scheduling` (`cerca(listaP, listaS, conExtra)`), `Prepara Riepilogo` (`talentBase` e `talent`);
  - `i0ZbyVmKy5ae7XuM`: `Code - Trova Slot Alternativi` (`raccogli(..., conExtra)`);
  - `m7WJCsjBYyPYRmUT`: `Trova 3 Slot Sposta` (e ora valorizza anche `shT`, prima `undefined`), `Prepara Resp Assegna` e `Show Talent`, `Scelte Cambio` e `Prepara Cambio` (callback `col.indiceFoglio` quando il Talent viene da un altro foglio);
  - `iD9mAViOGLgKM2M9`: `Code - Calcola Slot` (foglio Talent virtuale: persone T&DA più extra, ognuno con il suo foglio `f`). La forma della risposta non cambia, quindi `app.js` non è stato toccato.
- **Simulazione sui dati del 10/10 sera:** chi aveva già uno slot con un Talent lo tiene identico, e gli slot trovati passano da 15 a 33. Quasi tutti i colloqui in più hanno Diego Campanale come Talent (18), perché è l'unico degli extra con molte ore spuntate nelle settimane 3 e 4.
- Ogni responsabile ha la protezione della sua colonna (editor: la persona più hr@).
- La pagina web delle disponibilità (`VlgUU8NTZLZeCrNW`) è pronta ma si userà dal prossimo REC: oggi si usa ancora il foglio.

**Commit nel repo JEToP**: identità `Claude <noreply@anthropic.com>`, con le righe finali `Co-Authored-By` e `Claude-Session` indicate dal sistema nella sessione.

## Files Changed

### Repo JEToP/recruitment-form-jetop (privato; si modifica solo il codice del sito, niente note)
- `rec-dashboard/colloqui/index.html`, nuova pagina, conforme alla CSP (niente script inline):
  - login;
  - topbar con "📊 Statistiche" ed Esci;
  - sezioni Da gestire, Settimana (weekbar, daytabs, cal, agenda, legenda), Dopo il colloquio (`#dopo`) e In attesa di uno slot (`#waiting`);
  - stili picker `pk-*`, `dopo-h`, `pk-mail`.
- `rec-dashboard/colloqui/app.js`:
  - costanti `API = 'https://n8n.jetop.com/webhook'`, `LIST_PATH = 'rec-colloqui-lista'`, `ACTION_PATH = 'rec-colloqui-azione'`;
  - `api()` tratta come errore ogni risposta con `j.ok !== true`;
  - `azione()` restituisce true/false e ricarica i dati;
  - `finestra()` crea le finestre modali;
  - funzioni `apriPicker`, `pickerScelta`, `pickerRiepilogo`, `apriGiorno`, `apriCommissione`, `commissioneScelta`, `commissionePersona`, `commissioneConferma`, `dopoButtons`, `apriTecnico`, `apriAccetta`, `renderDopo`;
  - drawer con i pulsanti azione.
- `rec-dashboard/index.html`: link a Colloqui e Banchetti; il nome del brand è nascosto su mobile.
- **Redesign del 10/10 sera (`9a1a2f8`):** il sistema grafico è descritto in `.claude/design/DESIGN_rec-jetop.md` (questo repo) e va letto prima di toccare modulo, banchetti, statistiche o colloqui. In sintesi: foglio condiviso `assets/jetop-ui.css` (caricato con `?v=AAAAMMGG`: nginx non manda Cache-Control per i `.css`), token OKLCH indaco e iris, icone Phosphor (non più Lucide), intestazione comune con il selettore Statistiche / Colloqui / Banchetti, modulo a cinque tappe con avanzamento. **La pagina pubblica `/` non si tocca** (design Figma, richiesta di Daniel). Le regole sotto della rifinitura restano valide salvo le icone, ora Phosphor.
- **Rifinitura del 10/10 (`0767d8f`), regole da mantenere nelle due dashboard:**
  - icone solo SVG: in `colloqui/app.js` l'helper `icon(nome)` / `ib(nome, testo)` con i tracciati Lucide (ISC) in `ICONE`; niente emoji nell'interfaccia (il 🎉 resta solo nel testo dell'email di accettazione);
  - nei menu delle persone niente emoji: `gruppiPersone()` e `aggiungiGruppi()` creano gli `optgroup` «Disponibili», «Senza disponibilità», «Già impegnati in quest'ora»;
  - `:focus-visible` iris, `--ring` per i campi, `header`/`main`/`nav`, titoli di sezione `h2.section-t` (niente maiuscoletto), schede `h3`;
  - su telefono e touch tutti i comandi alti almeno 44 px, testi minimi 12 px, `tabular-nums`, `prefers-reduced-motion` rispettato (anche nei grafici);
  - niente `border-left` colorato su schede, risposte, avvisi e toast; resta solo sulle chip del calendario perché codifica l'area;
  - statistiche: griglie a colonne fisse (KPI e operatività 4 per riga, 2 su telefono; `.grid` a 2, `.grid.g3` a 3), date `dd/mm` con `ddmm()` e `titoloData()`, università raggruppate con `gruppoUni()`, `emptyMsg(id, testo)` per gli stati vuoti, `etichetta()` trasforma «(nessuno)» in «Non indicato»;
  - colloqui: «Cerca candidati» mostra 10 risultati senza filtri (50 con filtri), «Dopo il colloquio» e «In attesa» 8 righe con «Mostra tutti» (`riempiRichiudibile`, stato in `APERTI`).
- **Liste richiudibili (`c07c196`, 10/10):** richiesta di Daniel («quando estendo la scheda non posso nasconderla di nuovo, solo alla fine e solo in parte»).
  - Ogni sezione (Cerca candidati `#finder`, Dopo il colloquio `#dopo`, In attesa `#waiting`) ha un titolo `.sec-head` con il numero (`#nCerca`, `#nDopo`, `#nAttesa`) e un pulsante `.sec-toggle` («Nascondi»/«Mostra», `aria-controls`, `aria-expanded`). Le sezioni chiuse sono salvate in localStorage (`jetop_rec_sezioni`, `SEZ_CHIUSE`, `applicaSezioni()`), sempre in try/catch.
  - Liste aperte: «Mostra meno» sia in cima (`.more.top`) sia in fondo; richiudendo, `tornaAllaLista()` riporta all'inizio della lista e mette il focus sul pulsante.
  - Ricerca: dopo «Mostra altri» c'è «Mostra meno» (in cima e in fondo) che torna ai primi 10 (o 50 con filtri).
- **Una sola lista «Candidati» (`a804190`, 10/10):** le sezioni «Dopo il colloquio» (`#dopo`, `renderDopo`) e «In attesa di uno slot» (`#waiting`, `renderWaiting`) non esistono più, e con loro `riempiRichiudibile`/`APERTI`.
  - Sopra la ricerca ci sono le viste rapide `VISTE` (`.f-view`, `aria-pressed`): Tutti, In attesa di slot (`__attesa`), Da decidere (`__decidere`: Colloquio Effettuato con esito diverso da Escluso), Al tecnico (`Colloquio Tecnico`), Ripescabili (`__ripescabili`: Effettuato ed esito Escluso), Accettati (`Accettato`). Ogni vista imposta solo `FSTATE.fStato`, mostra il numero calcolato con gli altri filtri e una riga di aiuto (`#fHint`).
  - I gruppi stanno in `GRUPPI_STATO` e valgono anche nel filtro «Stato» della finestra (in più `__dopo` e `__fuori`). Lo stato scelto da una vista non compare tra le etichette dei filtri; uno stato scelto dalla finestra sì.
  - Ordine «data colloquio»: chi non ha colloquio va in fondo, in ordine di data di candidatura (così «In attesa» parte da chi aspetta da più tempo).
  - Le righe (`rigaCerca`) mostrano anche data di candidatura (se manca il colloquio), aree del tecnico, area di ingresso ed esito conoscitivo. Le azioni (tecnico, escludi, ripesca, accetta, rifiuta) restano nella scheda (`dopoButtons` in `openDrawer`).
- `banchetti/app.js`: impostazione "Ore massime a socio (0 = nessun limite)" (`#setMaxH`, `#setMaxHSave`, chiave `max_hours`).
- `Dockerfile` copia nel sito solo `index.html`, `app.js`, `landing.js`, `meta-pixel.js`, `candidati/`, `assets/`, `fonts/`, `banchetti/`, `rec-dashboard/`. `docs/` non viene pubblicata.

### Workflow n8n modificati l'08/10
- **Principale `HirNCuA0RjYvVPp1`:**
  - `Postgres - Lock Convocazione`: aggiunta la condizione sopra;
  - `Code - Chi Sollecitare`: `MAX_SOLLECITI = 3`, soglia di 24h e paragrafo "Se non riceviamo la tua conferma entro 24 ore prima del colloquio, dovremo annullarlo."
- **Error handler `PhG1qNhFlOhzhMME`:**
  - regex dei transitori estesa al DNS;
  - `periodico = isTriggerErr || execution.mode === 'trigger'` (soglia di 3 ripetizioni).
- **Sub `i0ZbyVmKy5ae7XuM`:** nuova regex delle citazioni.
- **Nuovo `AsjcIF8Sab5ZpXFH`** (annullamento) e **`iD9mAViOGLgKM2M9`** (dashboard API, fasi 1-3).

### Repo siti-torino (questo)
- `.claude/skills/handoff/`, `.claude/skills/handoffplan/`: skill di handoff (MIT, REMvisual/claude-handoff), con LICENSE.
- `.claude/skills/hallmark/` (MIT, nutlope/hallmark): skill usata per il redesign; `.claude/design/` contiene il sistema grafico e il registro di hallmark.
- `.claude/skills/ui-ux-pro-max/` (MIT), `.claude/skills/design-taste-frontend/` e `.claude/skills/redesign-existing-projects/` (MIT, Leonxlnx/taste-skill), `.claude/skills/impeccable/` (Apache 2.0 con NOTICE): skill di design usate per la rifinitura delle dashboard. Non eseguire `impeccable/scripts/impeccable`: scarica ed esegue un programma esterno; il contesto del progetto si legge a mano.
- `.claude/handoffs/HANDOFF_rec-ottobre-2026_2026-10-08.md`: questo file. Il sito è pubblicato da Vercel: il file `.vercelignore` esclude `.claude/` dal deploy. Il file resta comunque visibile su GitHub, perché il repo è pubblico.

### Script di lavoro (persi con il container, da ricreare)
- **Accesso:** `n8n.py` (API n8n), `gch.py` (canale temporaneo verso Google o Notion), `sqlch.py` (canale Postgres).
- **Creazione e deploy dei workflow:**
  - `crea_wf_dash_colloqui.py`, `fase2_dash.py`, `fase3_dash.py` (`python3 fase3_dash.py go` rifà il deploy mantenendo id e webhookId);
  - `crea_wf_annulla.py`.
- **Controlli e test:**
  - `salute.py` (controllo di coerenza tra fogli, DB, Notion ed errori di 11 workflow);
  - `check_staging_col.py` (fetch dello staging tramite n8n);
  - `test_*.py`, `smoke_prod.py`, `leggi_mail.py`.
- **Backup JSON dei workflow** prima di ogni PUT, per esempio `dash_backup_prima_fase3.json`, `main_backup_prima_max_solleciti.json`, `errhandler_backup_prima_dns.json`, `sub_backup_prima_taglio.json`. Tutti persi: prima di una modifica rifare sempre un backup con GET.

## User Feedback & Preferences (REQUIRED — never omit)
- **Prima di portare modifiche su `main` (rec.jetop.com), mostrarle a Daniel e aspettare il suo ok** (richiesta del 10/10, dopo che il redesign è andato online senza anteprima). Strumenti: schermate prima/dopo (script `pw/confronto.js`) e lo staging su `dev`. Deve sempre essere possibile tornare alla versione precedente.
- **Ultimo merge su main: `021a7a9`** (viste rapide nella lista Candidati); si annulla con `git revert -m 1 021a7a9` e torna a `b8e7545`.
- **Come tornare indietro dal redesign del 10/10:** su `main`, `git revert -m 1 1e8d618` e push. Provato in una copia separata: i file tornano identici a `60d7082` (la versione online prima del redesign). Per altre modifiche vale lo stesso schema: ogni pubblicazione è un merge `--no-ff` di `dev` su `main`, quindi si annulla con `git revert -m 1 <merge>`.

- **Rispondere sempre in italiano.** Quando sono passato all'inglese l'utente ha scritto "parla italiano".
- **Email ai candidati:**
  - non devono sembrare scritte da un'AI: niente lineette lunghe "—" e niente firma n8n;
  - **preparare una bozza e aspettare "inviala"/"inviale"** prima di inviare, salvo i percorsi già automatici;
  - mostrare il testo dell'email direttamente in chat: l'anteprima di AskUserQuestion non si vede ("non la vedo").
- **Mai chiedere token, chiavi o password in chat**: le credenziali vanno nelle impostazioni dell'ambiente.
- "togli la cosa che dobbiamo approvarle noi, dev'essere automatico."
- "non possiamo sempre dire di si": non promettere date di spostamento senza disponibilità reale.
- "non voglio avere un limite di tempo per scegliere il luogo" (secondi colloqui).
- "quando loro chiedono di spostarlo non ci dev'essere il limite di 2 giorni" (minimo 3h); lo stesso vale per lo Sposta di Telegram.
- "voglio che vengano inviati massimo 3 solleciti, e che a prescindere, in assenza di risposta 24 ore prima del colloquio, quest'ultimo venga annullato". Email di annullamento senza invito a riscrivere.
- "il colore si deve cambiare in automatico nel foglio, e devono essere messi uno accanto all'altro, senza nomi misti, attento alle disponibilità che hanno già dato".
- "nel messaggio che ci manda il bot telegram con le info sul candidato deve specificare le preferenze delle sottoaree".
- "deve dirmi anche chi fa i colloqui" (agenda).
- "in alcuni messaggi il bot telegram non fa tornare indietro… va uniformato".
- Nuove funzioni: verificarle prima, poi fare il merge. Esempi: "controlla che tutto funzioni nella parte 3 e fai il merge", "controlla che funzioni e poi fai direttamente il merge".
- Processo IT per il repo JEToP: push su `dev` (staging), verifica, poi merge su `main` (rec.jetop.com).
- 08/10: "non mettere nulla che ci serve nella repo di jetop, usa la mia per questa cose". Le note vanno in siti-torino, senza dati personali (repo pubblico).
- La config è modificabile solo dai Talent; ogni responsabile spunta solo la propria colonna.
- Su richiesta: niente risposte automatiche Gemini ai messaggi generici; un caso specifico "Non rispondere".
- Va bene agire subito sui casi urgenti ("sposta subito i possibili e agli altri manda un email…").

## Where We're Going

1. **Avvio della nuova sessione:** ricreare gli script di supporto (Quick Start), poi fare un controllo di salute di sola lettura:
   - esecuzioni con errori nelle ultime 24h;
   - coerenza tra celle grigie, protezioni e `slot_assignments`;
   - Mod3 e annullamenti regolari.
2. **Disponibilità Talent:** se sono comparse nuove ore, spostare Gri. (non il lunedì) e Ro. (martedì mattina; lunedì, mercoledì e venerdì pomeriggio; giovedì dalle 16).
   - Si fa dalla dashboard (picker `prenota`) o con lo Sposta di Telegram.
   - Mod3 manda in automatico l'email "Spostamento colloquio JEToP: nuova data".
3. **Dopo le 9:00:** verificare che lo scheduling abbia assegnato i 4 "Da Ricontrollare" e quanti dei 14 in attesa. Il riepilogo delle 10:00 dice il motivo per chi resta fuori.
4. **Monitorare gli annullamenti automatici** dei primi giorni: niente falsi positivi, cioè candidati che avevano confermato ma con Conferma Presenza non aggiornata.
5. **Prima del 25/10 (cambio dell'ora legale):** controllare gli orari degli eventi creati per i colloqui dopo il 25/10.
6. **Proposte ancora senza risposta,** da riproporre solo se utili:
   - prenotazione automatica della Sala Ex Allievi;
   - nascondere le date passate nelle liste di assegnazione;
   - togliere il limite 8-16 dello Sposta di Telegram;
   - vista Notion dei "non confermati";
   - modello di esclusione alternativo per i "Da Verificare";
   - rendere il bot admin del gruppo per cancellare i messaggi di comando.

## Risks & Blockers

- **DNS dell'host n8n:** errori di risoluzione intermittenti. Va sistemato dall'IT; i workflow lo tollerano, gli avvisi sono limitati. Anche i canali temporanei possono fallire per DNS: riprovare.
- **Disponibilità Talent:** è il collo di bottiglia, con 14 candidati in attesa. Non si risolve con il codice: serve che i Talent spuntino ore.
- **Deploy di rec.jetop.com:** a volte ci mette ore, una volta meno di un minuto. È fuori dal nostro controllo.
- **Classificatore in modalità auto:** nega le azioni distruttive o di modifica dei permessi su sistemi condivisi (cancellare workflow non creati nella sessione, branch remoti, token admin, canali generici). Non aggirare: spiegare all'utente e lasciargli l'azione.
- **Repo siti-torino pubblico:** mai committare dati dei candidati, email personali, ID di credenziali o segreti.
- **Quota di traffico Supabase (avviso del 09/10).**
  - Il Postgres "Postgres account" è un progetto Supabase sul piano gratuito. Contiene **anche il database interno di n8n** (esecuzioni, workflow, utenti, credenziali), per un totale di 845 MB, di cui 819 MB di `execution_data`.
  - Supabase ha segnalato il superamento della quota di traffico in uscita: il periodo attuale è tollerato; dall'08/11 vale la Fair Use e il limite è sotto i 5,5 GB al mese.
  - **Ogni richiesta autenticata a n8n** carica l'utente con tutti i permessi del ruolo (147 righe): circa 150 KB per l'editor e circa 430 KB per ogni chiamata API. Stima da `pg_stat_statements` dal 20/05: circa 4 GB per l'editor, circa 3 GB per le API (in gran parte le nostre), più credenziali, ruoli e caricamento dei workflow (Telegram è 511 KB).
  - **Quindi: ridurre al minimo le chiamate all'API di n8n.** Raggruppare le letture e creare canali temporanei solo quando servono: ogni canale costa 4 o più chiamate.
  - Le tabelle del REC sono minuscole e non sono il problema.
  - Ogni esecuzione costa circa 10-15 KB, soprattutto perché le credenziali vengono rilette a ogni nodo (circa 8 letture). Ci sono circa 8.000 nuove connessioni al giorno verso il pooler.
  - **Interventi del 09/10, approvati dall'utente:**
    - dashboard statistiche ogni 5 minuti e solo con la scheda visibile (prima ogni 60 secondi anche nascosta: circa 450 chiamate al giorno); commit `30bbd11` su dev, merge `cc8ffad` su main;
    - principale: `Trigger 2min - Risposte` gira ogni 2 minuti dalle 7 alle 22:59 e ogni 15 minuti di notte; `Trigger 5min - Mod3` gira ogni 5 minuti di giorno e ogni 30 di notte (Schedule Trigger con due regole cron);
    - Pulizia: `20,50 7-22 * * *`;
    - `saveDataSuccessExecution: 'none'` (le esecuzioni riuscite non vengono salvate) per Stats, Banchetti, Sotto-aree, Pulizia e Reminder banchetti. Principale, sub, Telegram, form, annullamento e Dashboard Colloqui restano salvati.
  - **Fotografie dei contatori** (`pg_stat_statements`, chiamate e righe per categoria) al 09/10 05:26 e 09:25 UTC. Per confrontarle, ripetere la stessa query raggruppata per categoria.
  - Soluzione strutturale da proporre all'IT: database interno di n8n su un Postgres locale al server di n8n. In alternativa: alzare `DB_POSTGRESDB_IDLE_CONNECTION_TIMEOUT`, abbassare `EXECUTIONS_DATA_PRUNE_MAX_COUNT`, oppure piano Pro di Supabase.
  - **Inventario del database del 09/10** (Supabase, database `postgres`, PostgreSQL 17.6, circa 900 MB):
    - **91 tabelle di n8n** in `public`, circa 888 MB, di cui `execution_data` circa 855 MB. Si riconoscono dalle colonne `createdAt`/`updatedAt`, più `migrations`, `settings`, `role_scope`, `scope`, `webhook_entity`, `workflow_statistics`, `execution_*`, `insights_*`, `oauth_*` e altre. L'ultima migrazione di n8n è `CreateAgentObservationTables1784000000000`. Funzione e trigger di n8n: `increment_workflow_version` su `workflow_entity`.
    - **16 tabelle REC e app**, circa 1,3 MB, da lasciare su Supabase:
      - `app_settings`, `automation_deliveries`, `blocked_slots`, `pending_reschedule`, `slot_assignments`, `slot_extra`, `telegram_pending`;
      - `banchetto_auth`, `banchetto_bookings`, `banchetto_config`, `banchetto_locations`, `banchetto_members`;
      - `colloqui_auth`, `form_email_verifications`, `n8n_error_occurrences`, `n8n_error_throttle`.
    - **Funzioni e trigger REC:** `update_updated_at` su `slot_assignments`, `banchetto_check_capacity` su `banchetto_bookings`.
    - **Supabase Auth e Storage non sono usati:** 0 utenti, 0 file.
    - **Migrazione (proposta da Ivan dell'IT):** PostgreSQL sul server JEToP solo per n8n, dump selettivo senza le 16 tabelle REC, conservando `N8N_ENCRYPTION_KEY`. La credenziale "Postgres account" dei workflow resta su Supabase.

## Open Questions

- Le disponibilità di Daniel dal 17 al 20/10 (non disponibile dalla sera del 15 al 20) sono state tolte o non c'erano? Verificare la sua colonna nel foglio T&DA.
- I casi più vecchi sono risolti?
  - 3 candidati non schedulabili di fine settembre (iniziali A.Z., C.T., E.S.);
  - un candidato in "Da Riprocessare" (C.R.);
  - un candidato straniero con il form tradotto (A.R.M.A.), a cui era stata offerta la ricandidatura senza risposta.
- Il Membro Talent di un candidato (N.B.) è rimasto una persona di Data Analysis per scelta dell'utente ("lasciamo così"): ok, ma va ricontrollato se viene spostato.

## Quick Start for Next Session

```bash
# 0. Contesto: leggi questo file per intero. Rispondi in italiano.

# REGOLA: ogni chiamata all'API di n8n costa circa 430 KB di traffico Supabase (quota gratuita).
#    Niente GET /workflows (l'elenco completo); una GET e una PUT per workflow; tutte le query SQL
#    in un solo canale; per leggere l'output di un canale usa responseMode lastNode invece di
#    interrogare /executions; niente controlli periodici via API.
# 1. Ambiente: N8N_API_KEY deve essere già impostata nelle variabili d'ambiente.
#    Mai chiederla in chat. Verifica che esista senza stamparla:
test -n "$N8N_API_KEY" && echo ok

# 2. Repo del sito REC (solo per modifiche al codice del sito, mai note):
#    collegalo con add_repo (owner JEToP, repo recruitment-form-jetop, access push), poi
git clone https://github.com/JEToP/recruitment-form-jetop /home/user/recruitment-form-jetop
cd /home/user/recruitment-form-jetop && git config user.name Claude && git config user.email noreply@anthropic.com
#    Lavora su dev, verifica lo staging, poi fai il merge su main. Per vedere i rami remoti usa git ls-remote.

# 3. Ricrea gli script di supporto nello scratchpad (NON nel repo pubblico):
#    - n8n.py: wrapper curl su https://n8n.jetop.com/api/v1 con l'header X-N8N-API-KEY.
#      Funzioni get(wid), put(wid, wf) (manda solo name/nodes/connections/settings ammesse),
#      node(wf, name), create(wf), delete(wid), activate(wid, on).
#    - gch.py: Channel(kind, write) crea un workflow temporaneo
#      (Webhook → Code che controlla un segreto casuale → HTTP Request con credenziale
#      predefinita → Respond), lo usa con .call(url, method, payload) e lo cancella
#      in __exit__. Tipi: googleSheetsOAuth2Api, googleCalendarOAuth2Api, notionApi,
#      googleDriveOAuth2Api, gmailOAuth2 (account hr@).
#    - sqlch.py: lo stesso schema con un nodo Postgres executeQuery, .q(sql).
#      Con 0 righe restituisce {"_raw": ""}.
#    Gli ID delle credenziali NON sono qui: leggili dal campo "credentials" dei nodi
#    del workflow principale (GET /workflows/HirNCuA0RjYvVPp1).
#    Cancella sempre i canali in __exit__, altrimenti restano attivi.

# 4. File chiave da leggere per primi:
#    - workflow HirNCuA0RjYvVPp1 (nodi "Algoritmo Scheduling", "Code - Chi Sollecitare", "Postgres - Lock Convocazione")
#    - workflow AsjcIF8Sab5ZpXFH (annullamento)
#    - workflow iD9mAViOGLgKM2M9 (API della dashboard)
#    - recruitment-form-jetop/rec-dashboard/colloqui/app.js

# 5. Verifica lo stato (sola lettura): esecuzioni con errori nelle ultime 24h
#    GET /executions?status=error&limit=50, poi incrocia Notion (Colloquio Schedulato)
#    con slot_assignments e con le celle grigie.

# 6. Prossima azione: controllare se ci sono nuove ore spuntate dai Talent e spostare
#    Gri. e Ro. nei loro vincoli (dalla dashboard o con lo Sposta di Telegram), dopo
#    averlo detto all'utente.
```

**Cose che l'utente deve fare a mano** (ricordarle se non sono fatte):
- Eliminare dalla UI di n8n i 3 workflow temporanei `y9BO62IWZzbqHYUe`, `ZZm9X9wIqTs09Whf`, `hqWHW9U5HbXUSdW9` (nome `__tmp_g` / `__tmp_sql`).
- Eliminare il branch `feat/banchetti-ore-massime` dal repo JEToP (GitHub → Branches → cestino).
- Far correggere le email nel foglio soci, colonna F, righe 30, 35, 46, 54, 64. Le mail giuste sono nella mappa `_MAIL_GIUSTE` del bot Telegram; serve qualcuno con permesso di modifica.
- Segnalare all'IT i problemi DNS dell'host n8n.
