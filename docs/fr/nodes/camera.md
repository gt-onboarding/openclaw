---
title: Caméra
summary: "Capture via la caméra (nœud iOS + app macOS) à usage de l’agent : photos (jpg) et courts clips vidéo (mp4)"
read_when:
  - Ajout ou modification de la capture caméra sur des nœuds iOS ou sur macOS
  - Extension des workflows de fichiers temporaires MEDIA accessibles par l’agent
---

<div id="camera-capture-agent">
  # Capture de caméra (agent)
</div>

OpenClaw prend en charge la **capture via la caméra** pour les workflows d’agent :

- **Nœud iOS** (appairé via le Gateway) : permet de capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio facultatif) via `node.invoke`.
- **Nœud Android** (appairé via le Gateway) : permet de capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio facultatif) via `node.invoke`.
- **Application macOS** (nœud via le Gateway) : permet de capturer une **photo** (`jpg`) ou un **court clip vidéo** (`mp4`, avec audio facultatif) via `node.invoke`.

Tout accès à la caméra est soumis à des **paramètres contrôlés par l’utilisateur**.

<div id="ios-node">
  ## Nœud iOS
</div>

<div id="user-setting-default-on">
  ### Paramètre utilisateur (activé par défaut)
</div>

- Onglet Réglages iOS → **Appareil photo** → **Autoriser l’appareil photo** (`camera.enabled`)
  - Par défaut : **activé** (une clé absente est considérée comme activée).
  - Lorsque désactivé : les commandes `camera.*` renvoient `CAMERA_DISABLED`.

<div id="commands-via-gateway-nodeinvoke">
  ### Commandes (via Gateway `node.invoke`)
</div>

- `camera.list`
  - Corps de la réponse :
    - `devices`: tableau de `{ id, name, position, deviceType }`

- `camera.snap`
  - Paramètres :
    - `facing`: `front|back` (par défaut : `front`)
    - `maxWidth`: nombre (facultatif ; par défaut `1600` sur le nœud iOS)
    - `quality`: `0..1` (facultatif ; par défaut `0.9`)
    - `format`: actuellement `jpg`
    - `delayMs`: nombre (facultatif ; par défaut `0`)
    - `deviceId`: chaîne (facultatif ; issu de `camera.list`)
  - Corps de la réponse :
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - Limitation de charge utile : les photos sont recompressées pour maintenir la charge utile base64 en dessous de 5 Mo.

- `camera.clip`
  - Paramètres :
    - `facing`: `front|back` (par défaut : `front`)
    - `durationMs`: nombre (par défaut `3000`, plafonné à `60000` maximum)
    - `includeAudio`: booléen (par défaut `true`)
    - `format`: actuellement `mp4`
    - `deviceId`: chaîne (facultatif ; issu de `camera.list`)
  - Corps de la réponse :
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

<div id="foreground-requirement">
  ### Exigence de fonctionnement au premier plan
</div>

Comme pour `canvas.*`, le nœud iOS n’autorise les commandes `camera.*` qu’au **premier plan**. Les appels effectués en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`.

<div id="cli-helper-temp-files-media">
  ### Utilitaire CLI (fichiers temporaires + MEDIA)
</div>

La manière la plus simple d&#39;obtenir des pièces jointes consiste à utiliser l&#39;utilitaire CLI, qui écrit le média décodé dans un fichier temporaire et affiche `MEDIA:<path>`.

Exemples :

```bash
openclaw nodes camera snap --node <id>               # par défaut : avant + arrière (2 lignes MEDIA)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

Remarques :

* `nodes camera snap` utilise par défaut **les deux** objectifs (avant et arrière) pour donner à l’agent les deux vues.
* Les fichiers de sortie sont temporaires (dans le répertoire temporaire du système d’exploitation), sauf si vous créez votre propre wrapper.


<div id="android-node">
  ## Nœud Android
</div>

<div id="user-setting-default-on">
  ### Paramètre utilisateur (activé par défaut)
</div>

- Panneau des paramètres Android → **Camera** → **Allow Camera** (`camera.enabled`)
  - Valeur par défaut : **activé** (une clé manquante est interprétée comme « activé »).
  - Lorsque désactivé : les commandes `camera.*` renvoient `CAMERA_DISABLED`.

<div id="permissions">
  ### Autorisations
</div>

- Android requiert des autorisations à l'exécution :
  - `CAMERA` pour `camera.snap` et `camera.clip`.
  - `RECORD_AUDIO` pour `camera.clip` lorsque `includeAudio=true`.

Si des autorisations sont manquantes, l'application les demandera lorsque c'est possible ; si elles sont refusées, les requêtes `camera.*` échouent avec une erreur
`*_PERMISSION_REQUIRED`.

<div id="foreground-requirement">
  ### Exigence de fonctionnement au premier plan
</div>

Comme pour `canvas.*`, le nœud Android n’autorise les commandes `camera.*` qu’au **premier plan**. Les appels en arrière-plan renvoient `NODE_BACKGROUND_UNAVAILABLE`.

<div id="payload-guard">
  ### Limite de taille du payload
</div>

Les photos sont recompressées pour que le payload base64 reste inférieur à 5 Mo.

<div id="macos-app">
  ## Application macOS
</div>

<div id="user-setting-default-off">
  ### Paramètre utilisateur (désactivé par défaut)
</div>

L’app compagnon macOS affiche une case à cocher :

- **Settings → General → Allow Camera** (`openclaw.cameraEnabled`)
  - Valeur par défaut : **off**
  - Lorsque désactivé : les requêtes de la caméra renvoient « Camera disabled by user ».

<div id="cli-helper-node-invoke">
  ### Aide CLI (node invoke)
</div>

Utilisez la CLI principale `openclaw` pour appeler des commandes de caméra sur le nœud macOS.

Exemples :

```bash
openclaw nodes camera list --node <id>            # liste les ID de caméra
openclaw nodes camera snap --node <id>            # affiche MEDIA:<chemin>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # affiche MEDIA:<chemin>
openclaw nodes camera clip --node <id> --duration-ms 3000      # affiche MEDIA:<chemin> (option obsolète)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

Remarques :

* `openclaw nodes camera snap` utilise par défaut `maxWidth=1600`, sauf si cette valeur est remplacée.
* Sous macOS, `camera.snap` attend `delayMs` (2000 ms par défaut) après la phase de préchauffage et la stabilisation de l’exposition avant de capturer.
* Les données photo sont recompressées pour maintenir la charge utile encodée en base64 en dessous de 5 Mo.


<div id="safety-practical-limits">
  ## Sécurité et limites pratiques
</div>

- L’accès à la caméra et au microphone déclenche les boîtes de dialogue d’autorisation habituelles du système d’exploitation (et nécessite des chaînes d’utilisation dans Info.plist).
- Les clips vidéo sont plafonnés (actuellement `<= 60s`) afin d’éviter des charges utiles de nœud trop volumineuses (surcoût lié au base64 + limites de messages).

<div id="macos-screen-video-os-level">
  ## Vidéo d’écran macOS (niveau système)
</div>

Pour la capture vidéo de l’écran (et non de la caméra), utilisez l’application compagnon macOS :

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # affiche MEDIA:<chemin>
```

Remarques :

* Nécessite l’autorisation **Enregistrement de l’écran** de macOS (TCC).
