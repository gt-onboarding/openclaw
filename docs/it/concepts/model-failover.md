---
title: Failover dei modelli
summary: "Come OpenClaw ruota i profili di autenticazione e gestisce il failover tra i modelli"
read_when:
  - Diagnosi della rotazione dei profili di autenticazione, dei periodi di cooldown o del comportamento di failover dei modelli
  - Aggiornamento delle regole di failover per i profili di autenticazione o i modelli
---

<div id="model-failover">
  # Failover dei modelli
</div>

OpenClaw gestisce gli errori in due fasi:

1. **Rotazione del profilo di autenticazione** all&#39;interno del provider corrente.
2. **Fallback del modello** al modello successivo in `agents.defaults.model.fallbacks`.

Questo documento spiega le regole di runtime e i dati su cui si basano.

<div id="auth-storage-keys-oauth">
  ## Archiviazione dell&#39;autenticazione (chiavi + OAuth)
</div>

OpenClaw utilizza **profili di autenticazione** sia per le chiavi API che per i token OAuth.

* I segreti sono salvati in `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (legacy: `~/.openclaw/agent/auth-profiles.json`).
* Le voci di configurazione `auth.profiles` / `auth.order` contengono **solo metadati e regole di instradamento** (nessun segreto).
* File OAuth legacy solo per import: `~/.openclaw/credentials/oauth.json` (importato in `auth-profiles.json` al primo utilizzo).

Ulteriori dettagli: [/concepts/oauth](/it/concepts/oauth)

Tipi di credenziali:

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` per alcuni provider)

<div id="profile-ids">
  ## ID dei profili
</div>

Gli accessi OAuth creano profili distinti, affinché più account possano coesistere.

* Predefinito: `provider:default` quando non è disponibile alcuna email.
* OAuth con email: `provider:&lt;email&gt;` (per esempio `google-antigravity:user@gmail.com`).

I profili si trovano in `~/.openclaw/agents/&lt;agentId&gt;/agent/auth-profiles.json` sotto `profiles`.

<div id="rotation-order">
  ## Ordine di rotazione
</div>

Quando un provider ha più profili, OpenClaw sceglie un ordine come questo:

1. **Configurazione esplicita**: `auth.order[provider]` (se impostata).
2. **Profili configurati**: `auth.profiles` filtrati per provider.
3. **Profili memorizzati**: voci in `auth-profiles.json` per il provider.

Se non è configurato alcun ordine esplicito, OpenClaw usa un ordine round‑robin:

* **Chiave primaria:** tipo di profilo (**OAuth prima delle chiavi API**).
* **Chiave secondaria:** `usageStats.lastUsed` (il più vecchio per primo, all&#39;interno di ciascun tipo).
* I **profili in cooldown/disabilitati** vengono spostati alla fine, ordinati per scadenza più vicina.

<div id="session-stickiness-cache-friendly">
  ### Stickiness della sessione (cache-friendly)
</div>

OpenClaw **fissa il profilo di autenticazione scelto per ogni sessione** per mantenere calde le cache dei provider.
**Non** esegue una rotazione a ogni richiesta. Il profilo fissato viene riutilizzato fino a quando:

* la sessione viene reimpostata (`/new` / `/reset`)
* viene completata una compattazione (il contatore di compattazione viene incrementato)
* il profilo è in cooldown/disabilitato

La selezione manuale tramite `/model …@<profileId>` imposta un **override utente** per quella sessione
e il profilo non viene ruotato automaticamente finché non inizia una nuova sessione.

I profili fissati automaticamente (selezionati dal router di sessione) sono trattati come una **preferenza**:
vengono provati per primi, ma OpenClaw può passare a un altro profilo in caso di rate limit/timeout.
I profili fissati dall&#39;utente restano bloccati su quel profilo; se questo fallisce e sono configurati fallback di modello,
OpenClaw passa al modello successivo invece di cambiare profilo.

<div id="why-oauth-can-look-lost">
  ### Perché OAuth può “sembrare scomparso”
</div>

Se hai sia un profilo OAuth che un profilo con chiave API per lo stesso provider, il round‑robin può passare dall’uno all’altro tra i messaggi, a meno che non ne fissi uno. Per forzare un singolo profilo:

* Fissa (pin) con `auth.order[provider] = ["provider:profileId"]`, oppure
* Usa un override per sessione tramite `/model …` con un override di profilo (quando supportato dalla tua interfaccia UI/chat).

<div id="cooldowns">
  ## Intervalli di cooldown
</div>

Quando un profilo va in errore a causa di problemi di autenticazione o di rate‑limit (o di un timeout che sembra
un rate limit), OpenClaw lo contrassegna in cooldown e passa al profilo successivo.
Gli errori di formato/richiesta non valida (ad esempio i fallimenti di validazione degli ID di chiamata
dello strumento Cloud Code Assist) sono trattati come errori che richiedono il failover e usano gli stessi intervalli di cooldown.

Gli intervalli di cooldown usano un backoff esponenziale:

* 1 minuto
* 5 minuti
* 25 minuti
* 1 ora (limite massimo)

Lo stato viene memorizzato in `auth-profiles.json` sotto `usageStats`:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## Disattivazioni per fatturazione
</div>

I problemi di fatturazione/credito (ad esempio “crediti insufficienti” / “saldo crediti troppo basso”) sono trattati come condizioni che richiedono il failover, ma di solito non sono transitori. Invece di un breve periodo di cooldown, OpenClaw contrassegna il profilo come **disabilitato** (con un backoff più lungo) e passa al profilo/provider successivo.

Lo stato viene salvato in `auth-profiles.json`:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Impostazioni predefinite:

* Il backoff di fatturazione parte da **5 ore**, raddoppia a ogni errore di fatturazione ed è limitato a un massimo di **24 ore**.
* I contatori di backoff vengono azzerati se il profilo non ha avuto errori per **24 ore** (configurabile).

<div id="model-fallback">
  ## Fallback del modello
</div>

Se tutti i profili per un provider vanno in errore, OpenClaw passa al modello successivo in
`agents.defaults.model.fallbacks`. Questo vale per errori di autenticazione, limiti di frequenza (rate limit) e
timeout che hanno esaurito la rotazione dei profili (altri errori non fanno procedere al fallback).

Quando un&#39;esecuzione inizia con un override del modello (hook o CLI), la catena di fallback termina comunque con
`agents.defaults.model.primary` dopo aver provato tutti i fallback configurati.

<div id="related-config">
  ## Configurazioni correlate
</div>

Vedi [Configurazione del Gateway](/it/gateway/configuration) per:

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* instradamento di `agents.defaults.imageModel`

Vedi [Modelli](/it/concepts/models) per una panoramica più ampia sulla selezione dei modelli e sui meccanismi di failover.