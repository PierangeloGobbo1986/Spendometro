# Spendometro

Contabilità familiare condivisa di Pierangelo e Martina. Dipartimento contabile del PMAS.

PWA in un solo file, React da CDN, database Firebase Firestore, pubblicazione su GitHub Pages.

- **App**: https://pierangelogobbo1986.github.io/Spendometro/ (la S maiuscola conta: GitHub Pages distingue maiuscole e minuscole nel percorso)
- **Console Firebase**: https://console.firebase.google.com/project/spendometro-618c4
- **Progetto Firebase**: `spendometro-618c4`

---

## File del repository

| File | Ruolo |
|---|---|
| `index.html` | Tutta l'app: interfaccia, logica, connessione Firebase |
| `manifest.json` | Configurazione PWA (nome, icone, schermo intero) |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png` | Icone della schermata Home |
| `README.md` | Questo file |

Non c'è alcun passaggio di compilazione. Si modifica `index.html`, si fa commit, ed entro un minuto o due il sito è aggiornato.

---

## Installazione sull'iPhone

Va fatta una volta per ciascun telefono, **in quest'ordine**.

1. Aprire l'indirizzo dell'app **in Safari** (non Chrome: su iOS solo Safari può installare).
2. Premere **Condividi**, scorrere, scegliere **Aggiungi a Home**.
3. Aprire l'app **dall'icona**, non da Safari.
4. Fare il login (email e password) **dentro l'app installata**.
5. Chiudere del tutto e riaprire dall'icona per verificare che la sessione persista.

Il punto 4 è il più importante: Safari e l'app installata sono due contenitori separati, quindi un login fatto in Safari non vale per l'icona. E solo l'app installata è protetta dalle pulizie automatiche di iOS che cancellerebbero la sessione dopo alcuni giorni di inattività.

---

## Backup mensile

Il primo di ogni mese l'app si **blocca per Pierangelo** finché il backup non viene eseguito. Martina non vede né blocchi né avvisi.

1. Premere **Backup database**.
2. Nella schermata di condivisione scegliere **Mail**.
3. Digitare le prime lettere del destinatario, iOS completa l'indirizzo.
4. Inviare.

Il file si chiama `SPENDOMETRO_backup_AAAA-MM-GG_v1_Ntx.json`, dove N è il numero di movimenti. La data ISA all'inizio fa sì che l'ordine alfabetico coincida con quello cronologico, e il conteggio permette di notare a colpo d'occhio un backup anomalo.

Il destinatario si imposta dentro l'app, in **Dati**, ed è salvato su Firestore e non nel codice pubblico.

### Filtro Gmail consigliato

Da fare una volta sola, in Gmail → Impostazioni → Filtri → Crea nuovo filtro:

- **Oggetto contiene**: `SPENDOMETRO_backup`
- Azioni: **Applica etichetta** `Spendometro/Backup` e **Salta Posta in arrivo**

Così i backup si archiviano da soli in ordine, senza intasare la posta.

---

## Ripristino da backup

Il file JSON contiene tutto: `transactions`, `settlements`, `balances`, `categories`, `config`.

**Ripristino parziale (caso normale).** Se sono sparite alcune righe, la strada più semplice è aprire il JSON, ricavare le righe mancanti e reinserirle a mano, oppure usare l'import CSV se il numero è alto.

**Ripristino totale (caso raro).** Serve uno script Python con `firebase-admin` e una chiave di servizio scaricata dalla console. Da chiedere quando serve. La chiave va tenuta in `~/.config/` con permessi `600` e **mai** dentro una cartella sincronizzata.

**Verifica del backup.** Ogni tanto vale la pena aprire il file più recente e controllare che il numero di transazioni sia plausibile. Un backup mai verificato non è un backup.

---

## Funzionamento offline

L'app usa la cache persistente di Firestore, quindi il database vive anche dentro il telefono.

Registrando una spesa senza rete, questa compare subito nella lista con la dicitura **in attesa** e viene scritta su disco nel telefono, quindi sopravvive alla chiusura dell'app e al riavvio. Al ritorno della connessione Firestore sincronizza da solo e la dicitura sparisce: è la conferma effettiva del server, non una stima.

In alto compare una fascia gialla quando si è offline. In **Dati** c'è il contatore dei movimenti ancora in attesa.

In caso di modifica contemporanea dello stesso movimento da parte di entrambi mentre sono offline, vince l'ultima scrittura che arriva al server.

---

## Foto dell'intestazione

Si carica da **Dati → Foto dell'intestazione**, è condivisa, e viene ridimensionata e compressa nel browser prima del salvataggio.

Non usa Firebase Storage, che da febbraio 2026 richiede il piano Blaze: l'immagine è salvata come dato dentro Firestore, nel documento `meta/photo`. Per questo vale solo per l'intestazione e non per allegare scontrini ai singoli movimenti.

---

## Struttura dei dati

Tutto in un unico database condiviso: entrambi leggono e scrivono tutto, ed è l'interfaccia a mostrare a ciascuno la propria quota.

| Collection | Contenuto |
|---|---|
| `transactions` | Un documento per movimento |
| `settlements` | Un documento per conguaglio (bonifico di pareggio) |
| `balances` | Rilevazioni patrimoni Revolut, Fineco, Intesa |
| `meta/categories` | Albero categorie (gruppi e voci) in un unico documento |
| `meta/config` | Email di backup, data e autore dell'ultimo backup |
| `meta/photo` | Foto dell'intestazione, condivisa |

### Categorie: quattro livelli

- **L1** Spese / Entrate — fisso
- **L2** Fisse / Variabili, oppure Stipendio / Altre entrate — fisso
- **L3** Gruppo con emoji (Casa, Auto, Persona…) — creabile e rinominabile
- **L4** Voce specifica (Benzina, Barbiere…) — creabile, rinominabile, **facoltativa**

Un gruppo può esistere sotto rami diversi: "Persona" sta sia nelle fisse (Barbiere, Dentista) sia nelle variabili (Cosmesi, Abbigliamento). La natura si determina dal livello superiore.

**La rinomina è retroattiva.** I movimenti memorizzano l'identificativo, non il nome, quindi rinominando una categoria cambia anche tutto lo storico. È voluto.

**L'eliminazione è protetta.** Una categoria con movimenti collegati viene archiviata, quindi sparisce dal menu di inserimento ma resta leggibile nello storico. Solo le categorie vuote si eliminano davvero.

### Spese condivise

Ogni spesa condivisa è **un solo record**, mai duplicato. Le viste dei due utenti sono calcolate, non memorizzate, quindi non possono divergere.

I quattro modi, con l'esempio di una cena da 100 inserita da Pierangelo al 50%:

| Modo | Quota Pierangelo | Quota Martina | Saldo |
|---|---|---|---|
| Pagato da me e diviso | 50 | 50 | Martina deve 50 |
| Pagato dall'altro e diviso | 50 | 50 | Pierangelo deve 50 |
| Devo all'altro l'intera cifra | 100 | 0 | Pierangelo deve 100 |
| L'altro mi deve l'intera cifra | 0 | 100 | Martina deve 100 |

Il **Conguaglio** si attiva solo per chi è in debito, chiede conferma del bonifico, azzera il saldo e non tocca la lista movimenti, perché le spese erano già state contabilizzate una per una.

### Investimenti

Non entrano in questo database. Il patrimonio si registra come **rilevazione** nella sezione Report (Revolut, Fineco, Intesa), che alimenta il grafico storico.

---

## Sicurezza

Il repository è pubblico, ma **non espone alcun dato**. Nel codice ci sono solo la configurazione Firebase, che è progettata per stare nel browser, e la logica dell'app. I dati vivono su Firestore, protetti dalle regole.

Regole attive (Firestore → Regole): accesso consentito solo ai due UID autorizzati, in lettura e scrittura. Chiunque altro, anche conoscendo la configurazione, riceve un rifiuto.

Domini autorizzati in Authentication → Settings: `localhost`, `spendometro-618c4.web.app`, `spendometro-618c4.firebaseapp.com`, `pierangelogobbo1986.github.io`.

Piano Firebase: **Spark** (gratuito). A due utenti i limiti giornalieri non vengono mai sfiorati.

---

## Problemi frequenti

**Il login non funziona dopo aver cambiato dominio.** Aggiungere il nuovo dominio in Authentication → Settings → Domini autorizzati. È l'errore più comune.

**"Missing or insufficient permissions".** Gli UID nelle regole non corrispondono. Ricopiarli dalla tabella Users con il pulsante di copia, senza trascriverli a mano: sono sensibili a maiuscole e minuscole.

**La sessione scade e chiede di nuovo il login.** L'app è stata aperta da Safari invece che dall'icona, oppure non è mai stata aggiunta alla schermata Home.

**Pagina bianca dopo un aggiornamento del codice.** Errore JavaScript. Aprire l'app da Safari sul Mac e guardare la console per il messaggio.

**Le modifiche non compaiono.** GitHub Pages impiega uno o due minuti. Se persiste, ricaricare forzando l'aggiornamento della cache.

**404 sul sito.** Quasi sempre è l'indirizzo: il percorso è `/Spendometro/` con la S maiuscola. Se invece Actions mostra pubblicazioni annullate, è perché più commit ravvicinati si annullano a vicenda: caricare tutti i file **in un unico commit** e attendere il pallino verde prima di riprovare. Il contenuto di `index.html` non può causare un 404: un errore nel codice produce pagina bianca, non 404.

---

## Da rivedere in futuro

- Import dello storico completo Spendee e Budjet di entrambi (in sospeso, mancano alcuni mesi perché Spendee gratuito esporta solo 365 giorni).
- Generazione PDF con jsPDF al posto della stampa del browser.
- Confronto fra spese fisse reali e baseline del PMAS, per accorgersi degli sforamenti.
