---
title: Modelli di dispositivo
summary: "Come OpenClaw gestisce gli identificatori di modello dei dispositivi Apple e li mappa su nomi più leggibili nell'app macOS."
read_when:
  - Aggiornare le mappature degli identificatori di modello di dispositivo o i file NOTICE/license
  - Modificare il modo in cui la UI di Instances visualizza i nomi dei dispositivi
---

<div id="device-model-database-friendly-names">
  # Database dei modelli di dispositivo (nomi leggibili)
</div>

L'app companion per macOS mostra nomi leggibili dei modelli di dispositivi Apple nella UI **Instances**, associando gli identificatori di modello Apple (ad es. `iPad16,6`, `Mac16,6`) a nomi comprensibili.

La mappatura è inclusa come JSON in:

- `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

<div id="data-source">
  ## Origine dei dati
</div>

Attualmente integriamo la mappatura dal repository con licenza MIT:

- `kyle-seongwoo-jun/apple-device-identifiers`

Per mantenere le build deterministiche, i file JSON sono bloccati a specifici commit upstream (annotati in `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

<div id="updating-the-database">
  ## Aggiornamento del database
</div>

1. Seleziona i commit upstream a cui vuoi fare riferimento (uno per iOS, uno per macOS).
2. Aggiorna gli hash dei commit in `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3. Riscarica i file JSON, ancorati a quei commit:

```bash
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<SHA del commit per mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4. Assicurati che `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` sia ancora allineato all’upstream (sostituiscilo se la licenza upstream cambia).
5. Verifica che l’app macOS si compili senza avvisi (nessun warning):

```bash
swift build --package-path apps/macos
```
