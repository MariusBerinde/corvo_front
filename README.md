# 🐦‍⬛ Corvo — Frontend

> Componente di interfaccia utente del sistema [Corvo]( https://github.com/MariusBerinde/corvo.git), sviluppato come progetto di tesi magistrale in collaborazione con **Sinelec S.p.A.**

---

## 📋 Descrizione

`corvo_front` è la Single Page Application (SPA) del sistema Corvo, sviluppata con **Angular 19** e **Angular Material**. Permette agli operatori di visualizzare in tempo reale lo stato degli host monitorati, gestire le policy di auditing Lynis e amministrare gli utenti del sistema.

> **Nota:** Questo repository contiene esclusivamente il codice del frontend. Gli altri componenti si trovano nei repository [`corvo_back`](https://github.com/MariusBerinde/corvo_server.git) e [`corvo_agent`](https://github.com/MariusBerinde/agent.git).

---

## ✨ Funzionalità principali

- **Login e registrazione** — autenticazione con validazione password lato client (regex con requisiti di complessità) e verifica credenziali tramite backend
- **Dashboard** — panoramica di tutti gli agent registrati con stato online/offline, aggiornata automaticamente ogni 5 minuti
- **Dettagli server** — visualizzazione e modifica di nome/descrizione del server, lista regole di sicurezza attive per categoria
- **Gestione Lynis** — avvio scansioni, selezione regole da escludere, visualizzazione e download dell'ultimo report
- **Gestione utenti** (solo Supervisor) — visualizzazione ruoli, cambio ruolo, eliminazione utenti, gestione whitelist di registrazione
- **Log di audit** — visualizzazione dei propri log (Worker) o di tutti gli utenti (Supervisor), con ordinamento per colonna
- **Cambio password** — con verifica della vecchia password e validazione della nuova

---

## 🏗️ Struttura del progetto

```
src/
├── env/                   # Variabili d'ambiente (URL API, configurazioni di deploy)
├── interfaces/            # Interfacce TypeScript (contratti dati con il backend)
├── services/              # Servizi Angular singleton per la comunicazione con il backend
│   ├── auth.service.ts              # Autenticazione e gestione utenti
│   ├── local-write.service.ts       # Lettura/scrittura localStorage
│   ├── manage-server.service.ts     # Operazioni sui server/agent
│   ├── manage-service.service.ts    # Recupero servizi per IP
│   ├── manage-log.service.ts        # Creazione, caricamento e recupero log
│   ├── manage-lynis.service.ts      # Interazioni con Lynis
│   └── manage-rule.service.ts       # Stato regole e servizi per agent
├── components/            # Componenti Angular (una per vista/funzionalità)
│   ├── login/
│   ├── registrazione/
│   ├── home/
│   ├── detail-server/
│   ├── lynis/
│   │   ├── report-view/
│   │   └── lynis-rules/
│   └── user/
│       ├── table-log/
│       └── table-user/
└── app/                   # Modulo principale e configurazione routing
```

---

## 🖥️ Schermate principali

| Schermata | Componente | Descrizione |
|---|---|---|
| Login | `LoginComponent` | Accesso con email e password |
| Registrazione | `RegistrazioneComponent` | Attivazione account (richiede pre-approvazione) |
| Home | `HomeComponent` | Lista agent con stato online/offline |
| Dettagli server | `DetailServerComponent` | Regole attive, info server, avvio Lynis |
| Lynis | `lynis` + `report-view` + `lynis-rules` | Gestione scansioni e regole da escludere |
| Utente | `user` + `table-log` + `table-user` | Profilo, log, gestione utenti (Supervisor) |

---

## 🔐 Autenticazione e sessione

Il sistema adotta un modello **stateless semplificato** pensato per reti aziendali isolate:

1. L'utente inserisce email e password → il frontend invia le credenziali al backend
2. In caso di successo, `email` e `username` vengono salvati nel `localStorage` tramite `LocalWriteService`
3. Ogni richiesta successiva include `username` nel payload o negli header per autorizzazione e audit
4. Il logout svuota il `localStorage` e carica sul backend i log accumulati localmente

Le route protette (`/home`, `/lynis`, `/utente`) sono protette da **Angular Route Guards** (`AuthGuard`): in assenza di credenziali nel `localStorage`, l'utente viene reindirizzato a `/login`. Le route pubbliche sono solo `/login` e `/register`.

> ⚠️ La vera autorizzazione avviene **sempre server-side**. La protezione client-side ha valore di UX, non di sicurezza.

---

## 🔑 Requisiti di formato password

La validazione della password è eseguita tramite espressione regolare che impone:

- almeno **1 lettera minuscola**
- almeno **1 lettera maiuscola**
- almeno **1 cifra**
- almeno **1 carattere speciale** tra: `. , / # ! ? $ % ^ & * ; : { } = - _ ( ) [ ] | ' ~ @`
- lunghezza **minima di 6 caratteri**

---

## ⚙️ Configurazione

L'URL del backend è configurato tramite i file nella cartella `src/env/`. Per lo sviluppo locale, verificare che l'URL punti all'istanza di `corvo_back` in esecuzione (default: `http://localhost:8083`).

---

## 🚀 Avvio in sviluppo

### Prerequisiti

- Node.js 18+
- Angular CLI 19
- `corvo_back` in esecuzione e raggiungibile

### Installazione dipendenze

```bash
npm install
npm install bcryptjs
```

> **Nota:** La libreria `bcryptjs` è richiesta ed è necessario installarla esplicitamente prima di avviare l'applicazione.

### Avvio del server di sviluppo

```bash
ng serve
```

L'applicazione sarà disponibile su `http://localhost:4200/` e si ricaricherà automaticamente ad ogni modifica del codice sorgente.

---

## 🔧 Comandi utili

### Generare un nuovo componente

```bash
ng generate component nome-componente
```

Per la lista completa degli schemi disponibili (componenti, direttive, pipe, ecc.):

```bash
ng generate --help
```

### Build per produzione

```bash
ng build --configuration production
```

Gli artifact vengono generati nella cartella `dist/`. La build di produzione applica ottimizzazioni automatiche per performance e dimensioni del bundle.

### Eseguire i test unitari

```bash
ng test
```

I test vengono eseguiti con [Karma](https://karma-runner.github.io).

### Eseguire test end-to-end

```bash
ng e2e
```

Angular CLI non include un framework e2e di default; è possibile configurare quello preferito.

---

## 🐳 Build e pubblicazione immagine Docker

```bash
# 1. Build di produzione
ng build --configuration production

# 2. Build dell'immagine Docker
docker build -t altairuchica/corvo-front .
```

---

## 🔒 Note di sicurezza

- La comunicazione avviene in **HTTP non cifrato**: il sistema è progettato per reti aziendali isolate (intranet) e **non deve essere esposto su reti pubbliche**
- Il ruolo utente (Worker/Supervisor) viene caricato nel `localStorage` dopo la visita alla schermata "dettagli utente". Le funzionalità Supervisor vengono mostrate/nascoste tramite `*ngIf` lato client, ma ogni operazione privilegiata è **sempre verificata server-side**: manipolare il `localStorage` non consente di bypassare i controlli del backend (risposta `403 Forbidden`)
- I log accumulati localmente vengono caricati sul backend al momento del logout

---

## 📄 Contesto del progetto

`corvo_front` fa parte del sistema **Corvo**, sviluppato come progetto di tesi magistrale presso **Sinelec S.p.A.** Il sistema completo comprende anche il backend Java Spring Boot (`corvo_back`) e l'agent Python (`corvo_agent`).

Generato con [Angular CLI](https://github.com/angular/angular-cli) versione 19.2.7.

Per maggiori informazioni su Angular CLI: [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli).

---

## 📜 Licenza

Tutti i diritti riservati. Progetto sviluppato nell'ambito di un tirocinio magistrale presso Sinelec S.p.A.
