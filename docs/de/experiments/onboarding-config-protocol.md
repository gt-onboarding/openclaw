---
title: Onboarding- und Konfigurationsprotokoll
summary: "Notizen zum RPC-Protokoll für Onboarding-Assistent und Konfigurationsschema"
read_when: "Bei Änderungen an den Schritten des Onboarding-Assistenten oder an den Endpunkten des Konfigurationsschemas"
---

<div id="onboarding-config-protocol">
  # Onboarding- und Konfigurationsprotokoll
</div>

Zweck: Gemeinsame Onboarding- und Konfigurationsoberflächen über CLI, macOS-App und Web-UI hinweg.

<div id="components">
  ## Komponenten
</div>

- Wizard-Engine (gemeinsame Sitzung + Prompts + Onboarding-Status).
- CLI-Onboarding verwendet denselben Wizard-Flow wie die UI-Clients.
- Gateway-RPC stellt Wizard- und Konfigurationsschema-Endpunkte bereit.
- macOS-Onboarding verwendet das Schrittmodell des Wizards.
- Die Web-UI rendert Konfigurationsformulare aus dem JSON Schema + UI-Hinweisen.

<div id="gateway-rpc">
  ## Gateway-RPC
</div>

- `wizard.start` Parameter: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` Parameter: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` Parameter: `{ sessionId }`
- `wizard.status` Parameter: `{ sessionId }`
- `config.schema` Parameter: `{}`

Antworten (Struktur)

- Wizard: `{ sessionId, done, step?, status?, error? }`
- Konfigurationsschema: `{ schema, uiHints, version, generatedAt }`

<div id="ui-hints">
  ## UI-Hinweise
</div>

- `uiHints` nach Pfad geschlüsselt; optionale Metadaten (label/help/group/order/advanced/sensitive/placeholder).
- Vertrauliche Felder werden als Passwortfelder dargestellt; keine zusätzliche Maskierungs- oder Redaktionsschicht.
- Nicht unterstützte Schema-Knoten fallen auf den Roh-JSON-Editor zurück.

<div id="notes">
  ## Hinweise
</div>

- Dieses Dokument ist die zentrale Referenz für alle Protokoll-Überarbeitungen rund um Onboarding und Konfiguration.