---
title: Oracle
summary: "OpenClaw su Oracle Cloud (Always Free ARM)"
read_when:
  - Configurazione di OpenClaw su Oracle Cloud
  - Ricerca di un hosting VPS a basso costo per OpenClaw
  - Esecuzione di OpenClaw 24/7 su un piccolo server
---

<div id="openclaw-on-oracle-cloud-oci">
  # OpenClaw su Oracle Cloud (OCI)
</div>

<div id="goal">
  ## Obiettivo
</div>

Eseguire in modo persistente un Gateway OpenClaw sul livello ARM **Always Free** di Oracle Cloud.

Il livello gratuito di Oracle può essere una buona soluzione per OpenClaw (soprattutto se disponi già di un account OCI), ma comporta alcuni compromessi:

* Architettura ARM (la maggior parte delle funzionalità è supportata, ma alcuni binari potrebbero essere disponibili solo per x86)
* Capacità disponibile e procedura di registrazione possono essere un po’ imprevedibili

<div id="cost-comparison-2026">
  ## Confronto dei costi (2026)
</div>

| Provider | Piano | Specifiche | Prezzo/mese | Note |
|----------|-------|------------|-------------|------|
| Oracle Cloud | Always Free ARM | fino a 4 OCPU, 24GB RAM | $0 | ARM, capacità limitata |
| Hetzner | CX22 | 2 vCPU, 4GB RAM | ~ $4 | Opzione a pagamento più economica |
| DigitalOcean | Basic | 1 vCPU, 1GB RAM | $6 | UI intuitiva, buona documentazione |
| Vultr | Cloud Compute | 1 vCPU, 1GB RAM | $6 | Molte località |
| Linode | Nanode | 1 vCPU, 1GB RAM | $5 | Ora parte di Akamai |

***

<div id="prerequisites">
  ## Prerequisiti
</div>

* Account Oracle Cloud ([registrazione](https://www.oracle.com/cloud/free/)) — consulta la [guida della community alla registrazione](https://gist.github.com/rssnyder/51e3cfedd730e7dd5f4a816143b25dbd) se riscontri problemi
* Account Tailscale (gratuito su [tailscale.com](https://tailscale.com))
* ~30 minuti

<div id="1-create-an-oci-instance">
  ## 1) Crea un&#39;istanza OCI
</div>

1. Accedi alla [Oracle Cloud Console](https://cloud.oracle.com/)
2. Vai a **Compute → Instances → Create Instance**
3. Configura:
   * **Name:** `openclaw`
   * **Image:** Ubuntu 24.04 (aarch64)
   * **Shape:** `VM.Standard.A1.Flex` (Ampere ARM)
   * **OCPUs:** 2 (o fino a 4)
   * **Memory:** 12 GB (o fino a 24 GB)
   * **Boot volume:** 50 GB (fino a 200 GB di spazio libero)
   * **SSH key:** Aggiungi la tua chiave pubblica
4. Fai clic su **Create**
5. Prendi nota dell&#39;indirizzo IP pubblico

**Suggerimento:** Se la creazione dell&#39;istanza non riesce con &quot;Out of capacity&quot;, prova un dominio di disponibilità diverso oppure riprova più tardi. La capacità del free tier è limitata.

<div id="2-connect-and-update">
  ## 2) Connetti e aggiorna
</div>

```bash
# Connetti tramite IP pubblico
ssh ubuntu@YOUR_PUBLIC_IP

# Aggiorna il sistema
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential
```

**Nota:** `build-essential` è necessario per la compilazione su ARM di alcune dipendenze.

<div id="3-configure-user-and-hostname">
  ## 3) Configura utente e nome host
</div>

```bash
# Imposta l'hostname
sudo hostnamectl set-hostname openclaw

# Imposta la password per l'utente ubuntu
sudo passwd ubuntu

# Abilita lingering (mantiene i servizi utente in esecuzione dopo il logout)
sudo loginctl enable-linger ubuntu
```

<div id="4-install-tailscale">
  ## 4) Installa Tailscale
</div>

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up --ssh --hostname=openclaw
```

Questo attiva Tailscale SSH, in modo da poterti connettere tramite `ssh openclaw` da qualsiasi dispositivo sulla tua tailnet, senza bisogno di un IP pubblico.

Verifica:

```bash
tailscale status
```

**Da ora in poi collegati tramite Tailscale:** `ssh ubuntu@openclaw` (oppure usa l&#39;IP Tailscale).

<div id="5-install-openclaw">
  ## 5) Installare OpenClaw
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
source ~/.bashrc
```

Quando ti viene chiesto &quot;How do you want to hatch your bot?&quot;, seleziona **&quot;Do this later&quot;**.

> Nota: se riscontri problemi con build nativi per ARM, inizia dai pacchetti di sistema (ad es. `sudo apt install -y build-essential`) prima di ricorrere a Homebrew.

<div id="6-configure-gateway-loopback-token-auth-and-enable-tailscale-serve">
  ## 6) Configura il Gateway (loopback + token auth) e abilita Tailscale Serve
</div>

Usa l&#39;autenticazione tramite token come modalità predefinita. È prevedibile ed evita la necessità di qualsiasi flag della Control UI per l&#39;“autenticazione non sicura”.

```bash
# Keep the Gateway private on the VM
openclaw config set gateway.bind loopback

# Require auth for the Gateway + Control UI
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token

# Esponi tramite Tailscale Serve (HTTPS + accesso tailnet)
openclaw config set gateway.tailscale.mode serve
openclaw config set gateway.trustedProxies '["127.0.0.1"]'

systemctl --user restart openclaw-gateway
```

<div id="7-verify">
  ## 7) Verifica
</div>

```bash
# Check version
openclaw --version

# Controlla lo stato del daemon
systemctl --user status openclaw-gateway

# Check Tailscale Serve
tailscale serve status

# Test local response
curl http://localhost:18789
```

<div id="8-lock-down-vcn-security">
  ## 8) Mettere in sicurezza la VCN
</div>

Ora che tutto funziona, metti in sicurezza la VCN per bloccare tutto il traffico tranne Tailscale. La Virtual Cloud Network di OCI funge da firewall al perimetro della rete: il traffico viene bloccato prima che raggiunga la tua istanza.

1. Vai su **Networking → Virtual Cloud Networks** nella console OCI
2. Clicca sulla tua VCN → **Security Lists** → Default Security List
3. **Rimuovi** tutte le regole in ingresso tranne:
   * `0.0.0.0/0 UDP 41641` (Tailscale)
4. Mantieni le regole di uscita predefinite (consenti tutto il traffico in uscita)

Questo blocca SSH sulla porta 22, HTTP, HTTPS e tutto il resto al perimetro della rete. D&#39;ora in poi, puoi connetterti solo tramite Tailscale.

***

<div id="access-the-control-ui">
  ## Accedi al Control UI
</div>

Da qualsiasi dispositivo nella tua rete Tailscale:

```
https://openclaw.<tailnet-name>.ts.net/
```

Sostituisci `<tailnet-name>` con il nome del tuo tailnet (visibile eseguendo `tailscale status`).

Non serve alcun tunnel SSH. Tailscale fornisce:

* Crittografia HTTPS (certificati automatici)
* Autenticazione tramite identità Tailscale
* Accesso da qualsiasi dispositivo del tuo tailnet (laptop, telefono, ecc.)

***

<div id="security-vcn-tailscale-recommended-baseline">
  ## Sicurezza: VCN + Tailscale (configurazione di base consigliata)
</div>

Con la VCN bloccata (solo UDP 41641 aperta) e il Gateway vincolato all’interfaccia di loopback, ottieni una solida difesa in profondità: il traffico pubblico viene bloccato al perimetro di rete e l’accesso di amministrazione avviene tramite il tuo tailnet.

Questa configurazione spesso elimina la *necessità* di regole firewall aggiuntive a livello di host solo per fermare gli attacchi SSH brute force provenienti da Internet, ma dovresti comunque mantenere il sistema operativo aggiornato, eseguire `openclaw security audit` e verificare di non avere servizi in ascolto per errore su interfacce pubbliche.

<div id="whats-already-protected">
  ### Cosa è già protetto
</div>

| Passaggio tradizionale | Necessario? | Perché |
|------------------------|------------|--------|
| Firewall UFW | No | La VCN blocca il traffico prima che raggiunga l&#39;istanza |
| fail2ban | No | Nessun attacco brute force se la porta 22 è bloccata a livello di VCN |
| Hardening di sshd | No | Tailscale SSH non usa sshd |
| Disabilitare l&#39;accesso dell&#39;utente root | No | Tailscale usa l&#39;identità Tailscale, non gli utenti di sistema |
| Autenticazione solo con chiave SSH | No | Tailscale esegue l&#39;autenticazione tramite la tua tailnet |
| Hardening IPv6 | Di solito no | Dipende dalle impostazioni della tua VCN/subnet; verifica cosa è effettivamente assegnato/esposto |

<div id="still-recommended">
  ### Ulteriori raccomandazioni
</div>

* **Permessi per le credenziali:** `chmod 700 ~/.openclaw`
* **Verifica della sicurezza:** `openclaw security audit`
* **Aggiornamenti di sistema:** esegui regolarmente `sudo apt update && sudo apt upgrade`
* **Monitora Tailscale:** controlla i dispositivi nella [console di amministrazione Tailscale](https://login.tailscale.com/admin)

<div id="verify-security-posture">
  ### Verifica della postura di sicurezza
</div>

```bash
# Conferma che non ci siano porte pubbliche in ascolto
sudo ss -tlnp | grep -v '127.0.0.1\|::1'

# Verifica che Tailscale SSH sia attivo
tailscale status | grep -q 'offers: ssh' && echo "Tailscale SSH active"

# Opzionale: disabilita completamente sshd
sudo systemctl disable --now ssh
```

***

<div id="fallback-ssh-tunnel">
  ## Alternativa: tunnel SSH
</div>

Se Tailscale Serve non funziona, usa un tunnel SSH:

```bash
# Dalla tua macchina locale (tramite Tailscale)
ssh -L 18789:127.0.0.1:18789 ubuntu@openclaw
```

Poi apri `http://localhost:18789`.

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="instance-creation-fails-out-of-capacity">
  ### Creazione dell&#39;istanza non riuscita (&quot;Out of capacity&quot;)
</div>

Le istanze ARM del livello gratuito sono molto richieste. Prova a:

* Usare un dominio di disponibilità diverso
* Riprova nelle ore di minor traffico (prime ore del mattino)
* Usare il filtro &quot;Always Free&quot; quando selezioni la shape

<div id="tailscale-wont-connect">
  ### Tailscale non riesce a connettersi
</div>

```bash
# Controlla lo stato
sudo tailscale status

# Ri-autentica
sudo tailscale up --ssh --hostname=openclaw --reset
```

<div id="gateway-wont-start">
  ### Il Gateway non si avvia
</div>

```bash
openclaw gateway status
openclaw doctor --non-interactive
journalctl --user -u openclaw-gateway -n 50
```

<div id="cant-reach-control-ui">
  ### Impossibile accedere a Control UI
</div>

```bash
# Verifica che Tailscale Serve sia in esecuzione
tailscale serve status

# Check gateway is listening
curl http://localhost:18789

# Restart if needed
systemctl --user restart openclaw-gateway
```

<div id="arm-binary-issues">
  ### Problemi con i binari ARM
</div>

Alcuni strumenti potrebbero non disporre di build per ARM. Controlla:

```bash
uname -m  # Dovrebbe mostrare aarch64
```

La maggior parte dei pacchetti npm funziona senza problemi. Per i binari, cerca versioni `linux-arm64` o `aarch64`.

***

<div id="persistence">
  ## Persistenza
</div>

Tutto lo stato è conservato in:

* `~/.openclaw/` — configurazione, credenziali, dati di sessione
* `~/.openclaw/workspace/` — spazio di lavoro (SOUL.md, memoria, artefatti)

Esegui periodicamente un backup:

```bash
tar -czvf openclaw-backup.tar.gz ~/.openclaw ~/.openclaw/workspace
```

***

<div id="see-also">
  ## Vedi anche
</div>

* [Accesso remoto al Gateway](/it/gateway/remote) — altri pattern di accesso remoto
* [Integrazione con Tailscale](/it/gateway/tailscale) — documentazione completa su Tailscale
* [Configurazione del Gateway](/it/gateway/configuration) — tutte le opzioni di configurazione
* [Guida a DigitalOcean](/it/platforms/digitalocean) — se vuoi una soluzione a pagamento con procedura di registrazione più semplice
* [Guida a Hetzner](/it/platforms/hetzner) — alternativa basata su Docker