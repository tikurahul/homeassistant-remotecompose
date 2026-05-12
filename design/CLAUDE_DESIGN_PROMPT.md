# Prompt for Claude Design

You're being asked to design new card variants for **Terrazzo / homeassistant-remotecompose**, an open-source project at `github.com/yschimke/homeassistant-remotecompose`. Read the references below before sketching anything.

## 1. What this project is

A Kotlin Multiplatform library that converts Home Assistant Lovelace dashboard cards into **RemoteCompose** documents (`androidx.compose.remote`, alpha08). A RemoteCompose doc is a serialised byte stream that any RC player can render — Android apps, Glance home-screen widgets, Wear tiles, TV, ESP32 panels. The goal: a Lovelace card config + an HA state snapshot in → a `.rc` document out → identical-looking widget on every surface.

The project is one hand-written `CardConverter` per Lovelace card type, verified against real HA screenshots until pixel-delta is below threshold. We are NOT trying to replicate HA pixel-for-pixel — we are designing the *Terrazzo* take on each card: HA-flavoured, Material 3-grounded, readable on phone / watch / wall tablet / TV.

## 2. Surfaces & constraints

- **Mobile** (phone + tablet): light & dark, Material 3 + 4 Terrazzo palettes.
- **Wear**: dark-only, watch-face viewing distance, very short glance.
- **TV / wall kiosk**: dark-only, Kiosk palette, read from 1–3 m.
- **Glance widget**: same `.rc` doc as the in-app card, no continuous animation (`updateAppWidget` cadence >= 10 min), colours baked at capture-time.

Two RemoteCompose constraints to keep in mind while designing:

- `RemoteText` in alpha08 takes `fontSize` / `fontWeight` but **no `fontFamily`** — typography is system default inside cards (chrome around them respects the palette).
- The doc bakes a single colour set; on theme change the widget regenerates a new `.rc`. Don't design anything that needs cross-fade between palettes.

## 3. Design system — Terrazzo

Full spec: `design/STYLE_GUIDE.md` on `main`. The short version:

**Two-axis toggle**: *Theme style* x *Dark mode*. Style is one of five — Material 3 (dynamic colour on Android 12+) or four hand-picked Terrazzo palettes:

| Palette | Seed | PaletteStyle | Font | Persona |
|---|---|---|---|---|
| Home | `#03A9F4` HA blue | TonalSpot | Roboto Flex + Inter | default — "looks like HA" |
| Mushroom | `#E89F71` salmon | Fidelity | Figtree | warm community-card fans |
| Minimalist | `#3F4A5C` slate | Neutral | IBM Plex Sans | data-dense dashboards |
| Kiosk | `#00897B` teal | Vibrant + Contrast.High | Atkinson Hyperlegible | wall panels, distance reading |

Schemes are derived from the seed at runtime via `materialkolor.dynamicColorScheme`. All palettes use the stock M3 type scale — only `fontFamily` changes.

**Card colour roles** (`HaTheme` — renaming of M3 roles):

| M3 | HA-card name | Used for |
|---|---|---|
| `surface` | `cardBackground` | card body |
| `background` | `dashboardBackground` | grid backdrop |
| `onSurface` | `primaryText` | title / entity name |
| `onSurfaceVariant` | `secondaryText` | state value |
| `outline` | `divider` | stroke, separators |
| `secondary` | `placeholderAccent` | shimmer accent |
| `secondaryContainer` | `placeholderBackground` | shimmer backdrop |

**State colours** (`HaStateColor`) — red warning, green OK, amber unavailable, etc. — are **fixed across all palettes**. The palette is chrome, not state.

Rules:

- Never hardcode `Color(0xFF...)` — extend `HaTheme` if a role is missing.
- Stick to M3 elevation defaults; surface tonality differentiates palettes, not depth.
- No custom motion specs yet (cards re-capture as discrete docs anyway).

## 4. Cards in the project

Converters live in `rc-converter/src/main/kotlin/ee/schimke/ha/rc/cards/`. The current set:

`alarm-panel`, `area`, `button`, `calendar`, `clock`, `conditional`, `entities`, `entity`, `entity-filter`, `gauge`, `glance`, `heading`, `history-graph`, `humidifier`, `light`, `logbook`, `map`, `markdown`, `media-control`, `picture`, `picture-elements`, `picture-entity`, `picture-glance`, `sensor`, `statistic`, `statistics-graph`, `thermostat`, `tile`, `todo-list`, `weather-forecast`, plus stack containers and a Bambu Lab custom card. `picture-elements`, `map` and graph cards are known-hard; `iframe` and arbitrary custom cards are out of scope.

`@Preview` fixtures per card live in `previews/src/main/kotlin/ee/schimke/ha/previews/` (`CardPreviews.kt`, `TileCardPreviews.kt`, `CardPreviewMatrix.kt`, `DashboardPreviews.kt`, etc.).

## 5. Where to see the current state

Rendered baselines from `main` are committed to a sibling branch and served as raw images:

- **Index**: https://github.com/yschimke/homeassistant-remotecompose/blob/compose-preview/main/README.md
- **Per-card PNGs** (light + dark, M3 + 4 Terrazzo palettes where applicable):
  - `https://raw.githubusercontent.com/yschimke/homeassistant-remotecompose/compose-preview/main/renders/previews/CardPreviewsKt.<Name>_<Light|Dark>_<slug>.png`
  - Card matrix: `.../renders/previews/CardPreviewMatrixKt.CardPreviewMatrix_<Card>_matrix_<card>.png`
  - Full dashboards: `.../renders/app/Screen_DashboardView_*.png`
  - Theme-style sweeps: `.../renders/app/Screen_DashboardView_ThemeStyle_dashboard_theme_<ThemeName>.png`
- Trees: `renders/app/`, `renders/previews/`, `renders/wear/`, `renders/tv/`.

Read the index README first to pick the exact image URLs you need, then fetch the PNGs directly.

## 6. Your task

> *Replace this block with the specific brief, e.g. "Design new variants for the `gauge` card — segment-style and dual-needle — for the four Terrazzo palettes, light + dark, at both card-matrix size and Wear tile size."*

Deliverables:

- One image per (variant x palette x dark/light x surface) cell, on a transparent or `dashboardBackground` backdrop.
- A short rationale per variant: which persona it serves, which M3 roles map to which surfaces.
- Any new `HaTheme` role you'd need added, with a proposed name and the M3 role it derives from.

Constraints recap: M3 type scale only, no custom motion, no hardcoded hex outside `HaTheme` / `HaStateColor`, state colour stays fixed across palettes, designs must read on Wear and on a 3 m kiosk.
