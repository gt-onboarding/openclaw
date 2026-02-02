---
title: Configuration du modèle
summary: "Exploration : configuration des modèles, profils d’authentification et comportement de basculement"
read_when:
  - Exploration d’idées futures concernant la sélection de modèles et les profils d’authentification
---

<div id="model-config-exploration">
  # Configuration des modèles (exploration)
</div>

Ce document rassemble des **idées** pour de futures configurations de modèles. Ce n’est pas
une spécification prête pour la production. Pour le comportement actuel, voir :

* [Modèles](/fr/concepts/models)
* [Basculement de modèle](/fr/concepts/model-failover)
* [OAuth + profils](/fr/concepts/oauth)

<div id="motivation">
  ## Motivation
</div>

Les opérateurs veulent :

* Plusieurs profils d&#39;authentification par fournisseur (personnel vs professionnel).
* Une sélection `/model` simple avec des mécanismes de repli prévisibles.
* Une séparation claire entre les modèles texte et les modèles capables de générer des images.

<div id="possible-direction-high-level">
  ## Orientation possible (haut niveau)
</div>

* Conserver une sélection de modèles simple : `provider/model` avec des alias optionnels.
* Permettre aux fournisseurs d’avoir plusieurs profils d’authentification, avec un ordre explicite.
* Utiliser une liste de repli globale pour que toutes les sessions basculent de manière cohérente.
* Ne redéfinir le routage des images que lorsqu’il est explicitement configuré.

<div id="open-questions">
  ## Questions ouvertes
</div>

* La rotation des profils doit-elle se faire par fournisseur ou par modèle ?
* Comment l’UI doit-elle présenter la sélection de profil pour une session ?
* Quelle est la stratégie de migration la plus sûre à partir des anciennes clés de configuration ?