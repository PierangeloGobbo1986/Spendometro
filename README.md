# Spendometro

App di famiglia per la gestione delle spese condivise, sviluppata per due persone (Pierangelo e Martina) con estensione a spese di gruppo con terzi. Progressive Web App a file singolo, nessun passaggio di build, hosting su GitHub Pages.

**In uso su:** `https://pierangelogobbo1986.github.io/Spendometro/`

---

## Cos'è

Un ibrido fra Splitwise e Spendee: tiene traccia delle spese personali e condivise di una coppia, calcola in tempo reale chi deve cosa a chi, gestisce spese ricorrenti automatiche, e permette di dividere spese occasionali anche con persone esterne (per esempio durante un viaggio di gruppo), senza che queste ultime abbiano bisogno di un account.

## Stack tecnico

- **Frontend**: React 18 caricato da CDN, JSX trasformato al volo con Babel standalone. Nessun bundler, nessun passaggio di build: si modifica `index.html` e si carica direttamente.
- **Backend**: Firebase Firestore (piano gratuito Spark) per i dati, Firebase Authentication (email/password) per l'accesso.
- **Export**: SheetJS (xlsx) via CDN per la generazione dei report in Excel, interamente lato client.
- **Hosting**: GitHub Pages, servito direttamente dalla root del branch principale.
- **PWA**: installabile su iPhone e Android da browser ("Aggiungi a Home"), con manifest e icone dedicate.

## Funzionalità principali

**Spese e entrate**
- Inserimento rapido con calcolatrice integrata (supporta espressioni matematiche)
- Categorie a cinque livelli (L1 spese/entrate, L2 fisse/variabili/stipendio/altre entrate, L3/L4/L5 personalizzabili), con emoji automatica
- Etichette libere con autocompletamento, utili per raggruppare spese di viaggi o eventi
- Spese condivise al 50% o in percentuale personalizzata con l'altro utente, con calcolo automatico del saldo reciproco (Conguaglio)
- Spese condivise con persone esterne (terzi): pagina dedicata con gestione delle persone, calcolo dei debiti incrociati organizzato per persona, ed elenco dei movimenti con l'importo totale della spesa di gruppo

**Spese ricorrenti**
- Cadenza settimanale, mensile o annuale, con data di fine facoltativa
- Generazione automatica delle occorrenze mancanti all'apertura dell'app (fino a un tetto di 24 per regola, per evitare raffiche eccessive dopo una lunga inattività), con identificativi deterministici che evitano doppioni anche aprendo l'app da più dispositivi
- Visibilità filtrata: ciascun utente vede le proprie regole e quelle condivise dall'altro, non quelle personali altrui

**Report**
- Grafici a istogrammi scorrevoli per fisse/variabili/entrate, patrimoni, differenza entrate-uscite, selezionabili per mese o anno
- Analisi per categoria con selezione multipla a qualsiasi livello della tassonomia, confronto fianco a fianco, filtro per etichetta
- Rilevazioni patrimoniali (conti bancari e di investimento) con andamento storico
- Esportazione in Excel con un foglio per sezione, numeri nativi pronti per calcoli e grafici personalizzati

**Categorie**
- Struttura a cinque livelli, completamente personalizzabile su L3/L4/L5
- Rinomina retroattiva: cambiare il nome di una categoria aggiorna automaticamente tutto lo storico
- Archiviazione delle voci non più in uso senza perdere lo storico collegato

**Dati e backup**
- Backup manuale in formato JSON, condivisibile via email
- Import ed esportazione completa in CSV
- Lavoro offline con sincronizzazione automatica al ritorno della rete

## Struttura del progetto

```
index.html          — l'intera applicazione (markup, stile, logica)
manifest.json        — manifest PWA
icon-*.png            — icone per l'installazione come app
```

Non ci sono altri file sorgente: tutto il codice, compresi gli stili e i componenti React, vive in `index.html`.

## Modello dati (Firestore)

| Collection | Contenuto |
|---|---|
| `transactions` | Movimenti di spesa/entrata, personali o condivisi |
| `settlements` | Conguagli registrati fra i due utenti (o fra un utente e una persona esterna) |
| `balances` | Rilevazioni periodiche dei patrimoni (conti bancari e investimenti) |
| `recurring` | Regole di spesa ricorrente |
| `esterni` | Persone esterne coinvolte in spese di gruppo |
| `condivisioni` | Registro delle spese condivise con terzi: chi ha pagato, chi partecipava, quota a testa |
| `meta/categories` | Tassonomia delle categorie a cinque livelli |
| `meta/config` | Configurazione generale (backup, ecc.) |
| `meta/photo` | Foto di intestazione condivisa |

Le spese condivise con terzi seguono un principio semplice: la quota di ciascun utente interno (Pierangelo e/o Martina) diventa sempre una spesa vera sul suo conto, indipendentemente da chi ha materialmente pagato; i debiti verso le persone esterne restano invece solo nel registro `condivisioni`, senza generare movimenti fittizi, e si ricalcolano dal vivo ogni volta che il registro cambia.

## Sviluppo

Non serve alcun ambiente di sviluppo: `index.html` è autosufficiente. Per lavorarci:

1. Modifica `index.html` direttamente.
2. Apri il file in un browser per testare in locale, oppure carica su GitHub Pages per testare come PWA.
3. Aggiorna la costante `APP_VERSION` a ogni modifica pubblicata, così l'app può segnalare quando è disponibile una versione più recente.

Per pubblicare, è sufficiente un commit e push sul branch servito da GitHub Pages.

## Note

Progetto a uso privato di due persone, non pensato per un pubblico più ampio: alcune scelte progettuali (nomi utente fissi, assenza di un vero sistema multi-tenant) riflettono questo utilizzo specifico.
