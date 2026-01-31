---
title: Kopplung
summary: "Gateway-eigene Knoten-Kopplung (Option B) für iOS und andere Remote-Knoten"
read_when:
  - Implementieren von Freigaben für Knoten-Kopplungen ohne macOS UI
  - Hinzufügen von CLI-Flows zum Genehmigen entfernter Knoten
  - Erweitern des Gateway-Protokolls um Knotenverwaltung
---

<div id="gateway-owned-pairing-option-b">
  # Gateway-gesteuerte Kopplung (Option B)
</div>

Bei Gateway-gesteuerter Kopplung ist das **Gateway** die maßgebliche Instanz dafür, welche Knoten
sich verbinden dürfen. UIs (macOS-App, zukünftige Clients) sind nur Frontends, die
ausstehende Anfragen genehmigen oder ablehnen.

**Wichtig:** WS-Knoten verwenden **Gerätekopplung** (Rolle `node`) während `connect`.
`node.pair.*` ist ein separater Speicher für Kopplungen und beeinflusst **nicht** den WS-Handshake.
Nur Clients, die explizit `node.pair.*` aufrufen, verwenden diesen Ablauf.

<div id="concepts">
  ## Konzepte
</div>

- **Ausstehende Anfrage**: Ein Knoten hat angefragt, beizutreten; erfordert eine Genehmigung.
- **Gepaarter Knoten**: Genehmigter Knoten mit einem ausgestellten Auth-Token.
- **Transport**: Der Gateway-WS-Endpunkt leitet Anfragen weiter, entscheidet aber nicht
  über die Mitgliedschaft. (Die Unterstützung für die Legacy-TCP-Bridge ist veraltet und wurde entfernt.)

<div id="how-pairing-works">
  ## So funktioniert die Kopplung
</div>

1. Ein Knoten verbindet sich mit dem Gateway-WS und fordert eine Kopplung an.
2. Das Gateway speichert eine **ausstehende Anfrage** und emittiert `node.pair.requested`.
3. Du genehmigst oder lehnst die Anfrage ab (CLI oder UI).
4. Bei Genehmigung stellt das Gateway ein **neues Token** aus (Tokens werden bei erneuter Kopplung rotiert).
5. Der Knoten stellt die Verbindung mit dem Token erneut her und ist jetzt „gekoppelt“.

Ausstehende Anfragen verfallen automatisch nach **5 Minuten**.

<div id="cli-workflow-headless-friendly">
  ## CLI-Workflow (headless-geeignet)
</div>

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes reject <requestId>
openclaw nodes status
openclaw nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` zeigt gekoppelte bzw. verbundene Knoten und deren Fähigkeiten an.


<div id="api-surface-gateway-protocol">
  ## API-Oberfläche (Gateway-Protokoll)
</div>

Ereignisse:

- `node.pair.requested` — wird ausgelöst, wenn eine neue ausstehende Anfrage erstellt wird.
- `node.pair.resolved` — wird ausgelöst, wenn eine Anfrage genehmigt/abgelehnt/abgelaufen ist.

Methoden:

- `node.pair.request` — erstellt eine ausstehende Anfrage oder nutzt eine bestehende weiter.
- `node.pair.list` — listet ausstehende + gekoppelte Knoten auf.
- `node.pair.approve` — genehmigt eine ausstehende Anfrage (stellt ein Token aus).
- `node.pair.reject` — lehnt eine ausstehende Anfrage ab.
- `node.pair.verify` — verifiziert `{ nodeId, token }`.

Hinweise:

- `node.pair.request` ist pro Knoten idempotent: Wiederholte Aufrufe geben dieselbe
  ausstehende Anfrage zurück.
- Eine Genehmigung erzeugt **immer** ein neues Token; aus `node.pair.request` wird niemals ein Token zurückgegeben.
- Anfragen können `silent: true` enthalten, als Hinweis für automatische Genehmigungsabläufe.

<div id="auto-approval-macos-app">
  ## Auto-Genehmigung (macOS-App)
</div>

Die macOS-App kann optional versuchen, eine **stille Genehmigung** durchzuführen, wenn:

- die Anfrage als `silent` markiert ist und
- die App eine SSH-Verbindung zum Gateway-Host mit demselben Benutzerkonto überprüfen kann.

Wenn die stille Genehmigung fehlschlägt, fällt sie auf die normale „Approve/Reject“-Aufforderung zurück.

<div id="storage-local-private">
  ## Speicherung (lokal, privat)
</div>

Der Kopplungsstatus wird im Gateway-Statusverzeichnis gespeichert (Standard `~/.openclaw`):

- `~/.openclaw/nodes/paired.json`
- `~/.openclaw/nodes/pending.json`

Wenn du `OPENCLAW_STATE_DIR` änderst, wird der Ordner `nodes/` ebenfalls verschoben.

Sicherheitshinweise:

- Token sind vertraulich; behandle `paired.json` als besonders schützenswert.
- Das Erneuern eines Tokens erfordert eine erneute Freigabe (oder das Löschen des Knoteneintrags).

<div id="transport-behavior">
  ## Transportverhalten
</div>

- Der Transport ist **zustandslos**; er speichert keine Mitgliedschaftsdaten.
- Wenn das Gateway offline ist oder die Kopplung deaktiviert ist, lassen sich Knoten nicht koppeln.
- Befindet sich das Gateway im Remote-Modus, erfolgt die Kopplung nach wie vor gegen den Speicher des Remote-Gateways.