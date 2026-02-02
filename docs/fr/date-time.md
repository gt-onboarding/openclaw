---
title: Date et heure
summary: "Gestion de la date et de l’heure dans les enveloppes, prompts, outils et connecteurs"
read_when:
  - Vous modifiez la façon dont les horodatages sont affichés au modèle ou aux utilisateurs
  - Vous déboguez le formatage de l’heure dans les messages ou la sortie du prompt système
---

<div id="date-time">
  # Date et heure
</div>

Par défaut, OpenClaw utilise **l&#39;heure locale de l&#39;hôte pour les horodatages de transport** et **le fuseau horaire de l&#39;utilisateur uniquement dans le prompt système**.
Les horodatages des fournisseurs sont conservés afin que les outils conservent leur sémantique native (l&#39;heure actuelle est disponible via `session_status`).

<div id="message-envelopes-local-by-default">
  ## Enveloppes de message (heure locale par défaut)
</div>

Les messages entrants sont encapsulés avec un horodatage (précision à la minute) :

```
[Provider ... 2026-01-05 16:26 PST] message text
```

L’horodatage de cette enveloppe est, par défaut, **local à l’hôte**, quel que soit le fuseau horaire du fournisseur.

Vous pouvez modifier ce comportement :

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | fuseau horaire IANA
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

* `envelopeTimezone: "utc"` utilise UTC.
* `envelopeTimezone: "local"` utilise le fuseau horaire de la machine hôte.
* `envelopeTimezone: "user"` utilise `agents.defaults.userTimezone` (avec repli sur le fuseau horaire de la machine hôte).
* Utilisez un fuseau horaire IANA explicite (par exemple `"America/Chicago"`) pour une zone fixe.
* `envelopeTimestamp: "off"` supprime les horodatages absolus des en-têtes d’enveloppe.
* `envelopeElapsed: "off"` supprime les suffixes indiquant le temps écoulé (du style `+2m`).

<div id="examples">
  ### Exemples
</div>

**Local (par défaut) :**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**Fuseau horaire de l’utilisateur :**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Durée écoulée activée :**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] suivi
```

<div id="system-prompt-current-date-time">
  ## Prompt système : date et heure actuelles
</div>

Si le fuseau horaire de l&#39;utilisateur est connu, le prompt système inclut une section dédiée
**Date et heure actuelles** contenant **uniquement le fuseau horaire** (sans heure ni format horaire)
afin de garder la mise en cache des prompts stable :

```
Time zone: America/Chicago
```

Lorsque l&#39;agent a besoin de l&#39;heure actuelle, utilisez l&#39;outil `session_status` ; la carte d&#39;état contient une ligne d&#39;horodatage.

<div id="system-event-lines-local-by-default">
  ## Lignes d’événements système (locales par défaut)
</div>

Les événements système mis en file d’attente et insérés dans le contexte de l’agent sont préfixés par un horodatage utilisant les mêmes paramètres de fuseau horaire que les enveloppes de message (par défaut : fuseau horaire de l’hôte).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

<div id="configure-user-timezone-format">
  ### Configurer le fuseau horaire et le format horaire de l’utilisateur
</div>

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // auto | 12 | 24
    }
  }
}
```

* `userTimezone` définit le **fuseau horaire local de l’utilisateur** pour le contexte du prompt.
* `timeFormat` contrôle l’**affichage 12h/24h** dans le prompt. `auto` suit les préférences du système.

<div id="time-format-detection-auto">
  ## Détection automatique du format horaire
</div>

Lorsque `timeFormat: "auto"`, OpenClaw inspecte la préférence du système d’exploitation (macOS/Windows)
et revient au formatage local si nécessaire. La valeur détectée est **mise en cache pour chaque processus**
afin d’éviter des appels système répétés.

<div id="tool-payloads-connectors-raw-provider-time-normalized-fields">
  ## Charges utiles des outils + connecteurs (horodatage brut du fournisseur + champs normalisés)
</div>

Les outils de canal renvoient des **horodatages natifs au fournisseur** et ajoutent des champs normalisés pour assurer la cohérence :

* `timestampMs` : millisecondes depuis l&#39;époque Unix (UTC)
* `timestampUtc` : chaîne UTC ISO 8601

Les champs bruts du fournisseur sont préservés afin qu&#39;aucune information ne soit perdue.

* Slack : chaînes de type epoch renvoyées par l&#39;API
* Discord : horodatages ISO UTC
* Telegram/WhatsApp : horodatages numériques/ISO spécifiques au fournisseur

Si vous avez besoin de l&#39;heure locale, convertissez-la en aval en utilisant le fuseau horaire connu.

<div id="related-docs">
  ## Documentation associée
</div>

* [Prompt système](/fr/concepts/system-prompt)
* [Fuseaux horaires](/fr/concepts/timezone)
* [Messages](/fr/concepts/messages)