---
title: Windows
summary: "Supporto per Windows (WSL2) + stato dell'app companion"
read_when:
  - Installazione di OpenClaw su Windows
  - Verifica dello stato dell'app companion per Windows
---

<div id="windows-wsl2">
  # Windows (WSL2)
</div>

L’uso di OpenClaw su Windows è consigliato **tramite WSL2** (si raccomanda Ubuntu). La
CLI + Gateway vengono eseguiti all&#39;interno di Linux, il che mantiene il runtime coerente e rende
gli strumenti/tooling molto più compatibili (Node/Bun/pnpm, binari Linux, abilità). Le installazioni
native su Windows non sono state testate e risultano più problematiche.

Sono previste app companion native per Windows.

<div id="install-wsl2">
  ## Installazione (WSL2)
</div>

* [Guida introduttiva](/it/start/getting-started) (da usare all&#39;interno di WSL)
* [Installazione e aggiornamenti](/it/install/updating)
* Guida ufficiale WSL2 (Microsoft): https://learn.microsoft.com/windows/wsl/install

<div id="gateway">
  ## Gateway
</div>

* [Runbook del Gateway](/it/gateway)
* [Configurazione](/it/gateway/configuration)

<div id="gateway-service-install-cli">
  ## Installazione del servizio Gateway (CLI)
</div>

In WSL2:

```
openclaw onboard --install-daemon
```

In alternativa:

```
openclaw gateway install
```

In alternativa:

```
openclaw configure
```

Seleziona **Gateway service** quando richiesto.

Riparazione/migrazione:

```
openclaw doctor
```

<div id="advanced-expose-wsl-services-over-lan-portproxy">
  ## Avanzato: esporre i servizi WSL sulla LAN (portproxy)
</div>

WSL ha una propria rete virtuale. Se un&#39;altra macchina deve raggiungere un servizio
in esecuzione **all&#39;interno di WSL** (SSH, un server TTS locale o il Gateway), devi
inoltrare una porta di Windows all&#39;IP WSL corrente. L&#39;IP WSL cambia dopo i riavvii,
quindi potresti dover aggiornare la regola di inoltro.

Esempio (PowerShell **come amministratore**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "IP WSL non trovato." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Consenti questa porta nel firewall di Windows (una sola volta):

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Aggiorna il portproxy dopo aver riavviato WSL:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Note:

* Una connessione SSH da un&#39;altra macchina punta all&#39;**IP dell&#39;host Windows** (esempio: `ssh user@windows-host -p 2222`).
* I nodi remoti devono puntare a un URL del Gateway **raggiungibile** (non `127.0.0.1`); usa
  `openclaw status --all` per confermare.
* Usa `listenaddress=0.0.0.0` per l&#39;accesso LAN; `127.0.0.1` limita l’accesso alla sola macchina locale.
* Se vuoi che questo sia automatico, registra un&#39;attività pianificata (Scheduled Task) per eseguire il passaggio di refresh all&#39;accesso.

<div id="step-by-step-wsl2-install">
  ## Installazione di WSL2 passo per passo
</div>

<div id="1-install-wsl2-ubuntu">
  ### 1) Installa WSL2 + Ubuntu
</div>

Apri PowerShell come amministratore:

```powershell
wsl --install
# Oppure scegli una distribuzione specifica:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Riavvia il sistema se Windows lo richiede.

<div id="2-enable-systemd-required-for-gateway-install">
  ### 2) Abilita systemd (necessario per l&#39;installazione del Gateway)
</div>

Nel terminale WSL:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Quindi, da PowerShell:

```powershell
wsl --shutdown
```

Apri di nuovo Ubuntu e verifica:

```bash
systemctl --user status
```

<div id="3-install-openclaw-inside-wsl">
  ### 3) Installa OpenClaw (all&#39;interno di WSL)
</div>

Segui la procedura &quot;Linux Getting Started&quot; all&#39;interno di WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # installa automaticamente le dipendenze della UI alla prima esecuzione
pnpm build
openclaw onboard
```

Guida completa: [Per iniziare](/it/start/getting-started)

<div id="windows-companion-app">
  ## App companion per Windows
</div>

Non abbiamo ancora un&#39;app companion per Windows. I contributi sono benvenuti se vuoi aiutarci a renderla realtà.