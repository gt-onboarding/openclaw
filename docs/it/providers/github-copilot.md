---
title: GitHub Copilot
summary: "Accedi a GitHub Copilot da OpenClaw usando il device flow"
read_when:
  - Vuoi usare GitHub Copilot come provider di modelli
  - Hai bisogno del flusso `openclaw models auth login-github-copilot`
---

<div id="github-copilot">
  # GitHub Copilot
</div>

<div id="what-is-github-copilot">
  ## Che cos'è GitHub Copilot?
</div>

GitHub Copilot è l'assistente di programmazione basato sull'IA di GitHub. Fornisce accesso ai modelli
Copilot per il tuo account GitHub e il relativo piano. OpenClaw può usare Copilot come
provider di modelli in due modi diversi.

<div id="two-ways-to-use-copilot-in-openclaw">
  ## Due modi per usare Copilot in OpenClaw
</div>

<div id="1-built-in-github-copilot-provider-github-copilot">
  ### 1) provider GitHub Copilot integrato (`github-copilot`)
</div>

Usa il flusso nativo di device-login per ottenere un token GitHub, quindi
scambialo con token API di Copilot quando OpenClaw è in esecuzione. Questo è il
percorso **predefinito** e più semplice perché non richiede l’uso di VS Code.

<div id="2-copilot-proxy-plugin-copilot-proxy">
  ### 2) Copilot Proxy plugin (`copilot-proxy`)
</div>

Usa l’estensione **Copilot Proxy** per VS Code come bridge locale. OpenClaw comunica con
l’endpoint `/v1` del proxy e utilizza l’elenco di modelli che configuri lì. Scegli
questa opzione se esegui già Copilot Proxy in VS Code o se devi instradare il traffico attraverso di esso.
Devi abilitare il plugin e mantenere attiva l’estensione VS Code.

Usa GitHub Copilot come provider di modelli (`github-copilot`). Il comando di login esegue
il device flow di GitHub, salva un profilo di autenticazione e aggiorna la configurazione per usare
quel profilo.

<div id="cli-setup">
  ## Configurazione della CLI
</div>

```bash
openclaw models auth login-github-copilot
```

Ti verrà chiesto di visitare un URL e inserire un codice monouso. Mantieni il terminale aperto finché l’operazione non è completata.


<div id="optional-flags">
  ### Opzioni facoltative
</div>

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```


<div id="set-a-default-model">
  ## Impostare un modello predefinito
</div>

```bash
openclaw models set github-copilot/gpt-4o
```


<div id="config-snippet">
  ### Snippet di configurazione
</div>

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } }
}
```


<div id="notes">
  ## Note
</div>

- Richiede un TTY interattivo; esegui il comando direttamente in un terminale.
- La disponibilità dei modelli Copilot dipende dal tuo piano; se un modello viene rifiutato, prova
  un altro ID (ad esempio `github-copilot/gpt-4.1`).
- Il login memorizza un token GitHub nell'auth profile store e lo scambia con un
  token API di Copilot quando OpenClaw è in esecuzione.