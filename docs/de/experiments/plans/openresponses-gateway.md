---
title: OpenResponses Gateway
summary: "Plan: OpenResponses-Endpunkt /v1/responses hinzufügen und Chat-Completions geordnet auslaufen lassen"
owner: "openclaw"
status: "draft"
last_updated: "2026-01-19"
---

<div id="openresponses-gateway-integration-plan">
  # Integrationsplan für das OpenResponses Gateway
</div>

<div id="context">
  ## Kontext
</div>

Das OpenClaw Gateway stellt derzeit einen minimalen, OpenAI-kompatiblen Chat-Completions-Endpunkt unter
`/v1/chat/completions` bereit (siehe [OpenAI Chat Completions](/de/gateway/openai-http-api)).

Open Responses ist ein offener Inferenzstandard, der auf der OpenAI Responses API basiert. Er ist für agentenbasierte Workflows konzipiert und verwendet item-basierte Eingaben plus semantische Streaming-Events. Die OpenResponses-Spezifikation definiert `/v1/responses`, nicht `/v1/chat/completions`.

<div id="goals">
  ## Ziele
</div>

* Einführung eines `/v1/responses`-Endpoints, der der OpenResponses-Semantik entspricht.
* Beibehaltung von Chat Completions als Kompatibilitätsebene, die sich leicht deaktivieren und schließlich entfernen lässt.
* Standardisierung von Validierung und Parsing mit isolierten, wiederverwendbaren Schemata.

<div id="non-goals">
  ## Nichtziele
</div>

* Vollständige Feature-Parität mit OpenResponses in der ersten Phase (Bilder, Dateien, gehostete Tools).
* Ersetzen der internen Agent-Ausführungslogik oder der Tool-Orchestrierung.
* Änderung des bestehenden `/v1/chat/completions`-Verhaltens in der ersten Phase.

<div id="research-summary">
  ## Forschungszusammenfassung
</div>

Quellen: OpenResponses OpenAPI, OpenResponses-Spezifikationsseite und der Hugging-Face-Blogbeitrag.

Wesentliche Punkte:

* `POST /v1/responses` akzeptiert `CreateResponseBody`-Felder wie `model`, `input` (String oder
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens` und
  `max_tool_calls`.
* `ItemParam` ist eine diskriminierte Union aus:
  * `message`-Elementen mit Rollen `system`, `developer`, `user`, `assistant`
  * `function_call` und `function_call_output`
  * `reasoning`
  * `item_reference`
* Erfolgreiche Antworten liefern eine `ResponseResource` mit `object: "response"`, `status` und
  `output`-Elementen.
* Streaming verwendet semantische Events wie:
  * `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  * `response.output_item.added`, `response.output_item.done`
  * `response.content_part.added`, `response.content_part.done`
  * `response.output_text.delta`, `response.output_text.done`
* Die Spezifikation erfordert:
  * `Content-Type: text/event-stream`
  * `event:` muss dem JSON-`type`-Feld entsprechen
  * das terminale Event muss das Literal `[DONE]` sein
* Reasoning-Elemente können `content`, `encrypted_content` und `summary` enthalten.
* HF-Beispiele enthalten `OpenResponses-Version: latest` in Anfragen (optionaler Header).

<div id="proposed-architecture">
  ## Vorgeschlagene Architektur
</div>

* Füge `src/gateway/open-responses.schema.ts` hinzu, das nur Zod-Schemas enthält (keine Gateway-Imports).
* Füge `src/gateway/openresponses-http.ts` (oder `open-responses-http.ts`) für `/v1/responses` hinzu.
* Belasse `src/gateway/openai-http.ts` unverändert als Legacy-Kompatibilitätsadapter.
* Füge die Konfiguration `gateway.http.endpoints.responses.enabled` hinzu (Standardwert `false`).
* Belasse `gateway.http.endpoints.chatCompletions.enabled` unabhängig; erlaube, beide Endpunkte
  getrennt zu schalten.
* Gib beim Start eine Warnung aus, wenn Chat Completions aktiviert ist, um den Legacy-Status zu signalisieren.

<div id="deprecation-path-for-chat-completions">
  ## Deprecation-Pfad für Chat Completions
</div>

* Strikte Modulgrenzen einhalten: keine gemeinsam genutzten Schema-Typen zwischen Responses und Chat Completions.
* Chat Completions per Konfiguration optional machen, sodass sie ohne Codeänderungen deaktiviert werden können.
* Dokumentation aktualisieren, um Chat Completions als Legacy-Funktion zu kennzeichnen, sobald `/v1/responses` stabil ist.
* Optionaler zukünftiger Schritt: Chat-Completions-Anfragen auf den Responses-Handler mappen, um eine einfachere Entfernung zu ermöglichen.

<div id="phase-1-support-subset">
  ## Phase-1-Support-Teilmenge
</div>

* `input` als String oder `ItemParam[]` mit Nachrichtenrollen und `function_call_output` akzeptieren.
* System- und Entwicklernachrichten in `extraSystemPrompt` extrahieren.
* Die jüngste `user`- oder `function_call_output`-Nachricht als aktuelle Nachricht für agent-Ausführungen verwenden.
* Nicht unterstützte Inhaltsteile (Bild/Datei) mit `invalid_request_error` zurückweisen.
* Eine einzelne Assistant-Nachricht mit `output_text`-Inhalt zurückgeben.
* `usage` mit auf Null gesetzten Werten zurückgeben, bis die Token-Abrechnung integriert ist.

<div id="validation-strategy-no-sdk">
  ## Validierungsstrategie (ohne SDK)
</div>

* Implementiere Zod-Schemas für die unterstützte Teilmenge von:
  * `CreateResponseBody`
  * `ItemParam` + Vereinigungen der Nachricht-Inhaltssegmente
  * `ResponseResource`
  * Streaming-Event-Strukturen, die vom Gateway verwendet werden
* Halte die Schemas in einem einzelnen, isolierten Modul, um Drift zu vermeiden und zukünftige Codegenerierung zu ermöglichen.

<div id="streaming-implementation-phase-1">
  ## Streaming-Implementierung (Phase 1)
</div>

* SSE-Zeilen mit `event:`- und `data:`-Feldern.
* Erforderliche Sequenz (minimale Version):
  * `response.created`
  * `response.output_item.added`
  * `response.content_part.added`
  * `response.output_text.delta` (bei Bedarf wiederholen)
  * `response.output_text.done`
  * `response.content_part.done`
  * `response.completed`
  * `[DONE]`

<div id="tests-and-verification-plan">
  ## Test- und Verifizierungsplan
</div>

* E2E-Abdeckung für `/v1/responses` hinzufügen:
  * Authentifizierung erforderlich
  * Nicht-Streaming-Antwortformat
  * Stream-Ereignisreihenfolge und `[DONE]`
  * Sitzungsrouting mit Headern und `user`
* `src/gateway/openai-http.e2e.test.ts` unverändert lassen.
* Manuell: curl-Aufruf auf `/v1/responses` mit `stream: true` ausführen und Ereignisreihenfolge sowie abschließendes
  `[DONE]` verifizieren.

<div id="doc-updates-follow-up">
  ## Dokumentations-Updates (Follow-up)
</div>

* Füge eine neue Dokumentationsseite zur Verwendung von `/v1/responses` mit Beispielen hinzu.
* Aktualisiere `/gateway/openai-http-api` mit einem Legacy-Hinweis und einem Verweis auf `/v1/responses`.