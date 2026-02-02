---
title: Instradamento dei canali
summary: "Regole di instradamento per singolo canale (WhatsApp, Telegram, Discord, Slack) e contesto condiviso"
read_when:
  - Quando modifichi l'instradamento dei canali o il comportamento dell'inbox
---

<div id="channels-routing">
  # Canali e instradamento
</div>

OpenClaw instrada le risposte **di nuovo verso il canale da cui è arrivato il messaggio**. Il
modello non sceglie un canale; l&#39;instradamento è deterministico ed è controllato dalla
configurazione dell&#39;host.

<div id="key-terms">
  ## Termini chiave
</div>

* **Channel**: `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`, `webchat`.
* **AccountId**: istanza di account specifica per canale (quando supportato).
* **AgentId**: spazio di lavoro isolato + archivio delle sessioni (“cervello”).
* **SessionKey**: la chiave del bucket usata per memorizzare il contesto e controllare la concorrenza.

<div id="session-key-shapes-examples">
  ## Formati delle chiavi di sessione (esempi)
</div>

I messaggi diretti vengono accorpati nella **sessione principale** dell’agente:

* `agent:<agentId>:<mainKey>` (valore predefinito: `agent:main:main`)

I gruppi e i canali restano isolati per canale:

* Gruppi: `agent:<agentId>:<channel>:group:<id>`
* Canali/stanze: `agent:<agentId>:<channel>:channel:<id>`

Thread:

* I thread Slack/Discord aggiungono `:thread:<threadId>` alla chiave di base.
* Gli argomenti dei forum Telegram incorporano `:topic:<topicId>` nella chiave del gruppo.

Esempi:

* `agent:main:telegram:group:-1001234567890:topic:42`
* `agent:main:discord:channel:123456:thread:987654`

<div id="routing-rules-how-an-agent-is-chosen">
  ## Regole di instradamento (come viene scelto un agente)
</div>

L’instradamento seleziona **un solo agente** per ogni messaggio in ingresso:

1. **Corrispondenza esatta del peer** (`bindings` con `peer.kind` + `peer.id`).
2. **Corrispondenza della guild** (Discord) tramite `guildId`.
3. **Corrispondenza del team** (Slack) tramite `teamId`.
4. **Corrispondenza dell’account** (`accountId` sul canale).
5. **Corrispondenza del canale** (qualsiasi account su quel canale).
6. **Agente predefinito** (`agents.list[].default`, altrimenti la prima voce dell’elenco, in ultima istanza `main`).

L’agente selezionato determina quale spazio di lavoro e quale archivio delle sessioni vengono utilizzati.

<div id="broadcast-groups-run-multiple-agents">
  ## Gruppi broadcast (eseguire più agenti)
</div>

I gruppi broadcast consentono di eseguire **più agenti** per lo stesso peer **nei casi in cui OpenClaw normalmente risponderebbe** (ad esempio: nei gruppi WhatsApp, dopo il gating su menzione/attivazione).

Config:

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

Vedi anche: [Broadcast Groups](/it/broadcast-groups).

<div id="config-overview">
  ## Panoramica della configurazione
</div>

* `agents.list`: definizioni di agenti con nome (spazio di lavoro, modello, ecc.).
* `bindings`: associa canali/account/peer in ingresso agli agenti.

Esempio:

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

<div id="session-storage">
  ## Archiviazione delle sessioni
</div>

Gli archivi delle sessioni si trovano nella directory di stato (predefinita `~/.openclaw`):

* `~/.openclaw/agents/<agentId>/sessions/sessions.json`
* Le trascrizioni in formato JSONL si trovano accanto all&#39;archivio

Puoi sovrascrivere il percorso dell&#39;archivio tramite `session.store` e il templating `{agentId}`.

<div id="webchat-behavior">
  ## Comportamento di WebChat
</div>

WebChat si associa all’**agente selezionato** e per impostazione predefinita usa la sessione principale di quell’agente. In questo modo, WebChat ti consente di vedere in un unico punto il contesto tra canali per quell’agente.

<div id="reply-context">
  ## Contesto della risposta
</div>

Le risposte ricevute includono:

* `ReplyToId`, `ReplyToBody` e `ReplyToSender` quando disponibili.
* Il contenuto citato viene concatenato a `Body` come blocco `[Replying to ...]`.

Questo comportamento è uniforme in tutti i canali.