# Forest City Tech Coop — design assets

Brand identity, hi-fi HTML mocks, and a Figma design system for **Forest City Tech Coop**, a worker-owned tech cooperative (Cleveland Heights, OH). This repo holds design artifacts, not application code — there's no build/test step.

## Directory map

| Path                                                               | What it is                                                                                                                                                                                                                                            |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `brand_kit/fctc-style-tile.html`                                   | Source of truth for brand tokens — colors, type ramp, buttons, the circuit-branch motif system. Read this before inventing a new color or font size.                                                                                                  |
| `graphics/ForestCityTechCoop_branch_*.svg`                         | The 5 authentic "branch" circuit-tree artworks (`br-a`–`br-e`) used as background decoration across the site. Each is a single organic filled path in a 581×581 viewBox.                                                                              |
| `hifi_mocks/fctc-home.html`                                        | High-fidelity static HTML build of the homepage. This is the canonical reference the Figma file should match.                                                                                                                                         |
| `hifi_mocks/assets/`                                               | Images used by `fctc-home.html` (work-card screenshots for Gather Well / Cofa House).                                                                                                                                                                 |
| `hifi_mocks/screenshots/fctc-home-1440w.png`, `fctc-home-320w.png` | **Golden rendered reference screenshots** of the homepage at desktop and mobile widths. When something looks off in Figma (or a future browser render), diff against these PNGs, not against a fresh headless-browser capture — see the gotcha below. |
| `scanned_sketches/*.pdf`                                           | Early wireframe sketches (home, about, portfolio, contact, global nav) — content/structure planning, pre-visual-design.                                                                                                                               |
| `static_mocks/fctc-wire-*.html`                                    | Lower-fidelity structural HTML mocks (global nav shell, home wireframe) that predate `hifi_mocks/`.                                                                                                                                                   |
| `.mcp.json.example`                                                | Template for local Penpot MCP config. Copy to `.mcp.json` and fill in your own token — real `.mcp.json` is gitignored because it holds a live auth token.                                                                                             |

## Brand tokens (from `brand_kit/fctc-style-tile.html` / `hifi_mocks/fctc-home.html`)

- **Color scales**: `green` 50–950 (primary/accent, e.g. `green-400 #96ca51`), `neutral` 50–950 (text/bg, warm-gray), `amber` 50–700 (defined, currently unused on the homepage).
- **Fonts**: Plus Jakarta Sans (display/headings), Inter (body), JetBrains Mono (overlines/mono labels).
- **Circuit-branch motif system**: small reusable SVG motifs (`--motif-lead`, `--motif-pad`, `--motif-corner`, `--motif-rule`) plus the 5 large branch artworks in `graphics/`. This is the brand's signature visual device — expect it in overlines, card corners, dividers, and section backgrounds on every page, not just the homepage.

## Figma file

**[Forest City Tech Coop — Design System & Homepage](https://www.figma.com/design/6EVaUnDG6k1jOoDZVV7Vd4)**

Built via the Figma MCP server (`use_figma` / Plugin API), not hand-authored. Structure:

- **Foundations** (file-level, no dedicated page): 31 color primitives → 21 semantic tokens (`bg/*`, `text/*`, `border/*`, `accent/*`), 10 text styles, 2 shadow styles (`shadow/sm`, `shadow/md`).
- **Components page**: Brand Mark, Button (Primary/Outline/Ghost/OnDark), Overline, SectionHead, Header, Footer, ServiceCard, WorkCard, PrincipleRow, plus the imported branch-decoration assets (`decoration/Branch A`, `decoration/Branch B`, icon components).
- **Homepage page**: the assembled screen, built entirely from instances of the above components — not one-off frames. Future pages (Our Story, Our Work, Contact — referenced in nav but not yet built) should reuse Header/Footer/Button/SectionHead directly.

### Gotchas learned the hard way (read before touching the branch decorations)

1. **Match the PNG screenshots, not a fresh browser capture.** A `headless Chrome --screenshot` render of `fctc-home.html` does _not_ visually match `hifi_mocks/screenshots/fctc-home-1440w.png` (the branch decorations render far more subtly than intended). Treat the checked-in PNGs as ground truth for what the design should look like; don't re-derive intent from a live re-render.
2. **The branch-decoration crop math is exact `viewBox` + `preserveAspectRatio="slice"` math**, replicated from the CSS (`vx,vy,vw,vh`, `xMid/xMax/xMin`+`YMid/YMin/YMax` alignment, `macro--left`/`macro--right` = fade-mask direction, `macro--right` also mirrors). Values live in `hifi_mocks/fctc-home.html` inline SVGs — grep for `class="macro`.
3. **Figma's screenshot/export pipeline silently renders blank** when a single node is scaled to a large size (e.g. an instance blown up ~4–5×, 2500px+) and clipped by a much smaller parent frame with a large negative offset — confirmed via isolated testing, not just this file. The same content renders fine unscaled, and renders fine if scaled up only ~1.7–2×. **Fix: crop first, scale second.** Build a small frame at the crop's _natural_ size (`vw`×`vh`) with `clipsContent=true`, position the source vector inside it, and only _then_ resize the whole (already-cropped) frame up to the target display size. Scaling a pre-cropped, self-contained unit is reliable; scaling first and clipping second is not.
4. **`figma.intersect()` (boolean ops) computes a correct bounding box but does not reliably re-render the clipped geometry** in this pipeline — a `BOOLEAN_OPERATION` node's children still export at full, uncropped size. Use frame `clipsContent` masking instead of boolean intersect for cropping.
5. **Mirroring an SVG via a baked-in `<g transform="scale(-1,1)...">` before import, combined with a forced `resize()` on the imported frame, corrupts the geometry** (pushes it outside its own canvas). Import unmirrored, then apply the mirror as a native Figma `relativeTransform` matrix (`[[-1,0,W],[0,1,0]]`) _after_ import.
6. **Tint fills go on the nested `VECTOR` child, not the wrapper `FRAME`** that `createNodeFromSvg` returns. Setting `.fills` on the wrapper is a no-op for visible color.
7. **Footer link columns must be fixed-width, not hug-content.** The CSS uses `grid-template-columns: 1.4fr 1fr 1fr; gap: 40px` over a 1120px inner width (1440 − 160px×2 padding) → Brand ≈428px, each link column ≈306px. Hug-content auto-layout columns bunch the links together instead of spreading across their grid track.

## Working conventions

- No package manager, build step, or tests — this is a static-asset/design repo.
- When editing `hifi_mocks/fctc-home.html`, keep it framework-free static HTML (matches the rest of the mocks).
- When editing the Figma file, load the `figma-use` skill (and `figma-generate-design`/`figma-generate-library` as relevant) before any `use_figma` call — see gotchas above for pitfalls already discovered in this file specifically.
