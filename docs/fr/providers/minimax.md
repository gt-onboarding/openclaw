---
title: Minimax
summary: "Utiliser MiniMax M2.1 dans OpenClaw"
read_when:
  - Vous souhaitez utiliser des modèles MiniMax dans OpenClaw
  - Vous avez besoin d’un guide de configuration pour MiniMax
---

<div id="minimax">
  # MiniMax
</div>

MiniMax est une entreprise spécialisée dans l&#39;IA qui développe la famille de modèles **M2/M2.1**. La version actuelle, orientée développement, est **MiniMax M2.1** (23 décembre 2025), conçue pour des tâches complexes en conditions réelles.

Source : [Notes de version de MiniMax M2.1](https://www.minimax.io/news/minimax-m21)

<div id="model-overview-m21">
  ## Vue d’ensemble du modèle (M2.1)
</div>

MiniMax présente les améliororations suivantes dans M2.1 :

* **Programmation multilingue** plus robuste (Rust, Java, Go, C++, Kotlin, Objective-C, TS/JS).
* Meilleur **développement web et applicatif** et meilleure qualité esthétique des rendus (y compris pour les applications mobiles natives).
* Gestion améliorée des **instructions composites** pour les workflows de type bureautique, en s’appuyant sur
  un raisonnement imbriqué et une exécution intégrée des contraintes.
* **Réponses plus concises** avec une consommation de tokens réduite et des boucles d’itération plus rapides.
* Compatibilité renforcée avec les **frameworks d’outils et d’agents** et meilleure gestion du contexte (Claude Code,
  Droid/Factory AI, Cline, Kilo Code, Roo Code, BlackBox).
* Produits de **dialogue et de rédaction technique** de meilleure qualité.

<div id="minimax-m21-vs-minimax-m21-lightning">
  ## MiniMax M2.1 vs MiniMax M2.1 Lightning
</div>

* **Vitesse :** Lightning est la variante « rapide » dans la documentation tarifaire de MiniMax.
* **Coût :** La tarification affiche le même coût pour l’entrée, mais Lightning a un coût de sortie plus élevé.
* **Routage du plan de développement :** Le back-end Lightning n’est pas directement disponible avec le plan de développement MiniMax. MiniMax redirige automatiquement la plupart des requêtes vers Lightning, mais bascule sur le back-end M2.1 classique lors des pics de trafic.

<div id="choose-a-setup">
  ## Choisir une configuration
</div>

<div id="minimax-m21-recommended">
  ### MiniMax M2.1 — recommandé
</div>

**Idéal pour :** MiniMax hébergé avec une API compatible avec Anthropic.

Configurez via la CLI :

* Exécutez `openclaw configure`
* Sélectionnez **Model/auth**
* Choisissez **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="minimax-m21-as-fallback-opus-primary">
  ### MiniMax M2.1 comme solution de repli (Opus principal)
</div>

**Idéal pour :** conserver Opus 4.5 comme modèle principal, avec bascule vers MiniMax M2.1 en cas de panne.

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

<div id="optional-local-via-lm-studio-manual">
  ### Facultatif : en local via LM Studio (manuel)
</div>

**Idéal pour :** l’inférence locale avec LM Studio.
Nous avons observé d’excellents résultats avec MiniMax M2.1 sur du matériel puissant (par exemple un
ordinateur de bureau ou un serveur) en utilisant le serveur local de LM Studio.

Configurez manuellement via `openclaw.json` :

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

<div id="configure-via-openclaw-configure">
  ## Configurer avec `openclaw configure`
</div>

Utilisez l’assistant de configuration interactif pour paramétrer MiniMax sans modifier le JSON :

1. Exécutez `openclaw configure`.
2. Sélectionnez **Model/auth**.
3. Choisissez **MiniMax M2.1**.
4. Choisissez votre modèle par défaut lorsque vous y êtes invité.

<div id="configuration-options">
  ## Options de configuration
</div>

* `models.providers.minimax.baseUrl` : préférez `https://api.minimax.io/anthropic` (compatible Anthropic) ; `https://api.minimax.io/v1` est optionnel pour les charges utiles compatibles OpenAI.
* `models.providers.minimax.api` : préférez `anthropic-messages` ; `openai-completions` est optionnel pour les charges utiles compatibles OpenAI.
* `models.providers.minimax.apiKey` : clé API MiniMax (`MINIMAX_API_KEY`).
* `models.providers.minimax.models` : définissez `id`, `name`, `reasoning`, `contextWindow`, `maxTokens`, `cost`.
* `agents.defaults.models` : créez des alias pour les modèles que vous souhaitez inclure dans la liste d’autorisation.
* `models.mode` : conservez `merge` si vous voulez ajouter MiniMax en plus des modèles intégrés.

<div id="notes">
  ## Remarques
</div>

* Les références de modèle sont `minimax/<model>`.
* API d’utilisation du MiniMax Coding Plan : `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (nécessite une clé de Coding Plan).
* Mettez à jour les valeurs de tarification dans `models.json` si vous avez besoin d’un suivi précis des coûts.
* Lien de parrainage pour MiniMax Coding Plan (10 % de réduction) : https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&amp;source=link
* Consultez [/concepts/model-providers](/fr/concepts/model-providers) pour les règles des fournisseurs.
* Utilisez `openclaw models list` et `openclaw models set minimax/MiniMax-M2.1` pour changer de modèle.

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="unknown-model-minimaxminimax-m21">
  ### « Unknown model: minimax/MiniMax-M2.1 »
</div>

Cela signifie généralement que le **fournisseur MiniMax n’est pas configuré** (aucune entrée de fournisseur
et aucun profil d’authentification MiniMax/clé d’environnement trouvés). Une correction pour cette détection est incluse dans
**2026.1.12** (non publiée au moment de la rédaction). Pour corriger :

* Mettre à jour vers la version **2026.1.12** (ou exécuter depuis la branche `main`), puis redémarrer le Gateway.
* Exécuter `openclaw configure` et sélectionner **MiniMax M2.1**, ou
* Ajouter manuellement le bloc `models.providers.minimax`, ou
* Définir `MINIMAX_API_KEY` (ou un profil d’authentification MiniMax) afin que le fournisseur puisse être injecté.

Assurez-vous que l’identifiant du modèle est **sensible à la casse** :

* `minimax/MiniMax-M2.1`
* `minimax/MiniMax-M2.1-lightning`

Ensuite, revérifiez avec :

```bash
openclaw models list
```
