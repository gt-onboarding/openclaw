---
title: Soul Evil
summary: "Hook SOUL Evil (sostituisce SOUL.md con SOUL_EVIL.md)"
read_when:
  - Vuoi abilitare o configurare l'hook SOUL Evil
  - Vuoi una finestra di purge o un cambio casuale di persona
---

<div id="soul-evil-hook">
  # SOUL Evil Hook
</div>

L&#39;hook SOUL Evil sostituisce il contenuto **iniettato** di `SOUL.md` con `SOUL_EVIL.md` durante
una finestra di eliminazione (purge) o in modo casuale. Non **modifica** i file su disco.

<div id="how-it-works">
  ## Come funziona
</div>

Quando viene eseguito `agent:bootstrap`, l&#39;hook può sostituire in memoria il contenuto di `SOUL.md`
prima che venga assemblato il prompt di sistema. Se `SOUL_EVIL.md` è assente o vuoto,
OpenClaw scrive un avviso nei log e mantiene il normale `SOUL.md`.

Le esecuzioni dei sotto-agenti **non** includono `SOUL.md` nei file di bootstrap, quindi questo hook
non ha alcun effetto sui sotto-agenti.

<div id="enable">
  ## Abilita
</div>

```bash
openclaw hooks enable soul-evil
```

Poi imposta la configurazione:

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

Crea `SOUL_EVIL.md` nella cartella principale dello spazio di lavoro dell&#39;agente (accanto a `SOUL.md`).

<div id="options">
  ## Opzioni
</div>

* `file` (string): nome di file SOUL alternativo (predefinito: `SOUL_EVIL.md`)
* `chance` (number 0–1): probabilità casuale, per ogni esecuzione, di usare `SOUL_EVIL.md`
* `purge.at` (HH:mm): orario di inizio dell&#39;eliminazione giornaliera (formato 24 ore)
* `purge.duration` (duration): durata della finestra (ad es. `30s`, `10m`, `1h`)

**Precedenza:** la finestra di eliminazione ha la precedenza sull&#39;opzione `chance`.

**Fuso orario:** usa `agents.defaults.userTimezone` quando impostato; altrimenti il fuso orario dell&#39;host.

<div id="notes">
  ## Note
</div>

* Nessun file viene scritto o modificato su disco.
* Se `SOUL.md` non è presente nell&#39;elenco di bootstrap, l&#39;hook non fa nulla.

<div id="see-also">
  ## Vedi anche
</div>

* [Hooks](/it/hooks)