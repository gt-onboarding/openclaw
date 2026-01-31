---
title: Tippindikatoren
summary: "Wann OpenClaw Tippindikatoren anzeigt und wie du sie anpasst"
read_when:
  - Ändern des Verhaltens oder der Standardwerte der Tippindikatoren
---

<div id="typing-indicators">
  # Tippindikatoren
</div>

Tippindikatoren werden in den Chat-Kanal gesendet, während eine Ausführung aktiv ist. Verwende
`agents.defaults.typingMode`, um zu steuern, **wann** das Tippen beginnt, und `typingIntervalSeconds`,
um zu steuern, **wie oft** die Anzeige aktualisiert wird.

<div id="defaults">
  ## Standardwerte
</div>

Wenn `agents.defaults.typingMode` **nicht gesetzt** ist, behält OpenClaw das Legacy-Verhalten bei:

- **Direktchats**: Die Eingabeanzeige beginnt sofort, sobald die Modellschleife startet.
- **Gruppenchats mit Erwähnung**: Die Eingabeanzeige beginnt sofort.
- **Gruppenchats ohne Erwähnung**: Die Eingabeanzeige beginnt erst, wenn das Streaming des Nachrichtentexts beginnt.
- **Herzschlag-Läufe**: Die Eingabeanzeige ist deaktiviert.

<div id="modes">
  ## Modi
</div>

Setze `agents.defaults.typingMode` auf einen der folgenden Werte:

- `never` — es wird **niemals** ein Tippindikator angezeigt.
- `instant` — beginne zu tippen, **sobald die Modellschleife startet**, selbst wenn die Ausführung später nur das Silent-Antwort-Token zurückgibt.
- `thinking` — beginne zu tippen beim **ersten Reasoning-Delta** (erfordert
  `reasoningLevel: "stream"` für die Ausführung).
- `message` — beginne zu tippen beim **ersten Text-Delta mit Inhalt** (ignoriert
  das Silent-Token `NO_REPLY`).

Reihenfolge danach, **wie früh es ausgelöst wird**:
`never` → `message` → `thinking` → `instant`

<div id="configuration">
  ## Konfiguration
</div>

```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

Du kannst den Modus oder die Frequenz pro Sitzung überschreiben:

```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```


<div id="notes">
  ## Hinweise
</div>

- Im `message`-Modus wird bei ausschließlich stillen Antworten (z. B. dem `NO_REPLY`-
  Token zur Unterdrückung der Ausgabe) kein Tippen angezeigt.
- `thinking` wird nur ausgelöst, wenn der Lauf Reasoning streamt (`reasoningLevel: "stream"`).
  Wenn das Modell keine Reasoning-Deltas ausgibt, beginnt keine Tippanzeige.
- Herzschläge zeigen niemals Tippen an, unabhängig vom Modus.
- `typingIntervalSeconds` steuert die **Aktualisierungsfrequenz**, nicht die Startzeit.
  Der Standardwert ist 6 Sekunden.