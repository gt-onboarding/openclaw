---
title: Ollama
summary: "Exécuter OpenClaw avec Ollama (moteur LLM local)"
read_when:
  - Vous souhaitez exécuter OpenClaw avec des modèles locaux via Ollama
  - Vous avez besoin de conseils pour l'installation et la configuration d'Ollama
---

<div id="ollama">
  # Ollama
</div>

Ollama est un runtime LLM local qui facilite l&#39;exécution de modèles open source sur votre machine. OpenClaw s&#39;intègre à l&#39;API compatible OpenAI d&#39;Ollama et peut **découvrir automatiquement les modèles compatibles avec les outils** lorsque vous activez cette option avec `OLLAMA_API_KEY` (ou un profil d&#39;authentification) et que vous ne définissez pas d&#39;entrée `models.providers.ollama` explicite.

<div id="quick-start">
  ## Démarrage rapide
</div>

1. Installez Ollama : https://ollama.ai

2. Téléchargez un modèle :

```bash
ollama pull llama3.3
# ou
ollama pull qwen2.5-coder:32b
# ou
ollama pull deepseek-r1:32b
```

3. Activer Ollama pour OpenClaw (n&#39;importe quelle valeur convient ; Ollama ne nécessite pas de clé API réelle) :

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Utilisez les modèles Ollama :

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/llama3.3" }
    }
  }
}
```

<div id="model-discovery-implicit-provider">
  ## Découverte de modèles (fournisseur implicite)
</div>

Quand vous définissez `OLLAMA_API_KEY` (ou un profil d’authentification) et que vous **ne** définissez **pas** `models.providers.ollama`, OpenClaw découvre les modèles depuis l’instance Ollama locale à `http://127.0.0.1:11434` :

* Interroge `/api/tags` et `/api/show`
* Ne conserve que les modèles qui annoncent la capacité `tools`
* Marque `reasoning` quand le modèle annonce `thinking`
* Lit `contextWindow` à partir de `model_info["<arch>.context_length"]` lorsqu’il est disponible
* Définit `maxTokens` à 10× la fenêtre de contexte
* Définit tous les coûts à `0`

Cela évite d’ajouter manuellement des modèles tout en gardant le catalogue aligné sur les capacités d’Ollama.

Pour voir quels modèles sont disponibles :

```bash
ollama list
openclaw models list
```

Pour ajouter un nouveau modèle, il vous suffit de le télécharger avec Ollama :

```bash
ollama pull mistral
```

Le nouveau modèle sera automatiquement détecté et prêt à l&#39;emploi.

Si vous définissez explicitement la clé `models.providers.ollama`, la détection automatique est désactivée et vous devez définir les modèles manuellement (voir ci-dessous).

<div id="configuration">
  ## Configuration
</div>

<div id="basic-setup-implicit-discovery">
  ### Configuration de base (découverte implicite)
</div>

La façon la plus simple d’activer Ollama consiste à utiliser une variable d’environnement :

```bash
export OLLAMA_API_KEY="ollama-local"
```

<div id="explicit-setup-manual-models">
  ### Configuration explicite (modèles manuels)
</div>

Utilisez une configuration explicite lorsque :

* Ollama s’exécute sur un autre hôte/port.
* Vous voulez imposer des fenêtres de contexte ou des listes de modèles spécifiques.
* Vous voulez inclure des modèles qui ne déclarent pas la prise en charge des outils.

```json5
{
  models: {
    providers: {
      ollama: {
        // Utilisez un hôte incluant /v1 pour les API compatibles OpenAI
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "llama3.3",
            name: "Llama 3.3",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Si `OLLAMA_API_KEY` est défini, vous pouvez omettre `apiKey` dans l&#39;entrée du fournisseur et OpenClaw le renseignera automatiquement pour les vérifications de disponibilité.

<div id="custom-base-url-explicit-config">
  ### URL de base personnalisée (configuration explicite)
</div>

Si Ollama s’exécute sur un autre hôte ou un autre port (la configuration explicite désactive l’auto‑découverte ; vous devez donc définir les modèles manuellement) :

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1"
      }
    }
  }
}
```

<div id="model-selection">
  ### Sélection du modèle
</div>

Une fois la configuration effectuée, tous vos modèles Ollama sont disponibles :

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/llama3.3",
        fallback: ["ollama/qwen2.5-coder:32b"]
      }
    }
  }
}
```

<div id="advanced">
  ## Fonctionnalités avancées
</div>

<div id="reasoning-models">
  ### Modèles de raisonnement
</div>

OpenClaw marque les modèles comme compatibles avec le raisonnement lorsqu’Ollama indique `thinking` dans `/api/show` :

```bash
ollama pull deepseek-r1:32b
```

<div id="model-costs">
  ### Coût des modèles
</div>

Ollama est gratuit et s’exécute en local, donc tous les coûts de modèle sont à 0 $.

<div id="context-windows">
  ### Fenêtres de contexte
</div>

Pour les modèles découverts automatiquement, OpenClaw utilise la fenêtre de contexte indiquée par Ollama lorsqu’elle est disponible, sinon elle est définie par défaut sur `8192`. Vous pouvez surcharger `contextWindow` et `maxTokens` dans la configuration explicite du fournisseur.

<div id="troubleshooting">
  ## Dépannage
</div>

<div id="ollama-not-detected">
  ### Ollama non détecté
</div>

Assurez-vous qu&#39;Ollama est en cours d&#39;exécution, que vous avez défini `OLLAMA_API_KEY` (ou un profil d&#39;authentification) et que vous **n&#39;avez pas** défini d&#39;entrée explicite `models.providers.ollama` :

```bash
ollama serve
```

Et que l&#39;API est accessible :

```bash
curl http://localhost:11434/api/tags
```

<div id="no-models-available">
  ### Aucun modèle disponible
</div>

OpenClaw ne détecte automatiquement que les modèles qui indiquent la prise en charge des outils. Si votre modèle n’est pas répertorié, soit :

* Téléchargez un modèle compatible avec les outils, ou
* Déclarez explicitement le modèle dans `models.providers.ollama`.

Pour ajouter des modèles :

```bash
ollama list  # Voir ce qui est installé
ollama pull llama3.3  # Récupérer un modèle
```

<div id="connection-refused">
  ### Connexion refusée
</div>

Vérifiez qu&#39;Ollama est en cours d&#39;exécution sur le bon port :

```bash
# Vérifier si Ollama est en cours d'exécution
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

<div id="see-also">
  ## Voir aussi
</div>

* [Fournisseurs de modèles](/fr/concepts/model-providers) - Vue d’ensemble de tous les fournisseurs
* [Sélection de modèles](/fr/concepts/models) - Comment choisir des modèles
* [Configuration](/fr/gateway/configuration) - Référence complète de la configuration