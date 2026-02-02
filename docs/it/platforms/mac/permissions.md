---
title: Autorizzazioni
summary: "Persistenza delle autorizzazioni macOS (TCC) e requisiti di firma"
read_when:
  - Eseguire il debug di prompt di autorizzazione di macOS mancanti o bloccati
  - Creare il pacchetto o firmare l'app macOS
  - Modificare gli ID bundle o i percorsi di installazione dell'app
---

<div id="macos-permissions-tcc">
  # permessi macOS (TCC)
</div>

Le autorizzazioni concesse in macOS sono delicate. TCC associa un'autorizzazione alla
firma del codice dell'app, all'identificatore del bundle e al percorso sul disco. Se uno qualsiasi di questi elementi cambia,
macOS considera l'app come nuova e può revocare o non mostrare più le richieste di autorizzazione.

<div id="requirements-for-stable-permissions">
  ## Requisiti per mantenere autorizzazioni stabili
</div>

- Stesso percorso: esegui l'app sempre dalla stessa posizione fissa (per OpenClaw, `dist/OpenClaw.app`).
- Stesso bundle identifier: cambiare il bundle identifier crea una nuova identità per le autorizzazioni.
- App firmata: le build non firmate o firmate ad-hoc non conservano le autorizzazioni.
- Firma coerente: usa un certificato reale Apple Development o Developer ID
  in modo che la firma rimanga stabile tra una ricompilazione e l'altra.

Le firme ad-hoc generano una nuova identità a ogni build. macOS dimenticherà le
autorizzazioni concesse in precedenza e i prompt possono scomparire del tutto finché le voci obsolete non vengono eliminate.

<div id="recovery-checklist-when-prompts-disappear">
  ## Checklist di ripristino quando i prompt scompaiono
</div>

1. Esci dall&#39;app.
2. Rimuovi la voce dell&#39;app in Impostazioni di Sistema -&gt; Privacy e sicurezza.
3. Riavvia l&#39;app dallo stesso percorso e concedi di nuovo le autorizzazioni.
4. Se il prompt ancora non compare, reimposta le voci di TCC con `tccutil` e riprova.
5. Alcune autorizzazioni ricompaiono solo dopo un riavvio completo di macOS.

Esempi di reimpostazione (sostituisci il bundle ID se necessario):

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

Se stai testando le autorizzazioni, firma sempre con un certificato reale. Le build ad hoc sono accettabili solo per esecuzioni locali rapide in cui le autorizzazioni non sono rilevanti.
