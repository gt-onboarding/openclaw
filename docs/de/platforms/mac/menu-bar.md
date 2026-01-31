---
title: Menüleiste
summary: "Statuslogik der Menüleiste und was Nutzer:innen angezeigt wird"
read_when:
  - Anpassen der macOS-Menüleisten-UI oder der Statuslogik
---

<div id="menu-bar-status-logic">
  # Statuslogik in der Menüleiste
</div>

<div id="what-is-shown">
  ## Was angezeigt wird
</div>

- Wir zeigen den aktuellen Arbeitsstatus des Agents im Menüleistensymbol und in der ersten Statuszeile des Menüs an.
- Der Gesundheitsstatus ist ausgeblendet, während Arbeit aktiv ist; er wird wieder angezeigt, wenn alle Sitzungen im Leerlauf sind.
- Der Block „Nodes“ im Menü listet nur **Geräte** auf (gekoppelte Knoten über `node.list`), nicht Client-/Präsenz-Einträge.
- Ein Abschnitt „Usage“ erscheint unter „Context“, wenn Nutzungs-Snapshots des Anbieters verfügbar sind.

<div id="state-model">
  ## Zustandsmodell
</div>

- Sitzungen: Ereignisse kommen mit `runId` (pro Ausführung) plus `sessionKey` im Payload. Die „Hauptsitzung“ ist der Schlüssel `main`; falls dieser fehlt, greifen wir auf die zuletzt aktualisierte Sitzung zurück.
- Priorität: `main` gewinnt immer. Wenn `main` aktiv ist, wird ihr Zustand sofort angezeigt. Wenn `main` inaktiv ist, wird die zuletzt aktive Nicht‑`main`‑Sitzung angezeigt. Wir springen während einer Aktivität nicht hin und her; wir wechseln nur, wenn die aktuelle Sitzung inaktiv wird oder `main` aktiv wird.
- Aktivitätstypen:
  - `job`: Ausführung eines übergeordneten Befehls (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` mit `toolName` und `meta/args`.

<div id="iconstate-enum-swift">
  ## IconState-Enum (Swift)
</div>

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (Debug-Override)

<div id="activitykind-glyph">
  ### ActivityKind → Symbol
</div>

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- Standard → 🛠️

<div id="visual-mapping">
  ### Visuelle Zuordnung
</div>

- `idle`: normaler Critter.
- `workingMain`: Badge mit Symbol, voll eingefärbt, Bein-„Working“-Animation.
- `workingOther`: Badge mit Symbol, abgeschwächter Farbton, keine Laufanimation.
- `overridden`: verwendet das gewählte Symbol/den gewählten Farbton unabhängig von der aktuellen Aktivität.

<div id="status-row-text-menu">
  ## Statuszeilentext (Menü)
</div>

- Während eine Aktivität läuft: `<Session role> · <activity label>`
  - Beispiele: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Im Leerlauf: wird stattdessen die Statusübersicht angezeigt.

<div id="event-ingestion">
  ## Ereigniserfassung
</div>

- Quelle: `agent`‑Ereignisse des Control‑Channels (`ControlChannel.handleAgentEvent`).
- Geparste Felder:
  - `stream: "job"` mit `data.state` für Start/Stopp.
  - `stream: "tool"` mit `data.phase`, `name`, optional `meta`/`args`.
- Labels:
  - `exec`: erste Zeile von `args.command`.
  - `read`/`write`: verkürzter Pfad.
  - `edit`: Pfad plus abgeleitete Änderungsart aus `meta`/Diff‑Zählern.
  - Fallback: Tool‑Name.

<div id="debug-override">
  ## Debug-Override
</div>

- Einstellungen ▸ Debug ▸ „Icon-Override“-Auswahlmenü:
  - `System (auto)` (Standard)
  - `Working: main` (pro Tool-Typ)
  - `Working: other` (pro Tool-Typ)
  - `Idle`
- Gespeichert in `@AppStorage("iconOverride")`; zugeordnet zu `IconState.overridden`.

<div id="testing-checklist">
  ## Test-Checkliste
</div>

- Job der Hauptsitzung auslösen: überprüfen, dass das Symbol sofort umschaltet und die Statuszeile das Label der Hauptsitzung anzeigt.
- Job einer Nicht-Hauptsitzung auslösen, während die Hauptsitzung im Leerlauf ist: Symbol/Status zeigt die Nicht-Hauptsitzung an und bleibt stabil, bis der Job abgeschlossen ist.
- Hauptsitzung starten, während eine andere Sitzung aktiv ist: Symbol wechselt sofort auf die Hauptsitzung.
- Schnelle, aufeinanderfolgende Tool-Ausführungen: sicherstellen, dass das Badge nicht flackert (TTL-Kulanz bei Tool-Ergebnissen).
- Die Health-Zeile erscheint wieder, sobald alle Sitzungen im Leerlauf sind.