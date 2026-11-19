# NAO CogniLab

Applicazione web interattiva per lo studio di caso con il robot umanoide **NAO** su un bambino con disturbo dello spettro autistico (ASD) e ADHD. Il robot guida il bambino attraverso una batteria di esercizi cognitivi, lo incoraggia con frasi e movimenti e raccoglie dati di telemetria per l'analisi.

Progetto di tesi magistrale in psicologia clinica (studio di caso su interazione uomo-robot, antropomorfismo e potenziamento cognitivo).

> **Attenzione:** il repository contiene solo il codice dell'app. Non devono mai finire nel repository dati di bambini, file audio, CSV o archivi esportati.

---

## Indice

1. [Cosa fa l'app](#cosa-fa-lapp)
2. [Struttura del percorso](#struttura-del-percorso)
3. [I cinque moduli](#i-cinque-moduli)
4. [Check-in emotivo](#check-in-emotivo)
5. [Monetine e NAO](#monetine-e-nao)
6. [Pannello del terapeuta](#pannello-del-terapeuta)
7. [Dati, telemetria e anonimizzazione](#dati-telemetria-e-anonimizzazione)
8. [Uso e installazione](#uso-e-installazione)
9. [Configurazione](#configurazione)
10. [Struttura del codice](#struttura-del-codice)
11. [Note tecniche e limiti](#note-tecniche-e-limiti)
12. [Prima di mettere il progetto su GitHub](#prima-di-mettere-il-progetto-su-github)

---

## Cosa fa l'app

- Un'unica pagina HTML, senza installazione e senza server. Funziona aperta da file o pubblicata come sito statico.
- Interfaccia in italiano, blu, con font pensati per la lettura (Lexend, Atkinson Hyperlegible, OpenDyslexic), scelta dal pannello del terapeuta.
- Il robot NAO è rappresentato da una mascotte che parla (sintesi vocale italiana del browser), dà feedback positivi e sblocca nuove azioni con le monetine.
- Ogni gioco registra eventi con marcature temporali precise (`performance.now()`), esportabili in CSV.
- Il terapeuta controlla l'app tramite un PIN: sblocco di "Aiuta NAO", fine sessione, accesso ai dati e impostazioni.
- Il bambino non vede mai dati, punteggi negativi o faccine punitive: il linguaggio è incoraggiante e i messaggi di errore sono neutri.

---

## Struttura del percorso

Il percorso ha una schermata iniziale ("Ciao! Sono NAO") e tre aree.

| Area | Contenuto | Sblocco |
|---|---|---|
| **Check-in** | Domande iniziali sul proprio stato | Obbligatorio all'inizio della sessione |
| **Sezione 1 – Allenamento cognitivo** | I moduli 1, 2, 3, 4 in sequenza | Dopo il check-in |
| **Sezione 2 – Aiuta NAO** | Modulo 5 (snake-path) | Sblocco con PIN del terapeuta, dopo la Sezione 1 |
| **Sezione 3 – Scelta libera** | Il terapeuta sceglie un singolo modulo da far fare | Sempre disponibile dopo il check-in |

A fine sezione compare la schermata "Bravissimo! Ora passa l'iPad a [ricercatrice]", con il pulsante **Scarica dati** per esportare il CSV della sezione.

Quando il bambino completa un modulo, il successivo parte con una breve spiegazione di NAO e una prova di 10 secondi, poi la schermata "Ora si inizia!" in cui il bambino tocca lo schermo per partire.

### Sessione

- Una sessione inizia al primo gioco e ha un numero progressivo (S01, S02, …).
- Durata massima configurabile (default 30 minuti). Allo scadere la sessione si chiude da sola.
- La sessione si può chiudere anche a mano con il pulsante **Concludi sessione** (richiede PIN).
- Alla chiusura il progresso del bambino viene azzerato e la home torna allo stato iniziale. I dati restano nell'archivio.

---

## I cinque moduli

Ogni modulo ha livelli. Per passare al livello successivo serve un'accuratezza di almeno l'80% (`masteryThreshold: 0.80`). Se il livello non viene superato, il bambino ripete lo stesso livello (sistema a checkpoint).

### Modulo 1 – Fase iniziale (Go/No-Go)
- Compaiono stimoli: il bambino tocca solo per lo stimolo "GO" e non tocca per "NOGO".
- 3 livelli con tempi di stimolo e di pausa decrescenti.
- Limite: 1 min 30 s.

### Modulo 2 – Attraversa la strada (rane)
- Canvas. Il bambino guida una rana con quattro frecce verso la sponda opposta, evitando auto e camion e salendo sui tronchi.
- 3 livelli, 5 attraversamenti per livello:
  - **Livello 1:** 2 corsie di auto, direzione verso l'alto.
  - **Livello 2:** 3 corsie (auto, tronchi, auto), direzione verso il basso. Almeno una corsia è più lenta.
  - **Livello 3:** 3 corsie di auto, poi tronchi, poi stagno con una ninfea. Direzione verso destra.
- Limite: 3 minuti.

### Modulo 3 – Occhio al puntatore (tracking)
- Un cerchio si muove sullo schermo; il bambino lo segue con il dito (livello 1).
- Livello 2: cerchi che compaiono in posizioni diverse da toccare entro un tempo.
- Livello 3: come il livello 2, con distrattori da non toccare.
- 3 blocchi da 30 secondi, totale 1 min 30 s.

### Modulo 4 – Ricerca visiva
- Griglie quadrate (3×3, 4×4, 5×5, 6×6) con celle di uguale dimensione, ognuna con un'icona.
- Il bambino trova il bersaglio indicato nel riquadro "Trova" (immagine o parola). Nel livello 4 il riquadro alterna immagine e parola.
- Limite: 3 minuti; allo scadere si passa al gioco successivo.

### Modulo 5 – Aiuta NAO (snake-path)
- Una griglia con NAO. Il bambino traccia un percorso che raccoglie tutti gli oggetti, nell'ordine indicato, senza toccare i blocchi.
- 4 livelli:
  1. Griglia 5×5, 2 oggetti.
  2. Griglia 5×5, 3 oggetti.
  3. Griglia 6×6, 3 oggetti, con blocchi da evitare.
  4. Griglia 6×6, 3 oggetti, con blocchi e un oggetto da **non** prendere.
- Dopo 3 errori il livello si ripete.

---

## Check-in emotivo

All'inizio di ogni sessione il bambino risponde a quattro passi. Senza il check-in completato i giochi restano bloccati.

1. **"Come ti senti in questo momento?"** – Benissimo / Bene / Un po' stanco / Stanco / Stanchissimo.
2. **"Ti senti un po' nervoso o arrabbiato?"** – Per niente / Un pochino / Abbastanza / Tanto / Tantissimo.
3. **"Com'è andata la scuola oggi?"** – Benissimo / Bene / Così così / Un po' male / Male.
4. **"Com'è andata la settimana fino adesso?"** – stessa scala.

Se il bambino risponde "Un po' male" o "Male" alle domande 3 e 4, NAO chiede **"Vuoi parlarne?"**. Se sceglie di parlarne, la risposta viene registrata in audio e trascritta (riconoscimento vocale italiano). Il bambino può sempre rispondere "Non ne voglio parlare" e proseguire.

Ogni risposta produce una riga di telemetria con il valore da 1 a 5. Le etichette sono in linguaggio semplice e non parlano di batterie o energia in senso tecnico.

---

## Monetine e NAO

- Ogni gioco completato dà monetine. Il totale dipende da: un bonus base per il gioco, l'accuratezza e un bonus per ogni livello completato. Il check-in dà 20 monetine.
- Le monetine si accumulano nel **salvadanaio** di NAO, con il totale nella barra in basso.
- Ogni 500 monetine si sbloccano nuove azioni di NAO (saluta, balla, salta, piroetta, canta, …). Le azioni bloccate mostrano il lucchetto e la soglia.
- Suono "cling!" e animazione delle monetine al guadagno.

---

## Pannello del terapeuta

Accessibile dalla barra in basso con PIN (default `1234`, cambiabile nelle impostazioni).

- **Risultati / Archivio:** elenco delle prove per sessione, con filtri, download del CSV di una prova, di una sessione o di tutto l'archivio, e download dell'audio registrato.
- **Scelta libera:** far fare al bambino un singolo modulo a scelta.
- **Impostazioni:** font, voce del robot (scelta tra le voci italiane del dispositivo), separatore CSV (virgola o punto e virgola, per Excel italiano), durata massima della sessione, suoni, PIN.
- **Sblocco "Aiuta NAO"** e **Concludi sessione** richiedono il PIN.

---

## Dati, telemetria e anonimizzazione

### Cosa viene registrato
Ogni evento è una riga con queste colonne:

`participant_id, session_id, study_day, t_session_ms, module, phase, level, trial, event, stimulus, response, correct, rt_ms, error_type, x, y, value, notes, run_mode, run_id`

Il tempo di risposta (`rt_ms`) e gli eventi sono calcolati con `performance.now()`.

### Anonimizzazione
- Il nome del bambino compare **solo sullo schermo**. Nei dati è sostituito dal codice pseudonimo (`P01`), e nei testi esportati il nome viene sostituito con "[bambino]".
- Giorno dello studio (`study_day`) al posto della data: il giorno 0 è il primo giorno di sessione.
- Non vengono registrati user agent, modello del dispositivo, indirizzo IP o orari assoluti.
- I file esportati hanno nomi del tipo `P01_sezione1_allenamento_cognitivo_S03.csv`.

### Esportazioni
- **CSV di sezione:** scaricato a fine sezione (Sezione 1, Sezione 2, o modulo della Scelta libera).
- **CSV di archivio:** tutte le prove, con filtri.
- **CSV telemetria completa:** tutti gli eventi.
- **Audio** delle risposte "Vuoi parlarne?" in formato del browser.

I CSV usano codifica UTF-8 con BOM, così Excel mostra correttamente le lettere accentate.

### Dove sono salvati
- Eventi: `localStorage` del browser (`naolab_rows`).
- Audio: IndexedDB del browser (`naolab_audio`).
- Impostazioni, progresso, monetine e sessioni: `localStorage` (`naolab_settings`, `naolab_progress`, `naolab_coins`, `naolab_study`).

I dati restano sul dispositivo finché non vengono esportati. Svuotare i dati del browser li cancella.

---

## Uso e installazione

### Uso locale (consigliato per le sessioni)
1. Aprire `nao_cognilab.html` in Chrome o Safari.
2. Il microfono (registrazione e trascrizione) richiede una pagina servita in HTTPS: in locale funziona solo dove il browser lo permette. Per le sessioni con registrazione usare la versione pubblicata in HTTPS.

### Pubblicazione statica (Cloudflare Pages)
Il pacchetto `nao_cognilab_cloudflare.zip` contiene:

- `index.html` – l'app, con meta `noindex`
- `manifest.webmanifest`, `icon-180.png`, `icon-512.png` – icona per l'iPad (aggiungi a schermata Home)
- `robots.txt` – blocca l'indicizzazione (`Disallow: /`)
- `_headers` – intestazioni di sicurezza: `Permissions-Policy: microphone=(self)`, `X-Robots-Tag: noindex`, `Referrer-Policy: no-referrer`, cache

Per pubblicare: caricare il contenuto dello zip (o la cartella) su Cloudflare Pages come sito statico, senza build.

### Su iPad
Aprire il link in Safari, poi **Condividi → Aggiungi a schermata Home**. Si apre a schermo intero come un'app.

---

## Configurazione

Tutte le impostazioni principali sono nell'oggetto `CONFIG` all'inizio dello script:

| Chiave | Significato | Default |
|---|---|---|
| `childName` | Nome mostrato sullo schermo (mai nei dati) | – |
| `participantCode` | Codice pseudonimo usato nei dati | `P01` |
| `researcherName` | Nome di chi riceve l'iPad a fine sezione | – |
| `sessionMinutes` | Durata massima della sessione (0 = nessun limite) | `30` |
| `masteryThreshold` | Soglia di accuratezza per passare di livello | `0.80` |
| `practiceMs` | Durata della prova iniziale | `10000` |
| `gng`, `frog`, `track`, `search`, `snake` | Parametri dei moduli (durate, livelli, velocità, griglie) | vedi codice |

Le monetine sono definite in `COINS`, le azioni di NAO in `NAO_MOVES`. Il PIN predefinito è in `Settings.pin`.

---

## Struttura del codice

Il file è unico (HTML + CSS + JavaScript). Lo script è diviso in sezioni numerate:

| Sezione | Contenuto |
|---|---|
| 1–4 | Configurazione, stato, utilità |
| `Settings`, `Voice`, `Sfx` | Impostazioni, sintesi vocale, suoni |
| `Study`, `Telemetry` | Sessioni, eventi, CSV |
| `Progress`, `Coins` | Sblocchi e monetine |
| `runGNG`, `runFrog`, `runTrack`, `runSearch`, `runSnake` | I cinque moduli |
| `SECTIONS`, `runSection1`, `runSection2`, `runSingle` | Flusso delle sezioni |
| `runCheckin` | Check-in emotivo e registrazione |
| `Archive`, `ResultsUI` | Archivio e pannello risultati |
| `endSession` | Chiusura sessione |

Nel codice, i commenti in italiano spiegano ogni parte.

---

## Note tecniche e limiti

- **Font incorporati:** Lexend, Atkinson Hyperlegible e OpenDyslexic sono inclusi in forma base64, quindi l'app funziona anche offline. Le licenze sono OFL (SIL Open Font License).
- **Voce:** dipende dalle voci italiane installate sul dispositivo (Web Speech API). Su alcuni dispositivi la voce più naturale va scelta nelle impostazioni.
- **Riconoscimento vocale:** usa `webkitSpeechRecognition` (Chrome, Safari). Su browser che non lo supportano la trascrizione non viene salvata, ma l'audio sì.
- **Registrazione:** `MediaRecorder` richiede HTTPS (o localhost).
- **Archiviazione locale:** i dati sono legati al browser e al dispositivo. Non c'è sincronizzazione tra dispositivi: per consolidare i dati, esportare i CSV a ogni sessione.
- **Account opzionali:** l'app può predisporre un archivio su database con account, ma nella versione locale non è attivo.
- **Non testato su tutti i dispositivi:** i test automatici sono stati fatti con Chromium. La prova su iPad e Mac reale va fatta prima dell'uso con il bambino.

---

## Prima di mettere il progetto su GitHub

Il codice attuale contiene dati da proteggere. Prima di qualsiasi push:

1. **Rendere il repository privato.** Anche con il solo codice, il nome del bambino è nel file.
2. **Sostituire il nome del bambino** in `CONFIG.childName` con un valore neutro (per esempio `"Bambino"`), oppure lasciarlo solo sul dispositivo di sessione.
3. **Cambiare il PIN predefinito** `1234` in `Settings.pin`.
4. **Non committare mai** CSV, file audio, esportazioni o l'archivio del browser.
5. Aggiungere un `.gitignore` con `*.csv`, `*.webm`, `*.mp4`, `*.zip` (se contiene dati), `*.log`.
6. Verificare con `grep -i` che nel repository non compaiano il nome del bambino né altri dati personali prima del primo commit.

Per la pubblicazione su GitHub con questa struttura:
```
nao-cognilab/
├── README.md
├── index.html          (copia di nao_cognilab.html)
├── manifest.webmanifest
├── icon-180.png
├── icon-512.png
├── robots.txt
├── _headers
└── .gitignore
```

---

## Stato del progetto

Versione attuale: sezioni 1–2 complete, scelta libera, check-in emotivo con registrazione, monetine e NAO, sessioni con limite di tempo, anonimizzazione dei dati, esportazione CSV, archivio in app.

Da fare:
- Verificare il pacchetto Cloudflare su dispositivo reale (iPad e Mac).
- Confermare con la ricercatrice le formulazioni del check-in e la soglia di accuratezza.
- Decidere se aggiungere il PIN anche alla Scelta libera.
- Predisporre il modulo di consenso informato per la registrazione vocale.

---

## Note

Il progetto è sviluppato per uso di ricerca. I dati dei partecipanti vanno trattati secondo il consenso informato, il parere del comitato etico e la normativa sulla protezione dei dati (GDPR), e non devono essere pubblicati.
