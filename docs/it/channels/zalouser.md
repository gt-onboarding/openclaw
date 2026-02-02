---
title: Zalouser
summary: "Supporto per account personale Zalo tramite zca-cli (accesso via QR), funzionalità e configurazione"
read_when:
  - Durante la configurazione di Zalo Personal per OpenClaw
  - Per eseguire il debug dell'accesso o del flusso di messaggi di Zalo Personal
---

<div id="zalo-personal-unofficial">
  # Zalo Personal (non ufficiale)
</div>

Stato: sperimentale. Questa integrazione automatizza un **account Zalo personale** tramite `zca-cli`.

> **Avvertenza:** si tratta di un&#39;integrazione non ufficiale e potrebbe comportare la sospensione o il blocco dell&#39;account. Utilizzala a tuo rischio e pericolo.

<div id="plugin-required">
  ## Plugin richiesto
</div>

Zalo Personal viene distribuito come plugin e non è incluso nell&#39;installazione principale.

* Installa tramite CLI: `openclaw plugins install @openclaw/zalouser`
* Oppure dal codice sorgente: `openclaw plugins install ./extensions/zalouser`
* Dettagli: [Plugin](/it/plugin)

<div id="prerequisite-zca-cli">
  ## Prerequisito: zca-cli
</div>

La macchina che esegue il Gateway deve avere il binario `zca` disponibile nel `PATH`.

* Verifica con: `zca --version`
* Se non è presente, installa zca-cli (vedi `extensions/zalouser/README.md` o la documentazione ufficiale di zca-cli).

<div id="quick-setup-beginner">
  ## Configurazione rapida (per principianti)
</div>

1. Installa il plugin (vedi sopra).
2. Effettua l’accesso (QR, sulla macchina del Gateway):
   * `openclaw channels login --channel zalouser`
   * Scansiona il codice QR nel terminale con l’app mobile Zalo.
3. Abilita il canale:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "abbinamento"
    }
  }
}
```

4. Riavvia il Gateway (o completa l&#39;onboarding).
5. L&#39;accesso tramite DM richiede per impostazione predefinita l&#39;abbinamento; approva il codice di abbinamento al primo contatto.

<div id="what-it-is">
  ## Che cos&#39;è
</div>

* Usa `zca listen` per ricevere i messaggi in ingresso.
* Usa `zca msg ...` per inviare risposte (testo/media/link).
* Progettato per casi d&#39;uso con “account personale” in cui la Zalo Bot API non è disponibile.

<div id="naming">
  ## Denominazione
</div>

L&#39;ID del canale è `zalouser` per rendere esplicito che serve ad automatizzare un **account utente personale Zalo** (non ufficiale). Manteniamo `zalo` riservato per una potenziale futura integrazione ufficiale con le api Zalo.

<div id="finding-ids-directory">
  ## Trovare gli ID (directory)
</div>

Usa la CLI directory per individuare peer e gruppi e i relativi ID:

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

<div id="limits">
  ## Limiti
</div>

* Il testo in uscita è suddiviso in segmenti di circa 2000 caratteri (limiti del client Zalo).
* Lo streaming è bloccato per impostazione predefinita.

<div id="access-control-dms">
  ## Controllo degli accessi (messaggi diretti)
</div>

`channels.zalouser.dmPolicy` supporta: `pairing | allowlist | open | disabled` (predefinito: `pairing`; `open` indica che i messaggi sono accettati senza restrizioni da qualsiasi utente).
`channels.zalouser.allowFrom` accetta ID utente o nomi. La procedura guidata risolve i nomi in ID tramite `zca friend find` quando disponibile.

Approva tramite:

* `openclaw pairing list zalouser`
* `openclaw pairing approve zalouser <code>`

<div id="group-access-optional">
  ## Accesso ai gruppi (opzionale)
</div>

* Predefinito: `channels.zalouser.groupPolicy = "open"` (gruppi consentiti; l&#39;impostazione `open` permette l&#39;accettazione illimitata di messaggi da qualsiasi utente). Usa `channels.defaults.groupPolicy` per sovrascrivere il valore predefinito quando non è impostato.
* Limita a una lista di autorizzati con:
  * `channels.zalouser.groupPolicy = "allowlist"`
  * `channels.zalouser.groups` (le chiavi sono ID o nomi dei gruppi)
* Blocca tutti i gruppi: `channels.zalouser.groupPolicy = "disabled"`.
* La procedura guidata di configurazione può chiedere le liste di autorizzati per i gruppi.
* All&#39;avvio, OpenClaw risolve i nomi dei gruppi/utenti nelle liste di autorizzati in ID e registra l&#39;associazione; le voci non risolte vengono mantenute così come sono state digitate.

Esempio:

```json5
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true }
      }
    }
  }
}
```

<div id="multi-account">
  ## Multi-account
</div>

Gli account sono associati ai profili zca. Esempio:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" }
      }
    }
  }
}
```

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

**`zca` non trovato:**

* Installa zca-cli e assicurati che sia nel `PATH` per il processo Gateway.

**L’accesso non rimane attivo:**

* `openclaw channels status --probe`
* Esegui di nuovo il login: `openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`