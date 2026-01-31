---
title: Configuración de modelos
summary: "Exploración: configuración de modelos, perfiles de autenticación y comportamiento de conmutación por error"
read_when:
  - Exploración de ideas futuras sobre selección de modelos y perfiles de autenticación
---

<div id="model-config-exploration">
  # Configuración de modelos (Exploración)
</div>

Este documento presenta **ideas** para una futura configuración de modelos. No es una
especificación lista para producción. Para el comportamiento actual, consulta:

* [Modelos](/es/concepts/models)
* [Conmutación por error de modelos](/es/concepts/model-failover)
* [OAuth + perfiles](/es/concepts/oauth)

<div id="motivation">
  ## Motivación
</div>

Los operadores quieren:

* Varios perfiles de autenticación por proveedor (personal vs trabajo).
* Una selección sencilla de `/model` con opciones de respaldo predecibles.
* Una separación clara entre modelos de texto y modelos con capacidad de generar imágenes.

<div id="possible-direction-high-level">
  ## Posible orientación (a nivel general)
</div>

* Mantén la selección de modelos simple: `provider/model` con alias opcionales.
* Permite que los proveedores tengan varios perfiles de autenticación, con un orden explícito.
* Usa una lista de reserva global para que todas las sesiones hagan conmutación por error de forma consistente.
* Solo sobrescribe el enrutamiento de imágenes cuando se configure de forma explícita.

<div id="open-questions">
  ## Preguntas abiertas
</div>

* ¿La rotación de perfiles debería ser por proveedor o por modelo?
* ¿Cómo debería la UI mostrar la selección de perfil para una sesión?
* ¿Cuál es la vía de migración más segura desde las claves de configuración heredadas?