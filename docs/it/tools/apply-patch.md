---
title: Applica patch
summary: "Applica patch su più file con lo strumento apply_patch"
read_when:
  - Hai bisogno di modifiche strutturate a più file
  - Vuoi documentare o eseguire il debug di modifiche basate su patch
---

<div id="apply_patch-tool">
  # apply_patch tool
</div>

Applica modifiche ai file usando un formato di patch strutturato. È ideale per
modifiche multi-file o con più hunk, in cui una singola chiamata a `edit` sarebbe poco robusta.

Lo strumento accetta un&#39;unica stringa `input` che racchiude una o più operazioni sui file:

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
  ## Parametri
</div>

- `input` (obbligatorio): Contenuto completo della patch, comprese le righe `*** Begin Patch` e `*** End Patch`.

<div id="notes">
  ## Note
</div>

- I percorsi vengono risolti rispetto alla radice dello spazio di lavoro.
- Usa `*** Move to:` all'interno di un hunk `*** Update File:` per rinominare i file.
- `*** End of File` indica un inserimento che riguarda solo l'EOF, quando necessario.
- Funzionalità sperimentale, disabilitata per impostazione predefinita. Abilitala con `tools.exec.applyPatch.enabled`.
- Solo OpenAI (incluso OpenAI Codex). Limitabile opzionalmente per modello tramite
  `tools.exec.applyPatch.allowModels`.
- La configurazione è disponibile solo sotto `tools.exec`.

<div id="example">
  ## Esempio
</div>

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```
