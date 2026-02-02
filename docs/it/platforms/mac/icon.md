---
title: Icona
summary: "Stati e animazioni dell'icona nella barra dei menu di OpenClaw su macOS"
read_when:
  - Modifica del comportamento dell'icona nella barra dei menu
---

<div id="menu-bar-icon-states">
  # Stati dell'icona nella barra dei menu
</div>

Autore: steipete · Aggiornato: 2025-12-06 · Scope: macOS app (`apps/macos`)

- **Idle:** Animazione normale dell'icona (ammiccamento, oscillazione occasionale).
- **In pausa:** L'elemento di stato usa `appearsDisabled`; nessun movimento.
- **Attivazione vocale (orecchie grandi):** Il rilevatore di attivazione vocale chiama `AppState.triggerVoiceEars(ttl: nil)` quando viene rilevata la parola di attivazione (wake word), mantenendo `earBoostActive=true` mentre l’enunciato viene acquisito. Le orecchie si ingrandiscono (1,9x), mostrano fori circolari per migliorarne la leggibilità, quindi tornano allo stato normale tramite `stopVoiceEars()` dopo 1s di silenzio. Attivato solo dalla pipeline vocale in-app.
- **Al lavoro (agente in esecuzione):** `AppState.isWorking=true` gestisce una micro-animazione di “coda/zampe che sgambettano”: oscillazione delle zampe più veloce e leggero spostamento mentre il lavoro è in corso. Attualmente attivata attorno alle esecuzioni dell'agente WebChat; aggiungi la stessa attivazione attorno ad altri task lunghi quando li colleghi.

Punti di collegamento

- Attivazione vocale: le chiamate di runtime/test usano `AppState.triggerVoiceEars(ttl: nil)` sul trigger e `stopVoiceEars()` dopo 1s di silenzio per allinearsi alla finestra di acquisizione.
- Attività dell'agente: imposta `AppStateStore.shared.setWorking(true/false)` attorno agli intervalli di lavoro (già fatto nella chiamata dell'agente WebChat). Mantieni gli intervalli brevi e ripristina in blocchi `defer` per evitare animazioni bloccate.

Forme e dimensioni

- Icona di base disegnata in `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`.
- La scala delle orecchie è impostata di default a `1.0`; il boost vocale imposta `earScale=1.9` e attiva `earHoles=true` senza modificare il frame complessivo (immagine template 18×18 pt renderizzata in un backing store Retina 36×36 px).
- Lo “scurry” usa un'oscillazione delle zampe fino a ~1.0 con una piccola oscillazione orizzontale; è additiva rispetto a qualsiasi oscillazione idle esistente.

Note sul comportamento

- Nessun toggle esterno via CLI/broker per orecchie/stato di lavoro; mantieni tutto interno ai segnali dell'app per evitare oscillazioni indesiderate.
- Mantieni i TTL brevi (&lt;10s) in modo che l'icona ritorni rapidamente allo stato base se un job si blocca.