---
title: Sécurité
summary: "Référence CLI pour `openclaw security` (auditer et corriger les pièges de sécurité classiques)"
read_when:
  - Vous voulez lancer un audit de sécurité rapide sur la configuration et l’état
  - Vous voulez appliquer des suggestions de correctifs sûrs (chmod, durcissement des valeurs par défaut)
---

<div id="openclaw-security">
  # `openclaw security`
</div>

Outils de sécurité (audit + correctifs facultatifs).

Contenu associé :

* Guide de sécurité : [Sécurité](/fr/gateway/security)

<div id="audit">
  ## Audit
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

L&#39;audit signale lorsque plusieurs expéditeurs de DM partagent la session principale et recommande `session.dmScope="per-channel-peer"` (ou `per-account-channel-peer` pour les canaux multi‑comptes) pour les boîtes de réception partagées.
Il signale également lorsque de petits modèles (`<=300B`) sont utilisés sans sandboxing et avec les outils Web/navigateur activés.
