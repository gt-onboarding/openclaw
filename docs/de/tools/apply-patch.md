---
title: Patch anwenden
summary: "Mehrdatei-Patches mit dem apply_patch-Tool anwenden"
read_when:
  - Du strukturierte Dateiänderungen in mehreren Dateien benötigst
  - Du patchbasierte Änderungen dokumentieren oder debuggen möchtest
---

<div id="apply_patch-tool">
  # apply_patch tool
</div>

Wende Dateiänderungen mit einem strukturierten Patch-Format an. Dies ist ideal für
Änderungen an mehreren Dateien oder mehreren Hunks, bei denen ein einzelner `edit`-Aufruf fehleranfällig wäre.

Das Tool akzeptiert einen einzelnen `input`-String, der eine oder mehrere Dateioperationen umfasst:

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```


<div id="parameters">
  ## Parameter
</div>

- `input` (erforderlich): Vollständiger Patchinhalt, einschließlich `*** Begin Patch` und `*** End Patch`.

<div id="notes">
  ## Hinweise
</div>

- Pfade werden relativ zum Root des Arbeitsbereichs aufgelöst.
- Verwende `*** Move to:` innerhalb eines `*** Update File:`-Hunks, um Dateien umzubenennen.
- `*** End of File` kennzeichnet bei Bedarf eine Einfügung ausschließlich am Dateiende (EOF-only-Insert).
- Experimentell und standardmäßig deaktiviert. Aktiviere es mit `tools.exec.applyPatch.enabled`.
- Nur für OpenAI (einschließlich OpenAI Codex). Optional nach Modell einschränkbar über
  `tools.exec.applyPatch.allowModels`.
- Konfiguration befindet sich nur unter `tools.exec`.

<div id="example">
  ## Beispiel
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
