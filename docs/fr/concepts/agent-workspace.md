---
title: Espace de travail de l’Agent
summary: "Espace de travail de l’agent : emplacement, organisation et stratégie de sauvegarde"
read_when:
  - Vous devez expliquer l’espace de travail de l’agent ou l’organisation de ses fichiers
  - Vous souhaitez sauvegarder ou migrer l’espace de travail d’un agent
---

<div id="agent-workspace">
  # Espace de travail de l&#39;agent
</div>

L&#39;espace de travail est l&#39;environnement principal de l&#39;agent. C&#39;est le seul répertoire de travail utilisé pour
les outils de fichiers et pour le contexte associé à l&#39;espace de travail. Gardez-le privé et traitez-le comme de la mémoire.

Il est distinct de `~/.openclaw/`, qui stocke la configuration, les identifiants d&#39;accès et
les sessions.

**Important :** l&#39;espace de travail est le **cwd par défaut**, et non un sandbox strictement isolé. Les outils
résolvent les chemins relatifs par rapport à l&#39;espace de travail, mais les chemins absolus peuvent toujours atteindre
d&#39;autres emplacements sur l&#39;hôte tant que le sandboxing n&#39;est pas activé. Si vous avez besoin d&#39;isolation, utilisez
[`agents.defaults.sandbox`](/fr/gateway/sandboxing) (et/ou une configuration de sandbox par agent).
Lorsque le sandboxing est activé et que `workspaceAccess` n&#39;est pas `"rw"`, les outils fonctionnent
dans un espace de travail de sandbox sous `~/.openclaw/sandboxes`, et non dans votre espace de travail sur l&#39;hôte.

<div id="default-location">
  ## Emplacement par défaut
</div>

* Par défaut : `~/.openclaw/workspace`
* Si `OPENCLAW_PROFILE` est défini et différent de `"default"`, l’emplacement par défaut devient
  `~/.openclaw/workspace-<profile>`.
* Peut être redéfini dans `~/.openclaw/openclaw.json` :

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace"
  }
}
```

`openclaw onboard`, `openclaw configure` ou `openclaw setup` créeront l’espace de travail et généreront les fichiers bootstrap s’ils n’existent pas.

Si vous gérez déjà vous-même les fichiers de l’espace de travail, vous pouvez désactiver la création automatique des fichiers bootstrap :

```json5
{ agent: { skipBootstrap: true } }
```

<div id="extra-workspace-folders">
  ## Dossiers d&#39;espace de travail supplémentaires
</div>

D&#39;anciennes installations ont pu créer `~/openclaw`. Conserver plusieurs
répertoires d&#39;espace de travail peut provoquer des incohérences déroutantes au niveau de l&#39;authentification ou de l&#39;état, car un seul espace de travail est actif à la fois.

**Recommandation :** conservez un seul espace de travail actif. Si vous n&#39;utilisez
plus les dossiers supplémentaires, archivez-les ou déplacez-les vers la corbeille (par exemple `trash ~/openclaw`).
Si vous conservez délibérément plusieurs espaces de travail, assurez-vous
que `agents.defaults.workspace` pointe vers l&#39;espace actif.

`openclaw doctor` émet un avertissement lorsqu&#39;il détecte des répertoires d&#39;espace de travail supplémentaires.

<div id="workspace-file-map-what-each-file-means">
  ## Carte des fichiers de l’espace de travail (ce que signifie chaque fichier)
</div>

Voici les fichiers standard qu’OpenClaw s’attend à trouver dans l’espace de travail :

* `AGENTS.md`
  * Instructions de fonctionnement pour l’agent et la façon dont il doit utiliser la mémoire.
  * Chargé au début de chaque session.
  * Bon endroit pour les règles, priorités et détails sur « comment se comporter ».

* `SOUL.md`
  * Persona, ton et limites.
  * Chargé à chaque session.

* `USER.md`
  * Qui est l’utilisateur et comment s’adresser à lui.
  * Chargé à chaque session.

* `IDENTITY.md`
  * Le nom, l’ambiance et les emoji de l’agent.
  * Créé/mis à jour pendant le rituel de bootstrap.

* `TOOLS.md`
  * Notes sur vos outils locaux et conventions.
  * Ne contrôle pas la disponibilité des outils ; c’est uniquement une aide.

* `HEARTBEAT.md`
  * Liste de contrôle minimale facultative pour les exécutions de signal de vie.
  * Gardez-la courte pour éviter de consommer trop de jetons.

* `BOOT.md`
  * Liste de contrôle de démarrage facultative exécutée au redémarrage du Gateway lorsque les hooks internes sont activés.
  * Gardez-la courte ; utilisez l’outil de message pour les envois sortants.

* `BOOTSTRAP.md`
  * Rituel de première exécution unique.
  * Créé uniquement pour un espace de travail tout neuf.
  * Supprimez-le une fois le rituel terminé.

* `memory/YYYY-MM-DD.md`
  * Journal de mémoire quotidien (un fichier par jour).
  * Il est recommandé de lire aujourd’hui + hier au démarrage de la session.

* `MEMORY.md` (optionnel)
  * Mémoire à long terme organisée.
  * À charger uniquement dans la session principale et privée (pas dans les contextes partagés/de groupe).

Voir [Memory](/fr/concepts/memory) pour le workflow et la vidange automatique de la mémoire.

* `skills/` (optionnel)
  * Compétences spécifiques à l’espace de travail.
  * Remplace les compétences gérées/intégrées lorsque les noms entrent en collision.

* `canvas/` (optionnel)
  * Fichiers de l’UI Canvas pour les affichages de nœuds (par exemple `canvas/index.html`).

Si un fichier de bootstrap manque, OpenClaw injecte un marqueur de « fichier manquant » dans
la session et continue. Les gros fichiers de bootstrap sont tronqués lors de l’injection ;
ajustez la limite avec `agents.defaults.bootstrapMaxChars` (valeur par défaut : 20000).
`openclaw setup` peut recréer les valeurs par défaut manquantes sans écraser les fichiers
existants.

<div id="what-is-not-in-the-workspace">
  ## Ce qui N’EST PAS dans l’espace de travail
</div>

Ces éléments se trouvent sous `~/.openclaw/` et ne DOIVENT PAS être ajoutés au dépôt Git de l’espace de travail :

* `~/.openclaw/openclaw.json` (configuration)
* `~/.openclaw/credentials/` (jetons OAuth, clés API)
* `~/.openclaw/agents/<agentId>/sessions/` (transcriptions de sessions + métadonnées)
* `~/.openclaw/skills/` (compétences gérées)

Si vous devez migrer des sessions ou la configuration, copiez-les séparément et gardez-les en dehors du système de gestion de versions.

<div id="git-backup-recommended-private">
  ## Sauvegarde Git (recommandée, privée)
</div>

Considérer l’espace de travail comme une mémoire privée. Le placer dans un dépôt Git **privé** afin qu’il soit sauvegardé et récupérable.

Exécuter ces étapes sur la machine où le Gateway est exécuté (c’est là que se trouve l’espace de travail).

<div id="1-initialize-the-repo">
  ### 1) Initialiser le dépôt
</div>

Si Git est installé, tout nouvel espace de travail est initialisé automatiquement. Si cet
espace de travail n’est pas déjà un dépôt, exécutez la commande suivante :

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/
git commit -m "Ajout de l'espace de travail de l'agent"
```

<div id="2-add-a-private-remote-beginner-friendly-options">
  ### 2) Ajouter un dépôt distant privé (options adaptées aux débutants)
</div>

Option A : UI web GitHub

1. Crée un nouveau dépôt **privé** sur GitHub.
2. Ne l’initialise pas avec un README (évite les conflits de fusion).
3. Copie l’URL HTTPS du dépôt distant.
4. Ajoute le dépôt distant et pousse :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

Option B : GitHub CLI (`gh`)

```bash
gh auth login
gh repo create openclaw-workspace --private --source . --remote origin --push
```

Option C : UI web de GitLab

1. Crée un nouveau dépôt **privé** sur GitLab.
2. Ne l’initialise pas avec un README (pour éviter les conflits de fusion).
3. Copie l’URL distante HTTPS.
4. Ajoute le remote et effectue un push :

```bash
git branch -M main
git remote add origin <https-url>
git push -u origin main
```

<div id="3-ongoing-updates">
  ### 3) Mises à jour régulières
</div>

```bash
git status
git add .
git commit -m "Mise à jour de la mémoire"
git push
```

<div id="do-not-commit-secrets">
  ## Ne validez pas de secrets
</div>

Même dans un dépôt privé, évitez de stocker des secrets dans l&#39;espace de travail :

* Clés API, jetons OAuth, mots de passe ou identifiants privés.
* Tout ce qui se trouve sous `~/.openclaw/`.
* Dumps bruts de conversations ou pièces jointes sensibles.

Si vous devez stocker des références sensibles, utilisez des espaces réservés (« placeholders ») et conservez le vrai
secret ailleurs (gestionnaire de mots de passe, variables d&#39;environnement ou `~/.openclaw/`).

Exemple de base de `.gitignore` :

```gitignore
.DS_Store
.env
**/*.key
**/*.pem
**/secrets*
```

<div id="moving-the-workspace-to-a-new-machine">
  ## Déplacement de l’espace de travail vers une nouvelle machine
</div>

1. Clonez le dépôt dans le chemin souhaité (par défaut `~/.openclaw/workspace`).
2. Définissez `agents.defaults.workspace` sur ce chemin dans `~/.openclaw/openclaw.json`.
3. Exécutez `openclaw setup --workspace <path>` pour créer les fichiers manquants si nécessaire.
4. Si vous avez besoin des sessions, copiez `~/.openclaw/agents/<agentId>/sessions/` depuis
   l’ancienne machine à part.

<div id="advanced-notes">
  ## Notes avancées
</div>

* Le routage multi-agent peut utiliser des espaces de travail différents par agent. Voir
  [Routage des canaux](/fr/concepts/channel-routing) pour la configuration du routage.
* Si `agents.defaults.sandbox` est activé, les sessions non principales peuvent utiliser des espaces de travail de sandbox par session
  dans `agents.defaults.sandbox.workspaceRoot`.