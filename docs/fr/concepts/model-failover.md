---
title: Basculement de modèles
summary: "Comment OpenClaw fait alterner les profils d'authentification et assure le basculement entre les modèles"
read_when:
  - Pour diagnostiquer la rotation des profils d'authentification, les périodes de cooldown ou le comportement de basculement entre modèles
  - Pour mettre à jour les règles de basculement pour les profils d'authentification ou les modèles
---

<div id="model-failover">
  # Basculement de modèle
</div>

OpenClaw gère les échecs en deux étapes :

1. **Rotation du profil d’authentification** au sein du fournisseur actuel.
2. **Basculement de modèle** vers le modèle suivant dans `agents.defaults.model.fallbacks`.

Ce document décrit les règles de fonctionnement à l’exécution et les données sur lesquelles elles s’appuient.

<div id="auth-storage-keys-oauth">
  ## Stockage d&#39;authentification (clés + OAuth)
</div>

OpenClaw utilise des **profils d&#39;authentification** à la fois pour les clés d&#39;API et les jetons OAuth.

* Les secrets sont stockés dans `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (ancien emplacement : `~/.openclaw/agent/auth-profiles.json`).
* Les clés de configuration `auth.profiles` / `auth.order` ne contiennent **que des métadonnées et du routage** (aucun secret).
* Fichier OAuth hérité, utilisé uniquement pour l&#39;import : `~/.openclaw/credentials/oauth.json` (importé dans `auth-profiles.json` lors de la première utilisation).

Plus de détails : [/concepts/oauth](/fr/concepts/oauth)

Types d&#39;identifiants :

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` pour certains fournisseurs)

<div id="profile-ids">
  ## Identifiants de profil
</div>

Les connexions OAuth créent des profils distincts afin que plusieurs comptes puissent coexister.

* Par défaut : `provider:default` lorsqu’aucune adresse e-mail n’est disponible.
* OAuth avec adresse e-mail : `provider:&lt;email&gt;` (par exemple `google-antigravity:user@gmail.com`).

Les profils sont stockés dans `~/.openclaw/agents/&lt;agentId&gt;/agent/auth-profiles.json`, sous la clé `profiles`.

<div id="rotation-order">
  ## Ordre de rotation
</div>

Lorsqu’un fournisseur dispose de plusieurs profils, OpenClaw choisit un ordre comme suit :

1. **Configuration explicite** : `auth.order[provider]` (si défini).
2. **Profils configurés** : `auth.profiles` filtrés par fournisseur.
3. **Profils stockés** : entrées dans `auth-profiles.json` pour le fournisseur.

Si aucun ordre explicite n’est configuré, OpenClaw utilise un ordre en round‑robin :

* **Clé primaire :** type de profil (**OAuth avant les clés API**).
* **Clé secondaire :** `usageStats.lastUsed` (du plus ancien au plus récent, pour chaque type).
* Les **profils en cooldown/désactivés** sont déplacés à la fin, ordonnés de la date d’expiration la plus proche à la plus lointaine.

<div id="session-stickiness-cache-friendly">
  ### Persistance de session (optimisée pour le cache)
</div>

OpenClaw **épingle le profil d’auth choisi par session** afin de garder les caches du fournisseur pré‑chauffés.
Il **ne** fait **pas** de rotation à chaque requête. Le profil épinglé est réutilisé jusqu’à :

* réinitialisation de la session (`/new` / `/reset`)
* fin d’une compaction (incrément du compteur de compaction)
* profil en cooldown / désactivé

Une sélection manuelle via `/model …@<profileId>` définit une **surcharge utilisateur** pour cette session
et n’est pas soumise à la rotation automatique tant qu’une nouvelle session n’est pas démarrée.

Les profils auto‑épinglés (sélectionnés par le routeur de session) sont traités comme une **préférence** :
ils sont essayés en premier, mais OpenClaw peut passer à un autre profil en cas de rate limits/timeouts.
Les profils épinglés par l’utilisateur restent verrouillés sur ce profil ; s’il échoue et que des solutions de repli de modèle
sont configurées, OpenClaw passe au modèle suivant au lieu de changer de profil.

<div id="why-oauth-can-look-lost">
  ### Pourquoi OAuth peut « avoir l’air perdu »
</div>

Si vous avez à la fois un profil OAuth et un profil de clé API pour le même fournisseur, le round‑robin peut basculer entre eux d’un message à l’autre, sauf s’ils sont fixés. Pour forcer l’utilisation d’un seul profil :

* Fixez‑le avec `auth.order[provider] = ["provider:profileId"]`, ou
* Utilisez une surcharge par session via `/model …` avec une surcharge de profil (lorsque c’est pris en charge par votre interface de chat/UI).

<div id="cooldowns">
  ## Délais de refroidissement
</div>

Lorsqu’un profil échoue en raison d’erreurs d’authentification ou de limitation de débit (ou d’un délai d’expiration assimilé à une limitation de débit), OpenClaw le place en délai de refroidissement et passe au profil suivant.
Les erreurs de format ou de requête invalide (par exemple les échecs de validation d’ID d’appel d’outil Cloud Code Assist) sont considérées comme devant déclencher un basculement et utilisent les mêmes délais de refroidissement.

Les délais de refroidissement suivent un backoff exponentiel :

* 1 minute
* 5 minutes
* 25 minutes
* 1 heure (plafond)

L’état est stocké dans `auth-profiles.json` sous `usageStats` :

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## Désactivation liée à la facturation
</div>

Les échecs de facturation/de crédits (par exemple « insufficient credits » / « credit balance too low ») sont considérés comme des motifs de basculement, mais ils ne sont généralement pas transitoires. Plutôt qu’un court délai d’attente, OpenClaw marque le profil comme **désactivé** (avec un délai de reprise plus long) et bascule vers le profil/fournisseur suivant.

L’état est stocké dans `auth-profiles.json` :

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Par défaut :

* Le backoff de facturation commence à **5 heures**, double à chaque échec de facturation et est plafonné à **24 heures**.
* Les compteurs de backoff sont réinitialisés si le profil ne connaît aucun échec pendant **24 heures** (configurable).

<div id="model-fallback">
  ## Repli de modèle
</div>

Si tous les profils d’un fournisseur échouent, OpenClaw passe au modèle suivant dans
`agents.defaults.model.fallbacks`. Cela s’applique aux échecs d’authentification, aux limites de débit et
aux dépassements de délai qui ont épuisé la rotation des profils (les autres erreurs n’entraînent pas de progression dans la chaîne de repli).

Lorsqu’une exécution démarre avec un remplacement de modèle (hooks ou CLI), les replis aboutissent tout de même à
`agents.defaults.model.primary` après avoir essayé tous les modèles de repli configurés.

<div id="related-config">
  ## Configuration associée
</div>

Voir la [configuration du Gateway](/fr/gateway/configuration) pour :

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* le routage de `agents.defaults.imageModel`

Voir [Modèles](/fr/concepts/models) pour une vue d’ensemble plus complète de la sélection des modèles et des mécanismes de repli.