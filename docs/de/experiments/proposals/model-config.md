---
title: Modellkonfiguration
summary: "Exploration: Modellkonfiguration, Auth-Profile und Fallback-Verhalten"
read_when:
  - Beim Erkunden zukünftiger Ideen zur Modellauswahl und zu Auth-Profilen
---

<div id="model-config-exploration">
  # Modellkonfiguration (Explorativ)
</div>

In diesem Dokument werden **Ideen** für zukünftige Modellkonfigurationen festgehalten. Es ist
keine produktionsreife Spezifikation. Für das aktuelle Verhalten siehe:

* [Modelle](/de/concepts/models)
* [Modell-Failover](/de/concepts/model-failover)
* [OAuth + Profile](/de/concepts/oauth)

<div id="motivation">
  ## Motivation
</div>

Betreiber möchten:

* Mehrere Authentifizierungsprofile pro Anbieter (privat vs. beruflich).
* Einfache `/model`-Auswahl mit vorhersehbaren Fallbacks.
* Klare Trennung zwischen Textmodellen und Modellen mit Bildunterstützung.

<div id="possible-direction-high-level">
  ## Mögliche Richtung (High-Level)
</div>

* Modellauswahl einfach halten: `provider/model` mit optionalen Aliasen.
* Anbietern mehrere Auth-Profile mit expliziter Reihenfolge erlauben.
* Eine globale Fallback-Liste verwenden, damit alle Sitzungen konsistent auf Failover umschalten.
* Routing für Bilder nur überschreiben, wenn es explizit konfiguriert ist.

<div id="open-questions">
  ## Offene Fragen
</div>

* Sollte die Profilrotation pro Anbieter oder pro Modell erfolgen?
* Wie sollte die UI die Profilauswahl für eine Sitzung anzeigen?
* Was ist der sicherste Migrationspfad für die Migration von Legacy-Konfigurationsschlüsseln?