---
title: Tlon
summary: "Unterstützungsstatus, Funktionsumfang und Konfiguration von Tlon/Urbit"
read_when:
  - Bei der Arbeit an Tlon/Urbit-Kanalfunktionen
---

<div id="tlon-plugin">
  # Tlon (Plugin)
</div>

Tlon ist ein dezentraler Messenger, der auf Urbit basiert. OpenClaw verbindet sich mit deinem Urbit-Ship und kann
auf DMs und Gruppenchats antworten. Gruppenantworten erfordern standardmäßig eine @-Erwähnung und können
über eine Allowlist weiter eingeschränkt werden.

Status: unterstützt über Plugin. DMs, Gruppenerwähnungen, Thread-Antworten und Fallback auf Nur-Text-Medien
(URL an die Beschriftung angehängt). Reaktionen, Umfragen und native Medien-Uploads werden nicht unterstützt.

<div id="plugin-required">
  ## Plugin erforderlich
</div>

Tlon wird als Plugin ausgeliefert und ist nicht in der Core-Installation enthalten.

Installation über die CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/tlon
```

Lokaler Checkout (Ausführung aus einem Git-Repository):

```bash
openclaw plugins install ./extensions/tlon
```

Details: [Plugins](/de/plugin)

<div id="setup">
  ## Setup
</div>

1. Installiere das Tlon-Plugin.
2. Rufe deine Ship-URL und deinen Login-Code ab.
3. Konfiguriere `channels.tlon`.
4. Starte den Gateway neu.
5. Schreibe dem Bot eine DM oder erwähne ihn in einem Gruppen-Channel.

Minimale Konfiguration (einzelner Account):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup"
    }
  }
}
```

<div id="group-channels">
  ## Gruppenkanäle
</div>

Die automatische Erkennung ist standardmäßig aktiviert. Du kannst Kanäle auch manuell anheften:

```json5
{
  channels: {
    tlon: {
      groupChannels: [
        "chat/~host-ship/general",
        "chat/~host-ship/support"
      ]
    }
  }
}
```

Automatische Erkennung deaktivieren:

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false
    }
  }
}
```

<div id="access-control">
  ## Zugriffskontrolle
</div>

DM-Allowlist (leer = alle zulassen):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"]
    }
  }
}
```

Gruppenberechtigungen (standardmäßig eingeschränkt):

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"]
          },
          "chat/~host-ship/announcements": {
            mode: "open"
          }
        }
      }
    }
  }
}
```

<div id="delivery-targets-clicron">
  ## Zustellziele (CLI/cron)
</div>

Verwende diese mit `openclaw message send` oder für Cron-Zustellung:

* DM: `~sampel-palnet` oder `dm/~sampel-palnet`
* Gruppe: `chat/~host-ship/channel` oder `group:~host-ship/channel`

<div id="notes">
  ## Hinweise
</div>

* Antworten in Gruppen erfordern eine Erwähnung (z. B. `~your-bot-ship`), damit reagiert wird.
* Antworten in Threads: Wenn die eingehende Nachricht in einem Thread ist, antwortet OpenClaw im selben Thread.
* Medien: `sendMedia` fällt auf Text + URL zurück (kein nativer Upload).