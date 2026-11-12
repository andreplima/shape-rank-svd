# Shape-Rank-SVD — Functional & Technical Specification

**Version:** 1.0 (deployed as `index.html` on GitHub Pages, November 2026)
**Scope:** This document describes what the Shape-Rank-SVD applet does, how it is structured, and the design choices behind it. It is meant for anyone who maintains, extends, or learns from the codebase — authors of lecture materials who want to adapt the applet, students taking it apart to understand it, reviewers assessing its claims, and the author's future self.

---

## 1. Overview

### 1.1 Purpose

Shape-Rank-SVD is a single-page interactive applet that lets a user reshape a quadrilateral polygon $P$ inside the unit square $[0,1]^2$ over a configurable lattice of cells, and see — in real time — both the **coverage matrix** $A$ that records cell-by-cell intersection areas and its **singular value decomposition**. The applet is designed to make visible a mathematical fact that is otherwise hidden in formulas: the rank of $A$ (and the shape of its singular-value spectrum) is determined by the geometry of $P$ relative to the lattice axes.

### 1.2 Target audience

- Undergraduate and early-graduate students learning linear algebra and the SVD.
- Instructors preparing lectures or classroom demonstrations.
- Researchers studying the relationship between diagram geometry and feature-matrix rank in learning tasks.

### 1.3 In scope

- Four-vertex polygons anchored at the origin, drawn on a square lattice with independent $n_x, n_y \in \{2, \ldots, 10\}$.
- Exact (polygon-clipping) computation of cell–polygon intersection areas.
- Full SVD of the coverage matrix and visual presentation of each rank-1 additive component.
- Linear and logarithmic scree plots.
- Four guided exercises and a numeric-details inspector.

### 1.4 Out of scope

- Polygons with more than four vertices or with holes.
- Non-rectangular or non-axis-aligned lattices.
- Session persistence (reload resets to defaults).
- URL-encoded state sharing.
- Mobile/touch optimisation (functional but not tuned).
- Programmatic APIs or data export.

---

## 2. Functional specification

The applet is divided into two side-by-side panels. The left panel holds the shape and lattice controls; the right panel holds the SVD visualisations. Both panels update whenever any input changes.

### 2.1 Left panel — Shape & lattice

**Header.** The panel title "Shape & lattice" sits at the top-left; an "Instructions ↗" link on the top-right opens the project README (GitHub repo) in a new tab.

**Drawing area.** A square SVG shows:

- The unit square frame.
- The $(n_x + 1) \times (n_y + 1)$ lattice lines.
- A polygon $P$ drawn as a magenta outline connecting the four vertices.
- Cells shaded according to a three-category classification:
  - **Fully covered** (translucent magenta fill) — the cell lies entirely inside $P$.
  - **Partially covered** (diagonal magenta hatching) — the cell is crossed by an edge of $P$.
  - **Uncovered** (no fill) — the cell lies entirely outside $P$.
- Four vertex markers: a black marker at the fixed origin $(0, 0)$ and three blue markers at the free vertices.

**Vertex manipulation.** The three free vertices are draggable. During a drag, the vertex follows the cursor continuously. On release, the vertex snaps to the nearest lattice joint $(i/n_x, j/n_y)$. The applet then validates the resulting configuration against the following rules:

1. The origin vertex must remain at $(0, 0)$ (enforced by making it non-draggable).
2. No two vertices may coincide.
3. The four vertices must not all be collinear.
4. The polygon is always traversed in counterclockwise order around the centroid of the four vertices; after any successful drag the free vertices are silently re-sorted by polar angle to maintain this order.

If any of rules 2 or 3 fails, the drag is rejected and the vertex snaps back to its pre-drag position; a transient status message is displayed below the sliders.

**Lattice sliders.** Two range inputs labelled $n_x$ and $n_y$ control the lattice resolution from 2 to 10 independently. Changing a slider resnaps every free vertex to the nearest joint of the new lattice; if resnapping would create an invalid configuration, the slider change is rejected and reverted.

**Reset button.** Returns the applet to its default state: a parallelogram with vertices $(0, 0)$, $(4/5, 0)$, $(1, 4/5)$, $(1/5, 4/5)$ on a $5 \times 5$ lattice.

**Status message.** A single-line area between the sliders and the Reset button shows transient feedback (red for rejections, green for successful resets), auto-clearing after ~2.5 seconds.

**Legend.** A horizontal strip at the bottom names the three cell categories (fully covered, partially covered, uncovered) with matching visual swatches.

### 2.2 Right panel — Decomposition into rank-1 additive components (SVD)

**Header.** The panel title explains that the panel shows the SVD decomposition of the coverage matrix as a sum of rank-1 terms.

**Colorbar.** A horizontal canvas gradient from the negative endpoint (blue) through zero (gray) to the positive endpoint (orange) communicates the shared colour scale used by every component heatmap. Labels on either side show the current value of the shared maximum magnitude $M$ (in scientific notation), prefixed by $\pm$. A small italic note below describes the current colour-mapping mode.

**Normalise toggle.** A checkbox labelled "Normalize each component independently" switches between the default **shared scale** (each heatmap uses the same $[-M, +M]$ range so magnitudes are comparable) and a **per-component scale** (each heatmap rescales to its own range, revealing the internal structure of small components at the cost of cross-component comparability). The colorbar labels update to reflect the current mode.

**Component heatmaps.** A $2 \times 5$ grid of up to ten slots, one per rank-1 additive component $\sigma_k \mathbf{u}_k \mathbf{v}_k^\top$ for $k = 0, \ldots, 9$. Each slot contains:

- A label showing $\sigma_k \cdot \mathbf{u}_k \mathbf{v}_k^\top$.
- A heatmap (SVG) showing the matrix entries of the component, with $j = 0$ at the bottom and $i = 0$ at the left (matching the orientation of the unit square in the left panel).
- A caption showing the numeric value of $\sigma_k$ and, for $k \geq 1$, the ratio $\sigma_k / \sigma_0$ as a percentage or in scientific notation.
- A coloured top border matching the colour of the corresponding bar in the scree plot (D3's `schemeTableau10` palette keyed by $k$).

Slots for which $k \geq \min(n_x, n_y)$ are shown in a faded "absent" style with no heatmap and "$\sigma_k = —$" instead of a numeric value, reinforcing that rank is bounded by $\min(n_x, n_y)$.

**Scree plot.** A D3-rendered bar chart of $\sigma_k$ versus $k$, with the same slot colours as the component heatmaps. A "Switch to log scale / linear scale" button toggles the y-axis mode with a 300 ms animated transition. Bars whose magnitude is below $\sigma_0 \cdot 10^{-12}$ are rendered as faded 4-pixel stubs on the log scale, so numerical-noise singular values remain visible without distorting the plot. The plot resizes responsively via `ResizeObserver`.

**Exercises.** A collapsible section with four guided exercises, each with a prompt and an expandable hint. The exercises walk students through the rank-1 case (axis-aligned rectangle), the near-rank-1 case (parallelogram), the triangle case (via three-collinear-vertices degeneracy), and the relationship between lattice resolution and effective rank.

**Numeric details.** A second collapsible section exposing:

- Metadata: $n_x, n_y$, $\alpha_x, \alpha_y$, cell area, total coverage, effective rank at tolerance $10^{-9}$, reconstruction error ($\max |M - U \, \mathrm{diag}(\sigma) \, V^\top|$) with a PASS/FAIL indicator.
- The coverage matrix $A$ as a formatted table, with row/column sums, coloured by cell category (full/partial/empty).
- A one-row table of the singular values.
- The left and right singular-vector matrices $U$ and $V$, both printed with row index in natural ascending order.

### 2.3 Cross-panel interactions

- Hovering over any cell of any heatmap shows a tooltip with $k$, $j$, $i$, the raw entry value, and the normalised value in $[-1, +1]$.
- Hovering over a scree-plot bar shows a tooltip with $k$, $\sigma_k$, and (for $k > 0$) the ratio $\sigma_k / \sigma_0$.
- Vertex drags in the left panel trigger a full re-render of the right panel on each drag event, so heatmaps and the scree plot animate in real time as the shape changes.

---

## 3. Technical specification

### 3.1 Technology stack

- **Language.** HTML5, CSS3, plain JavaScript (ES2020+). No TypeScript, no build step, no bundler.
- **Dependencies** (all loaded from jsDelivr CDN):
  - `d3@7` — SVG rendering, scales, axes, data-join pattern, drag behaviour, transitions, `ResizeObserver` consumers.
  - `polygon-clipping@0.15.3` — robust polygon-polygon intersection (for computing partial-cell coverage).
  - `ml-matrix@6.12.2` — singular value decomposition with the `autoTranspose` option for numerical robustness on wide matrices.
- **No server.** The applet runs entirely client-side.
- **Deployment.** Static hosting (currently GitHub Pages at `https://andreplima.github.io/shape-rank-svd/`).

### 3.2 File structure

```
index.html     # the applet: HTML, CSS, JS inline in one file
README.md      # project introduction and user documentation
SPEC.md        # this document
LICENSE        # CC BY-NC-SA 4.0 licence text
```

### 3.3 High-level architecture

The applet follows a simple **state → render** pattern. All mutable data lives in a single top-level `state` object. Any user action (drag, slider change, toggle, button click) mutates `state` and then calls the appropriate render function, which recomputes everything derived and updates the DOM. There is no reactive framework; D3's data-join pattern handles DOM reconciliation for the SVG elements, and plain `innerHTML` is used for the HTML tables and component slots.

### 3.4 State model

```javascript
state = {
  nx: integer in [2, 10],                   // horizontal lattice resolution
  ny: integer in [2, 10],                   // vertical lattice resolution
  vertices: [                               // always length 4, CCW order
    { x, y, role: "origin" | "free" }       // x, y in [0, 1]
  ],
  normalizeIndependently: boolean,           // colorbar/heatmap scaling mode
  logScale: boolean,                        // scree-plot y-axis mode
}
```

Transient drag state (grab offset, pre-drag position) lives in a separate `dragState` variable that is created on drag start and destroyed on drag end.

### 3.5 Data flow

```
                     state
                       │
                       ▼
         ┌─────────────────────────┐
         │   computeCoverage()     │  ← polygon-clipping library
         └──────────┬──────────────┘
                    │  { A, categories, ax, ay, cellArea }
                    ▼
         ┌─────────────────────────┐
         │     computeSVD()        │  ← ml-matrix library
         └──────────┬──────────────┘
                    │  { M, U, S, V, r }
                    ▼
    ┌──────────────┬─────────────────┬──────────────────┐
    ▼              ▼                 ▼                  ▼
renderLattice  renderCells  renderComponentsGrid  updateScreePlot
renderPolygon                renderNumericDetails
renderVertices
```

`renderAll()` calls every render function; `renderDuringDrag()` skips `renderLattice()` and `renderVertices()` (which don't change on vertex drag) and updates the vertex DOM directly from the drag handler for performance.

### 3.6 Core algorithms

**Coverage matrix computation (`computeCoverage`).**
For each cell $C_{ij}$, compute the intersection polygon $P \cap C_{ij}$ via `polygonClipping.intersection(V, C)` and sum the absolute signed areas of its rings (shoelace formula). A tolerance of $10^{-9} \cdot \alpha_x \alpha_y$ decides which cells are classified as empty (area below tolerance), full (area within tolerance of the cell area), or partial (anything in between). Negative areas or areas exceeding the cell area are clamped defensively.

**SVD computation (`computeSVD`).**
The coverage matrix $A$ is transposed into a row-major $n_y \times n_x$ matrix $M$ (so that $M[j][i] = A[i][j]$, keeping the visual orientation consistent across the applet) and passed to `mlMatrix.SingularValueDecomposition` with `{ autoTranspose: true }`. This returns the thin-form decomposition $M = U \, \mathrm{diag}(\sigma) \, V^\top$ where $U$ is $n_y \times r$, $V$ is $n_x \times r$, and $r = \min(n_x, n_y)$.

**Vertex validation (`validate`).**
Runs three checks in order: (1) any coincident vertices (pairwise equality), (2) all four collinear (two cross-product checks against the fixed origin), and (3) counterclockwise ordering (unconditional polar-angle sort around the centroid of the four vertices, keeping the origin at index 0). The function returns `{ valid, reason, sorted }`.

**Lattice snapping (`snap`).**
Clamps the input to $[0, 1]^2$ and rounds each coordinate to the nearest multiple of $1/n_x$ (resp. $1/n_y$).

**Color mapping (`colormap` + `powerMap`).**
Entries are first normalised to $[-1, +1]$ by dividing by the shared or per-component maximum magnitude. A power-law map $t \mapsto \mathrm{sgn}(t) \cdot |t|^{\gamma}$ with $\gamma = 1/2$ compresses the dynamic range so small-magnitude entries remain visually distinguishable from zero. The mapped value is then interpolated linearly through a three-stop diverging palette: blue (`#2166ac`) → gray (`#f2f2f2`) → orange (`#d6604d`). White is reserved for "absent" slots.

**Scree-plot rendering (`updateScreePlot`).**
A D3 band x-scale over $k \in \{0, \ldots, r-1\}$ paired with either a linear or log y-scale over the singular-value range. Bars receive stable identities via the data key $d \to d.k$; attributes `x`, `width`, `class`, `fill` are set directly on both entering and updating selections, while `y` and `height` go through a 300 ms transition only on the explicit log/linear toggle and are set directly otherwise (the `.interrupt()` call prevents in-flight transitions from stale frames bleeding into the current render). This design avoids a numerical-stability bug I encountered when using a uniformly-applied D3 transition with duration 0 during rapid drags.

### 3.7 Coordinate systems

Three coordinate systems coexist:

| System | Range | Used for |
|---|---|---|
| Unit square (math) | $x, y \in [0, 1]$ | Vertex positions, lattice cells, coverage computation |
| SVG viewBox (left) | $[\text{PAD}, 100 - \text{PAD}]^2$ with $y$ flipped | Rendering shapes in the left panel |
| Heatmap cell index | $(i, j) \in \{0, \ldots, n_x - 1\} \times \{0, \ldots, n_y - 1\}$ | Coverage matrix entries |

D3 linear scales (`xScale`, `yScale`, `xScaleInv`, `yScaleInv`) convert between the first two. The y-axis is flipped in SVG so that $(0, 0)$ sits at the bottom-left visually, matching the math convention.

### 3.8 Styling and theming

All CSS lives in a single `<style>` block at the top of `index.html`. Font sizes are centralised in four CSS custom properties at `:root`:

```css
--fs-title: 15px;   /* panel titles */
--fs-body:  14px;   /* exercise prompts, primary text */
--fs-meta:  13px;   /* slider labels, buttons, section headings */
--fs-small: 12px;   /* tooltips, axis ticks, table cells */
```

Changing any one rescales the whole applet uniformly — useful for classroom projection.

Colours are specified inline as hex strings; the diverging palette is hard-coded in the JavaScript (`BLUE`, `GRAY`, `ORANGE` constants).

---

## 4. Mathematical specification

Let $P \subset [0,1]^2$ be a simple polygon with four lattice-aligned vertices, the first fixed at the origin and the others at lattice joints. Let $\alpha_x = 1/n_x$, $\alpha_y = 1/n_y$. Define the cell

$$C_{ij} \;=\; \bigl[i \alpha_x,\, (i+1) \alpha_x\bigr] \times \bigl[j \alpha_y,\, (j+1) \alpha_y\bigr] \qquad i \in \{0, \ldots, n_x - 1\},\ j \in \{0, \ldots, n_y - 1\}$$

and the **coverage matrix** $A \in \mathbb{R}^{n_x \times n_y}$ by

$$a_{ij} \;=\; \mathrm{area}(P \cap C_{ij}).$$

For visualisation we work with the transposed form $M \in \mathbb{R}^{n_y \times n_x}$ with $M_{ji} = a_{ij}$, so that the row index matches the vertical direction of the picture.

The singular value decomposition is

$$M \;=\; \sum_{k=0}^{r-1} \sigma_k \, \mathbf{u}_k \mathbf{v}_k^\top, \qquad r \;=\; \min(n_x, n_y), \qquad \sigma_0 \geq \sigma_1 \geq \cdots \geq 0.$$

The applet renders each additive term $\sigma_k \mathbf{u}_k \mathbf{v}_k^\top$ as a heatmap and the sequence $(\sigma_k)$ as a scree plot. The **effective rank** reported in the numeric details panel is the number of indices $k$ for which $\sigma_k > 10^{-9} \sigma_0$ — a practical rank that tolerates machine-epsilon noise.

---

## 5. Non-functional requirements

- **Performance.** All rendering must keep up with real-time vertex dragging at $n_x = n_y = 10$ (100 cells × up to 10 SVD components rendering per drag event, measured at ~20 ms per full render on a modern laptop). Achieved by avoiding unnecessary re-computations in `renderDuringDrag` and by interrupting in-flight D3 transitions.
- **Browser support.** Modern evergreen browsers (Chrome, Firefox, Safari, Edge). Requires `ResizeObserver`, CSS `aspect-ratio`, CSS custom properties, flexbox, and ES2020 features. Not tested on Internet Explorer; no polyfills included.
- **Offline use.** The applet requires network access to load its three CDN dependencies on first load. Classroom environments with restricted networks should pre-cache these libraries locally.
- **Accessibility.** Keyboard navigation is not implemented for vertex dragging. Colour choices (blue/gray/orange diverging palette, blue/black/magenta accents) are approximately colour-blind-friendly but not WCAG-audited.
- **Internationalisation.** Interface strings are hard-coded in English.

---

## 6. Known limitations and design trade-offs

1. **Fixed four-vertex polygon.** The applet is a specific pedagogical tool, not a general polygon explorer. Triangles are modelled as degenerate quadrilaterals (three collinear vertices).
2. **Silent re-ordering on bowtie risk.** When a vertex drag would produce a self-intersecting quadrilateral, the applet silently reorders the free vertices to maintain counterclockwise traversal. This produces a valid simple polygon but may surprise users whose intended cyclic order was different. The alternative (reject bowtie drags) was considered and rejected: the current behaviour maximises exploration freedom at the cost of occasional visual surprise.
3. **CDN dependency.** Offline classroom use requires pre-caching three library files and editing the script tags.
4. **No session persistence.** The default parallelogram is restored on every page load.
5. **Lattice resolution capped at 10.** For $n_x, n_y > 10$ the SVD computation remains fast, but the component-grid layout (fixed $2 \times 5$ slots) and heatmap legibility degrade. Raising the cap would require UI rethinking.
6. **Numeric-details reconstruction check assumes well-conditioned SVD.** The PASS/FAIL threshold of $10^{-10}$ is generous and may need tightening for small matrices.

---

## 7. Extension points

Likely extensions, in increasing order of effort:

- **Persistence.** Save `state` to `localStorage` on each render; restore on page load.
- **URL-encoded state.** Serialise `state` into a URL hash so configurations can be shared.
- **Export.** A button to download the current coverage matrix as CSV and the SVD components as JSON.
- **More vertices.** Generalise the polygon to $n$ vertices, using the same validity rules (with coincidence, collinearity, and simplicity checks adapted accordingly).
- **Other decompositions.** Add NMF, eigendecomposition, or CUR alongside SVD as alternative views of the coverage matrix.
- **Alternative lattices.** Triangular or hexagonal cells, with polygon-clipping already supporting the arithmetic.

---

## 8. Change log

| Version | Date | Notes |
|---|---|---|
| 1.0 | 2026-11-11 | Initial public release on GitHub Pages. |

---

## 9. References

- Horn & Johnson, *Matrix Analysis*, 2nd ed. (Cambridge, 2012) — SVD and rank-1 factorisations.
- Golub & Van Loan, *Matrix Computations*, 4th ed. (JHU Press, 2013) — numerical SVD.
- Mike Bostock, *D3.js* documentation — https://d3js.org/
- Michael Fogel, *polygon-clipping* — https://github.com/mfogel/polygon-clipping
- `mljs/matrix` — https://github.com/mljs/matrix
- The *Polygrid* research repository — https://github.com/andreplima/polygrid

---

*This specification is a living document; update it when the applet changes substantively.*
