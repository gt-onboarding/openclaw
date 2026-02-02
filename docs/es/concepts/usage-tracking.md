---
title: Seguimiento de uso
summary: "Vistas de uso y cuota y requisitos de credenciales"
read_when:
  - Estás integrando las vistas de uso/cuota de proveedores
  - Necesitas explicar el comportamiento del seguimiento de uso o los requisitos de autenticación
---

<div id="usage-tracking">
  # Seguimiento del uso
</div>

<div id="what-it-is">
  ## Qué es
</div>

- Extrae el uso y la cuota del proveedor directamente de sus endpoints de uso.
- Sin costos estimados; solo las ventanas informadas por el proveedor.

<div id="where-it-shows-up">
  ## Dónde se muestra
</div>

- `/status` en chats: tarjeta de estado rica en emojis con tokens de sesión + costo estimado (solo con API key). El uso del proveedor se muestra para el **proveedor de modelo actual** cuando está disponible.
- `/usage off|tokens|full` en chats: pie de uso por respuesta (con OAuth solo muestra tokens).
- `/usage cost` en chats: resumen local de costos, agregado a partir de los registros de sesión de OpenClaw.
- CLI: `openclaw status --usage` imprime un desglose completo por proveedor.
- CLI: `openclaw channels list` imprime la misma instantánea de uso junto con la configuración del proveedor (usa `--no-usage` para omitirlo).
- Barra de menús de macOS: sección "Usage" dentro de Context (solo si está disponible).

<div id="providers-credentials">
  ## Proveedores + credenciales
</div>

- **Anthropic (Claude)**: tokens OAuth en perfiles de autenticación.
- **GitHub Copilot**: tokens OAuth en perfiles de autenticación.
- **Gemini CLI**: tokens OAuth en perfiles de autenticación.
- **Antigravity**: tokens OAuth en perfiles de autenticación.
- **OpenAI Codex**: tokens OAuth en perfiles de autenticación (se usa `accountId` cuando está presente).
- **MiniMax**: clave de API (clave del plan de codificación; `MINIMAX_CODE_PLAN_KEY` o `MINIMAX_API_KEY`); usa la ventana del plan de codificación de 5 horas.
- **z.ai**: clave de API mediante env/config/auth store.

El uso no se muestra si no existen credenciales OAuth/API correspondientes.