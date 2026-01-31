---
title: Appareils
summary: "Référence CLI pour `openclaw devices` (appairage d’appareils + rotation/révocation de jetons d’appareil)"
read_when:
  - Vous approuvez des demandes d’appairage d’appareils
  - Vous devez effectuer une rotation ou révoquer des jetons d’appareil
---

<div id="openclaw-devices">
  # `openclaw devices`
</div>

Gérer les demandes d’appairage d’appareils et les jetons dont la portée est l’appareil.

<div id="commands">
  ## Commandes
</div>

<div id="openclaw-devices-list">
  ### `openclaw devices list`
</div>

Liste les demandes d&#39;appairage en attente et les appareils appairés.

```
openclaw devices list
openclaw devices list --json
```


<div id="openclaw-devices-approve-requestid">
  ### `openclaw devices approve <requestId>`
</div>

Approuve une demande d’appairage d’appareil en attente.

```
openclaw devices approve <requestId>
```


<div id="openclaw-devices-reject-requestid">
  ### `openclaw devices reject <requestId>`
</div>

Refuser une demande d’appairage d’appareil en attente.

```
openclaw devices reject <requestId>
```


<div id="openclaw-devices-rotate-device-id-role-role-scope-scope">
  ### `openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; [--scope &lt;scope...&gt;]`
</div>

Fait tourner un jeton d’appareil pour un rôle donné (en mettant éventuellement à jour les portées).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```


<div id="openclaw-devices-revoke-device-id-role-role">
  ### `openclaw devices revoke --device <id> --role <role>`
</div>

Révoque le jeton d’un appareil pour un rôle donné.

```
openclaw devices revoke --device <deviceId> --role node
```


<div id="common-options">
  ## Options courantes
</div>

- `--url <url>` : URL WebSocket du Gateway (par défaut `gateway.remote.url` lorsqu'il est configuré).
- `--token <token>` : jeton Gateway (si nécessaire).
- `--password <password>` : mot de passe Gateway (authentification par mot de passe).
- `--timeout <ms>` : délai d'expiration RPC.
- `--json` : sortie JSON (recommandée pour l'écriture de scripts).

<div id="notes">
  ## Remarques
</div>

- La rotation de jetons renvoie un nouveau jeton (donnée sensible). Traitez-le comme un secret.
- Ces commandes requièrent la portée `operator.pairing` (ou `operator.admin`).