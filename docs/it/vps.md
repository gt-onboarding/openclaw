---
title: VPS
summary: "Hub di hosting VPS per OpenClaw (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Vuoi eseguire Gateway nel cloud
  - Ti serve una panoramica rapida delle guide VPS/hosting
---

<div id="vps-hosting">
  # Hosting VPS
</div>

Questo hub collega alle guide supportate su VPS/hosting e spiega, a livello generale, come funzionano i deployment nel cloud.

<div id="pick-a-provider">
  ## Scegli un provider
</div>

* **Railway** (configurazione con un clic dal browser): [Railway](/it/railway)
* **Northflank** (configurazione con un clic dal browser): [Northflank](/it/northflank)
* **Oracle Cloud (Always Free)**: [Oracle](/it/platforms/oracle) — $0/mese (Always Free, ARM; capacità e procedura di registrazione possono essere un po’ macchinose)
* **Fly.io**: [Fly.io](/it/platforms/fly)
* **Hetzner (Docker)**: [Hetzner](/it/platforms/hetzner)
* **GCP (Compute Engine)**: [GCP](/it/platforms/gcp)
* **exe.dev** (VM + proxy HTTPS): [exe.dev](/it/platforms/exe-dev)
* **AWS (EC2/Lightsail/free tier)**: funziona bene anche AWS. Guida video:
  https://x.com/techfrenAJ/status/2014934471095812547

<div id="how-cloud-setups-work">
  ## Come funzionano i setup cloud
</div>

* Il **Gateway viene eseguito sul VPS** e gestisce stato e spazio di lavoro.
* Ti connetti dal tuo laptop/telefono tramite la **Control UI** o **Tailscale/SSH**.
* Considera il VPS come source of truth ed esegui il **backup** di stato e spazio di lavoro.
* Impostazione predefinita sicura: mantieni il Gateway in ascolto su loopback e accedi tramite tunnel SSH o Tailscale Serve.
  Se lo esponi su `lan`/`tailnet`, richiedi `gateway.auth.token` o `gateway.auth.password`.

Accesso remoto al Gateway: [Gateway remote](/it/gateway/remote)\
Hub piattaforme: [Platforms](/it/platforms)

<div id="using-nodes-with-a-vps">
  ## Utilizzo dei nodi con un VPS
</div>

Puoi mantenere il Gateway in esecuzione nel cloud e associare **nodi** ai tuoi dispositivi locali
(Mac/iOS/Android/headless). I nodi forniscono funzionalità locali per schermo/fotocamera/canvas e il comando `system.run`
mentre il Gateway rimane nel cloud.

Documentazione: [Nodes](/it/nodes), [Nodes CLI](/it/cli/nodes)