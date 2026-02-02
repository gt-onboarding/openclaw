---
title: Configurazione delle abilità
summary: "Schema di configurazione delle abilità ed esempi"
read_when:
  - Aggiunta o modifica della configurazione delle abilità
  - Modifica della lista di autorizzati inclusa nel pacchetto o del comportamento di installazione
---

<div id="skills-config">
  # Configurazione delle abilità
</div>

Tutta la configurazione delle abilità è contenuta nella sezione `skills` in `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: [
        "~/Projects/agent-scripts/skills",
        "~/Projects/oss/some-skill-pack/skills"
      ],
      watch: true,
      watchDebounceMs: 250
    },
    install: {
      preferBrew: true,
      nodeManager: "npm" // npm | pnpm | yarn | bun (il runtime del Gateway è ancora Node; bun non consigliato)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE"
        }
      },
      peekaboo: { enabled: true },
      sag: { enabled: false }
    }
  }
}
```


<div id="fields">
  ## Campi
</div>

- `allowBundled`: lista di autorizzati opzionale solo per le abilità **bundled**. Quando impostata, solo
  le abilità bundled presenti nella lista sono ammesse (le abilità gestite/spazio di lavoro non sono interessate).
- `load.extraDirs`: directory aggiuntive di abilità da scandire (precedenza più bassa).
- `load.watch`: monitora le cartelle delle abilità e aggiorna lo snapshot delle abilità (default: true).
- `load.watchDebounceMs`: debounce per gli eventi del watcher delle abilità in millisecondi (default: 250).
- `install.preferBrew`: preferisce gli installer di brew quando disponibili (default: true).
- `install.nodeManager`: preferenza per l'installer node (`npm` | `pnpm` | `yarn` | `bun`, default: npm).
  Questo influisce solo sulle **installazioni delle abilità**; il runtime del Gateway deve comunque essere Node
  (Bun non è consigliato per WhatsApp/Telegram).
- `entries.<skillKey>`: override per singola abilità.

Campi per singola abilità:

- `enabled`: imposta `false` per disabilitare un'abilità anche se è bundled/installata.
- `env`: variabili di ambiente iniettate per l'esecuzione dell'agente (solo se non già impostate).
- `apiKey`: comodità opzionale per le abilità che dichiarano una variabile di ambiente primaria.

<div id="notes">
  ## Note
</div>

- Le chiavi in `entries` vengono associate per impostazione predefinita al nome dell'abilità. Se un'abilità definisce
  `metadata.openclaw.skillKey`, viene invece utilizzata quella chiave.
- Le modifiche alle abilità vengono rilevate al turno successivo dell'agente, quando il watcher è abilitato.

<div id="sandboxed-skills-env-vars">
  ### Abilità in sandbox + variabili d'ambiente
</div>

Quando una sessione è in esecuzione in **sandbox**, i processi delle abilità vengono eseguiti all'interno di Docker. La sandbox
**non** eredita il `process.env` dell'host.

Usa una delle seguenti opzioni:

- `agents.defaults.sandbox.docker.env` (oppure, per singolo agente, `agents.list[].sandbox.docker.env`)
- incorpora le variabili d'ambiente direttamente nella tua immagine sandbox personalizzata

Le impostazioni globali `env` e `skills.entries.<skill>.env/apiKey` si applicano solo alle esecuzioni sull'**host**.