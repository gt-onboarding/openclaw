---
title: Raspberry Pi
summary: "OpenClaw su Raspberry Pi (configurazione self-hosted economica)"
read_when:
  - Configurare OpenClaw su un Raspberry Pi
  - Eseguire OpenClaw su dispositivi ARM
  - Creare un'IA personale economica sempre attiva
---

<div id="openclaw-on-raspberry-pi">
  # OpenClaw su Raspberry Pi
</div>

<div id="goal">
  ## Obiettivo
</div>

Eseguire in modo persistente e sempre attivo un Gateway OpenClaw su un Raspberry Pi con un costo una tantum di **~$35-80** (nessun canone mensile).

Ideale per:

* Assistente IA personale 24/7
* Hub per l&#39;automazione domestica
* Bot Telegram/WhatsApp a basso consumo e sempre disponibile

<div id="hardware-requirements">
  ## Requisiti hardware
</div>

| Modello Pi | RAM | Funziona? | Note |
|------------|-----|----------|------|
| **Pi 5** | 4GB/8GB | ✅ Migliore | Il più veloce, consigliato |
| **Pi 4** | 4GB | ✅ Buono | Compromesso ideale per la maggior parte degli utenti |
| **Pi 4** | 2GB | ✅ OK | Funziona, aggiungi swap |
| **Pi 4** | 1GB | ⚠️ Al limite | Possibile con swap, configurazione minima |
| **Pi 3B+** | 1GB | ⚠️ Lento | Funziona, ma è piuttosto lento |
| **Pi Zero 2 W** | 512MB | ❌ | Non consigliato |

**Specifiche minime:** 1GB RAM, 1 core, 500MB di spazio su disco\
**Consigliato:** 2GB+ RAM, sistema operativo a 64 bit, scheda SD da 16GB+ (o SSD USB)

<div id="what-youll-need">
  ## Cosa ti serve
</div>

* Raspberry Pi 4 o 5 (consigliati almeno 2 GB di RAM)
* Scheda microSD (16 GB+) o SSD USB (prestazioni migliori)
* Alimentatore (consigliato l’alimentatore ufficiale per Pi)
* Connessione di rete (Ethernet o WiFi)
* ~30 minuti

<div id="1-flash-the-os">
  ## 1) Scrivere il sistema operativo (OS)
</div>

Usa **Raspberry Pi OS Lite (64-bit)** — non serve l&#39;ambiente desktop per un server headless.

1. Scarica [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Scegli come OS: **Raspberry Pi OS Lite (64-bit)**
3. Fai clic sull&#39;icona a forma di ingranaggio (⚙️) per preconfigurare:
   * Imposta l&#39;hostname: `gateway-host`
   * Abilita SSH
   * Imposta nome utente/password
   * Configura il WiFi (se non usi Ethernet)
4. Scrivi l&#39;immagine sulla scheda SD / sull&#39;unità USB
5. Inserisci la scheda/unità e avvia il Pi

<div id="2-connect-via-ssh">
  ## 2) Connettersi tramite SSH
</div>

```bash
ssh user@gateway-host
# oppure utilizza l'indirizzo IP
ssh user@192.168.x.x
```

<div id="3-system-setup">
  ## 3) Configurazione del sistema
</div>

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y git curl build-essential

# Imposta il fuso orario (importante per cron/promemoria)
sudo timedatectl set-timezone America/Chicago  # Sostituisci con il tuo fuso orario
```

<div id="4-install-nodejs-22-arm64">
  ## 4) Installa Node.js 22 (ARM64)
</div>

```bash
# Install Node.js via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version  # Dovrebbe mostrare v22.x.x
npm --version
```

<div id="5-add-swap-important-for-2gb-or-less">
  ## 5) Aggiungi swap (importante con 2 GB di RAM o meno)
</div>

Lo swap evita crash per esaurimento della memoria:

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Ottimizza per RAM ridotta (riduci swappiness)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

<div id="6-install-openclaw">
  ## 6) Installa OpenClaw
</div>

<div id="option-a-standard-install-recommended">
  ### Opzione A: Installazione standard (consigliata)
</div>

```bash
curl -fsSL https://openclaw.bot/install.sh | bash
```

<div id="option-b-hackable-install-for-tinkering">
  ### Opzione B: Installazione hackable (per smanettare)
</div>

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
npm install
npm run build
npm link
```

L&#39;installazione hackable ti offre accesso diretto ai log e al codice sorgente — utile per il debug di problemi specifici di ARM.

<div id="7-run-onboarding">
  ## 7) Esegui la procedura di onboarding
</div>

```bash
openclaw onboard --install-daemon
```

Segui la procedura guidata:

1. **Modalità Gateway:** Locale
2. **Autenticazione:** Chiavi API consigliate (OAuth può dare problemi su un Pi headless)
3. **Canali:** Telegram è il più semplice da cui partire
4. **Daemon:** Sì (systemd)

<div id="8-verify-installation">
  ## 8) Verifica l&#39;installazione
</div>

```bash
# Check status
openclaw status

# Controlla il servizio
sudo systemctl status openclaw

# View logs
journalctl -u openclaw -f
```

<div id="9-access-the-dashboard">
  ## 9) Accedi alla dashboard
</div>

Poiché il Pi è headless, usa un tunnel SSH:

```bash
# From your laptop/desktop
ssh -L 18789:localhost:18789 user@gateway-host

# Then open in browser
open http://localhost:18789
```

Oppure usa Tailscale per avere un accesso sempre disponibile:

```bash
# On the Pi
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Aggiorna la configurazione
openclaw config set gateway.bind tailnet
sudo systemctl restart openclaw
```

***

<div id="performance-optimizations">
  ## Ottimizzazioni delle prestazioni
</div>

<div id="use-a-usb-ssd-huge-improvement">
  ### Utilizza un SSD USB (miglioramento enorme)
</div>

Le schede SD sono lente e soggette a usura. Un SSD USB migliora in modo drastico le prestazioni:

```bash
# Verifica se l'avvio avviene da USB
lsblk
```

Vedi la [guida all&#39;avvio da USB per Raspberry Pi](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#usb-mass-storage-boot) per la configurazione.

<div id="reduce-memory-usage">
  ### Ridurre l&#39;utilizzo della memoria
</div>

```bash
# Disabilita l'allocazione della memoria GPU (headless)
echo 'gpu_mem=16' | sudo tee -a /boot/config.txt

# Disable Bluetooth if not needed
sudo systemctl disable bluetooth
```

<div id="monitor-resources">
  ### Monitoraggio delle risorse
</div>

```bash
# Verifica memoria
free -h

# Verifica temperatura CPU
vcgencmd measure_temp

# Monitoraggio live
htop
```

***

<div id="arm-specific-notes">
  ## Note specifiche per ARM
</div>

<div id="binary-compatibility">
  ### Compatibilità binaria
</div>

La maggior parte delle funzionalità di OpenClaw funziona su ARM64, ma alcuni binari esterni potrebbero richiedere build per ARM:

| Tool | Stato ARM64 | Note |
|------|--------------|-------|
| Node.js | ✅ | Funziona molto bene |
| WhatsApp (Baileys) | ✅ | JS puro, nessun problema |
| Telegram | ✅ | JS puro, nessun problema |
| gog (Gmail CLI) | ⚠️ | Verifica se è disponibile una release ARM |
| Chromium (browser) | ✅ | `sudo apt install chromium-browser` |

Se una skill va in errore, verifica se il relativo binario dispone di una build per ARM. Molti tool in Go/Rust la forniscono; alcuni no.

<div id="32-bit-vs-64-bit">
  ### 32-bit vs 64-bit
</div>

**Usa sempre un sistema operativo a 64 bit.** Node.js e molti strumenti moderni lo richiedono. Verifica con:

```bash
uname -m
# Dovrebbe mostrare: aarch64 (64-bit) e non armv7l (32-bit)
```

***

<div id="recommended-model-setup">
  ## Configurazione consigliata dei modelli
</div>

Poiché il Pi funge solo da Gateway (i modelli vengono eseguiti nel cloud), utilizza modelli basati su api:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514",
        "fallbacks": ["openai/gpt-4o-mini"]
      }
    }
  }
}
```

**Non cercare di eseguire LLM locali su un Pi** — anche i modelli più piccoli sono troppo lenti. Lascia che Claude/GPT si occupino del lavoro pesante.

***

<div id="auto-start-on-boot">
  ## Avvio automatico all&#39;avvio del sistema
</div>

La procedura guidata di onboarding lo imposta già, ma per verificarlo:

```bash
# Verifica che il servizio sia abilitato
sudo systemctl is-enabled openclaw

# Abilita se non è abilitato
sudo systemctl enable openclaw

# Avvia all'avvio del sistema
sudo systemctl start openclaw
```

***

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="out-of-memory-oom">
  ### Memoria esaurita (OOM)
</div>

```bash
# Controlla la memoria
free -h

# Aggiungi più swap (vedi Passaggio 5)
# Oppure riduci i servizi in esecuzione sul Pi
```

<div id="slow-performance">
  ### Prestazioni lente
</div>

* Usa un SSD USB invece di una scheda SD
* Disabilita i servizi non utilizzati: `sudo systemctl disable cups bluetooth avahi-daemon`
* Verifica il throttling della CPU: `vcgencmd get_throttled` (dovrebbe restituire `0x0`)

<div id="service-wont-start">
  ### Il servizio non si avvia
</div>

```bash
# Check logs
journalctl -u openclaw --no-pager -n 100

# Common fix: rebuild
cd ~/openclaw  # se si usa un'installazione hackable
npm run build
sudo systemctl restart openclaw
```

<div id="arm-binary-issues">
  ### Problemi con i binari ARM
</div>

Se una skill restituisce &quot;exec format error&quot;:

1. Verifica che il binario sia compilato per ARM64
2. Prova a compilarlo dai sorgenti
3. In alternativa, usa un container Docker con supporto ARM

<div id="wifi-drops">
  ### Disconnessioni Wi‑Fi
</div>

Per i Raspberry Pi headless su Wi‑Fi:

```bash
# Disabilita il risparmio energetico WiFi
sudo iwconfig wlan0 power off

# Make permanent
echo 'wireless-power off' | sudo tee -a /etc/network/interfaces
```

***

<div id="cost-comparison">
  ## Confronto dei costi
</div>

| Setup | Costo iniziale | Costo mensile | Note |
|-------|----------------|---------------|------|
| **Pi 4 (2GB)** | ~$45 | $0 | + corrente (~$5/anno) |
| **Pi 4 (4GB)** | ~$55 | $0 | Consigliato |
| **Pi 5 (4GB)** | ~$60 | $0 | Prestazioni migliori |
| **Pi 5 (8GB)** | ~$80 | $0 | Sovradimensionato ma a prova di futuro |
| DigitalOcean | $0 | $6/mese | $72/anno |
| Hetzner | $0 | €3,79/mese | ~$50/anno |

**Punto di pareggio:** un Pi si ripaga da solo in circa 6–12 mesi rispetto a un VPS cloud.

***

<div id="see-also">
  ## Vedi anche
</div>

* [Guida Linux](/it/platforms/linux) — configurazione generale di Linux
* [Guida DigitalOcean](/it/platforms/digitalocean) — alternativa cloud
* [Guida Hetzner](/it/platforms/hetzner) — configurazione Docker
* [Tailscale](/it/gateway/tailscale) — accesso remoto
* [Nodi](/it/nodes) — associa il tuo laptop o telefono al Gateway su Raspberry Pi