---
title: Accesso al browser
summary: "Accessi manuali per l'automazione del browser + pubblicazione su X/Twitter"
read_when:
  - Hai bisogno di accedere a siti web per l'automazione del browser
  - Vuoi pubblicare aggiornamenti su X/Twitter
---

<div id="browser-login-xtwitter-posting">
  # Accesso dal browser + pubblicazione su X/Twitter
</div>

<div id="manual-login-recommended">
  ## Accesso manuale (consigliato)
</div>

Quando un sito richiede l’accesso, **effettua l’accesso manualmente** nel profilo del browser **host** (il browser di OpenClaw).

**Non** fornire al modello le tue credenziali. Gli accessi automatizzati spesso attivano i sistemi di difesa anti‑bot e possono bloccare l’account.

Torna alla documentazione principale del browser: [Browser](/it/tools/browser).

<div id="which-chrome-profile-is-used">
  ## Quale profilo di Chrome viene utilizzato?
</div>

OpenClaw controlla un **profilo Chrome dedicato** (chiamato `openclaw`, UI con sfumature arancioni). Questo è separato dal profilo del browser che usi ogni giorno.

Due modi semplici per accedervi:

1. **Chiedi all&#39;agente di aprire il browser** e poi effettua tu stesso l&#39;accesso.
2. **Aprilo tramite CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Se utilizzi più profili, passa l&#39;opzione `--browser-profile <name>` (il valore predefinito è `openclaw`).

<div id="xtwitter-recommended-flow">
  ## X/Twitter: flusso consigliato
</div>

* **Lettura/ricerca/thread:** usa la skill CLI **bird** (senza browser, stabile).
  * Repo: https://github.com/steipete/bird
* **Pubblica aggiornamenti:** usa il browser dell&#39;host (login manuale).

<div id="sandboxing-host-browser-access">
  ## Sandbox e accesso al browser dell&#39;host
</div>

Le sessioni del browser eseguite in sandbox hanno **maggiore probabilità** di attivare i sistemi di rilevamento bot. Per X/Twitter (e altri siti con policy più restrittive), è preferibile usare il browser **dell&#39;host**.

Se l&#39;agente è in sandbox, lo strumento browser usa la sandbox come impostazione predefinita. Per consentire il controllo dall&#39;host:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true
        }
      }
    }
  }
}
```

Quindi punta al browser host:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Oppure disabilita la sandbox per l&#39;agente che pubblica gli aggiornamenti.
