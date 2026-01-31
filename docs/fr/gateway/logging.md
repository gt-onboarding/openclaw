---
title: Journalisation
summary: "Surfaces de logs, fichiers journaux, styles de logs WS et formatage de la console"
read_when:
  - Modifier la sortie ou les formats de journalisation
  - Déboguer la sortie du CLI ou du Gateway
---

<div id="logging">
  # Journalisation
</div>

Pour un aperçu orienté utilisateur (CLI + Control UI + config), voir [/logging](/fr/logging).

OpenClaw propose deux « surfaces » de logs :

* **Sortie de la console** (ce que vous voyez dans le terminal / Debug UI).
* **Journaux fichiers** (lignes JSON) écrits par le logger du Gateway.

<div id="file-based-logger">
  ## Journalisation sur fichier
</div>

* Le fichier journal rotatif par défaut se trouve dans `/tmp/openclaw/` (un fichier par jour) : `openclaw-YYYY-MM-DD.log`
  * La date utilise le fuseau horaire local de l’hôte exécutant le Gateway.
* Le chemin et le niveau du fichier journal peuvent être configurés via `~/.openclaw/openclaw.json` :
  * `logging.file`
  * `logging.level`

Le format du fichier est un objet JSON par ligne.

L’onglet Logs de la Control UI suit ce fichier via le Gateway (`logs.tail`).
La CLI peut faire la même chose :

```bash
openclaw logs --follow
```

**Mode verbeux vs. niveaux de log**

* Les **logs de fichier** sont contrôlés exclusivement par `logging.level`.
* `--verbose` n&#39;affecte que la **verbosité de la console** (et le style des logs WS) ; il n&#39;augmente **pas**
  le niveau de log des fichiers.
* Pour capturer dans les logs de fichier les détails visibles uniquement en mode verbeux, définissez `logging.level` sur `debug` ou
  `trace`.

<div id="console-capture">
  ## Capture de la console
</div>

La CLI capture `console.log/info/warn/error/debug/trace` et les écrit dans des fichiers journaux,
tout en les affichant toujours sur stdout/stderr.

Vous pouvez ajuster la verbosité de la console indépendamment via :

* `logging.consoleLevel` (valeur par défaut : `info`)
* `logging.consoleStyle` (`pretty` | `compact` | `json`)

<div id="tool-summary-redaction">
  ## Masquage dans les résumés d’outils
</div>

Les résumés détaillés d’outils (par ex. `🛠️ Exec: ...`) peuvent masquer des jetons sensibles avant qu’ils n’atteignent le flux de la console. Ceci est **limité aux outils** et ne modifie pas les journaux de fichiers.

* `logging.redactSensitive`: `off` | `tools` (par défaut : `tools`)
* `logging.redactPatterns`: tableau de chaînes regex (remplace les valeurs par défaut)
  * Utilisez des chaînes regex brutes (auto `gi`), ou `/pattern/flags` si vous avez besoin d’indicateurs personnalisés.
  * Les occurrences trouvées sont masquées en conservant les 6 premiers + 4 derniers caractères (longueur &gt;= 18), sinon `***`.
  * Les valeurs par défaut couvrent les affectations de clés courantes, les flags de CLI, les champs JSON, les en-têtes bearer, les blocs PEM et les préfixes de jetons courants.

<div id="gateway-websocket-logs">
  ## Journaux WebSocket du Gateway
</div>

Le Gateway journalise les événements du protocole WebSocket selon deux modes :

* **Mode normal (sans `--verbose`)** : seuls les résultats d’RPC « significatifs » sont affichés :
  * erreurs (`ok=false`)
  * appels lents (seuil par défaut : `>= 50ms`)
  * erreurs d’analyse
* **Mode verbeux (`--verbose`)** : affiche l’intégralité du trafic de requêtes/réponses WS.

<div id="ws-log-style">
  ### Style de journalisation WS
</div>

`openclaw gateway` prend en charge un style configurable par Gateway :

* `--ws-log auto` (par défaut) : le mode normal est optimisé ; le mode verbeux utilise une sortie compacte
* `--ws-log compact` : sortie compacte (requête/réponse associées) en mode verbeux
* `--ws-log full` : sortie détaillée par trame en mode verbeux
* `--compact` : alias de `--ws-log compact`

Exemples :

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# afficher tout le trafic WS (métadonnées complètes)
openclaw gateway --verbose --ws-log full
```

<div id="console-formatting-subsystem-logging">
  ## Formatage de la console (journalisation par sous-système)
</div>

Le formateur de console est **sensible au TTY** et affiche des lignes cohérentes, préfixées.
Les journaliseurs de sous-système gardent la sortie groupée et facile à analyser d’un coup d’œil.

Comportement :

* **Préfixes de sous-système** sur chaque ligne (par ex. `[gateway]`, `[canvas]`, `[tailscale]`)
* **Couleurs par sous-système** (stables par sous-système) plus coloration par niveau
* **Couleur lorsque la sortie est un TTY ou que l’environnement ressemble à un terminal riche** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respecte `NO_COLOR`
* **Préfixes de sous-système raccourcis** : supprime les préfixes initiaux `gateway/` et `channels/`, conserve les 2 derniers segments (par ex. `whatsapp/outbound`)
* **Sous-journaliseurs par sous-système** (préfixe automatique + champ structuré `{ subsystem }`)
* **`logRaw()`** pour la sortie QR/UX (pas de préfixe, pas de formatage)
* **Styles de console** (par ex. `pretty | compact | json`)
* **Niveau de journalisation console** distinct du niveau de journalisation fichier (le fichier conserve tous les détails lorsque `logging.level` est réglé sur `debug`/`trace`)
* **Les corps de messages WhatsApp** sont journalisés au niveau `debug` (utilisez `--verbose` pour les voir)

Cela maintient les journaux de fichiers existants stables tout en rendant la sortie interactive facile à parcourir.