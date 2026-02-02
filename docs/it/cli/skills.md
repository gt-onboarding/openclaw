---
title: Abilità
summary: "Riferimento CLI per `openclaw skills` (list/info/check) e stato di idoneità delle abilità"
read_when:
  - Vuoi vedere quali abilità sono disponibili e pronte per essere eseguite
  - Vuoi eseguire il debug di binari/variabili d'ambiente/configurazioni mancanti per le abilità
---

<div id="openclaw-skills">
  # `openclaw skills`
</div>

Ispeziona le abilità (incluse, nello spazio di lavoro e override gestiti) e verifica quali sono utilizzabili e quali non soddisfano i requisiti.

Correlato:

* Sistema delle abilità: [Abilità](/it/tools/skills)
* Configurazione delle abilità: [Configurazione delle abilità](/it/tools/skills-config)
* Installazioni da ClawHub: [ClawHub](/it/tools/clawhub)

<div id="commands">
  ## Comandi
</div>

```bash
openclaw skills list
openclaw skills list --eligible
openclaw skills info <name>
openclaw skills check
```
