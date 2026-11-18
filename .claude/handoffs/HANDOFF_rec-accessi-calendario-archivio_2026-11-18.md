# REC: accessi, Impostazioni, calendario senza foglio, archivio su Drive (aggiornato al 16/11)

**Date:** 2026-11-18
**Status:** IN PROGRESS (fasi 1, 2, 3a, 3c e archivio in produzione; nessun lavoro di codice a metà)
**Bead(s):** none
**Epic:** REC JEToP — piano «Accessi, calendario personale e pagina Impostazioni del REC» (6 fasi)
**Chain:** `standalone-e011a4a8` seq `2`
**Parent:** `HANDOFF_rec-ottobre-2026_2026-10-08.md`
**Prior chain:** `HANDOFF_rec-ottobre-2026_2026-10-08.md` > this

> **Nota privacy.** Il repo siti-torino è pubblico. Qui non ci sono dati personali: niente nomi di
> candidati (al massimo iniziali), niente email di persone, niente nomi dei soci in commissione,
> niente ID di credenziali n8n, niente ID di file o cartelle Drive con dati personali.
> Gli ID dei workflow n8n e i nomi delle tabelle invece ci sono: servono per lavorare.

---

## Since Last Handoff

- **Esiti del tecnico (punto 0 del passaggio precedente): chiusi.** Gli ingressi sono partiti in automatico (34 email `onboarding` `sent` in `automation_deliveries`), i rifiuti dopo il tecnico pure (41 `esclusione_tecnico` `sent`). A metà novembre restano 3 candidati di S&P in «Colloquio Tecnico» su Notion, mai chiusi.
- **Il focus è cambiato:** dall'11/11 Daniel ha chiesto di preparare il **prossimo REC (marzo 2027)**: rivedere chi entra dove nel sito, togliere il foglio Google delle disponibilità, avere un posto unico (Impostazioni) da cui gestire il REC, archiviare ogni edizione su Drive.
- **Il foglio delle disponibilità non si usa più dal 15/11:** tutti i workflow leggono le disponibilità dal database (Calendario del sito). I punti del passaggio precedente su celle grigie e protezioni sono superati.
- **Supabase:** il rischio della quota di traffico resta (vedi Risks), ma il database REC pesa 24 MB su 500; nessun problema di spazio.
- **Rischio «disponibilità Talent» del passaggio precedente:** a ottobre era il collo di bottiglia (14 in attesa). Con il Calendario ogni socio segna le ore dal telefono e T&DA vede «Disponibilità di tutti»: va verificato a marzo se basta.
- **Regola «mostrare prima di main»:** sempre rispettata (staging + foto + ok di Daniel) per fasi 1-3, archivio e griglia.
- **Proposte vecchie ancora aperte** (sala Ex Allievi, date passate nelle liste, limite 8-16 dello Sposta, vista Notion dei non confermati, ecc.): non riprese, nessuno le ha chieste.
- **16/11: il container di lavoro è stato sostituito.** Tutti i file locali sotto `/home/user/lavoro/` (script, banchi di prova, copie di sicurezza dei workflow) sono **spariti**. Produzione intatta. Vedi «Risks» e «Quick Start».

## Reference Documents

- Questo file e il precedente (`HANDOFF_rec-ottobre-2026_2026-10-08.md`, 970 righe, storia del REC di ottobre).
- `.claude/design/DESIGN_rec-jetop.md` (in questo repo): token, font, icone, sezioni «Impostazioni (12/11)» e «Calendario (13/11)». **Da aggiornare** con la griglia delle sezioni del 16/11 (vedi «Where We're Going»).
- Il piano completo stava in `/root/.claude/plans/abstract-orbiting-galaxy.md`: **perso con il container**. Il suo contenuto è riassunto qui sotto in «The Goal» e «Where We're Going».
- Repo privato `JEToP/recruitment-form-jetop` (sito rec.jetop.com): solo codice del sito, niente note.

## The Goal

Daniel Leocata (area Talent di JEToP) gestisce il REC, il recruitment dei nuovi soci, automatizzato su n8n (https://n8n.jetop.com) con Notion, Gmail, Google Calendar, Telegram e il sito rec.jetop.com. Dopo il REC di ottobre 2026 ha chiesto di preparare il sistema per **marzo 2027** (apertura candidature ipotizzata il **1° marzo**, «ancora da definire con certezza»), con una regola netta: **«a marzo dev'essere tutto pronto»**, niente doppio binario con il foglio.

Il piano in 6 fasi, approvato da Daniel:
1. **Accessi:** una regola unica nel database per chi vede cosa (banchetti per tutti i soci; Colloqui e Statistiche solo T&DA, IT e Board; Impostazioni solo admin). **Fatta.**
2. **Impostazioni:** pagina unica solo admin con persone e accessi, candidature, periodi dei colloqui, banchetti, esiti, registro. **Fatta.**
3. **Calendario personale e disponibilità senza foglio:** 3a pagina Calendario e tabelle (fatta), 3b scheduling sulle tabelle provato su ottobre (fatta), 3c passaggio di tutti i workflow dal foglio alle tabelle (**fatta il 15/11**), 3d preavviso/solleciti/annullamento nelle Impostazioni (**da fare**).
4. **Testi delle email** modificabili dalle Impostazioni (da fare).
5. **Sotto-aree** in un elenco unico (da fare).
6. **Chiusura e nuova edizione:** archivio, svuotamento delle tabelle, informativa privacy. **L'archivio su Drive è fatto (tasto nelle Impostazioni); il resto aspetta le decisioni di Daniel.**

## Where We Are

- **In produzione** (rec.jetop.com identico a `main` = `db85b51` al 16/11; n8n aggiornato; nessuna esecuzione in errore nelle ore successive alle pubblicazioni):
  - fase 1 accessi, fase 2 Impostazioni, fase 3a Calendario, **fase 3c** (15/11), **tabelle chiuse con RLS** (15/11), **archivio su Drive con tasto** (15/11), **griglia delle sezioni delle Impostazioni** (16/11).
- **Regola di accesso:** funzione Postgres `rec_accesso(email)` → `admin` | `rec` | `socio` | null. Config in `banchetto_config`: `aree_rec` = `T&DA,IT,Board`, `ruoli_senza_rec` = `Alumnus`, `admins` (indirizzi fissi). Conteggi al 12-13/11: admin 7, rec 30, socio 51, senza accesso 18.
- **Soci dal foglio:** workflow «REC — Soci dal foglio» `Pn9E3SiNg6jGxWAQ` legge ogni mattina la scheda «Lista soci attivi» del foglio soci giusto (quello indicato da Daniel il 12/11, letto dall'account hr@) e aggiorna `banchetto_members` (colonne `area`, `ruolo_je`, `fonte`, `attivo`); si ferma se il foglio è vuoto o dimezzato; salva l'esito in `banchetto_config.soci_ultimo_esito`. Chiamabile solo dall'Impostazioni API.
- **Impostazioni API** `FBOU4a97dlTvkPa3` (webhook `rec-impostazioni`, versione `f1135126-9cb4-4d23-acda-19f38c5139ed` dal 15/11): azioni `stato`, `persona_accesso`, `persona_aggiungi`, `persona_togli`, `soci_aggiorna`, `banchetti`, `colloqui`, `esiti`, `commissario_salva`, `commissario_togli`. Ogni modifica: una sola istruzione SQL con CTE `me`/`adm`/`err`/`fatto`/`log`, riga in `rec_impostazioni_log`, avviso nel gruppo Telegram.
  - Controlli aggiunti il 15/11: niente cambio di area per chi ha colloqui fissati; niente due persone con lo stesso nome tra chi fa i colloqui; l'inizio del periodo dei colloqui non può cambiare **settimana** se ci sono colloqui fissati (la fine sì).
- **Calendario API** `1ajgp9GLtPHDe6vq` (webhook `rec-calendario`): azioni `stato`, `salva`, `copia`, `tutti`. Il sabato l'ultima ora segnabile è le 13 (colloqui fino alle 14).
- **Archivio API** `1dh73nRHPk1e42gJ` (webhook `rec-archivio`, nuovo del 15/11, 20 nodi, versione `6cfcb0f0-dba8-478a-8bc2-ed76288aba45` al 16/11): azioni `stato` (edizioni da Notion, archivi creati) e `crea` (foglio Google). Solo admin.
- **Pagina Impostazioni** (`rec-dashboard/impostazioni/`): 8 sezioni in una **griglia di riquadri uguali con icona** (2 colonne fino a 720 px, 4 sopra), ordine: Persone e accessi, Chi fa i colloqui, Candidature, Colloqui, Banchetti, Esiti, **Archivio**, Registro.
- **Database (Supabase, Postgres 17.6, 24 MB su 500):**
  - tabelle nuove: `rec_commissari` (35 righe, 30 attive; colonna `colonna` dal 15/11), `rec_disponibilita` (email, giorno, ora; 0 righe in produzione al 14/11), `rec_impostazioni_log`, `rec_archivi`;
  - funzioni nuove della 3c: `rec_gid(area)`, `rec_foglio(p_oggi)`, `rec_spuntata(gid, col, giorno, ora)`, `rec_periodo(p_oggi)`;
  - trigger nuovi: `rec_commissari_colonna`, `rec_slot_email` (su `slot_assignments`), `rec_extra_email` (su `slot_extra`);
  - **RLS attiva su tutte le tabelle** di `public` (19 tabelle chiuse il 15/11; prima il ruolo pubblico `anon` le poteva leggere **e scrivere**, compresa `banchetto_auth` con i token).
- **Workflow passati dal foglio alle tabelle il 15/11** (nodi prima → dopo, versione nuova):
  - principale «Workflow REC JEToP» `HirNCuA0RjYvVPp1`: 92 → 79, `7eaa14d3-ce37-4096-84f2-2616d3267f81`;
  - «REC — Risposte candidati» `i0ZbyVmKy5ae7XuM`: 73 → 61, `73b887b1-a89c-4554-adb1-b66495b90da3`;
  - «REC — Annullamento colloqui non confermati» `AsjcIF8Sab5ZpXFH`: 18 → 13, `19a00593-c624-4d2c-bbc2-eb2071000e8d`;
  - «REC — Dashboard Colloqui API» `iD9mAViOGLgKM2M9`: 112 → 95, `3242fee7-2e52-455c-87e8-29c6158aee6c`;
  - bot «Workflow Telegram JEToP» `m7WJCsjBYyPYRmUT`: 351 → 315, `758d9c5f-2b79-481a-bb8d-8099ef799de3`.
  - Nessun nodo Google Sheets rimasto nei cinque workflow; 133 nodi Code tutti sintatticamente validi.
- **Archivio di ottobre 2026** su Drive, cartella «Archivio REC» dentro «[01] Talent» (drive condiviso dell'area Talent; proprietario l'account hr@). Versione buona del 16/11: **137 candidati, 104 colloqui, 35 entrati (7 in IT)**, 29 soci in commissione, 78 inviti al tecnico, 331 email, 49 soci ai banchetti (194 ore), 141 turni. Le tre versioni precedenti sono nel cestino di Drive. In `rec_archivi` c'è una sola riga (la buona).
- **Correzione su Notion del 16/11:** la candidata M.d.S. (entrata in IT) aveva la pagina senza edizione, esito e area: ora Edizione «Ottobre 2026», Stato «Accettato», Esito Tecnico «Entra», Area Ingresso «IT». Mancano ancora email, CV, prima/seconda area, aree ed esito del conoscitivo: li deve completare Daniel se servono.
- **Sito, cosa vede chi (fase 1):** menu in alto costruito dall'accesso. Tutti i soci: Calendario e Banchetti. Con accesso REC: anche Statistiche e Colloqui (`data-voci="4"`, due righe da due sotto i 520 px). Ingranaggio delle Impostazioni (`#impLink`) solo per gli admin. Senza accesso, `/rec-dashboard/…` riporta ai banchetti anche se il server risponde «non autorizzato». Il login è solo nella pagina dei banchetti, con ritorno alla pagina chiesta (`?torna=`). Il ruolo nel browser si aggiorna a ogni caricamento.
- **Pannello admin dei banchetti (12/11):** mostra le regole (aperte/chiuse, ore minime e massime, persone per turno) in sola lettura con un link a Impostazioni > Banchetti. Settimane e giorni dell'edizione arrivano dal server (`settimaneDa`, `impostaGiorni`, `edizioneRec`); niente più date scritte in `banchetti/app.js`.
- **Pagina Calendario (fase 3a, 13/11):** viste `disponibilita` (griglia della settimana, tocco per segnare, «Salva N modifiche», «Copia la settimana prima»; ore passate e ore con un colloquio non modificabili; il sabato dopo le 13 le celle sono `.fuori`), `colloqui` (i miei colloqui: giorno, ora, luogo, commissione) e `tutti` (disponibilità di tutti, solo T&DA e admin: persone libere per area e sotto-area, ore in cui una commissione completa è possibile, chi non ha segnato niente).
- **Pagina Impostazioni, contenuto delle sezioni:**
  - Persone e accessi: elenco con ricerca e filtri, accesso di ognuno, dai/togli accesso a mano, rendi admin, aggiungi socio fuori foglio, «Aggiorna dal foglio» con le anomalie del foglio soci;
  - Chi fa i colloqui: area, sotto-area, «può dirigere il colloquio» per persona; togli (vietato se ha colloqui fissati, e gli toglie le ore future);
  - Candidature: apertura/chiusura (azioni `candidature*` della Dashboard API);
  - Colloqui: assegnazione attiva/in pausa e periodi (`rec_scheduling_windows`, al massimo 8, senza sovrapposizioni);
  - Banchetti: aperte, giorni dell'edizione, regole; Esiti: link del gruppo e data dell'onboarding; Archivio; Registro (ultime modifiche).
- **Periodi dei colloqui in `app_settings` al 16/11:** «marzo» 2026-03-01 → 2026-03-31 e «ottobre» 2026-09-28 → 2026-10-22, entrambi finiti. **Il periodo di marzo 2027 non è ancora impostato.**
- **Workflow Esiti e Notion:** il workflow `19P81AlDynbuSyUu` ogni ora prende dall'edizione corrente (`Ottobre <anno>` da luglio, altrimenti `Marzo <anno>`) chi è «Colloquio Tecnico» con un «Esito Tecnico» e alle 18 manda l'email. Il solo trigger Notion attivo (`Notion - Trigger Escluso` nel principale) reagisce solo allo Stato «Da Escludere». Quindi: modificare a mano Stato ed Esito **insieme** non manda email.
- **Strumenti locali: persi.** Esiste solo `/home/user/lavoro/talent/n8n.py` ricostruito il 16/11 (vedi Quick Start) e `archivio/costruisci.js` + `archivio/bk_*_16-11.json` recuperati da n8n; anche questi spariranno al prossimo cambio di container.

## What We Tried (Chronological)

1. **(11-12/11) Fase 1, accessi.** Ipotesi: una funzione SQL `rec_accesso` chiamata a ogni richiesta dalle API rende l'accesso coerente ovunque e immediato. Fatto: colonne nuove in `banchetto_members`, workflow «Soci dal foglio», controlli in Banchetti API `QMElucsw0TaxcxoO`, Statistiche `zPRbwd7oiCUd647v`, Dashboard API, Convocazione tecnico `bN5TiyRrzWxmqdBm`, Reply API `biwo9PgpDHxAMz2m`; menu del sito costruito dall'accesso. Risultato: in produzione dal 12/11, elenchi confermati da Daniel (19 guadagnano l'accesso, 7 Head/AM e 3 Alumnus lo perdono, 16 non attivi escono, 38 nuovi soci entrano). Il classificatore della modalità auto bloccò i PUT la prima notte: niente aggiramenti, Daniel ha dato «vai» e sono passati.
2. **(12/11) Fase 2, Impostazioni.** Pagina solo admin + workflow «REC — Impostazioni API». Le date dei banchetti passate in `banchetto_config.giorni` (funzione `banchetto_giorni()`), le regole dei colloqui in `app_settings`. Daniel ha notato i comandi doppi nel pannello admin dei banchetti: tolti quattro (aperte/chiuse, ore minime, ore massime, persone per turno), ora solo in Impostazioni > Banchetti.
3. **(13/11) Fase 3a.** Tabelle `rec_commissari` (importate dalla scheda Config del foglio, 35 persone) e `rec_disponibilita`; pagina Calendario (le mie disponibilità con griglia 6 giorni × 12 ore, «Copia la settimana prima», i miei colloqui, disponibilità di tutti per T&DA e admin); colonne `email_rp/email_rs/email_talent` su `slot_assignments` (78 colloqui di ottobre abbinati).
4. **(13-14/11) Fase 3b, scheduling sulle tabelle.** `pianifica.js` riscriveva l'«Algoritmo Scheduling» con le email. Prova contro la storia vera di ottobre: **solo 4 colloqui su 42 coincidevano**, perché l'algoritmo è cambiato fino al 16/10 e del foglio resta solo lo stato finale. Criterio alternativo accettato da Daniel il 14/11: «stesse decisioni dell'algoritmo di oggi». Con quel criterio: 15 giri di ottobre (104 colloqui) e 600 scenari casuali (6.696 colloqui) identici, regole rispettate, tre errori messi apposta tutti rilevati.
5. **(14/11) Fase 3c, primo progetto scartato.** Il piano diceva di riscrivere ogni workflow per salvare email al posto delle celle. Inventario: **126 nodi** in 5 workflow toccano il foglio (principale 23, risposte 20, annullamento 3, Dashboard API 24, bot 56). Riscriverli tutti voleva dire cambiare molta logica non provabile dal vivo (Notion, Gmail, Telegram, Calendar insieme). Scartato.
6. **(14-15/11) Fase 3c, progetto adottato: il «foglio» ricostruito dal database.** Funzione `rec_foglio()` che restituisce le disponibilità nei tre formati che i workflow leggevano (righe del nodo Google Sheets, `values:batchGet`, tabella Config); ogni persona ha una **colonna fissa** (`rec_commissari.colonna`, uguale a quella del foglio di ottobre) e la scheda della sua area (`rec_gid`). I workflow restano uguali nella logica: cambia solo da dove leggono; spariscono le scritture sul foglio (celle grigie, protezioni). Le email della commissione le scrivono i trigger a partire da scheda+colonna.
   - Prova: il foglio ricostruito è **identico al foglio vero di ottobre casella per casella** (25 controlli).
   - Primo problema: i campi del formato «nodo Google Sheets» devono restare in ordine (`row_number`, colonna A, `col_2`, `col_3`…) perché il bot prende «il primo campo» come data; `jsonb` riordina le chiavi → righe costruite come testo `json`.
   - Secondo problema: una prima versione segnava come non spuntate le ore già occupate da un colloquio; il foglio vero no (la spunta resta, l'occupazione la dicono `slot_assignments`/`slot_extra`). Corretto.
7. **(15/11) Prove della 3c sui cinque workflow** (nodi veri eseguiti in node contro Postgres locale, uscita confrontata con i nodi di prima sugli stessi dati): tutte ok. La Dashboard dopo la fine del periodo non propone più ore (prima il foglio ne mostrava fino al sabato): differenza voluta.
8. **(15/11) Date del 2026 scritte a mano.** Trovate nell'ora legale (`2026-10-25` in «Formatta Data», «Code - Chi Sollecitare», «Code - Parse Scelta»), nelle settimane di `/calendario` del bot (7/10–28/11/2026), nell'anno di «Assegna» e di `/blocca_slot`. Sostituite con calcoli (fuso di Roma dalla data con `Intl`, periodo da `rec_periodo()`, anno corrente). Senza la correzione un colloquio del 30/03/2027 alle 15 sarebbe finito in calendario alle 16.
9. **(15/11) Prestazioni di `rec_foglio()`.** Prima misura: **2,4 s** a chiamata (troppo per il bot). Riscrittura con aggregazione per riga: ancora ~3 s. `EXPLAIN ANALYZE`: 1,43 s su 1,49 erano **compilazione JIT** (stime di costo enormi su una query piccola). `set jit = off` sulla funzione → **35 ms**, stesso risultato (md5 identico). In produzione `jit` risulta già `off`, ma la funzione lo forza comunque.
10. **(15/11) Pubblicazione 3c** con l'ok di Daniel («pubblica»): database → `controlla` (sola lettura) → Impostazioni API → principale → risposte → annullamento → Dashboard API → bot → sito (PR #49 su dev, PR #50 su main). Ogni PUT verificato con `activeVersionId == versionId`.
11. **(15/11) Sicurezza.** Daniel: «chiudi». Trovate 19 tabelle senza RLS (non 13 come stimato prima), scrivibili dal ruolo pubblico. RLS attivata su tutte; n8n (utente `postgres`, padrone delle tabelle) vede gli stessi conteggi di prima. Le funzioni nuove hanno `revoke execute` da `public`, `anon`, `authenticated`.
12. **(15/11) Archivio v1.** Script locale che legge Notion + database e crea un foglio Google formattato (8 schede) tramite workflow temporanei; prima un esempio con **dati finti** per guardare la grafica (PDF esportati), poi ottobre vero. Del file vero ho guardato solo il riepilogo (dati aggregati).
13. **(15/11) Archivio v2 sulle osservazioni di Daniel:** tabella aree × sotto-aree nel riepilogo, classifica delle ore ai banchetti, via la colonna «Ultima risposta del candidato» e la scheda «Registro», più leggibilità (righe più alte, giorno della settimana, stacco tra i giorni). In Notion la sotto-area di ingresso a volte è uguale all'area («M&C» per M&C): normalizzata ad «area intera».
14. **(15/11) Tasto «Crea l'archivio».** Daniel: «non vedo il tasto». Costruiti workflow «REC — Archivio API», tabella `rec_archivi`, scheda «Archivio» nelle Impostazioni. Prova del giro vero su una **copia temporanea dell'API senza controllo del token**: archivio creato in **16,2 s**, poi buttato e ripulito dal database. Pubblicato con l'ok di Daniel (PR #51/#52).
15. **(16/11) Griglia delle sezioni.** Daniel (foto dal telefono): «questi pulsanti a casaccio sono orribili, devono essere in ordine, lineari, facilmente individuabili». Le pillole che andavano a capo sono diventate una griglia di riquadri uguali con icone Phosphor; controllata da 360 a 1920 px. Pubblicata con l'ok (PR #53/#54).
16. **(16/11) Errore negli entrati.** Daniel: 35 entrati, 7 in IT; l'archivio diceva 34 e 6. Causa: una pagina Notion creata a mano il 22/10 senza edizione, esito e area. **Ho sbagliato a descriverla come «vuota, senza nome»**: il nome c'era, non l'avevo controllato. Daniel ha dato il nome; ho aggiornato la pagina con un solo PATCH (Stato «Accettato» insieme a Esito «Entra», così il workflow Esiti non manda l'email di benvenuto) e ricreato l'archivio.

17. **(15/11, sera) Come ho provato senza toccare la produzione** (da rifare allo stesso modo se servono altre modifiche ai workflow):
    - **Postgres locale** con una copia ridotta dello schema (`slot_assignments` con i vincoli NOT NULL veri, `slot_extra`, `blocked_slots`, `telegram_pending`, `pending_reschedule`, `rec_*`, `banchetto_*`, `app_settings`) e i dati di ottobre (34 persone con le colonne del foglio, 1.480 spunte, 78 colloqui, 6 aggiunti, 35 blocchi);
    - **codice dei nodi Code eseguito in node** con `new Function('$', '$json', codice)` e un finto `$('nodo')` che restituisce i dati dei nodi a monte; orologio fermato con una sottoclasse di `Date`; espressioni `{{ … }}` dei nodi Postgres valutate con lo stesso trucco e poi lanciate sul Postgres locale;
    - **confronto «vecchio contro nuovo»**: stesso scenario, nodo del backup sul foglio ricostruito come lo dava Google, nodo modificato su `rec_foglio()`; uscita confrontata con `JSON.stringify`;
    - **scenari casuali ripetibili** (generatore lineare con seme fisso): densità delle spunte, sotto-aree e «può dirigere» mescolati, candidati con aree sbagliate, colloqui già fissati, blocchi, aggiunti, orari di partenza diversi;
    - **sito**: server locale con la CSP vera (`srv_csp.py` sulla porta 3999), Playwright con le API simulate o con i nodi veri sul Postgres locale, controllo di visibilità da 360 a 1920 px, foto guardate a occhio;
    - **dopo la pubblicazione**: confronto dei file di rec.jetop.com con `origin/main` e prova di fumo (accessi, calendario, impostazioni) sullo staging e in produzione.
18. **(16/11) Container sostituito.** Alla richiesta sugli entrati `/home/user/lavoro` non esisteva più. Ricostruito un `n8n.py` minimo con `temporaneo()` e recuperato `costruisci.js` dal nodo «Code - Modello» dell'Archivio API. Per Notion ho usato workflow temporanei che restituiscono **solo conteggi** (nessun nome) e, per la correzione, una sola ricerca per cognome (indicato da Daniel).

## Key Decisions

- **Foglio ricostruito invece di workflow riscritti (3c).** Motivo: 126 nodi coinvolti, giro completo non provabile; con `rec_foglio()` la logica resta quella di ottobre e si prova confrontando le uscite dei nodi. Scartato: riscrivere i workflow su email (piano originale). Prezzo: i workflow ragionano ancora per «scheda e colonna» (debito tecnico, vedi Risks).
- **Criterio di equivalenza per lo scheduling:** «stesse decisioni dell'algoritmo di oggi» (Daniel, 14/11), al posto del confronto con la storia vera di ottobre (4/42 coincidenze, non significativo).
- **Colonna fissa per persona** (`rec_commissari.colonna`): nuova persona = prima colonna libera in fondo alla sua area; tolta e rimessa = stessa colonna; chi cambia area lascia una «(colonna libera N)» e gli altri non si spostano. Scartato: ricompattare le colonne a ogni modifica (romperebbe i colloqui già salvati).
- **Fuso di Roma calcolato** con `Intl.DateTimeFormat` (funzione `_fusoRoma`), non più date di cambio ora scritte a mano.
- **`/reset_sheets` del bot non cancellato:** il comando resta ma spiega che il foglio non si usa più (meno modifiche al bot, nessun comando che sparisce di colpo).
- **Sabato in `/calendario`** del bot (Daniel: «metti sabato»).
- **Archivio = foglio Google**, non file `.xlsx`: si legge da Drive e si scarica come Excel da File > Scarica. Con i dati personali (Daniel: «con i dati personali nel miglior modo possibile, facile da leggere ed esplorare»), nella cartella dell'area Talent.
- **Il tasto dell'archivio non cancella mai niente** e non avvisa il gruppo Telegram (non è una modifica); scrive nel Registro delle Impostazioni. Ogni clic crea un file nuovo: le versioni vecchie vanno buttate a mano.
- **Nelle email e nei turni dell'archivio** non c'è il filtro per edizione: finché le tabelle non vengono svuotate a fine edizione, l'archivio contiene tutto ciò che è nel database. Scelta consapevole in attesa della decisione di Daniel sulla chiusura.
- **Email dei soci dal database** (`rec_commissari`, poi `banchetto_members` attivi) al posto del foglio soci vecchio, in quattro posti: inviti del principale, nuova commissione nelle risposte, cambio/aggiunta nella Dashboard API e nel bot. La mappa `_MAIL_GIUSTE` per le email sbagliate del foglio resta nei nodi per sicurezza.
- **Il secondo colloquio della stessa ora** si salva ancora da `telegram_pending.slot_data` (scheda, riga, colonna), con `spreadsheet_id = 'calendario'`: i trigger ricavano le email. Scartato: riscrivere `slot_data` con le email (più nodi toccati, nessun guadagno).
- **Prove locali prima di tutto**, poi pubblicazione in un'unica sessione con controllo dopo ogni passo; per l'Archivio API la prova vera è stata fatta su una **copia temporanea senza controllo del token**, poi cancellata, perché non ho (e non devo avere) un token admin vero.
- **Archivio di prova creato dall'account hr@** (`creato_da = hr@jetop.com`, nome vuoto) quando è creato da me e non da Daniel, per non attribuirlo a una persona.
- **Griglia delle sezioni**: 2 colonne sul telefono, 4 da 720 px; sezione aperta piena di viola come la pagina attiva del menu in alto. Scartati: barra scorrevole orizzontale (nasconde le voci), barra laterale (troppo diversa dal resto del sito).
- **Aggiornamento Notion in un colpo solo** (Stato + Esito insieme) per non far partire il workflow Esiti, che manda l'email a chi è «Colloquio Tecnico» con un Esito.

## Evidence & Data

### Pubblicazioni del sito (repo JEToP/recruitment-form-jetop)

| PR | Ramo → base | Commit | Merge | Contenuto |
|---|---|---|---|---|
| #49 | claude/… → dev | `2bea78d` | `ffcb3bb` | Colloqui: testi senza «foglio disponibilità» |
| #50 | dev → main | — | `301b983` | pubblicazione della #49 |
| #51 | claude/… → dev | `fdef447` | `506d1a9` | Impostazioni: sezione Archivio |
| #52 | dev → main | — | `6b43607` | pubblicazione della #51 |
| #53 | claude/… → dev | `16ee4cb` | `bed2248` | Impostazioni: griglia delle sezioni |
| #54 | dev → main | — | `db85b51` | pubblicazione della #53 |

Prima del 15/11: `main` = `2859ebf` (calendario 3a), `dev` = `a12f754`. Ogni pubblicazione è un merge `--no-ff`: si annulla con `git revert -m 1 <merge>` su `main`.

### Versioni dei workflow per tornare indietro (cronologia versioni di n8n)

| Workflow | ID | Versione prima della 3c | Versione dopo (15/11) |
|---|---|---|---|
| Principale | `HirNCuA0RjYvVPp1` | `565a5f98-2727-4c80-a1ea-06cd4b2755bd` | `7eaa14d3-ce37-4096-84f2-2616d3267f81` |
| Risposte candidati | `i0ZbyVmKy5ae7XuM` | `e518667d-55ef-4d6e-8529-333108801a68` | `73b887b1-a89c-4554-adb1-b66495b90da3` |
| Annullamento | `AsjcIF8Sab5ZpXFH` | `bd0a160d-fa82-4784-b470-e03055bbd3bd` | `19a00593-c624-4d2c-bbc2-eb2071000e8d` |
| Dashboard API | `iD9mAViOGLgKM2M9` | `79cb5cd2-175a-4e35-a853-9576b3545997` | `3242fee7-2e52-455c-87e8-29c6158aee6c` |
| Bot Telegram | `m7WJCsjBYyPYRmUT` | `ede5a622-2bc7-4001-8d4f-8b9541bc777f` | `758d9c5f-2b79-481a-bb8d-8099ef799de3` |
| Impostazioni API | `FBOU4a97dlTvkPa3` | `4f2cabda-6a83-4efe-98e6-5575db085c5b` | `f1135126-9cb4-4d23-acda-19f38c5139ed` |

**Le copie di sicurezza locali (`bk_*_prima_fase3.json`) sono andate perse il 16/11.** Per tornare indietro: n8n → workflow → cronologia delle versioni → la versione «prima». Il database va riportato a mano solo se serve (le colonne e le funzioni nuove non danno fastidio ai workflow vecchi, tranne che i vecchi riscrivono celle sul foglio).

### Altri workflow del REC (stato al 16/11)

| Workflow | ID | Note |
|---|---|---|
| Statistiche API | `zPRbwd7oiCUd647v` | controllo accessi fase 1 |
| Banchetti API | `QMElucsw0TaxcxoO` | giorni da `banchetto_giorni()` |
| Convocazione tecnico | `bN5TiyRrzWxmqdBm` | |
| Reply API | `biwo9PgpDHxAMz2m` | banchetti Reply |
| Soci dal foglio | `Pn9E3SiNg6jGxWAQ` | chiamato solo dall'Impostazioni API |
| Calendario API | `1ajgp9GLtPHDe6vq` | webhook `rec-calendario` |
| Archivio API | `1dh73nRHPk1e42gJ` | webhook `rec-archivio`, nuovo 15/11 |
| Esiti finali del tecnico | `19P81AlDynbuSyUu` | ogni ora; email a chi è «Colloquio Tecnico» con Esito |
| Rifiuti dopo il Fantasocio | `jQunvRnQ8IMkfVrR` | |
| Error Handler | `PhG1qNhFlOhzhMME` | `errorWorkflow` di tutti |

### Nodi cambiati nella 3c, workflow per workflow (15/11)

**Principale `HirNCuA0RjYvVPp1` (92 → 79).**
- Tolti (19): `Sheets - M&C`, `Sheets - D&V`, `Sheets - IT`, `Sheets - S&P`, `Sheets - Talent`, `HTTP - Sotto-aree Config`, `Prepara Blocco Slot`, `HTTP Request` (celle grigie + protezioni), `Estrai Protection ID`, `HTTP - Disponibilità Riepilogo`, `HTTP - Sotto-aree Riepilogo`, `Get Config (Soci)`, `Parse Config (Soci)`, `Sheets - Email Lookup`, `HTTP - Fogli Escluso`, `IF - Celle Da Liberare`, `HTTP - Libera Celle Escluso`, `Postgres - Celle Escluso`, `Code - Richieste Escluso`.
- Nuovi o rinominati (6): `Postgres - Foglio Scheduling` (al posto della Config, legge `rec_foglio()`), `Persone Dello Slot` (era `Prepara Blocco Slot`), `Postgres - Foglio Riepilogo`, `Postgres - Email Lookup` (prima chi fa i colloqui, poi i soci attivi, con le colonne «Nome Completo» e «Mail della JE» che `Formatta Data` si aspetta), `Postgres - Colloquio Escluso`, `Code - Evento Escluso`.
- Cambiati (6 + ora legale): `Algoritmo Scheduling` (legge `_foglio(scheda)` dal database), `Prepara INSERT Supabase` (`spreadsheet_id = 'calendario'`), `Prepara Riepilogo`, `HTTP - Avvisa Slot Liberato` («ore della commissione di nuovo libere»), `HTTP - Cancella Evento Escluso`, `IF - Evento Escluso`, poi `Formatta Data` e `Code - Chi Sollecitare` (`_fusoRoma`).
- Il `Merge - Aspetta Tutti` ora ha 2 ingressi (candidati Notion e `Postgres - Leggi Slot Counts`) invece di 7.

**Risposte candidati `i0ZbyVmKy5ae7XuM` (73 → 61).**
- Tolti (15): `HTTP - GET CF Rules (Cancel)`, `Code - Prepara Liberation`, `HTTP - Liberation (Cancel)`, `Sheets M&C/D&V/IT/S&P/Talent - Mod4`, `HTTP - GET CF Rules (Reschedule)`, `Code - Prepara Liberation + Booking`, `HTTP - Liberation (Reschedule)`, `HTTP - Booking (Reschedule)`, `HTTP - Leggi Celle Scelte`, `HTTP - Sotto-aree Config`, `Sheets - Email Soci`.
- Nuovi: `Postgres - Foglio Alternativi`, `Postgres - Spunte Scelte` (usa `rec_spuntata` per i tre della commissione scelta), `Postgres - Email Soci`.
- Cambiati: `Code - Trova Slot Alternativi`, `Code - Celle Scelte` (senza `ranges`), `Postgres - Aggiorna Slot` (`protection_id_* = NULL`), `Code - Parse Scelta` (fuso).

**Annullamento `AsjcIF8Sab5ZpXFH` (18 → 13).** Tolti `Postgres - Celle`, `HTTP - Meta Foglio`, `Code - Richieste Foglio`, `IF - Celle Da Liberare`, `HTTP - Libera Celle`; `IF - Da Annullare` va dritto a `IF - Evento`; in `Code - Primo Da Annullare` via `celleSql` e il testo Telegram dice «ore della commissione di nuovo libere».

**Dashboard API `iD9mAViOGLgKM2M9` (112 → 95).**
- Tolti (19): le catene ritiro (`Postgres - Celle Ritiro`, `HTTP - Fogli Ritiro`, `Code - Richieste Ritiro`, `IF - Celle Ritiro`, `HTTP - Libera Celle`), rimetti (le stesse con «Rimetti»), prenota (`Postgres - Celle Prenota`, `HTTP - Fogli Prenota`, `Code - Richieste Prenota`, `HTTP - Aggiorna Foglio`), commissione (`HTTP - Meta Commissione`, `HTTP - Foglio Commissione`), ripresa (`HTTP - Blocca Celle Ripresa`), `HTTP - Leggi Fogli`, `HTTP - Leggi Soci`.
- Nuovi: `Postgres - Foglio` (`valueRanges` da `rec_foglio()`), `Postgres - Email Soci`.
- Cambiati: `Code - Calcola Slot` (10 messaggi d'errore senza «foglio»: «non fa i colloqui per…», «fuori dal periodo dei colloqui»), `Code - SQL Prenota`, `Code - SQL Ripresa` (`prot = []`, `'calendario'`), `Code - Prepara Ripresa`, `Code - Prepara Commissione`, `Postgres - Salva Commissione` (senza `__PROT__`), `Postgres - Salva Slot Luogo`.

**Bot `m7WJCsjBYyPYRmUT` (351 → 315).**
- Tolti (45): i 15 nodi `Sheets TG/Assegna/AT - <area>`, `HTTP - Sotto-aree Sposta/Assegna/Agg/Agg Ok/Cambia L/Cambia Ok` (al posto di alcuni ci sono i Postgres nuovi), `HTTP - Disponibilità Agg/Agg Ok/Cambia L/Cambia Ok`, `Sheets - Soci Agg/Cambia`, `HTTP - GET Sheets Info`, `Code - Prepara Reset Sheets`, `IF - Requests Non Vuote`, `HTTP - batchUpdate Reset Sheets`, `HTTP - Ripristina Checkbox`, `Postgres - Read Old Coords Sposta`, `HTTP - GET CF Rules Sposta`, `IF - Ha Liberation Requests`, `HTTP - Free Old Sheet Cells Sposta`, `HTTP - Book New Sheet Cells Sposta`, `HTTP - Blocca Cella Agg`, `HTTP - Celle Grigie Cambia`, `HTTP - Aggiorna Celle Cambio`, catena ritiro (`Postgres - Celle Ritiro`, `HTTP - Fogli Ritiro`, `Code - Richieste Ritiro`, `IF - Celle Ritiro`, `HTTP - Libera Celle Ritiro`).
- Nuovi (9): `Postgres - Foglio Sposta`, `Postgres - Foglio Assegna`, `Postgres - Foglio Assegna Resp`, `Postgres - Foglio Agg`, `Postgres - Foglio Agg Ok`, `Postgres - Foglio Cambia L`, `Postgres - Foglio Cambia Ok`, `Postgres - Email Soci Agg`, `Postgres - Email Soci Cambia`.
- Cambiati (18 + calendario): `Code - Trova 3 Slot Sposta`, `Code - Build Sposta Updates`, `Postgres - Update Slot Assignments`, `Code - Build Date Assegna`, `Code - Prepara Resp Assegna`, `Code - Scelte Aggiungi`, `Code - Prepara Aggiunta`, `Code - Scelte Cambio`, `Code - Prepara Cambio`, `Postgres - Salva Extra`, `Postgres - Aggiorna Commissione`, `Postgres - Salva Slot Secondo`, `Postgres - Stato Extra` e `Code - Formatta Stato` (niente più «protezioni mancanti»), `HTTP - Notifica Sheets OK`, `HTTP - Conferma Reset Sheets`, `HTTP - Benvenuto`, `HTTP - Invia Info`; poi `Postgres - Stato Giorni Cal/Nav`, `Code - Build Cal Week 0/Nav`, `Postgres - Orari per Data`, `Code - Build Orari Buttons` (settimane da `rec_periodo()`, sabato compreso), `Code - Parse Blocca Slot` (anno corrente).

### Iterazioni dell'archivio di ottobre

| Versione | Quando | Cosa cambiava | Candidati / entrati | Operazioni Sheets | Stato |
|---|---|---|---|---|---|
| esempio 1 | 15/11 | dati finti (40) per vedere la grafica | 40 / 10 | 140 | cestino |
| v1 | 15/11 mattina | 8 schede (con Registro), Candidati 35 colonne | 136 / 34 | 140 | cestino |
| esempio 2 | 15/11 sera | aree × sotto-aree, classifiche, leggibilità | 40 / 12 | 232 | cestino |
| v2 | 15/11 sera | senza Registro e «Ultima risposta» (33 colonne) | 136 / 34 | 226 | cestino |
| v3 | 15/11 sera | sotto-area uguale all'area = area intera | 136 / 34 | 222 | cestino |
| prova API | 15/11 | copia temporanea dell'API, 16,2 s | 136 / 34 | — | cestino, riga tolta |
| **v4** | 16/11 | pagina di M.d.S. corretta su Notion | **137 / 35** | — | **buona** |

### Struttura dell'archivio (v4)

| Scheda | Contenuto | Colonne principali |
|---|---|---|
| Riepilogo | «In breve» (6 righe), «Com'è finita» con «Cosa vuol dire», «Scelte per area e sotto-area» (prima scelta, seconda scelta, al tecnico, entrati; riga «Tutte le aree»), «Come ci hanno conosciuti», guida alle schede, avviso sui dati personali | griglia nascosta, 5 colonne |
| Candidati | una riga per candidato in ordine alfabetico; gruppi «In sintesi», «Chi è», «Candidatura», «Colloquio conoscitivo», «Colloquio tecnico», «Note» | 33 colonne; Nome, Stato finale, Entra in, email, telefono, università, corso, percorso, anno, CV, scheda Notion, candidatura del, aree e sotto-aree, canale, precedenti, motivazione, giorno, ora, luogo, commissione, presenza, solleciti, esito conoscitivo, motivo esclusione, aree e esito del tecnico, note del team |
| Colloqui | conoscitivi in ordine di tempo, linea tra un giorno e l'altro | giorno (con giorno della settimana), ora, candidato, luogo, aree, commissione, presenza, esito, stato |
| Commissione | classifica dei soci per colloqui | #, persona, area, colloqui come responsabile 1ª/2ª area, membro Talent, aggiunto, totale |
| Tecnico | inviti al colloquio tecnico per area | candidato, area convocata, invito del, mandato da, esito, area di ingresso, stato |
| Email inviate | tutte le consegne di `automation_deliveries` | quando, candidato, tipo (in italiano), riferimento («colloquio del 08/10 alle 10:00»), invio |
| Banchetti | classifica dei soci per ore | #, socio, area, turni, ore ai banchetti, ore ai banchetti Reply, ore in tutto |
| Turni ai banchetti | elenco giorno per giorno | giorno, orario (fascia), dove, socio, tipo |

Regole di stile: intestazione viola con testo bianco, bande alterne bianco/lilla chiaro, filtri sulla riga delle intestazioni, nome e intestazioni bloccati, colori sullo Stato finale (Accettato verde, Rifiutato rosso, Ritirato grigio, Da Escludere giallo, Colloquio Tecnico blu), scala lilla sulle colonne «totale» delle classifiche.

### Errori trovati e corretti (11-16/11)

| Errore | Causa | Correzione |
|---|---|---|
| PUT delle 5 API bloccati (12/11 notte) | classificatore della modalità auto | chiesto a Daniel, «vai», nessun aggiramento |
| merge rifiutato su PR #46 | `expectedHeadSha` corto | sempre i 40 caratteri |
| icone sbagliate nella pagina Impostazioni | digitate a memoria | copiate dal pacchetto `@phosphor-icons/core@2.1.1` |
| etichetta `.sr-only` segnalata come tagliata | controllo di visibilità | `aria-label` sull'input |
| pulsante morto «Riempi i giorni» | la rigenerazione è già immediata | tolto |
| link `.btn` sottolineati nel pannello banchetti | stile globale dei link | `a.btn { text-decoration: none }` |
| «Chi fa i colloqui»: menu e «Togli» su righe separate | flex | `.pe-com select { flex: 1 1 140px; width: auto }` |
| query di controllo dei permessi in errore | `has_table_privilege('anon', …)` | `information_schema.role_table_grants` |
| `updated_at` dei 78 colloqui di ottobre cambiato | trigger `slot_assignments_updated_at` durante l'import 3a | verificato che nessun workflow lo legge |
| storia di ottobre: 4 colloqui su 42 | algoritmo cambiato fino al 16/10 | criterio «stesse decisioni di oggi» |
| foglio ricostruito con ore occupate «non spuntate» | il foglio vero lascia la spunta | casella = solo disponibilità |
| ordine dei campi del nodo Sheets | `jsonb` riordina le chiavi | righe costruite come testo `json` |
| `rec_foglio()` lenta (2,4 s) | compilazione JIT | `set jit = off` → 35 ms |
| date del 2026 nei workflow | scritte a mano | fuso calcolato, periodo dal database |
| cestino Drive «File not found» | file in un drive condiviso | `supportsAllDrives=true` |
| «M&C · M&C» nell'archivio | sotto-area uguale all'area su Notion | normalizzata ad area intera |
| entrati 34 invece di 35 | pagina Notion senza edizione/esito/area | pagina corretta, archivio rifatto |

### Prove della 3c (15/11, banco locale, tutto ok)

| Prova | Cosa confronta | Numeri |
|---|---|---|
| foglio | `rec_foglio()` contro il foglio vero di ottobre | 25 controlli, 0 caselle diverse su 5 schede × 288 righe |
| query | le query nuove dei nodi «Postgres - Foglio …» | 11 nodi, formati giusti (294 righe per scheda, 6 `valueRanges`) |
| principale | «Algoritmo Scheduling» vecchio vs nuovo | ottobre 15 giri / 104 colloqui identici; 150 scenari identici; riepilogo delle 10 identico su 80 scenari; salvataggio con i vincoli NOT NULL veri |
| risposte | «Trova Slot Alternativi», ricontrollo spunte, spostamento | 120 scenari identici; spostamento e invito alla nuova commissione |
| annullamento | «Primo Da Annullare» | stesso colloquio scelto, stessi dati, testo senza «foglio» |
| dashboard | «Calcola Slot» + giro assegna/commissione/cambia/aggiungi/riprendi | 113 richieste identiche + 7 dopo il periodo senza ore |
| bot | Sposta, Assegna (date e persone), Aggiungi, Cambia, Ritira, /stato, /reset_sheets | 120 scenari identici; date di marzo 2027 corrette |
| date | fuso di Roma, solleciti, /calendario | 9 date 2026-2028 giuste; colloquio 30/03/2027 15:00 → `+02:00` |
| Impostazioni | colonna fissa, cambio area, omonimi, inizio periodo | tutti i casi |

### Prove del sito nel browser (Playwright, ultimo giro buono)

| Prova | Controlli | Cosa copre |
|---|---|---|
| `accessi_test` | 29 | menu per ruolo, rinvio ai banchetti, login unico |
| `impostazioni_test` | 93 | tutte le sezioni delle Impostazioni con i nodi veri sul Postgres locale |
| `calendario_test` | 69 | griglia, salva, copia, sabato, ore passate, vista di tutti |
| `commissari_test` | 40 | «Chi fa i colloqui» |
| `imp_header` | 10 | intestazione e ingranaggio |
| `edizione_test` | 5 | edizione e settimane dei banchetti dal server |
| `fanta_test` / `fanta_tec` | 96 / 103 | Fantasocio, regressione dopo i testi senza «foglio» |
| `vis_sweep` | 107 | leggibilità da 360 a 1920 px |
| `archivio_test` | 20 | scheda Archivio a 390 e 1366 px: caricamento, creazione, errore |
| `viste_foto` | 14 | griglia delle sezioni da 360 a 1920 px: righe, colonne, misure uguali, ordine |
| fumo staging/produzione | `accessi_staging`, `cal_staging`, `imp_staging` | dopo ogni pubblicazione: tutto ok |

Tutti i file di prova sono andati persi il 16/11: vanno riscritti se servono.

### Cronologia delle pubblicazioni e degli ok di Daniel

| Data | Cosa | Ok di Daniel |
|---|---|---|
| 12/11 | fase 1 (accessi) su n8n e sito | «vai. stai attento al limite di memoria di supabase» |
| 12/11 | fase 2 (Impostazioni) su n8n e staging | «vai» |
| 12/11 | doppioni del pannello banchetti tolti | «seguo il tuo consiglio» |
| 12-13/11 | fase 2 su main | «unisci su main» |
| 13/11 | fase 3a con il sabato fino alle 14, anche su main | «vai, sabato fino alle 14. anche su main» |
| 15/11 | fase 3c (database, Impostazioni API, 5 workflow, sito), sabato nel bot, tabelle chiuse | «1. pubblica. 2. metti sabato. 3. … 4. chiudi» |
| 15/11 | tasto «Crea l'archivio» su main | «procedi con il tasto e spiegami come funziona esattamente» |
| 16/11 | griglia delle sezioni su main | «procedi» |
| 16/11 | correzione della pagina Notion della candidata entrata in IT | ha indicato il nome dopo la mia domanda |

### Tabelle nuove del REC (colonne)

| Tabella | Colonne |
|---|---|
| `rec_commissari` | `email` PK, `nome`, `area` (M&C, D&V, IT, S&P, T&DA), `sotto_area`, `dirige`, `attivo`, `aggiornato_at`, `colonna` (unica per area) |
| `rec_disponibilita` | `email`, `giorno` date, `ora` testo `HH:00` (08-19), `creato_at`; PK (email, giorno, ora) |
| `rec_impostazioni_log` | `id`, `quando`, `email`, `nome`, `azione`, `testo`, `dettaglio` jsonb |
| `rec_archivi` | `id`, `edizione`, `file_id`, `link`, `numeri` jsonb, `creato_at`, `creato_da`, `nome` |
| `slot_assignments` (colonne aggiunte) | `email_rp`, `email_rs`, `email_talent` (le scrive il trigger `rec_slot_email`) |
| `banchetto_members` (colonne aggiunte) | `area`, `ruolo_je`, `fonte` (`foglio` o `manuale`), `attivo`, `aggiornato_at` |

Chiavi usate in `banchetto_config`: `admins`, `aree_rec`, `ruoli_senza_rec`, `giorni` (JSON dei giorni dei banchetti), `open`, `min_hours`, `max_hours`, `max_people`, `members_only`, `mail_corrette`, `soci_dal_foglio`, `soci_ultimo_esito`, `soci_ultimo_aggiornamento`. In `app_settings`: `rec_scheduling_windows`, `scheduling_paused`, `pending_reschedule_ttl_hours`.

### Spiegazione del tasto data a Daniel (15/11)

1. Impostazioni → Archivio (solo admin, ricontrollato dal server a ogni clic).
2. All'apertura: edizioni = opzioni di «Edizione REC» su Notion (dalla più recente; per una nuova edizione basta aggiungerla su Notion) ed elenco degli archivi già creati.
3. «Crea l'archivio»: legge i candidati dell'edizione da Notion, colloqui, commissioni, inviti al tecnico, email, turni dal database; crea il foglio, lo mette in «Archivio REC», lo aggiunge all'elenco e al Registro. 15-20 secondi, tasto bloccato nel frattempo.
4. Non cancella niente; ogni clic crea un file nuovo; si può usare a edizione in corso; email e turni non sono filtrati per edizione; se Google o Notion non rispondono, errore e niente salvato.

### Prestazioni di `rec_foglio()`

| Versione | Tempo | Note |
|---|---|---|
| prima stesura | 2.373 ms | sottoquery per ogni casella |
| aggregazione per riga | ~3.100 ms | stesso risultato |
| + `set jit = off` | **34-36 ms** | 1.432 ms erano compilazione JIT |

### Numeri di ottobre 2026 (archivio del 16/11)

| Voce | Valore |
|---|---|
| Candidature | 137 (136 + la pagina corretta il 16/11) |
| Colloqui conoscitivi | 104, dal 28/09 al 22/10 |
| Passati al tecnico | 66 (48,5%) |
| Entrati | 35: M&C 7, D&V 7, IT 7, S&P 8, T&DA 6 |
| Stati | Rifiutato 41, Accettato 35, Da Escludere 33, Ritirato 25, Colloquio Tecnico 3 |
| Commissione | 29 soci; il più impegnato 33 colloqui |
| Banchetti | 49 soci, 194 ore, 141 turni |
| Email automatiche | 331 (convocazione 134, invito_tecnico 77, esclusione_tecnico 41, email_esclusione 35, onboarding 34, annullamento 8, rinvio 2) |
| Come ci hanno conosciuti | Amici 45, Email 38, Instagram 16, Volantinaggio 16, Altro 8, Tiktok 7, Presentazione in aula 5 |

### Accessi e commissione (12-14/11)

| Voce | Valore |
|---|---|
| `rec_accesso` | admin 7, rec 30, socio 51, nessuno 18 |
| Foglio soci giusto | 88 soci attivi (87 all'ultimo aggiornamento) |
| `rec_commissari` | 35 persone, 30 attive, 6 «può dirigere»; per area M&C 8, D&V 7, IT 5, S&P 6, T&DA 9 |
| Colonne 15/11 | 35 persone, 35 colonne diverse, nessun omonimo; email dei 78 colloqui e dei 6 aggiunti di ottobre coerenti con le colonne |

### Sicurezza (15/11)

19 tabelle chiuse con RLS: `automation_deliveries`, `banchetto_auth`, `banchetto_bookings`, `banchetto_config`, `banchetto_locations`, `banchetto_members`, `colloqui_auth`, `esiti_impostazioni`, `esiti_programmati`, `form_email_verifications`, `n8n_error_occurrences`, `n8n_error_throttle`, `rec_storico`, `rec_storico_config`, `reply_giorni`, `reply_prenotazioni`, `rifiuti_programmati`, `slot_extra`, `tecnico_inviti`. Prima: `anon` e `authenticated` avevano SELECT/INSERT/UPDATE/DELETE/TRUNCATE. Conteggi visti da n8n uguali prima e dopo (es. `banchetto_members` 106, `automation_deliveries` 331, `form_email_verifications` 163).

## Code Analysis

- **`rec_foglio(p_oggi date default null) returns json language sql stable set jit = off`.** Periodo = primo periodo attivo di `app_settings.rec_scheduling_windows` non ancora finito. Settimane lunedì-sabato, ore 08-19 (12 righe al giorno, 73 righe per settimana: 1 titolo + 72). Righe 1-3 come il foglio (titolo, sottotitolo, intestazione «Data | Ora | nomi»). Casella TRUE = persona attiva con l'ora in `rec_disponibilita`. Restituisce `{valueRanges (5 schede + Config), fogli (righe nodo Sheets per scheda), config, periodo}`. Fuori periodo: solo intestazioni.
- **`rec_gid(area)`:** M&C 666945027, D&V 1496408726, IT 1064096795, S&P 1321516727, T&DA 161019212 (i gid delle schede del vecchio foglio, che i workflow usano ancora come chiavi).
- **Trigger `rec_commissari_colonna`:** su INSERT senza colonna o UPDATE di `area` → `max(colonna)+1` dell'area (minimo 2, cioè colonna C). Indice unico `(area, colonna)`.
- **Trigger `rec_slot_email`:** su INSERT/UPDATE di `slot_assignments` ricalcola `email_rp/rs/talent` da `(gid_*, col_*)` quando cambiano. `rec_extra_email`: idem per `slot_extra.email` se vuota.
- **`rec_periodo(p_oggi)`:** periodo in corso, altrimenti il prossimo, altrimenti l'ultimo finito. Usato da `/calendario` del bot e dai controlli delle Impostazioni.
- **`rec_spuntata(gid, col, giorno, ora)`:** sostituisce la rilettura delle tre caselle prima di uno spostamento scelto dal candidato.
- **Archivio API:** Webhook → Code Prepara (token, azione, edizione `^[A-Za-zÀ-ÿ0-9 ]{3,40}$`, una SQL con `me`/`adm` e i dati) → Postgres → Code Controlla (401/403) → Notion GET database (opzioni di «Edizione REC», ordinate per anno e mese) → Code Scegli → se `crea`: Notion query paginata (filtro edizione, max 40 pagine) → Code Modello (`costruisci.js`) → Sheets create → batchUpdate → Drive PATCH `addParents` (con `supportsAllDrives=true`) → Code Esito → Postgres (insert `rec_archivi` + `rec_impostazioni_log`) → Respond. Errori di Google → 502 con messaggio, niente salvato.
- **`costruisci.js` (dentro il nodo «Code - Modello»):** `costruisci({edizione, pagine, db, adesso, opzioni})` → modello con 8 schede; `fogli(modello)` → proprietà dei fogli (id fissi 1001…); `richieste(modello)` → richieste `batchUpdate` (celle, larghezze, altezze, unioni mai a cavallo delle colonne bloccate, bande alterne, filtri, colori dello stato finale, scala di colore sulle classifiche). Tutti i testi entrano come stringhe: un testo che comincia con «=» non diventa formula. Opzione `senzaSchedaEmail` pronta ma non usata.
- **Schema standard delle API nuove (fasi 1-3 e archivio):** Webhook POST (`responseMode: responseNode`, `allowedOrigins` = rec.jetop.com, staging, `http://localhost:3999`) → `Code - Prepara` (valida e costruisce **una** istruzione SQL) → `IF - Salta?` → Postgres (credenziale «Postgres account», utente `postgres`, padrone delle tabelle, quindi non toccato dalla RLS) → `Code - Esito` (status 401/403/400/200) → Respond con `responseCode`. Settings: `timezone: Europe/Rome`, `errorWorkflow` = Error Handler, `saveDataSuccessExecution: none`. Telegram via `https://api.telegram.org/bot{{ $env.TELEGRAM_BOT_TOKEN }}/sendMessage`, gruppo `-1003726144586`.
- **Autenticazione delle API:** il sito manda `Authorization: Bearer <token>`; la SQL cerca il token in `banchetto_auth` (valido 30 giorni) e chiama `rec_accesso(email)`. Un token falso o assente dà 401; un socio senza diritto 403 (la pagina allora torna ai banchetti).
- **Regola del Calendario per le ore segnabili** (`cal_prepara.js`, nodo «Code - Prepara» della Calendar API): giorno dentro un periodo attivo, lunedì-sabato, il sabato solo prima delle 14:00, e ora nel futuro (ora di Roma).
- **Chi può dirigere il colloquio:** un T&DA con sotto-area vuota, «Head» o «Talent», oppure chiunque abbia `dirige = true` (nel foglio era la colonna E «Sì» della Config). Lo scheduling prova prima i Talent di T&DA, poi gli altri.
- **Regole dello scheduling** (invariate dalla 3c, verificate sul banco): preavviso di 2 giorni di calendario (ora di Roma), al massimo 2 colloqui per ora (il secondo chiede il luogo su Telegram), tre persone diverse e libere (non già in un colloquio o aggiunte alla stessa ora), domenica esclusa, prima la sotto-area del candidato poi l'area intera, a parità di ora chi ha meno colloqui.
- **Fascia oraria dei banchetti:** 7 fasce da 1,5 ore (08:30-10:00 … 17:30-19:00); un'ora di banchetto Reply vale 1 ora.
- **`chiama(path, body, attesa)`** in `impostazioni/app.js`: tempo massimo 90 s, 240 s per «crea l'archivio».

## Files Changed

### Sito (repo JEToP/recruitment-form-jetop, tutto su `main`)
- `rec-dashboard/colloqui/app.js` — 12 testi senza «foglio» (ritiro, rinvio, ripresa, assegna, commissione: «la commissione torna libera a quell'ora», «Aggiorno Notion e calendario», etichetta «Area» al posto di «Foglio», «non ha segnato quest'ora nel Calendario»).
- `rec-dashboard/impostazioni/index.html` — sezione «Archivio» (`#v-archivio`, `#arForm`, `#arEdizione`, `#arCrea`, `#arLista`, link alla cartella `#arCartella`); griglia delle sezioni (`nav.viste` con 8 `button[data-vista]`, icone Phosphor `users`, `chats-circle`, `file-text`, `clock`, `storefront`, `seal-check`, `archive`, `clock-counter-clockwise`); `app.js?v=20261116`.
- `rec-dashboard/impostazioni/app.js` — vista `archivio` (`arCarica`, `disegnaArchivi`, `creaArchivio`), `ARCHIVIO_PATH = 'rec-archivio'`, `chiama(..., attesa)`.

### n8n e database
- Vedi le tabelle sopra. SQL della 3c (`setup_c.sql`, `import_c.sql`) e dell'archivio (`archivio/api/setup.sql`): **persi con il container**, ma già applicati in produzione.

### Questo repo (siti-torino)
- `.claude/handoffs/HANDOFF_rec-accessi-calendario-archivio_2026-11-18.md` — questo file.
- `.claude/design/DESIGN_rec-jetop.md` — **non** aggiornato con la griglia delle sezioni (da fare).

### Sito, file toccati nelle fasi 1-3a (11-13/11, già su `main`)
- `banchetti/app.js` — settimane dal server, `aggiornaMenu` per tutti i soci, `torna` verso colloqui/impostazioni/calendario, pannello admin con le regole in sola lettura (`IMPOSTAZIONI`, `IMPOSTAZIONI_BANCHETTI`); `banchetti/reply.js` — `edizioneRec(r.giorniRec)`; `banchetti/index.html`.
- `rec-dashboard/index.html`, `rec-dashboard/app.js` (Statistiche), `rec-dashboard/colloqui/index.html` e `app.js` — ingranaggio solo admin, menu a quattro voci.
- `rec-dashboard/impostazioni/` e `rec-dashboard/calendario/` — pagine nuove.
- `assets/jetop-ui.css` — regola `.app-switch[data-voci="4"]` (due righe sotto i 520 px); versione `?v=20261113`.
- `candidati/index.html` — solo il numero di versione del CSS.

### Il vecchio foglio delle disponibilità (solo per capire il codice)
Cinque schede (M&C, D&V, IT, S&P, «T&DA - Talent»), intestazione alla riga 3 («Data», «Ora», poi un nome per colonna da C), 4 settimane con una riga di titolo e 6 giorni × 12 ore (08:00-19:00), caselle TRUE/FALSE; scheda «⚙️ Config» con Nome, Area, Sotto-area, «Dirige» da B13. `rec_foglio()` riproduce esattamente questa forma: per questo i nodi parlano ancora di righe, colonne e gid. Il foglio vero non viene più letto né scritto da nessun workflow; resta su Drive com'era a ottobre.

### File locali persi il 16/11 (per sapere cosa ricostruire se serve)
- `talent/n8n.py`, `sql_leggi.py`, `reply/fixed.py`, `notion_leggi.py`, `fase3/foglio_leggi.py`;
- `fase3/`: `patchlib.py`, `patch_{main,risposte,annullamento,dashboard,bot}.py`, `pubblica_c.py`, `rls.py`, `errori_dopo.py`, `banco.js`, `scenari.js`, `ottobre.js`, `pianifica.js`, prove `prova_*.js`, `sintassi.js`, dati di ottobre (`dati/candidati.json`, `dati/slot.json`, `foglio_disp.json`);
- `archivio/`: `costruisci.js` (recuperato da n8n), `finti.js`, `prova_archivio.js`, `esegui.py`, `api/*` (prepara, controlla, scegli, esito, build.py, prova.js, prova_vera.py);
- `pw/`: prove del browser (`impostazioni_test.js`, `archivio_test.js`, `viste_foto.js`, `calendario_test.js`, `commissari_test.js`, `fanta_*`, `vis_sweep.js`, `visibilita.js`, `srv_csp.py`…).

## User Feedback & Preferences (REQUIRED — never omit)

- **Rispondere in italiano; diretto e critico;** non concordare per compiacenza; non inventare dati, chiedere prima.
- **«continua senza sprecare token nella spiegazione»** (ripetuto più volte, anche il 16/11): poche parole su cosa faccio, esito chiaro alla fine.
- **Prima di `main` mostrare e aspettare l'ok** (regola del 10/10, sempre rispettata: staging, foto, poi «procedi»).
- **Modifiche su n8n in produzione: con il suo ok.** Il 15/11 «pubblica» ha coperto tutta la 3c; il tasto dell'archivio è stato chiesto esplicitamente («non vedo il tasto»). Se il classificatore blocca: chiedere, mai aggirare.
- **14/11:** «Accetto il criterio "stesse decisioni dell'algoritmo di oggi"… usiamo il sistema nuovo, a marzo dev'essere tutto pronto. Ricordati che non abbiamo memoria infinita, serve un modo per immagazzinare le informazioni di ogni rec, magari esportando tutte le informazioni in un documento excel direttamente sul drive».
- **13/11:** «considera la data di apertura delle candidature del rec di marzo il 1 marzo, ma è ancora da definire con certezza»; «vai, sabato fino alle 14. anche su main».
- **12/11:** «vai. stai attento al limite di memoria di supabase».
- **15/11:** «1. pubblica. 2. metti sabato. 3. [cartella Drive]. con i dati personali nel miglior modo possibile, facile da leggere ed esplorare. 4. chiudi».
- **Sull'archivio (15/11):** «sul primo foglio: specificare meglio le scelte delle aree e sottoaree»; «banchetti: fare un conteggio delle ore come nel foglio commissione, con una sorta di classifica»; «togliere la colonna "ultima risposta del candidato" dalla scheda candidati, rimuovere il foglio registro»; «migliorare anche la leggibilità generale».
- **Chiusura dell'edizione:** «sulla chiusura di un'edizione te lo farò sapere in seguito». Non svuotare niente finché non lo dice.
- **«non vedo il tasto per generare il file nella piattaforma. poi dimmi quali sono i prossimi passi da fare»** → si aspetta le funzioni dentro il sito, non script miei.
- **«procedi con il tasto e spiegami come funziona esattamente»** → spiegazione passo per passo di cosa succede premendo.
- **16/11, con foto dal telefono:** «questi pulsanti a casaccio sono orribili, devono essere in ordine, lineari, facilmente individuabili». Ordine e regolarità contano quanto la leggibilità.
- **16/11:** «c'è un errore nel numero di persone entrate (sono 35 non 34, gli IT sono 7 non 6)» → controlla i numeri: li conosce a memoria.
- **Grafica:** leggibile ovunque, soprattutto dal telefono; giro da 360 a 1920 px e foto guardate a occhio prima di proporre.
- **Email ai candidati:** niente «—»; testi JEToP in prima persona come Daniel; email ufficiali firmate «Salvatore Daniel Leocata».
- **Mai chiedere token o chiavi in chat.**

## Where We're Going

1. **Ricostruire gli strumenti** (Quick Start) e fare un controllo di sola lettura: esecuzioni in errore dei workflow REC dal 16/11; `rec_archivi`; periodi in `app_settings`.
2. **«I miei colloqui» con nome e CV del candidato** (nella pagina Calendario, per chi è in commissione): previsto dal piano, promesso a Daniel come prossimo passo. Serve una lettura da Notion nella Calendar API per i candidati con colloquio fissato; niente contatti, note, esiti.
3. **Fase 3d:** preavviso dello scheduling (oggi `PREAVVISO_GIORNI = 2` / `ANTICIPO_GIORNI = 2` nel principale), numero massimo di solleciti (`MAX_SOLLECITI = 3` in «Code - Chi Sollecitare»), ore prima dell'annullamento automatico (24 h nell'annullamento), da costanti nei workflow a `app_settings` modificabili dalle Impostazioni (sezione Colloqui).
4. **Prima di marzo:** quando Daniel ha le date, impostare il periodo di marzo nelle Impostazioni (Colloqui) e la data di apertura delle candidature; dire ai soci di segnare le ore nel Calendario (il foglio non serve più). Su Notion l'opzione «Marzo 2027» di «Edizione REC» esiste già.
5. **Prova generale a febbraio** con 2-3 candidati finti: il giro completo dopo la 3c (form → scheduling → convocazione → risposte → bot → annullamento) non è mai stato provato dal vivo. Serve qualcuno che risponda alle email e usi il bot.
6. **Aggiornare `.claude/design/DESIGN_rec-jetop.md`** con la griglia delle sezioni (16/11) e la sezione Archivio.
7. **Fasi 4, 5, 6** del piano: testi delle email nelle Impostazioni; elenco unico delle sotto-aree; chiusura dell'edizione (svuotamento con anteprima e conferma, compattazione delle colonne di `rec_commissari`, informativa privacy) appena Daniel decide.

**Prossimi passi come li ho presentati a Daniel il 15/11** (lui ha risposto solo sull'archivio e sulla griglia):
- prima di marzo: periodo di marzo nelle Impostazioni quando ha le date (e avvisare i soci del Calendario); «I miei colloqui» con nome e CV (lo faccio io); 3d (lo faccio io); prova generale a febbraio (serve qualcuno di loro);
- meno urgenti: testi delle email, elenco unico delle sotto-aree, chiusura dell'edizione (aspetta la sua decisione);
- da lui: le date di marzo; per la privacy titolare, contatto e periodo di conservazione; chiudere su Notion i 3 S&P in «Colloquio Tecnico».

**Come verificare una pubblicazione del sito** (lo script `confronta_sito.sh` è perso; rifarlo così): per ogni file cambiato tra il commit di partenza e `origin/main` (`git diff --name-only`), scaricare `https://rec.jetop.com/<file>?v=<timestamp>` con `Cache-Control: no-cache` (per `…/index.html` l'indirizzo della cartella) e confrontarlo byte per byte con `git show origin/main:<file>`. Il deploy di solito arriva in 30-60 secondi, a volte molto di più. Per lo staging, stessa cosa con `origin/dev`.

## Risks & Blockers

- **Strumenti locali effimeri:** il container è stato sostituito il 16/11 e ha cancellato `/home/user/lavoro/`. Può ricapitare. Le copie di sicurezza dei workflow vanno prese dalla cronologia di n8n; gli script vanno ricreati. Non esiste un posto durevole per gli script: il repo JEToP è solo per il sito, questo repo è pubblico (niente dati, niente ID di credenziali).
- **3c mai provata dal vivo end-to-end:** prove solo nodo per nodo con dati veri e confronto con i nodi vecchi. Rischio maggiore: il primo scheduling vero di marzo.
- **Debito tecnico della 3c:** i workflow ragionano ancora per «scheda e colonna». Limiti: chi viene tolto dai colloqui resta come colonna senza disponibilità; chi cambia area lascia una «(colonna libera N)»; l'inizio del periodo non può cambiare settimana con colloqui fissati.
- **Archivio senza filtro per edizione su email e turni:** finché le tabelle operative non vengono svuotate, l'archivio di marzo conterrebbe anche dati di ottobre.
- **Quota di traffico Supabase** (vedi passaggio precedente): ogni chiamata all'API di n8n costa ~430 KB. Ridurre le chiamate; i workflow temporanei costano 4+ chiamate ciascuno.
- **Classificatore della modalità auto:** può bloccare PUT di workflow o scritture; non aggirare, chiedere a Daniel.
- **Modifiche a mano su Notion:** cambiare solo «Esito Tecnico» su un candidato in «Colloquio Tecnico» fa partire l'email del workflow Esiti alle 18 del giorno dopo; cambiare lo Stato in «Da Escludere» fa partire subito l'email di esclusione (trigger Notion del principale). Avvisare Daniel quando corregge pagine a mano.
- **Elenco di chi fa i colloqui fermo a ottobre:** se nessuno lo aggiorna prima di marzo, lo scheduling userà persone che magari non ci sono più (e chi è nuovo non potrà segnare le ore nel Calendario finché non è aggiunto).
- **Deploy di rec.jetop.com:** di solito 30-60 secondi dopo il merge, a volte ore; prima di dire «è online» confrontare i file.

## Open Questions

- **Chiusura dell'edizione:** cosa svuotare (solo database o anche candidati su Notion) e per quanto tenere i dati personali. Daniel: «te lo farò sapere in seguito».
- **Date di marzo 2027:** apertura candidature (ipotesi 1/3) e periodo dei colloqui.
- **Informativa privacy:** titolare, contatto privacy, periodo di conservazione.
- **Prova generale a febbraio:** da confermare con Daniel (chi fa i candidati finti).
- **Accesso con un account vero:** il percorso di login completo (account Google reale, token, ruolo) non è verificabile da me; il 12-13/11 avevo chiesto a Daniel di provarlo. Nessuna segnalazione di problemi da allora.
- **Chi fa i colloqui a marzo:** `rec_commissari` contiene ancora le persone di ottobre (30 attive). Prima di marzo Daniel o un admin devono aggiornare l'elenco dalle Impostazioni (chi esce, chi entra, sotto-aree).
- **Pagina di M.d.S. su Notion:** mancano email, CV, aree; i 3 candidati S&P in «Colloquio Tecnico» vanno chiusi. Li sistema Daniel.

## Quick Start for Next Session

```bash
# 0. Leggi questo file e il precedente (HANDOFF_rec-ottobre-2026_2026-10-08.md). Rispondi in italiano, poche spiegazioni.

# 1. Ambiente: N8N_API_KEY deve esserci (mai chiederla in chat).
test -n "$N8N_API_KEY" && echo ok

# 2. Ricrea l'aiuto per n8n in /home/user/lavoro/talent/n8n.py (NON nei repo):
#    _req(method, path, body) con curl su https://n8n.jetop.com/api/v1 e header X-N8N-API-KEY;
#    get(wid), put(wid, wf) (solo name/nodes/connections/settings ammesse), create(wf), delete(wid),
#    activate(wid, on), node(wf, name), save(obj, path);
#    temporaneo(nome, nodi): Webhook (responseMode lastNode, firstEntryJson) davanti ai nodi in fila,
#    lo attiva, lo chiama una volta con curl e lo cancella nel finally.
#    Gli ID delle credenziali NON sono qui: leggili dal campo "credentials" dei nodi di un workflow
#    (es. GET /workflows/1dh73nRHPk1e42gJ: Postgres, Notion, Sheets, Drive dell'account hr@).

# 3. Repo del sito (solo codice): add_repo JEToP/recruitment-form-jetop (access push), poi
git clone https://github.com/JEToP/recruitment-form-jetop /home/user/recruitment-form-jetop
#    Ramo di lavoro claude/awesome-ramanujan-mlqkdz ripartendo da origin/main; PR su dev (staging
#    https://ggs84s004oso4c8kwow44o84.server.jetop.com), ok di Daniel, PR dev -> main (rec.jetop.com).
#    Per il merge: expectedHeadSha con i 40 caratteri (git rev-parse).

# 4. Controllo di salute (sola lettura, un solo workflow temporaneo):
#    GET /executions?workflowId=<id>&status=error&limit=5 per i 6 workflow REC principali;
#    SELECT da rec_archivi, app_settings (rec_scheduling_windows), count da rec_disponibilita.

# 5. File da leggere per primi:
#    - recruitment-form-jetop/rec-dashboard/calendario/app.js e la Calendar API 1ajgp9GLtPHDe6vq
#      (per «I miei colloqui» con nome e CV);
#    - recruitment-form-jetop/rec-dashboard/impostazioni/app.js (sezione Colloqui, per la 3d);
#    - workflow HirNCuA0RjYvVPp1 nodi «Controlla Periodo», «Prepara Riepilogo», «Code - Chi Sollecitare»;
#      workflow AsjcIF8Sab5ZpXFH nodo «Code - Primo Da Annullare».

# 6. Prossima azione: «I miei colloqui» con nome e CV del candidato (Where We're Going punto 2),
#    in locale con prove nel browser, poi staging e ok di Daniel.
```

**Cose che Daniel deve fare a mano** (ricordarle se non fatte):
- Completare su Notion la pagina di M.d.S. e chiudere i 3 candidati S&P ancora in «Colloquio Tecnico»; poi ricreare l'archivio con il tasto e buttare la versione vecchia da Drive.
- Decidere chiusura dell'edizione, date di marzo, dati per l'informativa privacy.
- Dalle note precedenti: correggere le email nel foglio soci (righe segnalate il 30/10), segnalare all'IT i problemi DNS dell'host n8n.

## Session Closed
**Closed at:** 2026-11-18 (sera, ora italiana)
**Commit:** `f59ef55` (handoff) + questo commit di chiusura
**Session status:** Handed off to next session (Daniel apre una sessione nuova)
