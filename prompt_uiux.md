# Prompt — Diseño UI/UX: MeLi Explorer

## Contexto

Soy desarrollador iOS y estoy construyendo una app llamada **MeLi Explorer** para un challenge técnico. La app permite explorar productos del marketplace Mercado Libre, ver el detalle de cada uno y guardar productos de interés.

Necesito que actúes como un diseñador UI/UX senior especializado en apps iOS nativas y me entregues **mockups de alta fidelidad** para las 3 pantallas de la app.

---

## Lo que ya está definido (no cambiar)

### Plataforma
- iOS 17+, SwiftUI nativo
- Tema: sigue el modo del sistema (light/dark automático — la app debe verse bien en ambos)
- Orientación: solo portrait

### Estructura de navegación
- Stack lineal con `NavigationStack`. Sin TabBar.
- Flujo: **Search → Detail** (tap en producto) y **Search → Saved** (botón bookmark en navbar)

### Pantalla 1 — Search
Elementos presentes:
- Barra de navegación con título "Explorar" y botón bookmark (ícono) a la derecha
- Campo de búsqueda integrado en la navbar (`searchable`)
- Lista vertical de tarjetas de producto

Tarjeta de producto contiene:
- Imagen del producto (thumbnail)
- Nombre del producto (máximo 2 líneas)
- Precio formateado (ej: `$ 125.990`)

Estados que deben estar diseñados:
- `loading` — spinner centrado
- `results` — lista de tarjetas
- `empty` — mensaje "Sin resultados para '[query]'" con ícono
- `error` — mensaje de error con botón "Reintentar"

### Pantalla 2 — Detail
Elementos en orden vertical (de arriba a abajo):
1. Imagen grande del producto (full width, aspect ratio 4:3)
2. Nombre completo (título prominente)
3. Precio (destacado, mayor tamaño que el resto)
4. Categoría y Vendedor (texto secundario)
5. Rating con estrellas y cantidad de reseñas
6. Stock disponible
7. Descripción completa del producto (scrollable)
8. Botón "Guardar" fijo en la parte inferior, fuera del scroll

Estados del botón:
- `no guardado` → "Guardar" (filled, color primario)
- `guardado` → "Guardado ✓" (outlined, color secundario)

### Pantalla 3 — Saved
Elementos presentes:
- Barra de navegación con título "Guardados" y botón Back nativo
- Lista de tarjetas (mismo componente que en Search)
- Soporte visual para swipe-to-delete

Estados que deben estar diseñados:
- `empty` — mensaje "Todavía no guardaste ningún producto" con ícono bookmark
- `results` — lista de productos guardados

---

## Lo que necesito que decidas vos (decisiones de diseño libre)

- **Paleta de colores** — diseñala desde cero. Debe funcionar en light y dark mode. Debe tener un color primario para acciones principales (botón Guardar), un color secundario, y colores de fondo y texto adecuados para ambos modos.
- **Tipografía** — elegí una jerarquía tipográfica clara: título, subtítulo, cuerpo, precio, texto secundario. Puede ser SF Pro (la fuente del sistema iOS) u otra que funcione bien en SwiftUI.
- **Estilo visual general** — el tono puede ser moderno, limpio, premium, vibrante, etc. Lo que consideres más adecuado para una app de marketplace en iOS.
- **Iconografía** — SF Symbols de Apple preferentemente, pero podés proponer alternativas.
- **Espaciados y bordes** — definí un sistema de espaciado consistente (padding, corner radius de tarjetas, separación entre elementos).

---

## Entregables esperados

1. **Mockups de alta fidelidad** para las 3 pantallas en sus estados principales:
   - Search en estado `results` (con tarjetas visibles)
   - Search en estado `empty`
   - Detail con botón "Guardar"
   - Detail con botón "Guardado ✓"
   - Saved en estado `results`
   - Saved en estado `empty`

2. **Style Guide** que el desarrollador puede usar como referencia:
   - Paleta de colores con hex codes para light y dark mode
   - Escala tipográfica (tamaños, pesos)
   - Especificaciones de componentes: tarjeta de producto (corner radius, padding, sombra), botones (altura, radio, estados)
   - Sistema de espaciado (valores base usados en toda la app)

---

## Restricciones técnicas para el desarrollador

- Todo se implementa en SwiftUI nativo. No hay librerías de UI externas.
- Los colores deben definirse como `Color` assets con variantes light/dark, o como valores hex que el dev puede configurar en el Asset Catalog de Xcode.
- Las imágenes de productos se cargan desde URLs con `AsyncImage` — el diseño debe contemplar un estado de carga (placeholder) y un estado de error para las imágenes.
- El botón "Guardar" debe estar implementable como un botón SwiftUI estándar con dos estados visuales claramente diferenciados.
- El swipe-to-delete en Saved usa el componente nativo de SwiftUI (`.onDelete`) — no es necesario diseñar una UI personalizada para esto.
