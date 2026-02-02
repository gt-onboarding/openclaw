---
title: Reimposta
summary: "Riferimento CLI per `openclaw reset` (reimposta stato/configurazione locali)"
read_when:
  - Vuoi azzerare lo stato locale mantenendo installata la CLI
  - Vuoi una dry-run/anteprima di ciò che verrebbe rimosso
---

<div id="openclaw-reset">
  # `openclaw reset`
</div>

Ripristina la configurazione e lo stato locali (mantiene la CLI installata).

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```
