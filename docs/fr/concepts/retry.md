---
title: Réessai
summary: "Politique de réessai pour les appels sortants aux fournisseurs"
read_when:
  - Mise à jour du comportement ou des paramètres par défaut de réessai des fournisseurs
  - Débogage des erreurs d’envoi vers les fournisseurs ou des limitations de débit
---

<div id="retry-policy">
  # Politique de réessai
</div>

<div id="goals">
  ## Objectifs
</div>

- Réessayer par requête HTTP, et non par flux multi-étapes.
- Préserver l'ordre d'exécution en ne réessayant que l'étape en cours.
- Éviter de dupliquer les opérations non idempotentes.

<div id="defaults">
  ## Valeurs par défaut
</div>

- Nombre de tentatives : 3
- Plafond de délai maximal : 30000 ms
- Jitter : 0,1 (10 %)
- Valeurs par défaut du fournisseur :
  - Délai minimal Telegram : 400 ms
  - Délai minimal Discord : 500 ms

<div id="behavior">
  ## Fonctionnement
</div>

<div id="discord">
  ### Discord
</div>

- Ne réessaie qu’en cas d’erreur de limitation de débit (HTTP 429).
- Utilise le champ `retry_after` de Discord lorsqu’il est disponible, à défaut applique un backoff exponentiel.

<div id="telegram">
  ### Telegram
</div>

- Réessaie en cas d’erreurs temporaires (429, délai d’attente dépassé, connexion réinitialisée/fermée, indisponibilité temporaire).
- Utilise `retry_after` lorsqu’il est disponible, sinon un backoff exponentiel.
- Les erreurs d’analyse Markdown ne déclenchent pas de nouvelle tentative ; on revient alors au texte brut.

<div id="configuration">
  ## Configuration
</div>

Configurez la politique de réessai pour chaque fournisseur dans `~/.openclaw/openclaw.json` :

```json5
{
  channels: {
    telegram: {
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    },
    discord: {
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```


<div id="notes">
  ## Remarques
</div>

- Les réessais s’appliquent par requête (envoi de message, mise en ligne de média, réaction, sondage, sticker).
- Les flux composites ne relancent pas les étapes déjà terminées.