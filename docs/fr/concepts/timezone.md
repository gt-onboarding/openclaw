---
title: Fuseau horaire
summary: "Gestion des fuseaux horaires pour les agents, les enveloppes et les prompts"
read_when:
  - Vous devez comprendre comment les horodatages sont normalisés par le modèle
  - Configurer le fuseau horaire de l'utilisateur dans les prompts système
---

<div id="timezones">
  # Fuseaux horaires
</div>

OpenClaw normalise les horodatages afin que le modèle voie une **seule heure de référence**.

<div id="message-envelopes-local-by-default">
  ## Enveloppes de messages (locales par défaut)
</div>

Les messages entrants sont enveloppés dans une enveloppe de ce type :

```
[Provider ... 2026-01-05 16:26 PST] message text
```

L&#39;horodatage dans l&#39;enveloppe est **local à l&#39;hôte par défaut**, avec une précision à la minute.

Vous pouvez remplacer ce comportement par :

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

* `envelopeTimezone: "utc"` utilise l’UTC.
* `envelopeTimezone: "user"` utilise `agents.defaults.userTimezone` (et se rabat sinon sur le fuseau horaire de l’hôte).
* Utilisez un fuseau horaire IANA explicite (par exemple, `"Europe/Vienna"`) pour un décalage fixe.
* `envelopeTimestamp: "off"` supprime les horodatages absolus des en-têtes d’enveloppe.
* `envelopeElapsed: "off"` supprime les suffixes indiquant le temps écoulé (du type `+2m`).

<div id="examples">
  ### Exemples
</div>

**Local (par défaut) :**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**Fuseau horaire fixe :**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**Durée écoulée :**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

<div id="tool-payloads-raw-provider-data-normalized-fields">
  ## Charges utiles d’outils (données brutes du fournisseur + champs normalisés)
</div>

Les appels d’outils (`channels.discord.readMessages`, `channels.slack.readMessages`, etc.) renvoient des **horodatages bruts du fournisseur**.
Nous ajoutons également des champs normalisés pour la cohérence :

* `timestampMs` (millisecondes depuis l’époque Unix, en UTC)
* `timestampUtc` (chaîne UTC au format ISO 8601)

Les champs bruts du fournisseur sont conservés.

<div id="user-timezone-for-the-system-prompt">
  ## Fuseau horaire utilisateur pour le prompt système
</div>

Définissez `agents.defaults.userTimezone` pour indiquer au modèle le fuseau horaire local de l’utilisateur. S’il n’est pas défini, OpenClaw détermine **le fuseau horaire de l’hôte au moment de l’exécution** (sans modifier la configuration).

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

L’invite système inclut :

* une section `Current Date & Time` avec l’heure locale et le fuseau horaire
* `Time format: 12-hour` ou `24-hour`

Vous pouvez contrôler le format de l’invite avec `agents.defaults.timeFormat` (`auto` | `12` | `24`).

Voir [Date &amp; Time](/fr/date-time) pour une description complète du comportement et des exemples.
