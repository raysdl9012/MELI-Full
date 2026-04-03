# Validation Plan — MeLi Explorer

**Versión:** 1.1  
**Fecha:** 2026-04-02  
**Plataforma:** iOS 17+  
**Estado:** Inicial

---

## 1. Objetivo

Este documento responde una pregunta concreta: **¿cómo sabemos que la app funciona correctamente antes de entregarla?**

Para responderla, el plan define tres cosas:

1. **Qué se verifica con tests automatizados** — lógica de negocio y persistencia que puede ejecutarse sin intervención humana, de forma repetible en cualquier momento.
2. **Qué se verifica manualmente** — flujos de navegación, estados visuales y comportamientos que dependen del renderizado de SwiftUI y requieren correr la app en un simulador.
3. **Qué riesgo queda abierto** — lo que conscientemente no se cubre, con una justificación explícita de por qué se acepta ese riesgo para el scope actual.

La combinación de las tres cubre todos los criterios de aceptación definidos en la `functional_spec.md`.

---

## 2. Qué se valida con tests automatizados

Los tests automatizados cubren las capas que no dependen de SwiftUI ni de interacción visual. Se usan exclusivamente para lógica determinista y aislable.

### `SearchViewModel` — 7 casos

| # | Caso | Tipo |
|---|------|------|
| 1 | Carga inicial exitosa → estado `.results` con todos los productos | Unitario |
| 2 | Error en repositorio al cargar → estado `.error` con mensaje no vacío | Unitario |
| 3 | Query con 1 carácter → lista completa sin filtrar | Unitario |
| 4 | Query con 2 caracteres → lista completa sin filtrar | Unitario |
| 5 | Query con 3+ caracteres con coincidencias → lista filtrada correctamente | Unitario |
| 6 | Query con 3+ caracteres sin coincidencias → estado `.empty` con query correcto | Unitario |
| 7 | Borrar campo después de filtrar → lista vuelve a mostrar todos los productos | Unitario |

Todos los casos usan `MockProductRepository` inyectado por inicializador. No leen archivos ni tocan disco.

### `SavedProductStore` — 4 casos

| # | Caso | Tipo |
|---|------|------|
| 8 | Guardar un producto → `isSaved` retorna `true` | Unitario |
| 9 | Eliminar un producto guardado → `isSaved` retorna `false` | Unitario |
| 10 | Guardar y leer desde el contexto → producto recuperable vía `FetchDescriptor` | Integración (en memoria) |
| 11 | Guardar el mismo producto dos veces → no se crean duplicados | Integración (en memoria) |

Los casos 10 y 11 usan un `ModelContainer` configurado con `isStoredInMemoryOnly: true`. Esto aísla los tests del store real del dispositivo y garantiza que cada test parte de un estado limpio.

---

## 3. Qué se valida manualmente

La validación manual cubre flujos de UI, navegación, estados visuales y comportamientos que dependen del renderizado de SwiftUI. Se ejecuta corriendo la app en el simulador de iOS.

### Flujos de navegación

| # | Qué se valida | Resultado esperado |
|---|---------------|--------------------|
| M1 | Abrir la app | Se muestra spinner brevemente y luego la lista completa de productos |
| M2 | Tap en un producto desde Search | Navega a Detail con los datos correctos del producto |
| M3 | Tap en botón "Guardados" desde Search | Navega a Saved |
| M4 | Tap en Back desde Detail | Regresa a Search con el estado anterior intacto (filtro activo si lo había) |
| M5 | Tap en Back desde Saved | Regresa a Search |
| M6 | Tap en un producto desde Saved | Navega a Detail con los datos correctos |

### Estados de Search

| # | Qué se valida | Resultado esperado |
|---|---------------|--------------------|
| M7 | Escribir 1 carácter en el campo | La lista no se filtra, muestra todos los productos |
| M8 | Escribir 2 caracteres en el campo | La lista no se filtra, muestra todos los productos |
| M9 | Escribir 3+ caracteres con coincidencias | La lista se filtra en tiempo real |
| M10 | Escribir 3+ caracteres sin coincidencias | Se muestra estado empty con el query visible |
| M11 | Borrar el campo de búsqueda | La lista vuelve a mostrar todos los productos |

### Estados de Detail

| # | Qué se valida | Resultado esperado |
|---|---------------|--------------------|
| M12 | Abrir Detail de producto no guardado | Botón muestra "Guardar" en estado filled |
| M13 | Presionar "Guardar" | Botón cambia a "Guardado ✓" de forma inmediata |
| M14 | Cerrar y volver a abrir el mismo Detail | Botón aparece en estado "Guardado ✓" |
| M15 | Abrir Detail de producto con imagen | Imagen se carga correctamente desde URL |
| M16 | Abrir Detail de producto sin imagen | Se muestra placeholder visual |
| M17 | Contenido que excede la pantalla | El contenido es scrollable, el botón "Guardar" permanece fijo abajo |

### Estados de Saved

| # | Qué se valida | Resultado esperado |
|---|---------------|--------------------|
| M18 | Abrir Saved sin productos guardados | Se muestra estado empty con mensaje e ícono |
| M19 | Abrir Saved con productos guardados | Se muestra la lista ordenada (más reciente primero) |
| M20 | Swipe-to-delete sobre un producto | El producto se elimina de la lista |
| M21 | Eliminar el último producto | Aparece el estado empty |
| M22 | Reiniciar la app con productos guardados | Los productos persisten entre reinicios |

### Accesibilidad mínima

La accesibilidad se valida activando **VoiceOver**, el lector de pantalla integrado en iOS. VoiceOver está diseñado para personas con discapacidad visual: cuando está activo, el sistema lee en voz alta el contenido de cada elemento de la pantalla a medida que el usuario lo navega con gestos. Para que funcione correctamente, cada elemento visual necesita un label descriptivo — una imagen sin label es anunciada simplemente como "imagen", sin dar contexto al usuario.

Para activarlo en el simulador: **Configuración → Accesibilidad → VoiceOver → Activar.**

| # | Qué se valida | Resultado esperado |
|---|---------------|--------------------|
| M23 | Navegar Search con VoiceOver activo | Las imágenes de productos son anunciadas con el nombre del producto, los botones con su función |
| M24 | Navegar Detail con VoiceOver activo | El botón "Guardar" / "Guardado ✓" es anunciado con su estado actual |
| M25 | Cambiar tamaño de texto del sistema (Dynamic Type) | El texto de la app escala correctamente sin truncarse de forma incorrecta |

---

## 4. Riesgo residual

El riesgo residual documenta lo que conscientemente no está cubierto, con una justificación explícita de por qué se acepta para el scope actual.

| Riesgo | Impacto | Por qué se acepta |
|--------|---------|-------------------|
| No hay tests de UI automatizados (XCUITest) | Medio — los flujos de navegación podrían romperse sin detección automática | El scope del challenge no lo requiere; se mitiga con el plan de validación manual |
| No hay tests para `DetailViewModel` | Bajo — solo maneja `isSaved: Bool`, cubierto indirectamente por los tests de `SavedProductStore` | Agregar tests específicos tendría retorno marginal para el scope actual |
| `AsyncImage` no tiene retry manual | Bajo — una imagen que falla muestra placeholder, el usuario no puede reintentar | Aceptado para v1.0; en producción se consideraría una librería con caché y retry |
| Errores de escritura en SwiftData no se exponen al usuario | Bajo — en caso de fallo silencioso el producto simplemente no aparece en Saved | Aceptado para v1.0; en producción requeriría feedback al usuario y logging remoto |
| No se testea con datos reales de la API de Mercado Libre | Medio — el modelo podría no mapear correctamente campos reales | Aceptado; el JSON local está diseñado para simular la estructura real. La integración real requeriría un sprint dedicado |
| Sin tests para `@Query` reactivo de SwiftData en `SavedView` | Bajo — depende del framework de Apple, no de lógica propia | Se confía en el comportamiento del framework; se valida manualmente en M19–M22 |

---

## 5. Cambios respecto a versiones anteriores

| Versión | Cambio | Motivo |
|---------|--------|--------|
| v1.0 → v1.1 | Sección 1 — Objetivo | Reescrito para ser claro y directo; explica qué contiene el documento y cómo usarlo |
| v1.0 → v1.1 | Sección 3 — VoiceOver | Agregada explicación de qué es VoiceOver antes de los casos de validación |
Si