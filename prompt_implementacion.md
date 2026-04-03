# Prompt Maestro — Implementación iOS: MeLi Explorer

## Instrucciones para la IA

Sos un desarrollador iOS senior especializado en SwiftUI. Vas a ayudarme a implementar una app llamada **MeLi Explorer** bloque por bloque. Cada bloque es una unidad de trabajo independiente con archivos específicos a generar.

**Reglas generales que aplican a todos los bloques:**
- Usá Swift idiomático y convenciones de Apple (camelCase, nombres descriptivos, access control explícito)
- Cada archivo debe tener comentarios de documentación (`///`) en tipos y métodos públicos
- No uses librerías externas. Solo frameworks nativos de Apple
- El código debe compilar sin warnings en Xcode 16, iOS 17+
- Respetá estrictamente la arquitectura MVVM definida en este documento
- Cuando generes un bloque, generá **todos** los archivos del bloque completos, listos para copiar en Xcode
- Al final de cada bloque indicá claramente: qué archivos se crearon, en qué carpeta van, y si hay alguna configuración manual necesaria en Xcode

---

## Contexto del proyecto

### Nombre
MeLiExplorer

### Plataforma
iOS 17+, SwiftUI, Swift 5.9+, Xcode 16

### Arquitectura: MVVM estricto

La regla de dependencias es unidireccional y no se puede violar:

```
View → ViewModel → Repository → Model
```

- La **View** solo renderiza y captura eventos. Nunca tiene lógica de negocio.
- El **ViewModel** maneja el estado de UI y orquesta llamadas al repositorio. Usa `@Observable`.
- El **Repository** abstrae el origen de datos. Los ViewModels dependen del protocolo, nunca de la implementación concreta.
- El **Model** es el núcleo del dominio. Sin dependencias de SwiftUI ni SwiftData (excepto `SavedProduct`).

### Stack tecnológico
- Estado: `@Observable` (NO usar `ObservableObject`)
- Navegación: `NavigationStack` con `NavigationPath` tipado
- Persistencia: SwiftData (`@Model`)
- Imágenes: `AsyncImage` nativo
- Datos: JSON local en el bundle (`products.json`)
- Testing: Swift Testing (NO usar XCTest)

### Estructura de carpetas

```
MeLiExplorer/
├── App/
│   ├── MeLiExplorerApp.swift
│   └── ContentView.swift
├── Core/
│   ├── Models/
│   │   ├── Product.swift
│   │   └── SavedProduct.swift
│   ├── Repositories/
│   │   ├── ProductRepository.swift
│   │   └── JSONProductRepository.swift
│   └── Persistence/
│       └── SavedProductStore.swift
├── Features/
│   ├── Search/
│   │   ├── SearchViewModel.swift
│   │   └── SearchView.swift
│   ├── Detail/
│   │   ├── DetailViewModel.swift
│   │   └── DetailView.swift
│   └── Saved/
│       ├── SavedViewModel.swift
│       └── SavedView.swift
├── Shared/
│   └── Components/
│       ├── ProductCardView.swift
│       └── EmptyStateView.swift
└── Resources/
    └── products.json
```

### Navegación
- Stack lineal, sin TabBar
- Search es la pantalla raíz
- Search → Detail: tap en tarjeta de producto (navega pasando el `Product`)
- Search → Saved: botón bookmark en la navbar
- Back nativo de iOS para volver en ambos casos

---

## Diseño visual (Style Guide)

> ✅ Style Guide completado a partir del DESIGN.md y mockups de alta fidelidad entregados por la IA de UI/UX.
> El sistema de diseño se llama **"The Curated Marketplace"**. El tono es editorial, premium y limpio.
> Filosofía central: sin líneas divisoras, profundidad creada por capas de color, glassmorphism en navbars.

---

### Colores — Light Mode

Definir como `Color` assets en el Asset Catalog de Xcode con variantes light/dark.

| Token | Nombre en código | Light | Dark |
|-------|-----------------|-------|------|
| Canvas principal | `AppSurface` | `#fcf8fb` | `#1b1b1d` |
| Sección secundaria | `AppSurfaceContainerLow` | `#f6f3f5` | `#2a2a2c` |
| Tarjetas / inputs | `AppSurfaceContainerLowest` | `#ffffff` | `#2f2f31` |
| Acción primaria | `AppPrimary` | `#0058bc` | `#7ab3ff` |
| Contenedor primario | `AppPrimaryContainer` | `#0070eb` | `#004494` |
| Texto principal | `AppOnBackground` | `#1b1b1d` | `#e5e5e7` |
| Texto secundario | `AppOnSurfaceVariant` | `#414755` | `#9a9aac` |
| Texto sobre primario | `AppOnPrimary` | `#ffffff` | `#ffffff` |
| Contenedor secundario | `AppSecondaryContainer` | `#e8eaf6` | `#2d2f4a` |
| Texto sobre secundario | `AppOnSecondaryContainer` | `#1a1c3a` | `#c5c7f0` |
| Estado guardado | `AppTertiary` | `#9e3d00` | `#ffb68a` |
| Ghost border (dark only) | `AppOutlineVariant` | `transparent` | `rgba(255,255,255,0.15)` |

> **Regla crítica — "No-Line Rule":** No usar bordes de 1px para separar secciones. La separación se logra exclusivamente con cambios de color de fondo entre capas (`AppSurface` → `AppSurfaceContainerLow` → `AppSurfaceContainerLowest`).

> **Dark Mode:** En dark mode, usar `AppOutlineVariant` al 15% de opacidad como "Ghost Border" en tarjetas para mantener los límites visibles sin romper el flujo visual.

---

### Tipografía

Fuente: **Inter** (preferida por el design system para mayor carácter) o SF Pro como fallback nativo.
En SwiftUI: registrar Inter via `@font-face` o usar `.custom("Inter-Bold", size: 24)`. Si Inter no está disponible, usar `.system` con los pesos equivalentes.

| Rol | Nombre en código | Fuente | Peso | Tamaño | Color token |
|-----|-----------------|--------|------|--------|-------------|
| Título editorial hero | `AppDisplayMd` | Inter | Bold | 44pt | `AppOnBackground` |
| Título de sección (ej: "Selección Curada") | `AppHeadlineSm` | Inter | Bold | 24pt | `AppOnBackground` |
| Nombre de producto en tarjeta | `AppTitleMd` | Inter | SemiBold | 17pt | `AppOnBackground` |
| Precio destacado | `AppPriceLg` | Inter | Bold | 20pt | `AppOnBackground` |
| Precio en Detail (hero) | `AppPriceHero` | Inter | Bold | 28pt | `AppOnBackground` |
| Cuerpo / descripción | `AppBodyMd` | Inter | Regular | 15pt | `AppOnBackground` |
| Metadata / vendedor / categoría | `AppBodySm` | Inter | Regular | 14pt | `AppOnSurfaceVariant` |
| Labels de tags (ej: "RESULTADOS") | `AppLabelMd` | Inter | SemiBold | 12pt | `AppOnSurfaceVariant` |
| Tags de badge (ej: "TOP RATED") | `AppLabelSm` | Inter | SemiBold | 11pt | `AppPrimary` |

> **Regla de tags:** Los labels de categoría como "RESULTADOS" o "TOP RATED" usan ALL CAPS con letter spacing de +5% (`kerning: 0.6`) para evocar estética editorial de boutique.

---

### Espaciado

Sistema de espaciado basado en múltiplos de 4pt.

| Token | Valor | Uso típico |
|-------|-------|------------|
| `space1` | 4pt | Separación mínima entre elementos inline |
| `space2` | 8pt | Padding interno de badges y chips |
| `space3` | 12pt | Espaciado entre elementos dentro de una tarjeta |
| `space4` | 16pt | Padding horizontal estándar de pantalla, padding interno de tarjetas |
| `space5` | 20pt | Separación entre tarjetas en la lista |
| `space6` | 24pt | Separación entre secciones |
| `space8` | 32pt | Espaciado generoso entre bloques editoriales |
| `space10` | 40pt | Márgenes superiores de secciones hero |

> **Regla premium:** Un look "premium" es 50% tipografía y 50% espacio generoso. Preferir `space8` o `space10` para separar bloques, nunca usar `space2` donde cabe `space4`.

---

### Corner Radius

| Token | Valor | Uso |
|-------|-------|-----|
| `radiusSm` | 8pt | Badges, chips, tags pequeños |
| `radiusMd` | 12pt | Botones principales (`md` = 0.75rem) |
| `radiusLg` | 16pt | Tarjetas estándar de producto (`lg` = 1rem) |
| `radiusXl` | 24pt | Cards hero, imágenes de detalle |

---

### Sombras — Ambient Light

No usar sombras pesadas. Sistema de dos capas para elementos flotantes (barra inferior, botón Guardar fijo).

```swift
// Sombra ambiente — aplicar a botones flotantes y barras inferiores
.shadow(color: Color(hex: "1b1b1d").opacity(0.04), radius: 10, x: 0, y: 4)
.shadow(color: Color(hex: "1b1b1d").opacity(0.08), radius: 20, x: 0, y: 8)
```

Para tarjetas de producto: sombra sutil de una sola capa:
```swift
.shadow(color: Color(hex: "1b1b1d").opacity(0.06), radius: 8, x: 0, y: 2)
```

---

### Componentes — Especificaciones

#### Tarjeta de producto (`ProductCardView`)
Extraído del mockup de Search:
- Fondo: `AppSurfaceContainerLowest` (`#ffffff` / dark: `#2f2f31`)
- Corner radius: `radiusLg` (16pt)
- Padding interno: `space4` (16pt) en todos los lados
- Separación entre tarjetas: `space3` (12pt) — sin dividers, solo espacio
- Imagen: cuadrada (1:1), corner radius `radiusLg` (16pt), tamaño 80x80pt
- Layout horizontal: imagen a la izquierda, texto a la derecha con `space3` (12pt) de gap
- Nombre: `AppTitleMd` (Inter SemiBold 17pt), máximo 2 líneas, `.lineLimit(2)`
- Vendedor/Categoría: `AppBodySm` (Inter Regular 14pt, `AppOnSurfaceVariant`), 1 línea
- Precio: `AppPriceLg` (Inter Bold 20pt), separación `space1` (4pt) desde vendedor
- Sombra: sombra sutil de tarjeta (ver sección Sombras)
- En Dark Mode: agregar Ghost Border (`AppOutlineVariant` 15% opacidad)

#### Botón primario "Guardar"
- Estado no guardado: fondo `AppPrimary` (`#0058bc`), texto `AppOnPrimary` (`#ffffff`), texto "Guardar"
- Estado guardado: fondo `AppSecondaryContainer`, texto `AppOnSecondaryContainer`, texto "Guardado ✓", `.disabled(true)`
- Altura: 52pt
- Corner radius: `radiusMd` (12pt)
- Ancho: full width con padding horizontal `space4` (16pt) a cada lado
- Tipografía: Inter SemiBold 16pt
- Posición: fijo en la parte inferior con `safeAreaInset(edge: .bottom)`, fondo glassmorphism (`AppSurface` 80% opacidad + blur 20pt)

#### Barra de navegación
- Fondo: glassmorphism — `AppSurface` al 80% de opacidad con `ultraThinMaterial` de SwiftUI
- Sin línea separadora (`.toolbarBackground(.hidden, for: .navigationBar)` + implementación manual)
- Título: Inter Bold, tamaño nativo de iOS large title
- Botones de toolbar: SF Symbols con rendering `hierarchical`, color `AppPrimary`
- Ícono bookmark activo (Saved): `bookmark.fill`, color `AppPrimary`
- Ícono bookmark inactivo (Search): `bookmark`, color `AppPrimary`

#### Campo de búsqueda (`searchable`)
- Fondo: `AppSurfaceContainerLow` (`#f6f3f5`)
- Corner radius: `radiusLg` (16pt) — pill shape
- Placeholder: "Buscar en MeLi Explorer", color `AppOnSurfaceVariant`
- Ícono lupa: SF Symbol `magnifyingglass`, color `AppOnSurfaceVariant`

#### Badge / Tag editorial
- Fondo: `AppSurfaceContainerLow`
- Texto: ALL CAPS, `AppLabelSm` (Inter SemiBold 11pt), color `AppPrimary`
- Corner radius: `radiusSm` (8pt)
- Padding: 4pt vertical, 8pt horizontal
- Ejemplos: "TOP RATED", "BEST SELLER", "ULTRA LIMITADO", "RECIÉN LLEGADO"

---

### Pantalla Search — Observaciones del mockup

- Header editorial debajo del search bar: label "RESULTADOS" (ALL CAPS, `AppLabelMd`) + título grande "Selección Curada" (`AppHeadlineSm`)
- El fondo de la pantalla es `AppSurfaceContainerLow` (`#f6f3f5`), no blanco puro
- Las tarjetas flotan sobre el fondo con su fondo blanco (`AppSurfaceContainerLowest`)
- Sin separadores entre tarjetas — solo espacio `space3`

### Pantalla Detail — Observaciones del mockup

- Imagen hero: full width, sin corner radius en los bordes superiores (edge-to-edge), aspect ratio 16:9 aproximado
- Debajo de la imagen: badge "TOP RATED" + texto "CATEGORÍA • VENDEDOR" en la misma línea
- Precio con badge "X% OFF" al lado (chip pequeño con fondo `AppPrimaryContainer`)
- Rating: estrellas `star.fill` color amber/gold + número bold + "X.Xk opiniones" en `AppOnSurfaceVariant`
- Tarjetas de info (Stock / Envío): layout de dos columnas, fondo `AppSurfaceContainerLow`, corner radius `radiusLg`
- Sección vendedor: fondo `AppSurfaceContainerLow`, logo circular, nombre bold, badge verificado, métricas en 3 columnas

### Pantalla Saved — Observaciones del mockup

- Header editorial: título grande "Tu Selección" (`AppDisplayMd`) + subtítulo "X artículos guardados para más tarde." en `AppOnSurfaceVariant`
- Tarjetas tienen acento de color izquierdo (borde vertical `AppPrimary`, 3pt de ancho) para el estilo "curated"
- Ícono bookmark `bookmark.fill` color `AppPrimary` en la esquina superior derecha de cada tarjeta
- Al final de la lista: card "¿Buscás algo más?" con fondo `AppSurfaceContainerLow`, ícono corazón, CTA "Explorar Tendencias" en `AppPrimary`

---

## Modelos de datos

### `Product`
```swift
struct Product: Identifiable, Codable, Hashable {
    let id: String
    let name: String
    let price: Double
    let imageURL: String?
    let description: String
    let category: String
    let seller: String
    let rating: Double      // 0.0 - 5.0
    let reviewCount: Int
    let stock: Int
}
```

### `SavedProduct`
```swift
@Model class SavedProduct {
    var id: String
    var name: String
    var price: Double
    var imageURL: String?
    var category: String
    var seller: String
    var rating: Double
    var reviewCount: Int
    var stock: Int
    var savedAt: Date
}
```

### `products.json` — estructura
```json
[
  {
    "id": "MLA001",
    "name": "iPhone 15 Pro 256GB",
    "price": 2499990,
    "imageURL": "https://...",
    "description": "...",
    "category": "Celulares",
    "seller": "TiendaApple",
    "rating": 4.8,
    "reviewCount": 342,
    "stock": 15
  }
]
```

---

## Comportamiento de búsqueda

- Al abrir la app: se carga el JSON y se muestran **todos** los productos
- El filtro opera en memoria sobre los productos ya cargados (sin red)
- El filtro se activa con **3 o más caracteres** en el campo de búsqueda
- Con 0, 1 o 2 caracteres: se muestra la lista completa sin filtrar
- El filtro es case-insensitive y busca solo por el campo `name`
- Al borrar el campo: vuelve la lista completa

---

## Estados de UI

### `SearchViewState`
```swift
enum SearchViewState {
    case loading
    case results([Product])
    case empty(query: String)
    case error(message: String)
}
```

### Detail
- No tiene enum de estado
- Solo tiene `isSaved: Bool` que cambia al guardar

### Saved
- Sin enum de estado
- La View evalúa si el array de `@Query` está vacío para mostrar empty

---

## Errores tipados

```swift
enum ProductRepositoryError: Error, LocalizedError {
    case fileNotFound
    case decodingFailed(Error)
}
```

---

## BLOQUES DE IMPLEMENTACIÓN

---

### BLOQUE 1 — Setup del proyecto

**Objetivo:** Crear la estructura base del proyecto en Xcode con el entry point y la navegación raíz configurados.

**Archivos a generar:**
- `App/MeLiExplorerApp.swift` — entry point con `ModelContainer` configurado para `SavedProduct`
- `App/ContentView.swift` — `NavigationStack` con `NavigationPath`, dos `navigationDestination` (uno para `Product`, uno para `SavedDestination`)

**Definiciones adicionales a incluir en `ContentView.swift`:**
```swift
// Enum para navegar a Saved sin pasar datos
enum SavedDestination: Hashable {
    case list
}
```

**Resultado esperado:** La app compila y corre en simulador mostrando una pantalla vacía sin crashes.

**Configuración manual en Xcode:**
- Crear el proyecto: File → New → Project → App
- Nombre: `MeLiExplorer`, SwiftUI, Swift, iOS 17
- Crear la estructura de carpetas en el navegador de Xcode
- Agregar target de tests: File → New → Target → Unit Testing Bundle, nombrar `MeLiExplorerTests`

---

### BLOQUE 2 — Modelos y datos

**Objetivo:** Definir las entidades del dominio y el archivo JSON con productos mockeados reales.

**Archivos a generar:**
- `Core/Models/Product.swift` — struct con todos los campos, conforme a `Identifiable`, `Codable`, `Hashable`. Incluir una extensión con `static var mock: Product` para usar en tests.
- `Core/Models/SavedProduct.swift` — `@Model` class con todos los campos más `savedAt: Date`. Incluir `init(from product: Product)`.
- `Resources/products.json` — mínimo 20 productos con datos realistas de categorías variadas de Mercado Libre (electrónica, ropa, hogar, deportes). Usar URLs de imágenes reales y públicas (pueden ser de Wikimedia Commons, Unsplash o similares que no requieran autenticación).

**Resultado esperado:** Los modelos compilan sin errores. El JSON tiene estructura válida y es decodificable con `JSONDecoder()` estándar (sin configuración custom de keys).

---

### BLOQUE 3 — Repositorio y persistencia

**Objetivo:** Implementar la capa de datos completa: protocolo, implementación concreta y store de SwiftData.

**Archivos a generar:**
- `Core/Repositories/ProductRepository.swift` — protocolo con docstring explicando su propósito. Un solo método: `func fetchAll() async throws -> [Product]`
- `Core/Repositories/JSONProductRepository.swift` — implementación que lee `products.json` del bundle con `Bundle.main.url(forResource:withExtension:)` y decodifica con `JSONDecoder`. Lanza `ProductRepositoryError` tipado.
- `Core/Persistence/SavedProductStore.swift` — clase con tres métodos: `save(_:context:)`, `delete(_:context:)`, `isSaved(_:context:)`. El `ModelContext` se recibe como parámetro, no se almacena. Incluir protección contra duplicados en `save`.

**Errores a definir en `JSONProductRepository.swift`:**
```swift
enum ProductRepositoryError: Error, LocalizedError {
    case fileNotFound
    case decodingFailed(Error)
}
```

**Resultado esperado:** Los tres archivos compilan. `JSONProductRepository` puede instanciarse y llamarse desde un test o Playground retornando los 20 productos del JSON.

---

### BLOQUE 4 — Entry point y navegación

**Objetivo:** Conectar el `ModelContainer` con la navegación y verificar que el esqueleto de la app funciona antes de construir las pantallas.

**Actualizar archivos existentes:**
- `App/MeLiExplorerApp.swift` — agregar `.modelContainer(for: SavedProduct.self)` al `WindowGroup`
- `App/ContentView.swift` — implementar `NavigationStack` completo con ambos `navigationDestination`. Por ahora los destinos pueden ser vistas placeholder (`Text("Search")`, `Text("Detail")`, `Text("Saved")`).

**Resultado esperado:** La app compila, corre en simulador, y la navegación básica funciona (aunque las pantallas sean placeholders).

---

### BLOQUE 5 — Feature Search

**Objetivo:** Implementar la pantalla de búsqueda completa con todos sus estados.

**Archivos a generar:**
- `Shared/Components/ProductCardView.swift` — tarjeta reutilizable que recibe un `Product`. Contiene: `AsyncImage` con placeholder y estado de error, nombre (máx 2 líneas, `.lineLimit(2)`), precio formateado como moneda. Aplicar el Style Guide: corner radius, padding, sombra según especificación.
- `Features/Search/SearchViewModel.swift` — clase con `@Observable`. Propiedades: `state: SearchViewState`, `searchQuery: String`. Lógica: carga todos los productos al init con `Task`, filtra en tiempo real con `didSet` en `searchQuery` (activa filtro con 3+ chars). Recibe `ProductRepository` por inicializador.
- `Features/Search/SearchView.swift` — View con `.searchable(text:)` vinculado al ViewModel. Switch sobre `state` para renderizar cada caso. Botón bookmark en `.toolbar` que navega a `SavedDestination.list`.

**Comportamiento del filtro en `SearchViewModel`:**
```swift
var searchQuery: String = "" {
    didSet {
        if searchQuery.count >= 3 {
            let filtered = allProducts.filter {
                $0.name.localizedCaseInsensitiveContains(searchQuery)
            }
            state = filtered.isEmpty ? .empty(query: searchQuery) : .results(filtered)
        } else {
            state = allProducts.isEmpty ? .loading : .results(allProducts)
        }
    }
}
```

**Resultado esperado:** La pantalla muestra todos los productos al abrir, filtra en tiempo real con 3+ caracteres, y muestra el estado empty cuando no hay coincidencias. El botón bookmark navega a Saved (placeholder por ahora).

---

### BLOQUE 6 — Feature Detail

**Objetivo:** Implementar la pantalla de detalle con la jerarquía visual definida y el botón Guardar funcional.

**Archivos a generar:**
- `Features/Detail/DetailViewModel.swift` — clase con `@Observable`. Recibe `Product` y `ModelContext` por inicializador. Propiedades: `product: Product`, `isSaved: Bool`. Método: `toggleSave(context:)` que guarda el producto usando `SavedProductStore`. Al init, consulta `SavedProductStore.isSaved` para setear el estado inicial del botón.
- `Features/Detail/DetailView.swift` — View con `ScrollView` para el contenido y botón "Guardar" / "Guardado ✓" fijo en la parte inferior con `safeAreaInset(edge: .bottom)`. Jerarquía visual: imagen (AsyncImage full width 4:3), nombre, precio, categoría + vendedor, rating con SF Symbols (`star.fill`), stock, descripción. Aplicar Style Guide en tipografía y colores.

**Estados del botón Guardar:**
- No guardado: `.buttonStyle(.borderedProminent)` con color primario, texto "Guardar"
- Guardado: `.buttonStyle(.bordered)` con color secundario, texto "Guardado ✓", `.disabled(true)`

**Resultado esperado:** El Detail muestra todos los datos del producto. Al presionar "Guardar" el botón cambia de estado inmediatamente. Al volver y re-entrar al mismo producto, el botón mantiene su estado.

---

### BLOQUE 7 — Feature Saved

**Objetivo:** Implementar la pantalla de productos guardados con persistencia real entre reinicios.

**Archivos a generar:**
- `Shared/Components/EmptyStateView.swift` — componente genérico que recibe `systemImage: String`, `title: String`, `subtitle: String?`. Centrado vertical y horizontalmente. Aplicar Style Guide.
- `Features/Saved/SavedViewModel.swift` — clase con `@Observable`. No necesita lógica propia: solo expone una función `delete(_:from:)` que delega a `SavedProductStore`.
- `Features/Saved/SavedView.swift` — View con `@Query` para obtener `[SavedProduct]` ordenados por `savedAt` descendente. Si el array está vacío muestra `EmptyStateView`. Si tiene elementos muestra `List` con `ProductCardView` y `.onDelete` para swipe-to-delete. Tap en tarjeta navega al Detail del producto (convertir `SavedProduct` → `Product` para la navegación).

**Conversión `SavedProduct` → `Product` para navegación:**
```swift
extension SavedProduct {
    func toProduct() -> Product {
        Product(
            id: id, name: name, price: price,
            imageURL: imageURL, description: description,
            category: category, seller: seller,
            rating: rating, reviewCount: reviewCount, stock: stock
        )
    }
}
```

**Resultado esperado:** Los productos guardados aparecen en Saved. El orden es más reciente primero. Swipe-to-delete funciona. Al reiniciar la app los productos persisten. Al entrar al Detail desde Saved el botón aparece en estado "Guardado ✓".

---

### BLOQUE 8 — Unit Tests

**Objetivo:** Implementar los 11 casos de test definidos en el validation plan usando Swift Testing.

**Archivos a generar:**
- `MeLiExplorerTests/SearchViewModelTests.swift` — 7 tests para `SearchViewModel` usando `MockProductRepository`
- `MeLiExplorerTests/SavedProductStoreTests.swift` — 4 tests para `SavedProductStore` usando `ModelContainer(isStoredInMemoryOnly: true)`

**Mock a incluir en el target de tests:**
```swift
struct MockProductRepository: ProductRepository {
    var productsToReturn: [Product] = []
    var shouldThrow: Bool = false

    func fetchAll() async throws -> [Product] {
        if shouldThrow { throw ProductRepositoryError.fileNotFound }
        return productsToReturn
    }
}
```

**Casos a implementar:**

`SearchViewModelTests`:
1. Carga inicial exitosa → estado `.results` con todos los productos
2. Error en repositorio → estado `.error` con mensaje no vacío
3. Query con 1 carácter → lista completa sin filtrar
4. Query con 2 caracteres → lista completa sin filtrar
5. Query con 3+ caracteres con coincidencias → lista filtrada correctamente
6. Query con 3+ caracteres sin coincidencias → estado `.empty` con query correcto
7. Borrar campo después de filtrar → lista vuelve a todos los productos

`SavedProductStoreTests`:
8. Guardar un producto → `isSaved` retorna `true`
9. Eliminar un producto guardado → `isSaved` retorna `false`
10. Guardar y leer desde el contexto → producto recuperable vía `FetchDescriptor`
11. Guardar el mismo producto dos veces → no se crean duplicados

**Sintaxis Swift Testing a usar:**
```swift
import Testing

@Suite struct SearchViewModelTests {
    @Test func initialLoad_success_returnsResults() async {
        // ...
        #expect(viewModel.state == .results(...))
    }
}
```

**Resultado esperado:** Los 11 tests pasan en verde. Se ejecutan con Cmd+U en Xcode sin configuración adicional.

---

## Cómo usar este prompt

1. **Primero:** Completá la sección "Diseño visual (Style Guide)" con el output de la IA de UI/UX.
2. **Luego:** Pasá este documento completo a la IA de desarrollo al inicio de la conversación para que tenga todo el contexto.
3. **Por cada bloque:** Decile a la IA: *"Generá el Bloque N"*. Revisá el código, copialo en Xcode, verificá que compila, y recién entonces pasá al siguiente bloque.
4. **Si algo no compila o no se comporta como se espera:** Describí el error exacto a la IA antes de continuar con el siguiente bloque.
5. **No saltes bloques:** Cada bloque depende del anterior. El Bloque 5 necesita los modelos del Bloque 2 y el repositorio del Bloque 3.
