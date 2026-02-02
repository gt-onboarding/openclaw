---
title: Configurazione del modello
summary: "Esplorazione: configurazione del modello, profili di autenticazione e comportamento di fallback"
read_when:
  - Esplorazione di idee future per la selezione del modello e i profili di autenticazione
---

<div id="model-config-exploration">
  # Configurazione del modello (esplorazione)
</div>

Questo documento raccoglie **idee** per la futura configurazione del modello. Non è una
specifica pronta per il rilascio. Per il comportamento attuale, vedi:

* [Modelli](/it/concepts/models)
* [Failover del modello](/it/concepts/model-failover)
* [OAuth + profili](/it/concepts/oauth)

<div id="motivation">
  ## Motivazione
</div>

Gli operatori vogliono:

* Profili di autenticazione multipli per ogni provider (personale vs lavoro).
* Selezione semplice del modello tramite `/model` con fallback prevedibili.
* Chiara separazione tra modelli di solo testo e modelli con supporto alle immagini.

<div id="possible-direction-high-level">
  ## Possibile direzione (a livello generale)
</div>

* Mantieni semplice la selezione del modello: `provider/model` con alias opzionali.
* Consenti ai provider di avere più profili di autenticazione, con un ordine di priorità esplicito.
* Usa un elenco globale di fallback in modo che tutte le sessioni eseguano il failover in modo coerente.
* Sovrascrivi l’instradamento delle immagini solo quando è configurato esplicitamente.

<div id="open-questions">
  ## Questioni aperte
</div>

* La rotazione dei profili dovrebbe essere per provider o per modello?
* In che modo la UI dovrebbe presentare la selezione del profilo per una sessione?
* Qual è il percorso di migrazione più sicuro dalle chiavi di configurazione legacy?