---
title: Journalisation
summary: "Vue d’ensemble de la journalisation : journaux de fichiers, sortie sur la console, suivi en temps réel dans la CLI et le Control UI"
read_when:
  - Vous avez besoin d’une vue d’ensemble de la journalisation accessible aux débutants
  - Vous souhaitez configurer les niveaux ou les formats de journaux
  - Vous effectuez un dépannage et devez trouver rapidement les journaux
---

<div id="logging">
  # Journalisation
</div>

OpenClaw génère des journaux à deux emplacements :

* **Journaux de fichiers** (lignes JSON) écrits par le Gateway.
* **Sortie de console** affichée dans les terminaux et dans le Control UI.

Cette page explique où se trouvent les journaux, comment les lire et comment
configurer les niveaux et formats de journalisation.

<div id="where-logs-live">
  ## Où se trouvent les journaux
</div>

Par défaut, le Gateway écrit un fichier journal rotatif à l’emplacement suivant :

`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

La date utilise le fuseau horaire local de l’hôte du Gateway.

Vous pouvez redéfinir ce paramètre dans `~/.openclaw/openclaw.json` :

```json
{
  "logging": {
    "file": "/path/to/openclaw.log"
  }
}
```

<div id="how-to-read-logs">
  ## Comment lire les logs
</div>

<div id="cli-live-tail-recommended">
  ### CLI : lecture en continu (recommandé)
</div>

Utilisez la CLI pour consulter en continu le journal du Gateway via RPC :

```bash
openclaw logs --follow
```

Modes de sortie :

* **sessions TTY** : lignes de log lisibles, colorées et structurées.
* **sessions non TTY** : texte brut.
* `--json` : JSON délimité par ligne (un événement de log par ligne).
* `--plain` : force le texte brut dans les sessions TTY.
* `--no-color` : désactive les couleurs ANSI.

En mode JSON, la CLI émet des objets annotés avec un champ `type` :

* `meta` : métadonnées du flux (fichier, curseur, taille)
* `log` : entrée de log analysée
* `notice` : indications de troncation / rotation
* `raw` : ligne de log non analysée

Si le Gateway est inaccessible, la CLI affiche un court message suggérant d’exécuter :

```bash
openclaw doctor
```

<div id="control-ui-web">
  ### Control UI (web)
</div>

L’onglet **Logs** de la Control UI affiche en temps réel le même fichier via `logs.tail`.
Consultez [/web/control-ui](/fr/web/control-ui) pour savoir comment y accéder.

<div id="channel-only-logs">
  ### Journaux par canal uniquement
</div>

Pour filtrer l’activité d’un canal (WhatsApp/Telegram/etc.), utilisez :

```bash
openclaw channels logs --channel whatsapp
```

<div id="log-formats">
  ## Formats de journaux
</div>

<div id="file-logs-jsonl">
  ### Journaux de fichier (JSONL)
</div>

Chaque ligne du fichier journal est un objet JSON. La CLI et le Control UI
analysent ces entrées pour afficher des informations structurées (heure, niveau, sous-système, message).

<div id="console-output">
  ### Sortie de la console
</div>

Les journaux de console sont **adaptés aux TTY** et formatés pour rester lisibles :

* Préfixes de sous-système (par ex. `gateway/channels/whatsapp`)
* Coloration selon le niveau (info/warn/error)
* Mode compact ou JSON en option

Le formatage de la console est contrôlé par `logging.consoleStyle`.

<div id="configuring-logging">
  ## Configuration des logs
</div>

Toute la configuration des logs est définie dans la section `logging` de `~/.openclaw/openclaw.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/openclaw/openclaw-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

<div id="log-levels">
  ### Niveaux de logs
</div>

* `logging.level` : niveau de logs des **fichiers** (JSONL).
* `logging.consoleLevel` : niveau de verbosité de la **console**.

`--verbose` n’affecte que la sortie de la console ; il ne modifie pas les niveaux de logs des fichiers.

<div id="console-styles">
  ### Styles de console
</div>

`logging.consoleStyle` :

* `pretty` : lisible, en couleur, avec horodatage.
* `compact` : sortie plus compacte (idéal pour les longues sessions).
* `json` : un JSON par ligne (pour les outils de traitement des journaux).

<div id="redaction">
  ### Masquage
</div>

Les résumés d’outils peuvent masquer des jetons sensibles avant qu’ils n’apparaissent dans la console :

* `logging.redactSensitive` : `off` | `tools` (par défaut : `tools`)
* `logging.redactPatterns` : liste de chaînes d’expressions régulières (regex) pour remplacer l’ensemble par défaut

Le masquage affecte **uniquement la sortie de la console** et ne modifie pas les fichiers journaux.

<div id="diagnostics-opentelemetry">
  ## Diagnostics + OpenTelemetry
</div>

Les diagnostics sont des événements structurés et interprétables par les machines pour les exécutions de modèles **et**
la télémétrie des flux de messages (webhooks, mise en file d&#39;attente, état de session). Ils ne **remplacent pas**
les journaux ; ils existent pour alimenter les métriques, les traces et d&#39;autres exporteurs.

Les événements de diagnostic sont émis au sein du processus, mais les exportateurs ne sont associés que lorsque
les diagnostics + le plugin d&#39;exportation sont activés.

<div id="opentelemetry-vs-otlp">
  ### OpenTelemetry vs OTLP
</div>

* **OpenTelemetry (OTel)** : le modèle de données + les SDK pour les traces, métriques et journaux.
* **OTLP** : le protocole de transport utilisé pour exporter les données OTel vers un collecteur ou un backend.
* OpenClaw exporte aujourd’hui via **OTLP/HTTP (protobuf)**.

<div id="signals-exported">
  ### Signaux exportés
</div>

* **Métriques** : compteurs + histogrammes (utilisation de jetons, flux de messages, mise en file d&#39;attente).
* **Traces** : spans pour l&#39;utilisation des modèles + le traitement des webhooks/messages.
* **Logs** : exportés via OTLP lorsque `diagnostics.otel.logs` est activé. Le volume
  de logs peut être élevé ; gardez `logging.level` et les filtres de l’exporteur à l’esprit.

<div id="diagnostic-event-catalog">
  ### Catalogue d&#39;événements de diagnostic
</div>

Utilisation du modèle :

* `model.usage` : jetons, coût, durée, contexte, fournisseur/modèle/canal, identifiants de session.

Flux de messages :

* `webhook.received` : réception de webhook par canal.
* `webhook.processed` : webhook traité + durée.
* `webhook.error` : erreurs du gestionnaire de webhook.
* `message.queued` : message mis en file d’attente pour traitement.
* `message.processed` : résultat + durée + erreur facultative.

File d’attente + session :

* `queue.lane.enqueue` : mise en file d’attente d’une commande dans une voie + profondeur.
* `queue.lane.dequeue` : retrait de la file d’attente dans une voie de commandes + temps d’attente.
* `session.state` : transition d’état de la session + raison.
* `session.stuck` : avertissement de session bloquée + âge de la session.
* `run.attempt` : métadonnées de tentative d’exécution/de réessai.
* `diagnostic.heartbeat` : compteurs agrégés (webhooks/file d’attente/session).

<div id="enable-diagnostics-no-exporter">
  ### Activer les diagnostics (sans exporteur)
</div>

Utilisez cette option si vous voulez que les événements de diagnostic soient disponibles pour les plugins ou des collecteurs personnalisés :

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

<div id="diagnostics-flags-targeted-logs">
  ### Indicateurs de diagnostic (journaux ciblés)
</div>

Utilisez des indicateurs pour activer des journaux de débogage supplémentaires et ciblés sans augmenter `logging.level`.
Les indicateurs ne sont pas sensibles à la casse et prennent en charge les caractères génériques (par exemple `telegram.*` ou `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Remplacement ponctuel par variable d’environnement :

```
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

Remarques :

* Les logs de flags sont envoyés vers le fichier journal standard (même que `logging.file`).
* La sortie continue d’être masquée conformément à `logging.redactSensitive`.
* Guide complet : [/diagnostics/flags](/fr/diagnostics/flags).

<div id="export-to-opentelemetry">
  ### Export vers OpenTelemetry
</div>

Les diagnostics peuvent être exportés via le plugin `diagnostics-otel` (OTLP/HTTP). Cela fonctionne avec n&#39;importe quel collecteur ou backend OpenTelemetry acceptant OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "openclaw-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Notes :

* Vous pouvez également activer le plugin avec `openclaw plugins enable diagnostics-otel`.
* `protocol` ne prend actuellement en charge que `http/protobuf`. `grpc` est ignoré.
* Les métriques incluent l’utilisation de jetons, le coût, la taille du contexte, la durée d’exécution et des compteurs/histogrammes de flux de messages (webhooks, mise en file d’attente, état de session, profondeur de file d’attente/temps d’attente).
* Les traces/métriques peuvent être activées ou désactivées avec `traces` / `metrics` (par défaut : activé). Les traces incluent des spans d’utilisation du modèle ainsi que des spans de traitement des webhooks/messages lorsqu’elles sont activées.
* Définissez `headers` lorsque votre collecteur nécessite une authentification.
* Variables d’environnement prises en charge : `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

<div id="exported-metrics-names-types">
  ### Métriques exportées (noms + types)
</div>

Utilisation du modèle :

* `openclaw.tokens` (compteur, attributs : `openclaw.token`, `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.cost.usd` (compteur, attributs : `openclaw.channel`, `openclaw.provider`,
  `openclaw.model`)
* `openclaw.run.duration_ms` (histogramme, attributs : `openclaw.channel`,
  `openclaw.provider`, `openclaw.model`)
* `openclaw.context.tokens` (histogramme, attributs : `openclaw.context`,
  `openclaw.channel`, `openclaw.provider`, `openclaw.model`)

Flux de messages :

* `openclaw.webhook.received` (compteur, attributs : `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.error` (compteur, attributs : `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.webhook.duration_ms` (histogramme, attributs : `openclaw.channel`,
  `openclaw.webhook`)
* `openclaw.message.queued` (compteur, attributs : `openclaw.channel`,
  `openclaw.source`)
* `openclaw.message.processed` (compteur, attributs : `openclaw.channel`,
  `openclaw.outcome`)
* `openclaw.message.duration_ms` (histogramme, attributs : `openclaw.channel`,
  `openclaw.outcome`)

Files d&#39;attente + sessions :

* `openclaw.queue.lane.enqueue` (compteur, attributs : `openclaw.lane`)
* `openclaw.queue.lane.dequeue` (compteur, attributs : `openclaw.lane`)
* `openclaw.queue.depth` (histogramme, attributs : `openclaw.lane` ou
  `openclaw.channel=heartbeat`)
* `openclaw.queue.wait_ms` (histogramme, attributs : `openclaw.lane`)
* `openclaw.session.state` (compteur, attributs : `openclaw.state`, `openclaw.reason`)
* `openclaw.session.stuck` (compteur, attributs : `openclaw.state`)
* `openclaw.session.stuck_age_ms` (histogramme, attributs : `openclaw.state`)
* `openclaw.run.attempt` (compteur, attributs : `openclaw.attempt`)

<div id="exported-spans-names-key-attributes">
  ### Spans exportés (noms et attributs clés)
</div>

* `openclaw.model.usage`
  * `openclaw.channel`, `openclaw.provider`, `openclaw.model`
  * `openclaw.sessionKey`, `openclaw.sessionId`
  * `openclaw.tokens.*` (input/output/cache&#95;read/cache&#95;write/total)
* `openclaw.webhook.processed`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`
* `openclaw.webhook.error`
  * `openclaw.channel`, `openclaw.webhook`, `openclaw.chatId`,
    `openclaw.error`
* `openclaw.message.processed`
  * `openclaw.channel`, `openclaw.outcome`, `openclaw.chatId`,
    `openclaw.messageId`, `openclaw.sessionKey`, `openclaw.sessionId`,
    `openclaw.reason`
* `openclaw.session.stuck`
  * `openclaw.state`, `openclaw.ageMs`, `openclaw.queueDepth`,
    `openclaw.sessionKey`, `openclaw.sessionId`

<div id="sampling-flushing">
  ### Échantillonnage + vidage
</div>

* Échantillonnage des traces : `diagnostics.otel.sampleRate` (0,0–1,0, traces racines uniquement).
* Intervalle d’exportation des métriques : `diagnostics.otel.flushIntervalMs` (min. 1000 ms).

<div id="protocol-notes">
  ### Notes sur le protocole
</div>

* Les endpoints OTLP/HTTP peuvent être configurés via `diagnostics.otel.endpoint` ou
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
* Si l&#39;endpoint contient déjà `/v1/traces` ou `/v1/metrics`, il est utilisé tel quel.
* Si l&#39;endpoint contient déjà `/v1/logs`, il est utilisé tel quel pour les logs.
* `diagnostics.otel.logs` active l&#39;export de logs OTLP pour la sortie principale du logger.

<div id="log-export-behavior">
  ### Comportement d’exportation des journaux
</div>

* Les journaux OTLP utilisent les mêmes enregistrements structurés que ceux écrits dans `logging.file`.
* Le paramètre `logging.level` (niveau de journalisation du fichier) est respecté. Le masquage dans la console ne s’applique **pas** aux journaux OTLP.
* Les installations à fort volume devraient privilégier l’échantillonnage et le filtrage au niveau du collecteur OTLP.

<div id="troubleshooting-tips">
  ## Conseils de dépannage
</div>

* **Gateway inaccessible ?** Exécute d’abord `openclaw doctor`.
* **Journaux vides ?** Vérifie que le Gateway est en cours d’exécution et qu’il écrit bien dans le chemin de fichier indiqué dans `logging.file`.
* **Besoin de plus de détails ?** Définis `logging.level` sur `debug` ou `trace`, puis réessaie.