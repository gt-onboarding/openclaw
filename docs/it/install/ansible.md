---
title: Ansible
summary: "Installazione automatizzata e con hardening di OpenClaw tramite Ansible, VPN Tailscale e isolamento tramite firewall"
read_when:
  - Vuoi un deployment del server automatizzato con hardening della sicurezza
  - Ti serve una configurazione isolata tramite firewall con accesso tramite VPN
  - Stai effettuando il deployment su server Debian/Ubuntu remoti
---

<div id="ansible-installation">
  # Installazione con Ansible
</div>

Il modo consigliato per distribuire OpenClaw su server di produzione è tramite **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — un programma di installazione automatizzato con un&#39;architettura incentrata prima di tutto sulla sicurezza.

<div id="quick-start">
  ## Avvio rapido
</div>

Installazione con un solo comando:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 Guida completa: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> Il repository openclaw-ansible è il riferimento principale per la distribuzione con Ansible. Questa pagina è una rapida panoramica.

<div id="what-you-get">
  ## Cosa ottieni
</div>

* 🔒 **Sicurezza firewall-first**: isolamento UFW + Docker (sono accessibili solo SSH e Tailscale)
* 🔐 **VPN Tailscale**: accesso remoto sicuro senza esporre pubblicamente i servizi
* 🐳 **Docker**: container sandbox isolati, binding solo su localhost
* 🛡️ **Difesa in profondità**: architettura di sicurezza a 4 livelli
* 🚀 **Configurazione con un solo comando**: distribuzione completa in pochi minuti
* 🔧 **Integrazione con systemd**: avvio automatico all’avvio del sistema con hardening

<div id="requirements">
  ## Requisiti
</div>

* **OS**: Debian 11+ o Ubuntu 20.04+
* **Accesso**: permessi root o sudo
* **Rete**: connessione Internet per l&#39;installazione dei pacchetti
* **Ansible**: 2.14+ (installato automaticamente dallo script di quick-start)

<div id="what-gets-installed">
  ## Cosa viene installato
</div>

Il playbook Ansible installa e configura:

1. **Tailscale** (VPN mesh per l&#39;accesso remoto sicuro)
2. **Firewall UFW** (solo porte SSH + Tailscale)
3. **Docker CE + Compose V2** (per le sandbox degli agenti)
4. **Node.js 22.x + pnpm** (dipendenze di runtime)
5. **OpenClaw** (in esecuzione sull&#39;host, non containerizzato)
6. **Servizio systemd** (avvio automatico con hardening della sicurezza)

Nota: il Gateway viene eseguito **direttamente sull&#39;host** (non in Docker), ma le sandbox degli agenti usano Docker per l&#39;isolamento. Consulta [Sandboxing](/it/gateway/sandboxing) per i dettagli.

<div id="post-install-setup">
  ## Configurazione post-installazione
</div>

Una volta completata l&#39;installazione, passa all&#39;utente openclaw:

```bash
sudo -i -u openclaw
```

Lo script post-installazione ti guiderà nei passaggi seguenti:

1. **Procedura guidata di onboarding**: configura le impostazioni di OpenClaw
2. **Accesso al provider**: collega WhatsApp/Telegram/Discord/Signal
3. **Test del Gateway**: verifica l&#39;installazione
4. **Configurazione di Tailscale**: collegati alla tua rete mesh VPN

<div id="quick-commands">
  ### Comandi rapidi
</div>

```bash
# Verifica lo stato del servizio
sudo systemctl status openclaw

# Visualizza i log in tempo reale
sudo journalctl -u openclaw -f

# Riavvia il Gateway
sudo systemctl restart openclaw

# Login del provider (esegui come utente openclaw)
sudo -i -u openclaw
openclaw channels login
```

<div id="security-architecture">
  ## Architettura di sicurezza
</div>

<div id="4-layer-defense">
  ### Difesa su 4 livelli
</div>

1. **Firewall (UFW)**: Solo SSH (22) + Tailscale (41641/udp) esposti pubblicamente
2. **VPN (Tailscale)**: Gateway accessibile solo tramite rete mesh VPN
3. **Isolamento Docker**: La catena iptables DOCKER-USER impedisce l&#39;esposizione di porte verso l&#39;esterno
4. **Hardening di systemd**: NoNewPrivileges, PrivateTmp, utente non privilegiato

<div id="verification">
  ### Verifica
</div>

Testa la superficie di attacco esterna:

```bash
nmap -p- YOUR_SERVER_IP
```

Dovrebbe risultare **solo la porta 22** (SSH) aperta. Tutti gli altri servizi (Gateway, Docker) devono risultare chiusi.

<div id="docker-availability">
  ### Disponibilità di Docker
</div>

Docker è installato per le **sandbox degli agenti** (esecuzione isolata degli strumenti), non per eseguire il Gateway stesso. Il Gateway esegue il binding solo su localhost ed è accessibile tramite la VPN Tailscale.

Consulta [Sandbox e strumenti multi-agente](/it/multi-agent-sandbox-tools) per la configurazione delle sandbox.

<div id="manual-installation">
  ## Installazione manuale
</div>

Se preferisci gestire tutto manualmente invece di usare l&#39;automazione:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Oppure esegui direttamente (quindi esegui manualmente /tmp/openclaw-setup.sh in seguito)
# ansible-playbook playbook.yml --ask-become-pass
```

<div id="updating-openclaw">
  ## Aggiornare OpenClaw
</div>

Il programma di installazione Ansible configura OpenClaw per gli aggiornamenti manuali. Consulta [Aggiornamento](/it/install/updating) per il flusso di aggiornamento standard.

Per eseguire nuovamente il playbook Ansible (ad esempio per applicare modifiche di configurazione):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Nota: questa operazione è idempotente e può essere eseguita più volte in sicurezza.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="firewall-blocks-my-connection">
  ### Il firewall blocca la mia connessione
</div>

Se rimani bloccato fuori:

* Assicurati prima di poter accedere tramite VPN Tailscale
* L&#39;accesso SSH (porta 22) è sempre consentito
* Il Gateway è **accessibile solo** tramite Tailscale per progettazione

<div id="service-wont-start">
  ### Il servizio non si avvia
</div>

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Testa l'avvio manuale
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

<div id="docker-sandbox-issues">
  ### Problemi della sandbox Docker
</div>

```bash
# Verifica che Docker sia in esecuzione
sudo systemctl status docker

# Verifica l'immagine sandbox
sudo docker images | grep openclaw-sandbox

# Costruisci l'immagine sandbox se mancante
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

<div id="provider-login-fails">
  ### Accesso al provider non riuscito
</div>

Assicurati di eseguire l&#39;operazione come utente `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

<div id="advanced-configuration">
  ## Configurazione avanzata
</div>

Per ulteriori dettagli sull&#39;architettura di sicurezza e sulla risoluzione dei problemi:

* [Architettura di sicurezza](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
* [Dettagli tecnici](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
* [Guida alla risoluzione dei problemi](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

<div id="related">
  ## Correlati
</div>

* [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — guida completa alla distribuzione
* [Docker](/it/install/docker) — configurazione containerizzata del Gateway
* [Sandboxing](/it/gateway/sandboxing) — configurazione della sandbox degli agenti
* [Multi-Agent Sandbox &amp; Tools](/it/multi-agent-sandbox-tools) — isolamento per singolo agente