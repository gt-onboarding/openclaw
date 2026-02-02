---
title: Sicurezza
summary: "Riferimento CLI per `openclaw security` (analisi e correzione degli errori di sicurezza più comuni)"
read_when:
  - Vuoi eseguire un rapido controllo di sicurezza su configurazione e stato
  - Vuoi applicare suggerimenti di correzione sicuri (chmod, irrigidimento dei valori predefiniti)
---

<div id="openclaw-security">
  # `openclaw security`
</div>

Strumenti per la sicurezza (audit + correzioni opzionali).

Risorse correlate:

* Guida alla sicurezza: [Sicurezza](/it/gateway/security)

<div id="audit">
  ## Audit
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

L&#39;audit segnala quando più mittenti di DM condividono la sessione principale e raccomanda `session.dmScope="per-channel-peer"` (o `per-account-channel-peer` per i canali multi-account) per le caselle di posta condivise.
Segnala anche quando vengono utilizzati modelli di piccole dimensioni (`<=300B`) senza sandbox e con gli strumenti web/browser abilitati.
