---
title: Conmutación por error de modelos
summary: "Cómo OpenClaw rota los perfiles de autenticación y realiza conmutación por error entre modelos"
read_when:
  - Diagnosticar la rotación de perfiles de autenticación, los períodos de enfriamiento o el comportamiento de conmutación por error entre modelos
  - Actualizar las reglas de conmutación por error para perfiles de autenticación o modelos
---

<div id="model-failover">
  # Conmutación por error de modelos
</div>

OpenClaw gestiona los fallos en dos etapas:

1. **Rotación del perfil de autenticación** dentro del proveedor actual.
2. **Cambio al modelo de respaldo** (fallback) al siguiente modelo en `agents.defaults.model.fallbacks`.

Este documento explica las reglas en tiempo de ejecución y los datos que las respaldan.

<div id="auth-storage-keys-oauth">
  ## Almacenamiento de autenticación (claves + OAuth)
</div>

OpenClaw usa **perfiles de autenticación** tanto para claves de API como para tokens OAuth.

* Los secretos se almacenan en `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (legado: `~/.openclaw/agent/auth-profiles.json`).
* Las configuraciones `auth.profiles` / `auth.order` son **solo metadatos y enrutamiento** (sin secretos).
* Archivo OAuth legado, solo para importación: `~/.openclaw/credentials/oauth.json` (se importa en `auth-profiles.json` en el primer uso).

Más detalles: [/concepts/oauth](/es/concepts/oauth)

Tipos de credenciales:

* `type: "api_key"` → `{ provider, key }`
* `type: "oauth"` → `{ provider, access, refresh, expires, email? }` (+ `projectId`/`enterpriseUrl` para algunos proveedores)

<div id="profile-ids">
  ## IDs de perfil
</div>

Los inicios de sesión OAuth crean perfiles distintos para que varias cuentas puedan coexistir.

* Predeterminado: `provider:default` cuando no se dispone de un correo electrónico.
* OAuth con correo electrónico: `provider:<email>` (por ejemplo, `google-antigravity:user@gmail.com`).

Los perfiles se almacenan en `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` dentro de `profiles`.

<div id="rotation-order">
  ## Orden de rotación
</div>

Cuando un proveedor tiene varios perfiles, OpenClaw elige un orden como este:

1. **Configuración explícita**: `auth.order[provider]` (si está definido).
2. **Perfiles configurados**: `auth.profiles` filtrados por proveedor.
3. **Perfiles almacenados**: entradas en `auth-profiles.json` para el proveedor.

Si no se configura un orden explícito, OpenClaw usa una rotación de tipo round‑robin:

* **Clave primaria:** tipo de perfil (**OAuth antes que claves API**).
* **Clave secundaria:** `usageStats.lastUsed` (el más antiguo primero, dentro de cada tipo).
* **Perfiles en cooldown/deshabilitados** se mueven al final, ordenados por la expiración más próxima.

<div id="session-stickiness-cache-friendly">
  ### Persistencia de sesión (optimizada para la caché)
</div>

OpenClaw **fija el perfil de autenticación elegido por sesión** para mantener calientes las cachés del proveedor.
**No** alterna en cada solicitud. El perfil fijado se reutiliza hasta que:

* la sesión se restablece (`/new` / `/reset`)
* se completa una compactación (el contador de compactación se incrementa)
* el perfil está en cooldown/deshabilitado

La selección manual mediante `/model …@<profileId>` establece una **anulación definida por el usuario** para esa sesión
y no se alterna automáticamente hasta que comienza una nueva sesión.

Los perfiles fijados automáticamente (seleccionados por el enrutador de sesiones) se tratan como una **preferencia**:
se intentan primero, pero OpenClaw puede alternar a otro perfil en caso de límites de tasa o timeouts.
Los perfiles fijados por el usuario permanecen bloqueados a ese perfil; si falla y hay alternativas de modelo
configuradas, OpenClaw pasa al siguiente modelo en lugar de cambiar de perfil.

<div id="why-oauth-can-look-lost">
  ### Por qué OAuth puede “parecer perdido”
</div>

Si tienes tanto un perfil OAuth como un perfil de clave de API para el mismo proveedor, el round‑robin puede alternar entre ellos de un mensaje a otro a menos que estén fijados. Para forzar un único perfil:

* Fija con `auth.order[provider] = ["provider:profileId"]`, o
* Usa una anulación por sesión mediante `/model …` con una anulación de perfil (cuando tu UI/interfaz de chat lo admita).

<div id="cooldowns">
  ## Períodos de enfriamiento
</div>

Cuando un perfil falla debido a errores de autenticación/limitación de tasa (o un tiempo de espera que se comporta como una limitación de tasa), OpenClaw lo marca en estado de enfriamiento y pasa al siguiente perfil.
Los errores de formato/solicitud no válida (por ejemplo, fallos de validación del ID de la llamada a la herramienta Cloud Code Assist) se tratan como aptos para conmutación por error y usan los mismos períodos de enfriamiento.

Los períodos de enfriamiento usan backoff exponencial:

* 1 minuto
* 5 minutos
* 25 minutos
* 1 hora (límite máximo)

El estado se almacena en `auth-profiles.json` bajo `usageStats`:

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

<div id="billing-disables">
  ## Desactivaciones por facturación
</div>

Los fallos de facturación/crédito (por ejemplo, “insufficient credits” / “credit balance too low”) se tratan como motivo de conmutación por error, pero generalmente no son transitorios. En lugar de un período corto de espera, OpenClaw marca el perfil como **desactivado** (con un intervalo de reintento más largo) y pasa al siguiente perfil/proveedor.

El estado se almacena en `auth-profiles.json`:

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

Valores predeterminados:

* El backoff de facturación comienza en **5 horas**, se duplica con cada error de facturación y se limita a **24 horas**.
* Los contadores de backoff se reinician si el perfil no ha fallado durante **24 horas** (configurable).

<div id="model-fallback">
  ## Conmutación por error de modelo
</div>

Si todos los perfiles de un proveedor fallan, OpenClaw pasa al siguiente modelo en
`agents.defaults.model.fallbacks`. Esto se aplica a fallos de autenticación, límites de
uso y tiempos de espera que agotaron la rotación de perfiles (otros errores no hacen avanzar la conmutación por error).

Cuando una ejecución comienza con una anulación de modelo (hooks o CLI), la conmutación por error sigue terminando en
`agents.defaults.model.primary` después de probar todos los modelos de respaldo configurados.

<div id="related-config">
  ## Configuración relacionada
</div>

Consulta la [configuración de Gateway](/es/gateway/configuration) para:

* `auth.profiles` / `auth.order`
* `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
* `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
* `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
* enrutamiento de `agents.defaults.imageModel`

Consulta [Modelos](/es/concepts/models) para una descripción general más amplia de la selección de modelos y la conmutación por error.