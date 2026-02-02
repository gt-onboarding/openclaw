---
title: Estado
summary: "Referencia de la CLI para `openclaw health` (endpoint de salud del Gateway vía RPC)"
read_when:
  - Quieres comprobar rápidamente el estado del Gateway en ejecución
---

<div id="openclaw-health">
  # `openclaw health`
</div>

Recupera el estado de salud del Gateway en ejecución.

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

Notas:

* `--verbose` ejecuta sondas en tiempo real e imprime los tiempos por cuenta cuando hay varias cuentas configuradas.
* La salida incluye almacenes de sesión por agente cuando hay varios agentes configurados.
