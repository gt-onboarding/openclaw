---
title: Canvas
summary: "Panneau Canvas contrôlé par un Agent, intégré via WKWebView + schéma d’URL personnalisé"
read_when:
  - Implémentation du panneau Canvas sur macOS
  - Ajout de contrôles d’agent pour l’espace de travail visuel
  - Débogage du chargement de Canvas dans WKWebView
---

<div id="canvas-macos-app">
  # Canvas (app macOS)
</div>

L'app macOS intègre un **panneau Canvas** contrôlé par un agent à l'aide de `WKWebView`. Il s'agit d'un espace de travail visuel léger pour HTML/CSS/JS, A2UI et de petites surfaces d'UI interactives.

<div id="where-canvas-lives">
  ## Emplacement de Canvas
</div>

L’état de Canvas est stocké dans Application Support :

- `~/Library/Application Support/OpenClaw/canvas/<session>/...`

Le panneau Canvas expose ces fichiers via un **schéma d’URL personnalisé** :

- `openclaw-canvas://<session>/<path>`

Exemples :

- `openclaw-canvas://main/` → `<canvasRoot>/main/index.html`
- `openclaw-canvas://main/assets/app.css` → `<canvasRoot>/main/assets/app.css`
- `openclaw-canvas://main/widgets/todo/` → `<canvasRoot>/main/widgets/todo/index.html`

Si aucun `index.html` n’existe à la racine, l’application affiche une **page modèle intégrée**.

<div id="panel-behavior">
  ## Comportement du panneau
</div>

- Panneau sans bordure, redimensionnable, ancré près de la barre de menus (ou du curseur de la souris).
- Conserve la taille et la position pour chaque session.
- Se recharge automatiquement lorsque des fichiers Canvas locaux sont modifiés.
- Un seul panneau Canvas est visible à la fois (la session est changée si nécessaire).

Canvas peut être désactivé dans Réglages → **Autoriser Canvas**. Lorsqu’il est désactivé, les commandes du nœud canvas renvoient `CANVAS_DISABLED`.

<div id="agent-api-surface">
  ## Surface d’API de l’Agent
</div>

Canvas est exposé via le **Gateway WebSocket**, ce qui permet à l’agent de :

* afficher/masquer le panneau
* naviguer vers un chemin ou une URL
* évaluer du JavaScript
* capturer une image instantanée

Exemples CLI :

```bash
openclaw nodes canvas present --node <id>
openclaw nodes canvas navigate --node <id> --url "/"
openclaw nodes canvas eval --node <id> --js "document.title"
openclaw nodes canvas snapshot --node <id>
```

Remarques :

* `canvas.navigate` accepte les **chemins Canvas locaux**, les URL `http(s)` et les URL `file://`.
* Si vous fournissez `"/"`, Canvas affiche le squelette local ou `index.html`.


<div id="a2ui-in-canvas">
  ## A2UI dans Canvas
</div>

A2UI est hébergé par l’hôte Canvas du Gateway et rendu dans le panneau Canvas.
Quand le Gateway expose un hôte Canvas, l’application macOS navigue automatiquement
vers la page hôte A2UI lors de la première ouverture.

URL de l’hôte A2UI par défaut :

```
http://<gateway-host>:18793/__openclaw__/a2ui/
```


<div id="a2ui-commands-v08">
  ### Commandes A2UI (v0.8)
</div>

Canvas accepte actuellement les messages serveur→client **A2UI v0.8** suivants :

* `beginRendering`
* `surfaceUpdate`
* `dataModelUpdate`
* `deleteSurface`

`createSurface` (v0.9) n&#39;est pas pris en charge.

Exemple de commande CLI :

```bash
cat > /tmp/a2ui-v0.8.jsonl <<'EOFA2'
{"surfaceUpdate":{"surfaceId":"main","components":[{"id":"root","component":{"Column":{"children":{"explicitList":["title","content"]}}}},{"id":"title","component":{"Text":{"text":{"literalString":"Canvas (A2UI v0.8)"},"usageHint":"h1"}}},{"id":"content","component":{"Text":{"text":{"literalString":"If you can read this, A2UI push works."},"usageHint":"body"}}}]}}
{"beginRendering":{"surfaceId":"main","root":"root"}}
EOFA2

openclaw nodes canvas a2ui push --jsonl /tmp/a2ui-v0.8.jsonl --node <id>
```

Vérification rapide :

```bash
openclaw nodes canvas a2ui push --node <id> --text "Hello from A2UI"
```


<div id="triggering-agent-runs-from-canvas">
  ## Déclencher des exécutions d’agent depuis Canvas
</div>

Canvas peut déclencher de nouvelles exécutions d’agent via des liens profonds (deep links) :

* `openclaw://agent?...`

Exemple (en JS) :

```js
window.location.href = "openclaw://agent?message=Review%20this%20design";
```

L’app vous demande de confirmer, sauf si une clé valide est fournie.


<div id="security-notes">
  ## Notes de sécurité
</div>

- Le schéma Canvas empêche la traversée de répertoires ; tous les fichiers doivent se trouver sous la racine de la session.
- Le contenu Canvas local utilise un schéma personnalisé (aucun serveur loopback requis).
- Les URL externes `http(s)` ne sont autorisées que lorsqu’une navigation explicite vers celles‑ci est effectuée.