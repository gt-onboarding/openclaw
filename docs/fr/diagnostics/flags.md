---
title: Options
summary: "Options de diagnostic pour des journaux de débogage ciblés"
read_when:
  - Vous avez besoin de journaux de débogage ciblés sans augmenter le niveau de journalisation global
  - Vous devez capturer des journaux propres à un sous-système pour le support
---

<div id="diagnostics-flags">
  # Indicateurs de diagnostic
</div>

Les indicateurs de diagnostic vous permettent d’activer des journaux de débogage ciblés sans activer la journalisation verbeuse partout. Les indicateurs sont à activer explicitement (opt‑in) et restent sans effet tant qu’aucun sous-système ne les consulte.

<div id="how-it-works">
  ## Fonctionnement
</div>

* Les indicateurs sont des chaînes de caractères (sans distinction de casse).
* Vous pouvez activer des indicateurs dans la configuration ou via un override d’environnement.
* Les caractères génériques sont pris en charge :
  * `telegram.*` correspond à `telegram.http`
  * `*` active tous les indicateurs

<div id="enable-via-config">
  ## Activer via la config
</div>

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Plusieurs options :

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

Redémarrez Gateway après avoir modifié les flags.

<div id="env-override-one-off">
  ## Surcharge ponctuelle de l&#39;environnement
</div>

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Désactiver tous les indicateurs :

```bash
OPENCLAW_DIAGNOSTICS=0
```

<div id="where-logs-go">
  ## Emplacement des journaux
</div>

Les options écrivent les journaux dans le fichier journal de diagnostic standard. Par défaut :

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Si vous configurez `logging.file`, utilisez ce chemin à la place. Les logs sont au format JSONL (un objet JSON par ligne). La réédaction des données sensibles s’applique toujours, en fonction de `logging.redactSensitive`.

<div id="extract-logs">
  ## Extraire les journaux
</div>

Sélectionnez le fichier journal le plus récent :

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

Filtre des diagnostics HTTP pour Telegram :

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

Ou exécuter `tail` pendant que vous reproduisez le problème :

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Pour les Gateway distants, vous pouvez aussi utiliser `openclaw logs --follow` (voir [/cli/logs](/fr/cli/logs)).

<div id="notes">
  ## Remarques
</div>

* Si `logging.level` est défini à une valeur supérieure à `warn`, ces logs peuvent être masqués. La valeur par défaut `info` convient.
* Vous pouvez laisser les flags activés sans risque : ils n’affectent que le volume de logs pour le sous-système concerné.
* Utilisez [/logging](/fr/logging) pour modifier les destinations et niveaux de logs, ainsi que le masquage (redaction) des données.