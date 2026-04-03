# run.md — MeLi Explorer

## Requisitos previos

| Herramienta | Versión mínima | Cómo verificar |
|-------------|---------------|----------------|
| Xcode | 16.0+ | `xcodebuild -version` |
| iOS Simulator | iOS 17.0+ | Xcode → Settings → Platforms |
| macOS | 14.0 (Sonoma)+ | Apple → About This Mac |
| Swift | 5.9+ | `swift --version` |

> No se requieren dependencias externas. La app no usa Swift Package Manager ni CocoaPods.

---

## Clonar o abrir el proyecto

Si el proyecto está en un repositorio:
```bash
git clone <URL_DEL_REPOSITORIO>
cd MeLiExplorer
open MeLiExplorer.xcodeproj
```

Si recibiste la carpeta directamente:
```bash
open MeLiExplorer.xcodeproj
```

---

## Correr la app en simulador

### Opción A — Desde Xcode (recomendado)

1. Abrir `MeLiExplorer.xcodeproj` en Xcode
2. En la barra superior, seleccionar el scheme `MeLiExplorer`
3. Seleccionar un simulador: `iPhone 15 Pro` con iOS 17.0 o superior
4. Presionar `Cmd + R` o el botón ▶ para compilar y correr

### Opción B — Desde terminal

```bash
# Listar simuladores disponibles
xcrun simctl list devices available

# Correr en iPhone 15 Pro (iOS 17)
xcodebuild \
  -scheme MeLiExplorer \
  -destination 'platform=iOS Simulator,name=iPhone 15 Pro,OS=17.0' \
  -configuration Debug \
  build
```

### Resultado esperado al abrir la app

1. Pantalla Search aparece con un spinner brevemente
2. Se carga la lista completa de productos (mínimo 20 items)
3. El campo de búsqueda está visible en la barra superior
4. El ícono bookmark está visible en la esquina superior derecha

---

## Correr los tests

### Opción A — Desde Xcode

1. Seleccionar el scheme `MeLiExplorer`
2. Presionar `Cmd + U` para correr todos los tests
3. Verificar en el Test Navigator (`Cmd + 6`) que los 11 tests aparecen en verde

### Opción B — Desde terminal

```bash
xcodebuild test \
  -scheme MeLiExplorer \
  -destination 'platform=iOS Simulator,name=iPhone 15 Pro,OS=17.0' \
  -resultBundlePath TestResults.xcresult
```

### Tests incluidos

**`SearchViewModelTests`** — 7 casos

| # | Test | Qué verifica |
|---|------|-------------|
| 1 | `initialLoad_success_returnsResults` | Carga exitosa → estado `.results` |
| 2 | `initialLoad_failure_returnsError` | Error en repositorio → estado `.error` |
| 3 | `searchQuery_oneChar_returnsAllProducts` | 1 carácter → sin filtro |
| 4 | `searchQuery_twoChars_returnsAllProducts` | 2 caracteres → sin filtro |
| 5 | `searchQuery_threeChars_withMatches_returnsFiltered` | 3+ chars con coincidencias → filtrado |
| 6 | `searchQuery_threeChars_noMatches_returnsEmpty` | 3+ chars sin coincidencias → `.empty` |
| 7 | `searchQuery_cleared_returnsAllProducts` | Borrar campo → lista completa |

**`SavedProductStoreTests`** — 4 casos

| # | Test | Qué verifica |
|---|------|-------------|
| 8 | `save_product_isSavedReturnsTrue` | Guardar → `isSaved` = true |
| 9 | `delete_savedProduct_isSavedReturnsFalse` | Eliminar → `isSaved` = false |
| 10 | `save_product_persistsInContext` | Ciclo completo escritura + lectura desde SwiftData |
| 11 | `save_duplicateProduct_doesNotDuplicate` | Guardar dos veces → sin duplicados |

### Resultado esperado

```
Test Suite 'All tests' passed
    Executed 11 tests, with 0 failures (0 unexpected) in X.XXX seconds
```

---

## Validación manual — checklist rápido

Antes de entregar, verificar estos flujos en el simulador:

- [ ] La app abre y muestra la lista de productos sin errores en consola
- [ ] Escribir 1-2 caracteres en el buscador → la lista no se filtra
- [ ] Escribir 3+ caracteres → la lista filtra en tiempo real
- [ ] Borrar el campo → vuelven todos los productos
- [ ] Tap en un producto → navega al Detail con datos correctos
- [ ] Presionar "Guardar" en Detail → botón cambia a "Guardado ✓" inmediatamente
- [ ] Volver al Detail del mismo producto → botón sigue en "Guardado ✓"
- [ ] Tap en bookmark → navega a Saved con el producto guardado
- [ ] Swipe-to-delete en Saved → el producto se elimina
- [ ] Forzar cierre de la app (`Cmd + Shift + H` dos veces en simulador) y reabrir → los productos guardados persisten

---

## Solución de problemas comunes

**La app no compila — error de iOS target**
```
Verifica: Project → MeLiExplorer → Deployment Info → Minimum Deployments → iOS 17.0
```

**`products.json` no se encuentra en runtime**
```
Verifica: Project → MeLiExplorer → Build Phases → Copy Bundle Resources
El archivo products.json debe estar listado ahí. Si no está, arrastralo desde el navegador
de Xcode al grupo Resources y asegurate de marcar "Add to target: MeLiExplorer".
```

**SwiftData crash al primer arranque**
```
Verifica: MeLiExplorerApp.swift tiene .modelContainer(for: SavedProduct.self) en el WindowGroup.
Si el schema cambió durante desarrollo, borrá la app del simulador antes de volver a correr:
Simulator → Device → Erase All Content and Settings
```

**Los tests no aparecen en el Test Navigator**
```
Verifica: El target MeLiExplorerTests está correctamente configurado.
File → New → Target → Unit Testing Bundle si no existe.
Asegurate que el test file tiene: import Testing (no import XCTest)
```

**Inter font no carga (si se usa fuente custom)**
```
Verifica: Los archivos .ttf de Inter están en Resources/ y listados en:
1. Copy Bundle Resources (Build Phases)
2. Info.plist → Fonts provided by application
```
