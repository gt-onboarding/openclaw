---
title: Autorisations
summary: "Persistance des autorisations macOS (TCC) et exigences de signature"
read_when:
  - Dépannage des invites d’autorisation macOS manquantes ou bloquées
  - Création de paquets ou signature de l’app macOS
  - Modification des identifiants de bundle ou des chemins d’installation de l’app
---

<div id="macos-permissions-tcc">
  # Autorisations macOS (TCC)
</div>

Les autorisations macOS sont fragiles. TCC associe une autorisation à la
signature de code de l’application, à son identifiant de bundle et à son chemin sur le disque. Si l’un de ces éléments change,
macOS considère l’application comme nouvelle et peut supprimer ou masquer les invites de demande d’autorisation.

<div id="requirements-for-stable-permissions">
  ## Exigences pour des autorisations stables
</div>

- Même chemin : exécute l’app depuis un emplacement fixe (pour OpenClaw, `dist/OpenClaw.app`).
- Même identifiant de bundle : changer l’ID de bundle crée une nouvelle identité d’autorisation.
- App signée : les builds non signés ou signés en ad-hoc ne conservent pas les autorisations.
- Signature cohérente : utilise un véritable certificat Apple Development ou Developer ID
  afin que la signature reste stable entre les builds.

Les signatures ad-hoc génèrent une nouvelle identité à chaque build. macOS oubliera les
autorisations accordées auparavant, et les demandes d’autorisation peuvent disparaître complètement tant que les anciennes entrées n’ont pas été nettoyées.

<div id="recovery-checklist-when-prompts-disappear">
  ## Liste de vérification de récupération lorsque les demandes d&#39;autorisation disparaissent
</div>

1. Quittez l&#39;app.
2. Supprimez l&#39;entrée de l&#39;app dans Réglages Système -&gt; Confidentialité &amp; sécurité.
3. Relancez l&#39;app depuis le même chemin et accordez de nouveau les autorisations.
4. Si la demande d&#39;autorisation n&#39;apparaît toujours pas, réinitialisez les entrées TCC avec `tccutil` et réessayez.
5. Certaines autorisations ne réapparaissent qu&#39;après un redémarrage complet de macOS.

Exemples de réinitialisation (remplacez l&#39;ID de bundle si nécessaire) :

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

Si vous testez les autorisations, signez toujours avec un certificat réel. Les builds ad hoc ne sont acceptables que pour des exécutions locales rapides où les autorisations sont sans importance.
