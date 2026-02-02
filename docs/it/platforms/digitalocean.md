---
title: DigitalOcean
summary: "OpenClaw su DigitalOcean (semplice opzione VPS a pagamento)"
read_when:
  - Configurare OpenClaw su DigitalOcean
  - Alla ricerca di un hosting VPS economico per OpenClaw
---

<div id="openclaw-on-digitalocean">
  # OpenClaw su DigitalOcean
</div>

<div id="goal">
  ## Obiettivo
</div>

Eseguire un Gateway OpenClaw persistente su DigitalOcean per **6 $ al mese** (o 4 $ al mese con tariffe riservate).

Se vuoi un&#39;opzione da 0 $ al mese e non ti dispiace lavorare con ARM e una configurazione specifica del provider, consulta la [guida Oracle Cloud](/it/platforms/oracle).

<div id="cost-comparison-2026">
  ## Confronto dei costi (2026)
</div>

| Provider | Piano | Specifiche | Prezzo/mese | Note |
|----------|-------|------------|-------------|------|
| Oracle Cloud | Always Free ARM | fino a 4 OCPU, 24GB RAM | $0 | ARM, capacità limitata / particolarità nella procedura di registrazione |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | €3.79 (~$4) | Opzione a pagamento più economica |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | UI semplice, buona documentazione |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Molte location |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ora parte di Akamai |

**Scelta di un provider:**

* DigitalOcean: UX più semplice + configurazione prevedibile (questa guida)
* Hetzner: buon rapporto prezzo/prestazioni (vedi [guida Hetzner](/it/platforms/hetzner))
* Oracle Cloud: può costare $0/mese, ma è più macchinoso e solo ARM (vedi [guida Oracle](/it/platforms/oracle))

***

<div id="prerequisites">
  ## Prerequisiti
</div>

* Un account DigitalOcean ([registrati con 200 $ di credito gratuito](https://m.do.co/c/signup))
* Coppia di chiavi SSH (o disponibilità a usare l&#39;autenticazione con password)
* ~20 minuti

<div id="1-create-a-droplet">
  ## 1) Crea un Droplet
</div>

1. Effettua l&#39;accesso a [DigitalOcean](https://cloud.digitalocean.com/)
2. Fai clic su **Create → Droplets**
3. Scegli:
   * **Region:** la più vicina a te (o ai tuoi utenti)
   * **Image:** Ubuntu 24.04 LTS
   * **Size:** Basic → Regular → **$6/mese** (1 vCPU, 1GB RAM, 25GB SSD)
   * **Authentication:** chiave SSH (consigliata) o password
4. Fai clic su **Create Droplet**
5. Annota l&#39;indirizzo IP

<div id="2-connect-via-ssh">
  ## 2) Collegati via SSH
</div>

```bash
ssh root@YOUR_DROPLET_IP
```

<div id="3-install-openclaw">
  ## 3) Installa OpenClaw
</div>

```bash
# Update system
apt update && apt upgrade -y

# Installa Node.js 22
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Install OpenClaw
curl -fsSL https://openclaw.bot/install.sh | bash

# Verify
openclaw --version
```

<div id="4-run-onboarding">
  ## 4) Esegui la procedura di onboarding
</div>

```bash
openclaw onboard --install-daemon
```

La procedura guidata ti guiderà attraverso:

* Autenticazione dei modelli (chiavi API o OAuth)
* Configurazione dei canali (Telegram, WhatsApp, Discord, ecc.)
* Token del Gateway (generato automaticamente)
* Installazione del demone (systemd)

<div id="5-verify-the-gateway">
  ## 5) Verifica il Gateway
</div>

```bash
# Controlla lo stato
openclaw status

# Controlla il servizio
systemctl --user status openclaw-gateway.service

# Visualizza i log
journalctl --user -u openclaw-gateway.service -f
```

<div id="6-access-the-dashboard">
  ## 6) Accedi alla Dashboard
</div>

Per impostazione predefinita, il Gateway è in ascolto sull&#39;interfaccia di loopback. Per accedere alla Control UI:

**Opzione A: tunnel SSH (consigliato)**

```bash
# Dalla tua macchina locale
ssh -L 18789:localhost:18789 root@YOUR_DROPLET_IP

# Quindi apri: http://localhost:18789
```

**Opzione B: Tailscale Serve (HTTPS, solo su loopback)**

```bash
# On the droplet
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up

# Configura Gateway per usare Tailscale Serve
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

Apri: `https://<magicdns>/`

Note:

* Serve mantiene il Gateway in ascolto solo sull&#39;interfaccia loopback e autentica tramite le intestazioni di identità di Tailscale.
* Per richiedere invece un token/una password, imposta `gateway.auth.allowTailscale: false` oppure usa `gateway.auth.mode: "password"`.

**Opzione C: bind su Tailnet (senza Serve)**

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Apri `http://<tailscale-ip>:18789` (richiede token).

<div id="7-connect-your-channels">
  ## 7) Collega i canali
</div>

<div id="telegram">
  ### Telegram
</div>

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

<div id="whatsapp">
  ### WhatsApp
</div>

```bash
openclaw channels login whatsapp
# Scansiona il codice QR
```

Consulta la pagina [Channels](/it/channels) per altri provider.

***

<div id="optimizations-for-1gb-ram">
  ## Ottimizzazioni per 1 GB di RAM
</div>

Il droplet da 6$ ha solo 1 GB di RAM. Per mantenere il sistema stabile e reattivo:

<div id="add-swap-recommended">
  ### Aggiungi spazio di swap (consigliato)
</div>

```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

<div id="use-a-lighter-model">
  ### Usa un modello più leggero
</div>

Se incappi in errori OOM (out of memory), valuta di:

* Usare modelli basati su API (Claude, GPT) invece di modelli locali
* Impostare `agents.defaults.model.primary` su un modello più piccolo

<div id="monitor-memory">
  ### Monitoraggio della memoria
</div>

```bash
free -h
htop
```

***

<div id="persistence">
  ## Persistenza
</div>

Tutto lo stato risiede in:

* `~/.openclaw/` — configurazione, credenziali, dati di sessione
* `~/.openclaw/workspace/` — spazio di lavoro (SOUL.md, memoria, ecc.)

Questi dati persistono tra un riavvio e l&#39;altro. Esegui periodicamente un backup:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="oracle-cloud-free-alternative">
  ## Alternativa gratuita su Oracle Cloud
</div>

Oracle Cloud offre istanze ARM **Always Free** che sono significativamente più potenti di qualsiasi opzione a pagamento qui — per 0 $/mese.

| Cosa include | Specifiche |
|--------------|-----------|
| **4 OCPU** | ARM Ampere A1 |
| **24GB RAM** | Più che sufficiente |
| **200GB storage** | Volume a blocchi |
| **Sempre gratis** | Nessun addebito sulla carta di credito |

**Avvertenze:**

* La registrazione può essere un po’ problematica (riprova se non va a buon fine)
* Architettura ARM — quasi tutto funziona, ma alcuni binari richiedono build specifiche per ARM

Per la guida completa alla configurazione, consulta [Oracle Cloud](/it/platforms/oracle). Per suggerimenti sulla registrazione e per la risoluzione dei problemi nel processo di iscrizione, consulta questa [guida della community](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd).

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="gateway-wont-start">
  ### Il Gateway non si avvia
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl -u openclaw --no-pager -n 50
```

<div id="port-already-in-use">
  ### Porta già in uso
</div>

```bash
lsof -i :18789
kill <PID>
```

<div id="out-of-memory">
  ### Memoria esaurita
</div>

```bash
# Verifica la memoria
free -h

# Aggiungi più swap
# Oppure passa al droplet da $12/mese (2GB RAM)
```

***

<div id="see-also">
  ## Vedi anche
</div>

* [Guida Hetzner](/it/platforms/hetzner) — più economico e più potente
* [Installazione Docker](/it/install/docker) — setup containerizzato
* [Tailscale](/it/gateway/tailscale) — accesso remoto sicuro
* [Configurazione](/it/gateway/configuration) — riferimento completo per la configurazione