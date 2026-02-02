---
title: Messaggi di gruppo
summary: "Comportamento e configurazione per la gestione dei messaggi di gruppo WhatsApp (i mentionPatterns sono condivisi tra le diverse superfici)"
read_when:
  - Modificare le regole per i messaggi di gruppo o le menzioni
---

<div id="group-messages-whatsapp-web-channel">
  # Messaggi di gruppo (canale web WhatsApp)
</div>

Obiettivo: consentire a Clawd di partecipare ai gruppi WhatsApp, attivarsi solo quando viene menzionato e mantenere quel thread separato dalla sessione DM privata.

Nota: `agents.list[].groupChat.mentionPatterns` è ora usato anche da Telegram/Discord/Slack/iMessage; questo documento si concentra sul comportamento specifico di WhatsApp. Per configurazioni multi-agente, imposta `agents.list[].groupChat.mentionPatterns` per ogni agente (oppure usa `messages.groupChat.mentionPatterns` come fallback globale).

<div id="whats-implemented-2025-12-03">
  ## Funzionalità implementate (2025-12-03)
</div>

- Modalità di attivazione: `mention` (predefinita) o `always`. `mention` richiede un ping (vere @-menzioni WhatsApp tramite `mentionedJids`, pattern regex o l’E.164 del bot ovunque nel testo). `always` sveglia l’agente a ogni messaggio ma dovrebbe rispondere solo quando può aggiungere valore significativo; altrimenti restituisce il token silenzioso `NO_REPLY`. I valori predefiniti possono essere impostati nella config (`channels.whatsapp.groups`) e sovrascritti per singolo gruppo tramite `/activation`. Quando `channels.whatsapp.groups` è impostato, funge anche da lista di autorizzati per i gruppi (includi `"*"` per consentire tutti).
- Policy di gruppo: `channels.whatsapp.groupPolicy` controlla se i messaggi di gruppo vengono accettati (`open|disabled|allowlist`). `allowlist` usa `channels.whatsapp.groupAllowFrom` (fallback: `channels.whatsapp.allowFrom` esplicito). Il valore predefinito è `allowlist` (tutto bloccato finché non aggiungi i mittenti).
- Sessioni per gruppo: le chiavi di sessione hanno la forma `agent:<agentId>:whatsapp:group:<jid>` quindi comandi come `/verbose on` o `/think high` (inviati come messaggi autonomi) sono limitati a quello specifico gruppo; lo stato dei DM personali non viene toccato. Gli heartbeat vengono saltati per i thread di gruppo.
- Iniezione del contesto: i messaggi di gruppo **pending-only** (predefinito 50) che *non* hanno attivato un’esecuzione vengono inseriti sotto `[Chat messages since your last reply - for context]`, con la riga che ha attivato l’esecuzione sotto `[Current message - respond to this]`. I messaggi già presenti nella sessione non vengono reiniettati.
- Evidenziazione del mittente: ogni batch di gruppo ora termina con `[from: Sender Name (+E164)]` così Pi sa chi sta parlando.
- Messaggi effimeri/visualizzazione singola: li scartiamo dall’involucro prima di estrarre testo/menzioni, quindi i ping al loro interno attivano comunque l’agente.
- System prompt di gruppo: alla prima interazione di una sessione di gruppo (e ogni volta che `/activation` cambia la modalità) iniettiamo un breve testo nel system prompt come `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Se i metadati non sono disponibili diciamo comunque all’agente che si tratta di una chat di gruppo.

<div id="config-example-whatsapp">
  ## Esempio di configurazione (WhatsApp)
</div>

Aggiungi un blocco `groupChat` a `~/.openclaw/openclaw.json` in modo che le menzioni per display-name funzionino anche quando WhatsApp rimuove la `@` visiva dal corpo del messaggio:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?openclaw",
            "\\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

Note:

* Le regex sono case-insensitive; coprono un ping sul nome visualizzato come `@openclaw` e il numero grezzo con o senza `+`/spazi.
* WhatsApp continua a inviare menzioni canoniche tramite `mentionedJids` quando qualcuno tocca il contatto, quindi il fallback sul numero è raramente necessario ma rappresenta un&#39;utile rete di sicurezza.


<div id="activation-command-owner-only">
  ### Comando di attivazione (solo proprietario)
</div>

Usa il comando nella chat di gruppo:

- `/activation mention`
- `/activation always`

Solo il numero del proprietario (da `channels.whatsapp.allowFrom`, o l'E.164 del bot se non impostato) può modificarlo. Invia `/status` come messaggio singolo nel gruppo per vedere la modalità di attivazione corrente.

<div id="how-to-use">
  ## Come utilizzarlo
</div>

1) Aggiungi il tuo account WhatsApp (quello su cui esegui OpenClaw) al gruppo.
2) Scrivi `@openclaw …` (oppure includi il numero). Solo i mittenti presenti nella lista di autorizzati possono attivarlo, a meno che tu non imposti `groupPolicy: "open"` (che consente di accettare messaggi senza restrizioni da qualsiasi utente).
3) Il prompt dell'agente includerà il contesto recente del gruppo più il marcatore finale `[from: …]`, così da poter rispondere alla persona corretta.
4) Le direttive a livello di sessione (`/verbose on`, `/think high`, `/new` o `/reset`, `/compact`) si applicano solo alla sessione di quel gruppo; inviale come messaggi separati affinché vengano registrate. La tua sessione DM personale rimane indipendente.

<div id="testing-verification">
  ## Test / verifica
</div>

- Smoke test manuale:
  - Invia un ping `@openclaw` nel gruppo e verifica che la risposta faccia riferimento al nome del mittente.
  - Invia un secondo ping e verifica che il blocco di cronologia della conversazione venga incluso e poi cancellato al turno successivo.
- Controlla i log del Gateway (eseguilo con `--verbose`) per vedere le voci `inbound web message` che mostrano `from: <groupJid>` e il suffisso `[from: …]`.

<div id="known-considerations">
  ## Considerazioni note
</div>

- Gli heartbeat vengono intenzionalmente ignorati per i gruppi per evitare broadcast rumorosi.
- La soppressione dell'eco usa la stringa batch combinata; se invii due volte di seguito lo stesso testo senza menzioni, solo il primo riceverà una risposta.
- Le voci nel session store appariranno come `agent:<agentId>:whatsapp:group:<jid>` nel session store (`~/.openclaw/agents/<agentId>/sessions/sessions.json` per impostazione predefinita); una voce mancante significa semplicemente che il gruppo non ha ancora avviato un'esecuzione.
- Gli indicatori di digitazione nei gruppi seguono `agents.defaults.typingMode` (valore predefinito: `message` quando non viene menzionato).