---
title: Alma Maligna
summary: "Hook SOUL Evil (sustituye SOUL.md por SOUL_EVIL.md)"
read_when:
  - Quieres habilitar o ajustar el hook SOUL Evil
  - Quieres una ventana de purga o un cambio de personalidad aleatorio
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

El hook SOUL Evil sustituye el contenido `SOUL.md` **inyectado** por `SOUL_EVIL.md` durante
una ventana de purga o con cierta probabilidad aleatoria. **No** modifica archivos en el disco.

<div id="how-it-works">
  ## Cómo funciona
</div>

Cuando se ejecuta `agent:bootstrap`, el hook puede reemplazar el contenido de `SOUL.md` en memoria
antes de que se construya el prompt del sistema. Si `SOUL_EVIL.md` falta o está vacío,
OpenClaw genera una advertencia y mantiene el `SOUL.md` normal.

Las ejecuciones de subagentes **no** incluyen `SOUL.md` en sus archivos de bootstrap, por lo que este hook
no tiene efecto en los subagentes.

<div id="enable">
  ## Activar
</div>

```bash
openclaw hooks enable soul-evil
```

Luego establece la configuración:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

Crea `SOUL_EVIL.md` en la raíz del espacio de trabajo del agente (junto a `SOUL.md`).

<div id="options">
  ## Opciones
</div>

* `file` (string): nombre de archivo SOUL alternativo (predeterminado: `SOUL_EVIL.md`)
* `chance` (number 0–1): probabilidad aleatoria por ejecución de usar `SOUL_EVIL.md`
* `purge.at` (HH:mm): inicio diario de la purga (reloj de 24 horas)
* `purge.duration` (duration): duración de la ventana de tiempo (p. ej. `30s`, `10m`, `1h`)

**Precedencia:** la ventana de purga tiene prioridad sobre `chance`.

**Zona horaria:** usa `agents.defaults.userTimezone` cuando está configurada; en caso contrario, la zona horaria del host.

<div id="notes">
  ## Notas
</div>

* No se escribe ni se modifica ningún archivo en disco.
* Si `SOUL.md` no está en la lista de bootstrap, el hook no realiza ninguna acción.

<div id="see-also">
  ## Consulta también
</div>

* [Hooks](/es/hooks)