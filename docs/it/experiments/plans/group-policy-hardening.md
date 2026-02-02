---
title: Hardening dei criteri di gruppo
summary: "Hardening della lista di autorizzati Telegram: normalizzazione di prefissi e spazi bianchi"
read_when:
  - Durante la revisione delle modifiche storiche alla lista di autorizzati di Telegram
---

<div id="telegram-allowlist-hardening">
  # Rafforzamento della sicurezza della lista di autorizzati Telegram
</div>

**Data**: 2026-01-05\
**Stato**: Completato\
**PR**: #216

<div id="summary">
  ## Riepilogo
</div>

Le liste di autorizzati di Telegram ora accettano i prefissi `telegram:` e `tg:` in modo case-insensitive e tollerano spazi bianchi accidentali. Questo allinea i controlli in ingresso sulla lista di autorizzati con la normalizzazione dell’operazione di send in uscita.

<div id="what-changed">
  ## Cosa è cambiato
</div>

* I prefissi `telegram:` e `tg:` sono considerati equivalenti (senza distinzione tra maiuscole e minuscole).
* Le voci della lista di autorizzati vengono ripulite dagli spazi; le voci vuote vengono ignorate.

<div id="examples">
  ## Esempi
</div>

Tutti questi sono considerati validi per lo stesso ID:

* `telegram:123456`
* `TG:123456`
* `tg:123456`

<div id="why-it-matters">
  ## Perché è importante
</div>

Copia/incolla da log o ID chat include spesso prefissi e spazi bianchi. La normalizzazione evita
falsi negativi quando devi decidere se rispondere in DM o nei gruppi.

<div id="related-docs">
  ## Documenti correlati
</div>

* [Chat di gruppo](/it/concepts/groups)
* [Provider Telegram](/it/channels/telegram)