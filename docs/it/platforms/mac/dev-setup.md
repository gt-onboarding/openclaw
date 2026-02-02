---
title: Configurazione ambiente di sviluppo
summary: "Guida alla configurazione per sviluppatori che lavorano sull'app macOS di OpenClaw"
read_when:
  - Configurazione dell'ambiente di sviluppo su macOS
---

<div id="macos-developer-setup">
  # Configurazione dell'ambiente di sviluppo su macOS
</div>

Questa guida illustra i passaggi necessari per compilare ed eseguire l'applicazione OpenClaw per macOS a partire dal codice sorgente.

<div id="prerequisites">
  ## Prerequisiti
</div>

Prima di compilare l'app, assicurati di avere installato quanto segue:

1.  **Xcode 26.2+**: Necessario per lo sviluppo in Swift.
2.  **Node.js 22+ & pnpm**: Necessari per il Gateway, la CLI e gli script di packaging.

<div id="1-install-dependencies">
  ## 1. Installa le dipendenze
</div>

Installa le dipendenze globali del progetto:

```bash
pnpm install
```


<div id="2-build-and-package-the-app">
  ## 2. Compilare e creare il pacchetto dell&#39;app
</div>

Per compilare l&#39;app macOS e impacchettarla in `dist/OpenClaw.app`, esegui:

```bash
./scripts/package-mac-app.sh
```

Se non hai un certificato Apple Developer ID, lo script userà automaticamente la **firma ad-hoc** (`-`).

Per le modalità di esecuzione di sviluppo, i flag di firma e la risoluzione dei problemi relativi al Team ID, consulta il README dell&#39;app macOS:
https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md

> **Nota**: Le app firmate ad-hoc possono attivare avvisi di sicurezza. Se l&#39;app va in crash immediatamente con &quot;Abort trap 6&quot;, consulta la sezione [Troubleshooting](#troubleshooting).


<div id="3-install-the-cli">
  ## 3. Installa la CLI
</div>

L&#39;app macOS richiede un&#39;installazione globale della CLI `openclaw` per gestire le attività in background.

**Per installarla (consigliato):**

1. Apri l&#39;app OpenClaw.
2. Vai alla scheda **Generale** delle impostazioni.
3. Fai clic su **&quot;Install CLI&quot;**.

In alternativa, installala manualmente:

```bash
npm install -g openclaw@<version>
```


<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="build-fails-toolchain-or-sdk-mismatch">
  ### Compilazione non riuscita: incompatibilità tra toolchain e SDK
</div>

La compilazione dell&#39;app macOS richiede l&#39;SDK macOS più recente e la toolchain Swift 6.2.

**Dipendenze di sistema (obbligatorie):**

* **Ultima versione di macOS disponibile in Aggiornamento Software** (richiesta dagli SDK di Xcode 26.2)
* **Xcode 26.2** (toolchain Swift 6.2)

**Verifiche:**

```bash
xcodebuild -version
xcrun swift --version
```

Se le versioni non corrispondono, aggiorna macOS/Xcode e ricompila il progetto.


<div id="app-crashes-on-permission-grant">
  ### L'app va in crash quando concedi i permessi
</div>

Se l'app va in crash quando provi a consentire l'accesso al **Riconoscimento vocale** o al **Microfono**, potrebbe essere dovuto a una cache TCC danneggiata o a un'incongruenza nella firma.

**Soluzione:**

1. Reimposta le autorizzazioni TCC:
   ```bash
   tccutil reset All bot.molt.mac.debug
   ```
2. Se questo non funziona, modifica temporaneamente il `BUNDLE_ID` in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) per forzare una ripartenza da zero da parte di macOS.

<div id="gateway-starting-indefinitely">
  ### Gateway &quot;Starting...&quot; indefinitamente
</div>

Se lo stato del Gateway resta su &quot;Starting...&quot;, verifica che non ci sia un processo zombie che sta occupando la porta:

```bash
openclaw gateway status
openclaw gateway stop

# Se non stai utilizzando un LaunchAgent (modalità dev / esecuzioni manuali), trova il listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

Se un&#39;istanza avviata manualmente sta occupando la porta, interrompi quel processo (Ctrl+C). Come ultima risorsa, termina il processo con il PID che hai trovato sopra.
