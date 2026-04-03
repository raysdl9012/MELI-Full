# Functional Spec — MeLi Explorer

**Versión:** 1.1  
**Fecha:** 2026-04-02  
**Plataforma:** iOS 17+  
**Estado:** Inicial

---

## 1. Objetivo

MeLi Explorer es una app iOS que permite al usuario explorar productos del marketplace Mercado Libre, ver el detalle de cada producto y guardar productos de interés con persistencia local entre reinicios.

El alcance es intencionalmente acotado: se prioriza la solidez de las decisiones técnicas y la claridad de la spec por sobre la completitud funcional.

---

## 2. Navegación

La app usa una estructura de **stack lineal** (`NavigationStack`). No hay TabBar.

```
Search (lista completa al abrir)
  ├── [botón "Guardados"] ──────────────────────→ Saved
  └── [tap en producto] ────────────────────────→ Detail
                                                    └── [botón "Guardar"]
```

- Al abrir la app, Search carga y muestra todos los productos disponibles.
- El campo de búsqueda filtra la lista en tiempo real.
- Desde **Search** el usuario puede navegar a **Detail** (tap en producto) o a **Saved** (botón en la barra de navegación).
- Desde **Detail** el usuario puede guardar el producto.
- El botón Back nativo de iOS es el mecanismo de retorno en todos los casos.

---

## 3. Pantallas

### 3.1 Search — Exploración y búsqueda de productos

**Descripción:** Pantalla inicial. Muestra todos los productos al abrir y permite filtrarlos en tiempo real.

**Elementos de UI:**
- Barra de navegación con título "Explorar" y botón "Guardados" (ícono bookmark) a la derecha.
- Campo de búsqueda (`searchable`) integrado en la barra de navegación.
- Área de contenido principal que cambia según el estado.

**Estados:**

| Estado | Cuándo ocurre | Qué ve el usuario |
|--------|---------------|-------------------|
| `loading` | Al abrir la app, mientras se carga el JSON | Spinner (`ProgressView`) centrado |
| `results` | JSON cargado, sin filtro activo o filtro con 3+ caracteres con coincidencias | Lista vertical de tarjetas de producto |
| `filtering` | El usuario escribe 1 o 2 caracteres | Lista completa sin filtrar (igual que `results`) |
| `empty` | Filtro activo con 3+ caracteres y sin coincidencias | Mensaje: "Sin resultados para '[query]'" con ícono |
| `error` | Falla al cargar el JSON | Mensaje de error con botón "Reintentar" |

> **Nota:** El estado `filtering` (1-2 caracteres) muestra la lista completa intencionalmente. El filtro solo se activa con 3 o más caracteres. Esto evita resultados confusos con queries muy cortos y está declarado como decisión de UX.

**Tarjeta de producto en lista:**
- Imagen del producto (thumbnail, aspect ratio fijo 1:1)
- Nombre del producto (máximo 2 líneas, truncado con ellipsis)
- Precio formateado (ej: `$ 125.990`)

**Comportamiento de búsqueda:**
- El filtro se aplica en tiempo real sobre los productos ya cargados en memoria.
- El filtro se activa únicamente cuando el campo tiene **3 o más caracteres**.
- Con 0, 1 o 2 caracteres, se muestra la lista completa sin filtrar.
- El filtro es case-insensitive y busca coincidencias en el nombre del producto.
- Al borrar el campo de búsqueda, la lista vuelve a mostrar todos los productos.
- No hay llamadas de red ni recarga del JSON al buscar.

**Criterios de aceptación:**
- [ ] Al abrir la app se muestra el estado `loading` y luego la lista completa de productos.
- [ ] Con 1 o 2 caracteres en el campo, la lista no se filtra.
- [ ] Con 3 o más caracteres, la lista se filtra en tiempo real por nombre.
- [ ] Si no hay coincidencias con 3+ caracteres, se muestra el estado `empty` con el query visible.
- [ ] Si falla la carga del JSON, se muestra el estado `error` con opción de reintentar.
- [ ] El botón "Guardados" es visible en todo momento desde esta pantalla.
- [ ] Tap en una tarjeta navega al Detail del producto seleccionado.

---

### 3.2 Detail — Detalle de producto

**Descripción:** Muestra la información completa de un producto y permite guardarlo.

**Elementos de UI:**
- Barra de navegación con título del producto (truncado si es largo) y botón Back nativo.
- Contenido scrollable con la siguiente jerarquía visual de arriba a abajo:
  1. Imagen grande del producto (full width, aspect ratio 4:3)
  2. Nombre completo del producto (título prominente)
  3. Precio (destacado, tamaño mayor al resto del texto)
  4. Categoría y Vendedor (texto secundario)
  5. Rating con estrellas y cantidad de reseñas
  6. Stock disponible
  7. Descripción completa del producto
- Botón "Guardar" fijo en la parte inferior, fuera del área de scroll.

**Estados del botón Guardar:**

| Estado | Apariencia | Acción al tocar |
|--------|------------|-----------------|
| No guardado | "Guardar" (filled, color primario) | Persiste el producto y cambia al estado guardado |
| Guardado | "Guardado ✓" (outlined, color secundario) | Sin acción (el producto ya está guardado) |

**Criterios de aceptación:**
- [ ] Todos los campos se muestran con los datos del producto seleccionado.
- [ ] Si un campo no tiene datos (ej: sin imagen), se muestra un placeholder visual.
- [ ] Al presionar "Guardar", el botón cambia de estado de forma inmediata (sin delay perceptible).
- [ ] Al volver a abrir el Detail de un producto ya guardado, el botón aparece en estado "Guardado ✓".
- [ ] El contenido es scrollable cuando excede la altura de la pantalla.
- [ ] El botón "Guardar" siempre es visible, independientemente del scroll.

---

### 3.3 Saved — Productos guardados

**Descripción:** Lista de productos guardados por el usuario, con persistencia entre reinicios.

**Elementos de UI:**
- Barra de navegación con título "Guardados" y botón Back nativo.
- Lista de tarjetas de producto (mismo diseño que en Search).
- Soporte para eliminar con swipe-to-delete nativo de iOS.

**Estados:**

| Estado | Cuándo ocurre | Qué ve el usuario |
|--------|---------------|-------------------|
| `empty` | No hay productos guardados | Mensaje centrado: "Todavía no guardaste ningún producto" con ícono bookmark |
| `results` | Hay al menos un producto guardado | Lista de tarjetas ordenadas por fecha de guardado (más reciente primero) |

**Criterios de aceptación:**
- [ ] Los productos guardados persisten entre reinicios de la app.
- [ ] El orden es cronológico inverso (último guardado aparece primero).
- [ ] Swipe-to-delete elimina el producto de la lista y de la persistencia local.
- [ ] Tap en una tarjeta navega al Detail del producto.
- [ ] Al eliminar todos los productos, se muestra el estado `empty`.
- [ ] El estado del botón "Guardar" en Detail es consistente con el estado real de persistencia.

---

## 4. Flujos completos

### Flujo 1: Abrir la app y explorar productos
1. Usuario abre la app → ve estado `loading` brevemente.
2. App termina de cargar el JSON → muestra todos los productos en lista.
3. Usuario navega por la lista sin buscar.
4. Tap en un producto → navega a Detail.

### Flujo 2: Filtrar y guardar un producto
1. Desde Search, usuario escribe en el campo de búsqueda.
2. Con menos de 3 caracteres → la lista no se filtra.
3. Al tercer carácter → la lista filtra en tiempo real por nombre.
4. Usuario toca una tarjeta → navega a Detail.
5. Presiona "Guardar" → botón cambia a "Guardado ✓" inmediatamente.
6. Presiona Back → regresa a Search con el filtro activo.

### Flujo 3: Ver y gestionar productos guardados
1. Desde Search, usuario toca el ícono bookmark.
2. App navega a Saved con la lista de productos guardados.
3. Usuario hace swipe-to-delete sobre un producto → se elimina.
4. Si era el último, aparece el estado `empty`.
5. Usuario presiona Back → regresa a Search.

### Flujo 4: Búsqueda sin resultados
1. Usuario escribe 3+ caracteres sin coincidencias.
2. App muestra estado `empty` con el query ingresado.
3. Usuario borra el campo → la lista vuelve a mostrar todos los productos.

### Flujo 5: Error de carga
1. App intenta cargar el JSON al abrir.
2. Ocurre un error (archivo no encontrado o JSON malformado).
3. Se muestra el estado `error` con mensaje y botón "Reintentar".
4. Usuario toca "Reintentar" → se intenta cargar nuevamente.

---

## 5. Accesibilidad mínima

- Todas las imágenes tienen `accessibilityLabel` descriptivo.
- Botones tienen labels claros y legibles por VoiceOver.
- Los estados de carga y error tienen feedback visible (no dependen solo del color).
- Tamaños de fuente respetan Dynamic Type de iOS.
- El contraste de texto cumple con WCAG AA como mínimo.

---

## 6. Supuestos

- **Carga inicial:** El JSON se carga una sola vez al abrir la app. No se recarga mientras la sesión está activa.
- **Filtro local:** La búsqueda opera 100% en memoria sobre los productos ya cargados. No hay red.
- **Filtro por nombre:** La búsqueda filtra únicamente por el campo `name` del producto. No filtra por categoría, vendedor ni descripción.
- **Mínimo de caracteres:** Con menos de 3 caracteres en el campo de búsqueda, se muestra la lista completa. Decisión de UX deliberada para evitar resultados confusos.
- **Botón Guardar en Detail:** Al tocar un producto ya guardado, el botón no ejecuta ninguna acción. Para eliminar, el usuario debe ir a Saved y usar swipe-to-delete. Esto previene eliminaciones accidentales.
- **Datos del JSON:** Se asume que el JSON tiene al menos 20 productos con todos los campos cubiertos. Los campos opcionales tienen valores de fallback definidos en el modelo.
- **Layout:** Las decisiones de layout (espaciados, tamaños exactos, colores) son decisiones de implementación no especificadas en esta spec. La jerarquía visual de Detail sí está definida en la sección 3.2 por ser la pantalla más compleja.
- **Idioma:** La app está en español. No hay localización dinámica.
- **Orientación:** Solo portrait. El comportamiento en landscape no está definido.

---

## 7. No objetivos

Las siguientes funcionalidades están **explícitamente fuera de scope**:

- Checkout o proceso de compra.
- Recomendaciones de productos.
- Notificaciones push.
- Autenticación o cuentas de usuario.
- Paginación de resultados.
- Filtros, sorting o categorías de búsqueda.
- Búsqueda por campos distintos al nombre (categoría, vendedor, descripción).
- Compartir productos.
- Comparar productos.
- Modo oscuro explícito (se usa el modo del sistema por defecto de SwiftUI).
- Soporte landscape.
- Localización (solo español).
- Backend propio o integración de red real.
- Analytics o tracking.
- Tests de UI automatizados (XCUITest).

---

## 8. Cambios respecto a versiones anteriores

| Versión | Decisión cambiada | Cambio | Motivo |
|---------|-------------------|--------|--------|
| v1.0 → v1.1 | Estado inicial de Search | Se eliminó `idle`, se agregó `loading` al abrir + todos los productos visibles | Mejor experiencia: el usuario ve contenido de inmediato sin necesidad de buscar |
| v1.0 → v1.1 | Mecanismo de búsqueda | De botón "Buscar" a filtro en tiempo real con mínimo 3 caracteres | Más natural en iOS, consistente con patrones de la plataforma |
| v1.0 → v1.1 | Layout de pantallas | Removido de la spec, declarado como supuesto de implementación | No hay diseñador ni Figma; la jerarquía lógica de Detail es suficiente |
| v1.0 → v1.1 | Estado `filtering` | Agregado explícitamente para 1-2 caracteres | Documenta el comportamiento del umbral de 3 caracteres sin ambigüedad |
