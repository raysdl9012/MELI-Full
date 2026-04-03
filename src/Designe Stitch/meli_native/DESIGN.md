# Design System Strategy: The Curated Marketplace

## 1. Overview & Creative North Star
This design system is built upon the North Star of **"The Digital Curator."** In a marketplace flooded with noise, our objective is to provide a sense of editorial calm and premium precision. We move beyond the "standard grid" by leveraging intentional white space, sophisticated tonal layering, and an iOS-native soul that feels custom-crafted rather than templated.

The system rejects the "boxed-in" feeling of traditional marketplace apps. Instead, it utilizes **Organic Depth**—using soft transitions and glassmorphism to make the UI feel like a living, breathing environment. By prioritizing typography and surface-on-surface nesting, we create a layout that feels as much like a high-end magazine as it does a functional tool.

---

### 2. Colors: Tonal Depth & The "No-Line" Rule
Our palette is rooted in the interplay between deep indigos and soft surfaces. We do not use lines to define space; we use light.

*   **Primary Action:** Use `primary` (#0058bc) for high-intent actions. For a more premium feel in hero sections, use a gradient transition from `primary` to `primary_container` (#0070eb) at a 135-degree angle.
*   **The "No-Line" Rule:** Explicitly prohibit 1px solid borders for sectioning. Boundaries must be defined solely through background color shifts. A `surface_container_low` section sitting on a `surface` background provides all the separation the eye needs without the "cheapening" effect of a stroke.
*   **Surface Hierarchy:**
    *   **Level 0 (Base):** `surface` (#fcf8fb) - The canvas.
    *   **Level 1 (Nesting):** `surface_container_low` (#f6f3f5) - Secondary content areas.
    *   **Level 2 (Cards):** `surface_container_lowest` (#ffffff) - Interactive elements that need to "pop" from the background.
*   **The Glass Rule:** For top navigation bars and floating action footers, use `surface` at 80% opacity with a `backdrop-filter: blur(20px)`. This integrates the content into the environment.

---

### 3. Typography: Editorial Authority
We utilize **Inter** (as a high-end alternative to SF Pro for a more distinct character) or SF Pro, strictly following a scale that prioritizes hierarchy over size.

*   **Display (Editorial Hero):** Use `display-md` (2.75rem) for featured marketplace categories. It should feel authoritative and command the page.
*   **Headline & Title:** `headline-sm` (1.5rem) is our workhorse for section headers. Pair this with `body-md` (0.875rem) in `on_surface_variant` (#414755) to create a clear "Title vs. Metadata" distinction.
*   **Labeling:** `label-md` (0.75rem) should be used in All Caps with +5% letter spacing for small tags (e.g., "NEW ARRIVAL") to evoke a boutique fashion aesthetic.

---

### 4. Elevation & Depth: Tonal Layering
Traditional drop shadows are too heavy for a "clean" iOS aesthetic. We use **Ambient Light** principles.

*   **The Layering Principle:** To lift a card, place a `surface_container_lowest` card on a `surface_container_low` background. The subtle shift in hex code creates a "soft lift."
*   **Ambient Shadows:** For floating elements (like a "Buy" button bar), use a multi-layered shadow:
    *   `Shadow 1: 0px 4px 20px rgba(27, 27, 29, 0.04)`
    *   `Shadow 2: 0px 8px 40px rgba(27, 27, 29, 0.08)`
*   **The Ghost Border:** In Dark Mode, where surfaces merge, use a "Ghost Border": the `outline_variant` token at **15% opacity**. This provides a whisper of a boundary without interrupting the visual flow.

---

### 5. Components: The Primitive Set

#### Buttons
*   **Primary:** `primary` background with `on_primary` text. Radius: `md` (0.75rem). Full-width for mobile conversion.
*   **Secondary:** `secondary_container` background with `on_secondary_container` text. No border.
*   **Tertiary (Ghost):** No background. Use `primary` text for "See All" actions.

#### Cards & Lists (The "No Divider" Mandate)
*   **Cards:** Radius `xl` (1.5rem) for hero cards; `lg` (1rem) for standard grid items.
*   **Lists:** Forbid the use of horizontal divider lines. Use **Spacing 4** (1rem) to separate list items. The `on_surface_variant` text provides enough hierarchy to distinguish items without "cutting" the screen into slices.

#### SF Symbols 5 Integration
*   Use **Variable Weights**. Icons in active states (e.g., Tab Bar) should use a `semibold` weight, while inactive states use `light`. Use the `hierarchical` rendering mode with the `primary` color to add depth to icons.

#### Signature Component: The "Curated Float"
A horizontal scrolling carousel where cards use `surface_container_highest` for the background, but the image within the card slightly overlaps the top edge (asymmetric layout) to break the "boxed" feel.

---

### 6. Do’s and Don’ts

*   **DO:** Use `surface_container` tiers to create a "nested" look (e.g., a white card inside a light gray section).
*   **DO:** Use the **Spacing Scale** religiously. A "Premium" feel is 50% typography and 50% generous white space (typically `8` or `10` on our scale).
*   **DON’T:** Use 100% black (#000000) for text. Always use `on_background` (#1b1b1d) to maintain a soft, ink-on-paper feel.
*   **DON’T:** Use standard iOS separators. If you feel the need to separate, use a background color change or a larger gap.
*   **DO:** Ensure Dark Mode utilizes `inverse_surface` for high-contrast callouts, ensuring the experience remains "Rich" rather than just "Dim."

---

### 7. Implementation Tokens (Reference)

| Context | Token | Value |
| :--- | :--- | :--- |
| **App Canvas** | `surface` | #fcf8fb |
| **Section Backing** | `surface_container_low` | #f6f3f5 |
| **Card / Input Backing** | `surface_container_lowest`| #ffffff |
| **Primary Action** | `primary` | #0058bc |
| **Success/Saved** | `tertiary` (customized) | #9e3d00 (or System Green) |
| **Corner Radius (Main)**| `lg` | 1rem (16pt) |
| **Corner Radius (Button)**| `md` | 0.75rem (12pt) |