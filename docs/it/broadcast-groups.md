---
title: Gruppi broadcast
summary: "Invia un messaggio WhatsApp in broadcast a più agenti contemporaneamente"
read_when:
  - Configurare i gruppi broadcast
  - Eseguire il debug delle risposte multi-agente su WhatsApp
status: experimental
---

<div id="broadcast-groups">
  # Gruppi broadcast
</div>

**Stato:** Sperimentale\
**Versione:** Aggiunto nella versione 2026.1.9

<div id="overview">
  ## Panoramica
</div>

I Broadcast Groups consentono a più agenti di elaborare e rispondere allo stesso messaggio simultaneamente. Questo ti permette di creare team di agenti specializzati che lavorano insieme in un singolo gruppo WhatsApp o in DM, tutti usando un unico numero di telefono.

Ambito attuale: **solo WhatsApp** (canale web).

I Broadcast Groups vengono valutati dopo la lista di autorizzati del canale e le regole di attivazione dei gruppi. Nei gruppi WhatsApp, questo significa che i broadcast avvengono quando OpenClaw normalmente risponderebbe (per esempio: in caso di menzione, a seconda delle impostazioni del gruppo).

<div id="use-cases">
  ## Casi d&#39;uso
</div>

<div id="1-specialized-agent-teams">
  ### 1. Team di agenti specializzati
</div>

Distribuisci più agenti con responsabilità ben definite e mirate:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Ogni agente elabora lo stesso messaggio e offre il proprio punto di vista specializzato.

<div id="2-multi-language-support">
  ### 2. Supporto multilingue
</div>

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

<div id="3-quality-assurance-workflows">
  ### 3. Flussi di lavoro per la Quality Assurance
</div>

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

<div id="4-task-automation">
  ### 4. Automazione delle attività
</div>

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

<div id="configuration">
  ## Configurazione
</div>

<div id="basic-setup">
  ### Configurazione di base
</div>

Aggiungi una sezione `broadcast` di primo livello (accanto a `bindings`). Le chiavi sono gli ID dei peer WhatsApp:

* chat di gruppo: JID del gruppo (ad es. `120363403215116621@g.us`)
* messaggi diretti (DM): numero di telefono in formato E.164 (ad es. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Risultato:** quando OpenClaw deve rispondere in questa chat, esegue tutti e tre gli agenti.

<div id="processing-strategy">
  ### Strategia di elaborazione
</div>

Configura come gli agenti elaborano i messaggi:

<div id="parallel-default">
  #### Parallelo (predefinito)
</div>

Tutti gli agenti elaborano contemporaneamente:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="sequential">
  #### Sequenziale
</div>

Gli agenti vengono eseguiti uno dopo l&#39;altro (ognuno aspetta che il precedente abbia terminato):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

<div id="complete-example">
  ### Esempio completo
</div>

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

<div id="how-it-works">
  ## Funzionamento
</div>

<div id="message-flow">
  ### Flusso dei messaggi
</div>

1. **Messaggio in arrivo** in un gruppo WhatsApp
2. **Verifica del broadcast**: il sistema controlla se l&#39;ID del peer è presente in `broadcast`
3. **Se è nella lista broadcast**:
   * Tutti gli agenti elencati elaborano il messaggio
   * Ogni agente ha la propria chiave di sessione e un contesto isolato
   * Gli agenti elaborano in parallelo (impostazione predefinita) o in sequenza
4. **Se non è nella lista broadcast**:
   * Si applica il normale instradamento (prima associazione corrispondente)

Nota: i gruppi broadcast non ignorano le liste di autorizzati del canale o le regole di attivazione del gruppo (menzioni/comandi/ecc.). Modificano solo *quali agenti vengono eseguiti* quando un messaggio è idoneo all&#39;elaborazione.

<div id="session-isolation">
  ### Isolamento delle sessioni
</div>

Ogni agente in un gruppo di broadcast mantiene completamente separati:

* **Chiavi di sessione** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
* **Cronologia delle conversazioni** (l&#39;agente non vede i messaggi degli altri agenti)
* **Spazio di lavoro** (sandbox separate se configurate)
* **Accesso agli strumenti** (elenchi di allow/deny diversi)
* **Memoria/contesto** (IDENTITY.md, SOUL.md, ecc. separati)
* **Buffer di contesto del gruppo** (i messaggi recenti del gruppo usati per il contesto) è condiviso per singolo peer, quindi tutti gli agenti di broadcast vedono lo stesso contesto quando vengono attivati

Questo consente a ciascun agente di avere:

* Personalità diverse
* Accesso agli strumenti differente (ad es. sola lettura vs lettura/scrittura)
* Modelli diversi (ad es. opus vs sonnet)
* Abilità installate diverse

<div id="example-isolated-sessions">
  ### Esempio: sessioni isolate
</div>

Nel gruppo `120363403215116621@g.us` con gli agenti `["alfred", "baerbel"]`:

**Contesto di Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [messaggio utente, risposte precedenti di alfred]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Contesto di Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [messaggio utente, risposte precedenti di baerbel]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

<div id="best-practices">
  ## Best practice
</div>

<div id="1-keep-agents-focused">
  ### 1. Mantieni gli Agenti concentrati
</div>

Progetta ogni agente con una singola responsabilità ben definita:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Consigliato:** Ogni agente ha un solo compito
❌ **Sconsigliato:** Un unico agente generico &quot;dev-helper&quot;

<div id="2-use-descriptive-names">
  ### 2. Usa nomi descrittivi
</div>

Rendi chiaro che cosa fa ogni agente:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

<div id="3-configure-different-tool-access">
  ### 3. Configura diversi livelli di accesso agli strumenti
</div>

Concedi agli agenti solo gli strumenti di cui hanno bisogno:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // Lettura e scrittura
    }
  }
}
```

<div id="4-monitor-performance">
  ### 4. Monitora le prestazioni
</div>

Con molti agenti, valuta di:

* usare `"strategy": "parallel"` (predefinito) per aumentare la velocità
* limitare i gruppi di broadcast a 5-10 agenti
* usare modelli più veloci per gli agenti più semplici

<div id="5-handle-failures-gracefully">
  ### 5. Gestisci i guasti in modo elegante
</div>

Gli agenti possono fallire indipendentemente l’uno dall’altro. L’errore di un agente non blocca gli altri:

```
Messaggio → [Agente A ✓, Agente B ✗ errore, Agente C ✓]
Risultato: gli Agenti A e C rispondono, l'Agente B registra l'errore
```

<div id="compatibility">
  ## Compatibilità
</div>

<div id="providers">
  ### Provider
</div>

I gruppi broadcast sono attualmente compatibili con:

* ✅ WhatsApp (già implementato)
* 🚧 Telegram (pianificato)
* 🚧 Discord (pianificato)
* 🚧 Slack (pianificato)

<div id="routing">
  ### Instradamento
</div>

I gruppi di broadcast operano insieme all&#39;instradamento esistente:

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

* `GROUP_A`: risponde solo Alfred (instradamento normale)
* `GROUP_B`: rispondono sia agent1 che agent2 (broadcast)

**Precedenza:** `broadcast` ha priorità su `bindings`.

<div id="troubleshooting">
  ## Risoluzione dei problemi
</div>

<div id="agents-not-responding">
  ### Agenti che non rispondono
</div>

**Verifica:**

1. Gli ID degli agenti esistono in `agents.list`
2. Il formato dell&#39;ID peer è corretto (ad esempio `120363403215116621@g.us`)
3. Gli agenti non sono presenti in elenchi di blocco (deny list)

**Diagnostica:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

<div id="only-one-agent-responding">
  ### Solo un agente sta rispondendo
</div>

**Causa:** L&#39;ID del peer potrebbe essere in `bindings` ma non in `broadcast`.

**Soluzione:** Aggiungi l&#39;ID del peer alla configurazione di `broadcast` oppure rimuovilo da `bindings`.

<div id="performance-issues">
  ### Problemi di prestazioni
</div>

**Se noti lentezza con molti agenti:**

* Riduci il numero di agenti per gruppo
* Usa modelli più leggeri (sonnet invece di opus)
* Controlla i tempi di avvio della sandbox

<div id="examples">
  ## Esempi
</div>

<div id="example-1-code-review-team">
  ### Esempio 1: Team di revisione del codice
</div>

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**L&#39;utente invia:** frammento di codice
**Risposte:**

* code-formatter: &quot;Indentazione corretta e aggiunte annotazioni di tipo&quot;
* security-scanner: &quot;⚠️ Vulnerabilità di SQL injection alla riga 12&quot;
* test-coverage: &quot;La copertura dei test è al 45%, mancano test per i casi di errore&quot;
* docs-checker: &quot;Docstring mancante per la funzione `process_data`&quot;

<div id="example-2-multi-language-support">
  ### Esempio 2: Supporto multilingue
</div>

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

<div id="api-reference">
  ## Riferimento API
</div>

<div id="config-schema">
  ### Schema di configurazione
</div>

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

<div id="fields">
  ### Campi
</div>

* `strategy` (opzionale): Come elaborare gli agenti
  * `"parallel"` (predefinito): Tutti gli agenti elaborano simultaneamente
  * `"sequential"`: Gli agenti elaborano nell&#39;ordine dell&#39;array

* `[peerId]`: JID del gruppo WhatsApp, numero in formato E.164 o altro peer ID
  * Valore: Array di ID di agenti che devono elaborare i messaggi

<div id="limitations">
  ## Limitazioni
</div>

1. **Numero massimo di agenti:** Nessun limite fisso, ma con più di 10 agenti le prestazioni possono rallentare
2. **Contesto condiviso:** Gli agenti non possono vedere le risposte degli altri (per scelta progettuale)
3. **Ordine dei messaggi:** Le risposte in parallelo possono arrivare in qualsiasi ordine
4. **Rate limits:** Tutti gli agenti concorrono ai limiti di frequenza imposti da WhatsApp

<div id="future-enhancements">
  ## Miglioramenti futuri
</div>

Funzionalità previste:

* [ ] Modalità contesto condiviso (gli agenti vedono le risposte degli altri)
* [ ] Coordinamento tra agenti (gli agenti possono inviarsi segnali tra loro)
* [ ] Selezione dinamica degli agenti (scegli gli agenti in base al contenuto dei messaggi)
* [ ] Priorità degli agenti (alcuni agenti rispondono prima di altri)

<div id="see-also">
  ## Vedi anche
</div>

* [Configurazione multi-agente](/it/multi-agent-sandbox-tools)
* [Configurazione del routing](/it/concepts/channel-routing)
* [Gestione delle sessioni](/it/concepts/sessions)