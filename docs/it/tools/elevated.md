---
title: Elevato
summary: "Modalità di esecuzione elevata e direttive /elevated"
read_when:
  - Regolare i valori predefiniti della modalità elevata, le liste di autorizzati o il comportamento dei comandi slash
---

<div id="elevated-mode-elevated-directives">
  # Modalità con privilegi elevati (direttive /elevated)
</div>

<div id="what-it-does">
  ## Cosa fa
</div>

- `/elevated on` viene eseguito sull'host del Gateway e mantiene le approvazioni per exec (come `/elevated ask`).
- `/elevated full` viene eseguito sull'host del Gateway **e** approva automaticamente exec (salta le approvazioni per exec).
- `/elevated ask` viene eseguito sull'host del Gateway ma mantiene le approvazioni per exec (come `/elevated on`).
- `on`/`ask` **non** forzano `exec.security=full`; si applica comunque la policy di sicurezza/ask configurata.
- Cambia il comportamento solo quando l'agente è in **sandbox** (altrimenti exec viene già eseguito sull'host).
- Forme della direttiva: `/elevated on|off|ask|full`, `/elev on|off|ask|full`.
- Solo `on|off|ask|full` sono accettati; qualsiasi altro valore restituisce un suggerimento e non modifica lo stato.

<div id="what-it-controls-and-what-it-doesnt">
  ## Cosa controlla (e cosa no)
</div>

- **Vincoli di disponibilità**: `tools.elevated` è la baseline globale. `agents.list[].tools.elevated` può restringere ulteriormente elevated per singolo agente (entrambi devono consentire).
- **Stato per sessione**: `/elevated on|off|ask|full` imposta il livello elevated per la chiave di sessione corrente.
- **Direttiva inline**: `/elevated on|ask|full` all'interno di un messaggio si applica solo a quel messaggio.
- **Gruppi**: nelle chat di gruppo, le direttive elevated vengono rispettate solo quando l'agente è menzionato. I messaggi composti solo da comandi che aggirano i requisiti di menzione sono trattati come se l'agente fosse menzionato.
- **Esecuzione sull'host**: elevated forza `exec` sull'host del Gateway; `full` imposta anche `security=full`.
- **Approvazioni**: `full` salta le approvazioni di exec; `on`/`ask` le rispettano quando le regole di lista di autorizzati/ask lo richiedono.
- **Agenti non in sandbox**: nessun effetto sulla posizione di esecuzione; influisce solo su gating, logging e stato.
- **La policy dei tool si applica comunque**: se `exec` è negato dalla policy dei tool, elevated non può essere usato.
- **Separato da `/exec`**: `/exec` regola le impostazioni predefinite della sessione per i mittenti autorizzati e non richiede elevated.

<div id="resolution-order">
  ## Ordine di applicazione
</div>

1. Direttiva inline sul messaggio (si applica solo a quel messaggio).
2. Override della sessione (impostato inviando un messaggio contenente solo la direttiva).
3. Valore predefinito globale (`agents.defaults.elevatedDefault` nella configurazione).

<div id="setting-a-session-default">
  ## Impostare un valore predefinito della sessione
</div>

- Invia un messaggio che contenga **solo** la direttiva (spazi consentiti), ad es. `/elevated full`.
- Viene inviata una risposta di conferma (`Elevated mode set to full...` / `Elevated mode disabled.`).
- Se l'accesso elevato è disabilitato o il mittente non è nella lista di autorizzati configurata, la direttiva risponde con un errore con indicazioni operative e non modifica lo stato della sessione.
- Invia `/elevated` (o `/elevated:`) senza argomenti per vedere il livello di elevazione corrente.

<div id="availability-allowlists">
  ## Disponibilità + liste di autorizzati
</div>

- Feature gate: `tools.elevated.enabled` (il valore predefinito può essere disattivato tramite configurazione anche se il codice lo supporta).
- Lista di autorizzati dei mittenti: `tools.elevated.allowFrom` con liste di autorizzati per provider (ad es. `discord`, `whatsapp`).
- Gate per agente: `agents.list[].tools.elevated.enabled` (opzionale; può solo restringere ulteriormente).
- Lista di autorizzati per agente: `agents.list[].tools.elevated.allowFrom` (opzionale; se impostata, il mittente deve essere presente **sia** nella lista di autorizzati globale **sia** in quella per agente).
- Fallback Discord: se `tools.elevated.allowFrom.discord` è omesso, l'elenco `channels.discord.dm.allowFrom` viene usato come fallback. Imposta `tools.elevated.allowFrom.discord` (anche `[]`) per sovrascrivere il fallback. Le liste di autorizzati per agente **non** utilizzano il fallback.
- Tutti i gate devono essere superati; in caso contrario la modalità elevated è considerata non disponibile.

<div id="logging-status">
  ## Log e stato
</div>

- Le chiamate di esecuzione in modalità elevata vengono registrate a livello info.
- Lo stato della sessione include la modalità elevata (ad es. `elevated=ask`, `elevated=full`).