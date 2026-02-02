---
title: Refactoring del mirroring delle sessioni in uscita (Issue #1520)
description: Monitora note, decisioni, test e punti aperti relativi al refactoring del mirroring delle sessioni in uscita.
---

<div id="outbound-session-mirroring-refactor-issue-1520">
  # Refactoring del mirroring della sessione in uscita (Issue #1520)
</div>

<div id="status">
  ## Stato
</div>

- In corso.
- Routing dei canali core e plugin aggiornato per il mirroring in uscita.
- Gateway send ora determina la sessione di destinazione quando `sessionKey` è omesso.

<div id="context">
  ## Contesto
</div>

Gli invii in uscita venivano replicati nella sessione *corrente* dell'agente (chiave di sessione dello strumento) invece che nella sessione del canale di destinazione. L'instradamento in ingresso usa le chiavi di sessione del canale/peer, quindi le risposte in uscita finivano nella sessione sbagliata e i destinatari al primo contatto spesso non avevano alcuna voce di sessione.

<div id="goals">
  ## Obiettivi
</div>

- Replicare i messaggi in uscita nella chiave di sessione del canale di destinazione.
- Creare le voci di sessione in uscita quando non esistono.
- Mantenere l'ambito thread/argomento allineato con le chiavi di sessione in ingresso.
- Coprire i canali principali e le estensioni incluse.

<div id="implementation-summary">
  ## Riepilogo dell'implementazione
</div>

- Nuovo helper per il routing delle sessioni in uscita:
  - `src/infra/outbound/outbound-session.ts`
  - `resolveOutboundSessionRoute` costruisce il sessionKey di destinazione usando `buildAgentSessionKey` (dmScope + identityLinks).
  - `ensureOutboundSessionEntry` scrive un `MsgContext` minimale tramite `recordSessionMetaFromInbound`.
- `runMessageAction` (Invia) deriva il sessionKey di destinazione e lo passa a `executeSendAction` per il mirroring.
- `message-tool` non effettua più il mirroring direttamente; risolve solo l'agentId a partire dal sessionKey corrente.
- Il percorso di invio del plugin effettua il mirroring tramite `appendAssistantMessageToSessionTranscript` usando il sessionKey derivato.
- L'invio dal Gateway deriva un sessionKey di destinazione quando non ne viene fornito uno (agente predefinito) e garantisce l'esistenza di una voce di sessione.

<div id="threadtopic-handling">
  ## Gestione di thread/topic
</div>

- Slack: replyTo/threadId -> `resolveThreadSessionKeys` (suffisso).
- Discord: threadId/replyTo -> `resolveThreadSessionKeys` con `useSuffix=false` per corrispondere ai messaggi in ingresso (l'ID del canale del thread delimita già la sessione).
- Telegram: gli ID dei topic vengono mappati in `chatId:topic:<id>` tramite `buildTelegramGroupPeerId`.

<div id="extensions-covered">
  ## Estensioni coperte
</div>

- Matrix, MS Teams, Mattermost, BlueBubbles, Nextcloud Talk, Zalo, Zalo Personal, Nostr, Tlon.
- Note:
  - I target Mattermost ora rimuovono `@` per il routing della chiave di sessione DM.
  - Zalo Personal utilizza il tipo di peer DM per i target 1:1 (solo come gruppo quando è presente `group:`).
  - I target di gruppo BlueBubbles rimuovono i prefissi `chat_*` per allineare le chiavi di sessione in ingresso.
  - Il mirroring automatico dei thread Slack confronta gli ID dei canali senza distinzione tra maiuscole e minuscole.
  - Gateway send converte in minuscolo le chiavi di sessione fornite prima del mirroring.

<div id="decisions">
  ## Decisioni
</div>

- **Derivazione della sessione per l'operazione Gateway send**: se `sessionKey` è fornita, usala. Se omessa, deriva una sessionKey da target + agente predefinito ed esegui lì il mirroring.
- **Creazione della voce di sessione**: usa sempre `recordSessionMetaFromInbound` con `Provider/From/To/ChatType/AccountId/Originating*` allineati ai formati in ingresso.
- **Normalizzazione del target**: il routing in uscita usa i target risolti (dopo `resolveChannelTarget`) quando disponibili.
- **Normalizzazione delle chiavi di sessione**: imposta le chiavi di sessione in minuscolo in fase di scrittura e durante le migrazioni.

<div id="tests-addedupdated">
  ## Test aggiunti/aggiornati
</div>

- `src/infra/outbound/outbound-session.test.ts`
  - Chiave di sessione del thread Slack.
  - Chiave di sessione del topic di Telegram.
  - dmScope identityLinks con Discord.
- `src/agents/tools/message-tool.test.ts`
  - Deriva agentId dalla chiave di sessione (nessun sessionKey passato).
- `src/gateway/server-methods/send.test.ts`
  - Deriva la chiave di sessione se omessa e crea una voce di sessione.

<div id="open-items-follow-ups">
  ## Punti aperti / Azioni successive
</div>

- Il plugin di chiamata vocale utilizza chiavi di sessione personalizzate `voice:<phone>`. La mappatura in uscita non è standardizzata qui; se `message-tool` deve supportare invii di chiamate vocali, aggiungi una mappatura esplicita.
- Conferma se qualche plugin esterno utilizza formati `From/To` non standard oltre al set predefinito fornito a corredo.

<div id="files-touched">
  ## File interessati
</div>

- `src/infra/outbound/outbound-session.ts`
- `src/infra/outbound/outbound-send-service.ts`
- `src/infra/outbound/message-action-runner.ts`
- `src/agents/tools/message-tool.ts`
- `src/gateway/server-methods/send.ts`
- Test in:
  - `src/infra/outbound/outbound-session.test.ts`
  - `src/agents/tools/message-tool.test.ts`
  - `src/gateway/server-methods/send.test.ts`