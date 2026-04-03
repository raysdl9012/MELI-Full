# Technical Spec — MeLi Explorer

**Versión:** 1.1  
**Fecha:** 2026-04-02  
**Plataforma:** iOS 17+  
**Estado:** Inicial

---

## 1. Arquitectura

### Patrón: MVVM (Model — View — ViewModel)

La app sigue el patrón MVVM con separación estricta de responsabilidades:

| Capa | Responsabilidad | Tecnología |
|------|-----------------|------------|
| **View** | Renderizar UI y capturar eventos del usuario | SwiftUI |
| **ViewModel** | Manejar estado de UI, orquestar llamadas a repositorios | `@Observable` (iOS 17) |
| **Repository** | Abstraer el origen de los datos (JSON local, SwiftData) | Protocol + implementación concreta |
| **Model** | Representar entidades del dominio sin dependencias de UI | Swift structs / SwiftData `@Model` |

### Regla de dependencias — solo hacia abajo

Cada capa únicamente puede conocer a la capa inmediatamente inferior. Nunca al revés, nunca saltando capas.

```
View → ViewModel → Repository → Model
```

En términos concretos:

- `SearchView` conoce a `SearchViewModel`. No sabe que existe `JSONProductRepository` ni `Product`.
- `SearchViewModel` conoce a `ProductRepository` (el protocolo). No sabe que existe `SearchView` ni SwiftUI.
- `JSONProductRepository` conoce a `Product`. No sabe que existe ningún ViewModel.
- `Product` no conoce a nadie. Es el núcleo del dominio, sin dependencias externas.

El beneficio práctico es que podés reemplazar cualquier capa sin tocar las demás. Por ejemplo, cambiar `JSONProductRepository` por un `APIProductRepository` que consuma la API real de Mercado Libre no requeriría modificar ningún ViewModel ni ninguna View.

### Por qué MVVM y no otras alternativas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| MV (sin ViewModel) | Mezcla lógica de negocio con la View, difícil de testear |
| TCA (The Composable Architecture) | Overhead injustificado para el scope de este challenge |
| VIPER | Excesiva separación de capas para una app de 3 pantallas |

---

## 2. Stack tecnológico

| Categoría | Decisión | Justificación |
|-----------|----------|---------------|
| UI | SwiftUI | Framework nativo moderno de Apple para iOS 17+ |
| Estado | `@Observable` (macro) | Más simple y performante que `ObservableObject` en iOS 17+ |
| Navegación | `NavigationStack` | Navegación programática con path tipado |
| Persistencia | SwiftData | ORM nativo de Apple, integración natural con SwiftUI |
| Imágenes | `AsyncImage` | Carga asíncrona nativa, sin librerías externas |
| Datos mock | JSON local en bundle | Simple, reemplazable por API real sin cambiar el ViewModel |
| Testing | Swift Testing | Framework moderno de Apple, disponible desde Xcode 16 / iOS 17+ |
| Dependencias externas | Ninguna | El scope no justifica agregar paquetes SPM |

---

## 3. Estructura de carpetas

```
MeLiExplorer/
├── App/
│   ├── MeLiExplorerApp.swift          ← Entry point, configura ModelContainer
│   └── ContentView.swift              ← Root view, instancia NavigationStack
├── Core/
│   ├── Models/
│   │   ├── Product.swift              ← Struct del dominio (Codable, Hashable)
│   │   └── SavedProduct.swift         ← @Model de SwiftData
│   ├── Repositories/
│   │   ├── ProductRepository.swift    ← Protocol: define el contrato de fetching
│   │   └── JSONProductRepository.swift ← Lee y decodifica products.json del bundle
│   └── Persistence/
│       └── SavedProductStore.swift    ← Encapsula operaciones SwiftData (save, delete, isSaved)
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
│       ├── ProductCardView.swift      ← Tarjeta reutilizable (Search + Saved)
│       └── EmptyStateView.swift       ← Vista genérica para estados vacíos
└── Resources/
    └── products.json                  ← 20+ productos mockeados con URLs de imagen reales
```

---

## 4. Modelos de datos

### `Product` (dominio)

```swift
/// Representa un producto del catálogo de Mercado Libre.
/// Es una entidad del dominio: inmutable, sin dependencias de SwiftUI ni SwiftData.
/// Se deserializa desde el JSON local y se pasa entre capas por valor (struct).
struct Product: Identifiable, Codable, Hashable {
    let id: String
    let name: String
    let price: Double
    let imageURL: String?   // nil si el producto no tiene imagen; la UI muestra placeholder
    let description: String
    let category: String
    let seller: String
    let rating: Double      // rango 0.0 - 5.0
    let reviewCount: Int
    let stock: Int
}
```

### `SavedProduct` (persistencia)

```swift
/// Representa un producto guardado por el usuario, persistido con SwiftData.
/// Es una copia de los datos del producto al momento de guardarlo,
/// no una referencia al JSON. Esto garantiza que los productos guardados
/// no se pierdan si el JSON cambia en el futuro.
@Model
class SavedProduct {
    var id: String
    var name: String
    var price: Double
    var imageURL: String?
    var category: String
    var seller: String
    var rating: Double
    var reviewCount: Int
    var stock: Int
    var savedAt: Date       // permite ordenar por fecha de guardado (más reciente primero)

    init(from product: Product) { ... }
}
```

### `products.json` — estructura esperada

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

## 5. Repositorios y flujo de datos

### `ProductRepository` — protocolo

```swift
/// Define el contrato para obtener la lista de productos.
///
/// Cualquier implementación concreta (JSON local, API REST, mock de tests)
/// debe conformar este protocolo. Los ViewModels dependen exclusivamente
/// de este protocolo, nunca de una implementación concreta. Esto permite:
/// - Reemplazar la fuente de datos sin modificar ningún ViewModel.
/// - Inyectar un mock en tests sin infraestructura adicional.
protocol ProductRepository {
    func fetchAll() async throws -> [Product]
}
```

### `JSONProductRepository` — implementación concreta

- Lee `products.json` desde el bundle de la app en cada llamada a `fetchAll()`.
- Decodifica el JSON con `JSONDecoder`.
- Lanza `ProductRepositoryError` tipado si el archivo no existe o el JSON está malformado.
- No realiza filtrado — esa responsabilidad pertenece al `SearchViewModel`.

### `SavedProductStore` — persistencia

```swift
/// Encapsula todas las operaciones de lectura y escritura sobre SwiftData
/// para productos guardados. Recibe el ModelContext como parámetro en cada
/// operación (no lo almacena) para facilitar la inyección en tests.
class SavedProductStore {
    func save(_ product: Product, context: ModelContext) throws
    func delete(_ savedProduct: SavedProduct, context: ModelContext) throws
    func isSaved(_ productId: String, context: ModelContext) -> Bool
}
```

### Flujo de datos — Search

```
SearchView
  └── SearchViewModel (@Observable)
        ├── state: SearchViewState (enum)
        ├── allProducts: [Product]         ← cargados una vez al init
        ├── filteredProducts: [Product]    ← derivado de allProducts + query
        └── ProductRepository (protocolo)
              └── JSONProductRepository
                    └── products.json
```

### Flujo de datos — Detail

```
DetailView
  └── DetailViewModel (@Observable)
        ├── product: Product               ← recibido por parámetro de navegación
        ├── isSaved: Bool                  ← consultado al SavedProductStore al init
        └── SavedProductStore
              └── SwiftData (ModelContext)
```

### Flujo de datos — Saved

```
SavedView
  └── @Query [SavedProduct]                ← query reactiva de SwiftData
        ordenado por savedAt desc
```

---

## 6. Manejo de estados de UI

Cada ViewModel expone un enum de estado que la View consume con un `switch`. Esto evita combinaciones de flags booleanos que podrían representar estados imposibles (ej: `isLoading: true` + `hasResults: true` simultáneamente).

### `SearchViewState`

```swift
enum SearchViewState {
    case loading                    // cargando el JSON al abrir la app
    case results([Product])         // lista con productos (completa o filtrada)
    case empty(query: String)       // búsqueda activa sin coincidencias
    case error(message: String)     // fallo al cargar el JSON
}
```

> El caso `filtering` (1-2 caracteres) no es un estado del enum. Cuando el query tiene menos de 3 caracteres, el ViewModel emite `.results(allProducts)` directamente. El umbral de 3 caracteres vive en el ViewModel como lógica de negocio. El enum solo modela estados visualmente distinguibles para la View.

### `DetailViewState`

No requiere enum de estado. El Detail siempre recibe un `Product` completo por navegación. El único estado variable es `isSaved: Bool`, expuesto directamente por el ViewModel.

### `SavedViewState`

SwiftData maneja la reactividad vía `@Query`. La View evalúa si el array está vacío para renderizar el estado empty, sin necesidad de un enum adicional.

---

## 7. Navegación

```swift
// ContentView.swift
NavigationStack(path: $navigationPath) {
    SearchView()
        .navigationDestination(for: Product.self) { product in
            DetailView(product: product)
        }
        .navigationDestination(for: SavedDestination.self) { _ in
            SavedView()
        }
}
```

- `NavigationPath` es manejado en `ContentView` y pasado hacia abajo.
- `Product` conforma `Hashable` para usarse como valor de navegación tipado.
- `SavedDestination` es un enum simple para navegar a Saved sin necesidad de pasar datos.
- No hay coordinators ni routers externos — el scope no lo justifica.

---

## 8. Persistencia con SwiftData

### Configuración del `ModelContainer`

```swift
// MeLiExplorerApp.swift
@main
struct MeLiExplorerApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: SavedProduct.self)
    }
}
```

### Consideraciones

- Solo `SavedProduct` está registrado en el `ModelContainer`.
- `Product` (dominio) nunca se persiste con SwiftData — es efímero, vive solo en memoria durante la sesión.
- El `ModelContext` se accede desde el environment de SwiftUI en las Views que lo necesitan y se pasa al `SavedProductStore` por parámetro.
- No hay migraciones definidas (v1.0 inicial, sin historial de schema).

---

## 9. Carga de imágenes

Se usa `AsyncImage` nativo de SwiftUI. No se incorporan librerías externas (Kingfisher, SDWebImage) ya que el scope no lo justifica y el `URLCache` del sistema provee caché básica sin configuración adicional.

```swift
AsyncImage(url: URL(string: product.imageURL ?? "")) { phase in
    switch phase {
    case .success(let image): image.resizable()
    case .failure:            Image(systemName: "photo").foregroundColor(.gray)
    case .empty:              ProgressView()
    @unknown default:         EmptyView()
    }
}
```

- Si `imageURL` es `nil` o la carga falla, se muestra un ícono placeholder.
- No hay retry manual de imágenes fallidas — se acepta como riesgo residual.

---

## 10. Manejo de errores

### Errores definidos

```swift
/// Errores posibles al intentar cargar el catálogo de productos desde el JSON local.
enum ProductRepositoryError: Error, LocalizedError {
    case fileNotFound               // el archivo products.json no está en el bundle
    case decodingFailed(Error)      // el JSON existe pero no puede deserializarse

    var errorDescription: String? {
        switch self {
        case .fileNotFound:
            return "No se encontró el archivo de productos."
        case .decodingFailed:
            return "Error al procesar los datos de productos."
        }
    }
}
```

### Estrategia por capa

El manejo de errores sigue el mismo principio de separación de responsabilidades que la arquitectura general: cada capa maneja lo que le corresponde y propaga hacia arriba lo que no puede resolver.

- **`JSONProductRepository`** es el origen del error. Detecta si el archivo no existe o si el JSON está malformado, y lanza un `ProductRepositoryError` tipado con contexto suficiente para que la capa superior pueda decidir qué mostrar.
- **`SearchViewModel`** es el receptor del error. Captura la excepción con `do/catch` y traduce el error técnico en un estado de UI legible: `.error(message:)`. El ViewModel no muestra nada — solo actualiza su estado.
- **`SearchView`** es quien renderiza. Al observar el estado `.error(message:)`, muestra el mensaje al usuario junto con un botón "Reintentar". La View no sabe nada sobre el tipo de error original.
- **`SavedProductStore`** propaga errores de SwiftData hacia el ViewModel que lo invoca. En v1.0 los errores de escritura se loguean en consola y no se exponen al usuario (riesgo aceptado, ver sección 11).

### Lo que no se maneja (riesgo aceptado)

- Errores de red en `AsyncImage` — se muestra placeholder, sin retry manual.
- Corrupción del store de SwiftData — requeriría migración de schema, fuera de scope.
- Errores de escritura en SwiftData expuestos al usuario — logueados en consola en v1.0.

---

## 11. Estrategia de testing

### Framework: Swift Testing

Se usa el framework `Swift Testing` de Apple (Xcode 16+). La sintaxis usa macros (`#expect`, `#require`, `@Test`, `@Suite`) en lugar de `XCTAssert`.

### Cómo se mockea el repositorio

El `SearchViewModel` recibe un `ProductRepository` por inicializador. En tests se inyecta un mock que implementa el mismo protocolo:

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

Esto permite testear el ViewModel en total aislamiento, sin leer archivos ni tocar disco.

### Casos de test definidos

**`SearchViewModel`**

| # | Caso | Qué se verifica |
|---|------|-----------------|
| 1 | Carga inicial exitosa | Estado final es `.results` con todos los productos del mock |
| 2 | Error en repositorio al cargar | Estado final es `.error` con mensaje no vacío |
| 3 | Query con 1 carácter | `filteredProducts` contiene todos los productos (sin filtrar) |
| 4 | Query con 2 caracteres | `filteredProducts` contiene todos los productos (sin filtrar) |
| 5 | Query con 3+ caracteres con coincidencias | `filteredProducts` contiene solo los productos que coinciden |
| 6 | Query con 3+ caracteres sin coincidencias | Estado es `.empty(query:)` con el query correcto |
| 7 | Borrar el campo después de filtrar | `filteredProducts` vuelve a contener todos los productos |

**`SavedProductStore`**

| # | Caso | Qué se verifica |
|---|------|-----------------|
| 8 | Guardar un producto | `isSaved` retorna `true` para ese producto |
| 9 | Eliminar un producto guardado | `isSaved` retorna `false` para ese producto |
| 10 | Guardar y leer desde el contexto | El producto persiste y es recuperable via `FetchDescriptor` |
| 11 | Guardar el mismo producto dos veces | No se crean duplicados en el store |

El caso 10 es el más importante: valida el ciclo completo de persistencia usando un `ModelContainer` en memoria, aislado del store real del dispositivo:

```swift
@Test func savedProduct_persistsAndIsRetrievable() async throws {
    let container = try ModelContainer(
        for: SavedProduct.self,
        configurations: ModelConfiguration(isStoredInMemoryOnly: true)
    )
    let context = container.mainContext
    let store = SavedProductStore()
    let product = Product.mock

    try store.save(product, context: context)

    let saved = try context.fetch(FetchDescriptor<SavedProduct>())
    #expect(saved.count == 1)
    #expect(saved.first?.id == product.id)
    #expect(store.isSaved(product.id, context: context) == true)
}
```

### Qué no se testea con unit tests

| Qué | Por qué no | Cómo se cubre |
|-----|------------|---------------|
| Layout y UI | Requiere XCUITest, fuera de scope | Validación manual |
| Navegación entre pantallas | Requiere XCUITest, fuera de scope | Validación manual |
| `AsyncImage` con URLs reales | Requiere red, no es unit test | Validación manual |
| Estados vacíos de `SavedView` | Depende de `@Query` reactivo de SwiftData | Validación manual |

---

## 12. Cambios respecto a versiones anteriores

| Versión | Decisión cambiada | Cambio | Motivo |
|---------|-------------------|--------|--------|
| v1.0 → v1.1 | Regla de dependencias | Expandida con explicación narrativa y beneficio práctico | Faltaba claridad sobre el impacto real de la regla |
| v1.0 → v1.1 | `ProductRepository` protocolo | Agregado docstring explicando el propósito y los beneficios | Sin descripción era ambiguo su rol en la arquitectura |
| v1.0 → v1.1 | Estrategia de errores por capa | Reescrita en forma narrativa en lugar de tabla | La tabla no transmitía el flujo de propagación entre capas |
| v1.0 → v1.1 | Casos de test | Agregado caso 10: ciclo completo de persistencia con ModelContainer en memoria | Faltaba validar escritura + lectura real desde SwiftData |
| v1.0 → v1.1 | Casos de test | Agregado caso 11: guardar el mismo producto dos veces | Caso borde importante para integridad del store |
