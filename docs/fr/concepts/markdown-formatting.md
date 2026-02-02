---
title: Mise en forme Markdown
summary: "Pipeline de mise en forme Markdown pour les canaux sortants"
read_when:
  - Vous modifiez la mise en forme ou le découpage Markdown pour les canaux sortants
  - Vous ajoutez un nouveau formateur de canal ou un nouveau mappage de styles
  - Vous déboguez des régressions de mise en forme entre les canaux
---

<div id="markdown-formatting">
  # Mise en forme Markdown
</div>

OpenClaw met en forme le Markdown sortant en le convertissant d&#39;abord en une représentation intermédiaire (IR) partagée avant de générer un rendu spécifique à chaque canal. L&#39;IR conserve le texte source intact tout en portant les plages de style/liens, afin que le découpage et le rendu restent cohérents entre les canaux.

<div id="goals">
  ## Objectifs
</div>

* **Cohérence :** une seule étape d’analyse, plusieurs moteurs de rendu.
* **Découpage sécurisé :** découper le texte avant le rendu pour que la mise en forme inline ne soit jamais
  cassée entre les fragments.
* **Adaptation aux canaux :** faire correspondre la même IR à Slack mrkdwn, Telegram HTML et aux plages de style Signal sans avoir à réanalyser le Markdown.

<div id="pipeline">
  ## Pipeline
</div>

1. **Analyser le Markdown -&gt; IR**
   * L&#39;IR est du texte brut accompagné de plages de style (gras/italique/barré/code/spoiler) et de plages de lien.
   * Les décalages sont en unités de code UTF‑16 afin que les plages de style de Signal s&#39;alignent avec son API.
   * Les tableaux ne sont analysés que lorsqu&#39;un canal a activé la conversion de tableaux.
2. **Découper l&#39;IR (format-first)**
   * Le découpage s&#39;effectue sur le texte de l&#39;IR avant le rendu.
   * Le formatage en ligne n&#39;est pas scindé entre les segments ; les plages sont découpées par segment.
3. **Rendu par canal**
   * **Slack :** jetons mrkdwn (gras/italique/barré/code), liens sous la forme `<url|label>`.
   * **Telegram :** balises HTML (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   * **Signal :** texte brut + plages `text-style` ; les liens deviennent `label (url)` lorsque le libellé diffère.

<div id="ir-example">
  ## Exemple d’IR
</div>

Markdown en entrée :

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (schéma) :

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }
  ]
}
```

<div id="where-it-is-used">
  ## Où il est utilisé
</div>

* Les adaptateurs sortants Slack, Telegram et Signal effectuent le rendu à partir de l’IR.
* Les autres canaux (WhatsApp, iMessage, MS Teams, Discord) utilisent encore du texte brut ou
  leurs propres règles de mise en forme, avec conversion des tableaux Markdown appliquée avant
  le chunking lorsque celui-ci est activé.

<div id="table-handling">
  ## Gestion des tableaux
</div>

Les tableaux Markdown ne sont pas pris en charge de manière cohérente par tous les clients de discussion. Utilisez
`markdown.tables` pour contrôler la conversion par canal (et par compte).

* `code` : afficher les tableaux sous forme de blocs de code (valeur par défaut pour la plupart des canaux).
* `bullets` : convertir chaque ligne en puces (valeur par défaut pour Signal + WhatsApp).
* `off` : désactiver l’analyse et la conversion des tableaux ; le texte brut du tableau est transmis tel quel.

Clés de configuration :

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

<div id="chunking-rules">
  ## Règles de découpage en fragments
</div>

* Les limites de fragments proviennent des adaptateurs/configurations de canal et sont appliquées au texte IR.
* Les blocs de code délimités (code fences) sont conservés comme un seul bloc avec une nouvelle ligne finale afin que les canaux
  les affichent correctement.
* Les préfixes de liste et les préfixes de bloc de citation font partie du texte IR, de sorte que le découpage
  ne coupe jamais au milieu d&#39;un préfixe.
* Les styles en ligne (gras/italique/barré/code en ligne/spoiler) ne sont jamais scindés entre
  plusieurs fragments ; le moteur de rendu rouvre les styles à l&#39;intérieur de chaque fragment.

Si vous avez besoin de plus de détails sur le comportement du découpage entre canaux, consultez
[Streaming + chunking](/fr/concepts/streaming).

<div id="link-policy">
  ## Politique de liens
</div>

* **Slack :** `[label](url)` -&gt; `<url|label>` ; les URL brutes restent brutes. L’autolink
  est désactivé lors de l’analyse pour éviter la création de liens en double.
* **Telegram :** `[label](url)` -&gt; `<a href="url">label</a>` (mode d’analyse HTML).
* **Signal :** `[label](url)` -&gt; `label (url)` sauf si le label correspond à l’URL.

<div id="spoilers">
  ## Spoilers
</div>

Les marqueurs de spoiler (`||spoiler||`) ne sont interprétés que pour Signal, où ils correspondent à des plages de style SPOILER. Les autres canaux les traitent comme du texte brut.

<div id="how-to-add-or-update-a-channel-formatter">
  ## Comment ajouter ou mettre à jour un formateur de canal
</div>

1. **Analyse une seule fois :** utilise la fonction utilitaire partagée `markdownToIR(...)` avec des options adaptées au canal (autolink, style de titre, préfixe de citation).
2. **Rendu :** implémente un moteur de rendu avec `renderMarkdownWithMarkers(...)` et une table de correspondance de marqueurs de style (ou des plages de style Signal).
3. **Découpe :** appelle `chunkMarkdownIR(...)` avant le rendu, puis rends chaque fragment.
4. **Raccorder l’adaptateur :** mets à jour l’adaptateur sortant du canal pour utiliser le nouveau découpeur et le nouveau moteur de rendu.
5. **Tester :** ajoute ou mets à jour les tests de formatage et un test d’envoi sortant si le canal utilise le découpage.

<div id="common-gotchas">
  ## Pièges courants
</div>

* Les tokens Slack entre chevrons (`<@U123>`, `<#C123>`, `<https://...>`) doivent
  être conservés ; échappez le HTML brut de façon sûre.
* Le HTML de Telegram nécessite d’échapper le texte en dehors des balises pour éviter de casser le balisage.
* Les plages de style de Signal dépendent des décalages UTF-16 ; n’utilisez pas de décalages en points de code (Unicode).
* Préservez les retours à la ligne finaux pour les blocs de code délimités afin que les marqueurs de fermeture se retrouvent bien sur
  leur propre ligne.