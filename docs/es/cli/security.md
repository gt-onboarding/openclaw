---
title: Seguridad
summary: "Referencia de la CLI para `openclaw security` (audita y corrige errores de seguridad comunes y peligrosos)"
read_when:
  - Quieres ejecutar una auditoría rápida de seguridad de la configuración y el estado
  - Quieres aplicar correcciones seguras sugeridas (chmod, endurecer valores predeterminados)
---

<div id="openclaw-security">
  # `openclaw security`
</div>

Herramientas de seguridad (auditoría + correcciones opcionales).

Contenido relacionado:

* Guía de seguridad: [Seguridad](/es/gateway/security)

<div id="audit">
  ## Auditoría
</div>

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

La auditoría advierte cuando varios remitentes de DM comparten la sesión principal y recomienda `session.dmScope="per-channel-peer"` (o `per-account-channel-peer` para canales con múltiples cuentas) para bandejas de entrada compartidas.
También advierte cuando se usan modelos pequeños (`<=300B`) sin sandbox y con herramientas web/navegador habilitadas.
