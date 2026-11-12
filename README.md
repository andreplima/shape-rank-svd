
# Shape-Rank-SVD

Did you know that there is a relationship between the shape of a polygon and matrix rank? **Shape-Rank-SVD** is an interactive applet that makes this connection tangible. Drag the corners of a polygon, watch its coverage matrix change and decompose, and see the rank emerge from — or collapse into — the shape itself.

Why should anyone care about this relationship? Interpretability has become a mandatory feature of AI applications in many domains. A diagram that *faithfully* depicts what a learning algorithm does promotes its interpretability, offering a route to understanding that avoids post-hoc rationalisations. [We explore such diagrams in our research](https://github.com/andreplima/polygrid). To ensure faithfulness, we use a diagram as a template to extract a feature matrix $S$ from the original data $X$, and train a linear model $SW=Y$ to solve the task in hand. Since the rank of the feature matrix plays an important role in the solvability of the linear system, we should seek for diagrams that can somehow extract high-rank feature matrix. This applet aims to develop the basic intuition behind this mechanism.



**Live applet:** https://andreplima.github.io/shape-rank-svd/

---

## What the applet shows

Suppose your data $X \in (0,1]^{m \times d}, d \geq 2$, and select an arbitrary row $i$ and two arbitrary colums $(k,l)$.  Let's represent the pair $(x_{ik}, x_{il})$ by a polygon instead of a point (as we would usually do). Let $P$ be a polygon anchored at the origin inside the unit square, and consider a lattice that partitions $[0,1]^2$ into $n_x \times n_y$ cells. The **coverage matrix** $A$ records, for each cell, the area of its intersection with $P$:

$$a_{ij} = \mathrm{area}\bigl(P \cap C_{ij}\bigr)$$

The applet lets the user drag the vertices of $P$ on a lattice joint, change the lattice resolution with two sliders, and see in real time:

- the cells covered by $P$ (fully, partially, not at all),
- the coverage matrix $A$ and its singular value decomposition $A = U \mathrm{diag}(\sigma) \mathrm{ } V^\top$,
- the rank-1 components $\sigma_k \mathbf{u}_k \mathbf{v}_k^\top$, rendered as heatmaps,
- the singular value spectrum as a scree plot, on linear or log scale.

The pedagogical arc the applet supports in a classroom is:

1. **Axis-aligned rectangle — rank 1.** A Cartesian product of two intervals factors as an outer product of 1D profiles. Only $\sigma_0$ is nonzero.
2. **Parallelogram — effectively low rank.** Shearing introduces a coordinate-mixing constraint; a second singular value appears. The spectrum decays quickly.
3. **Triangle — richer spectrum.** A hypotenuse is a level set of $x_{ik} + x_{il}$, which genuinely mixes the coordinates. More singular values become meaningful.
4. **Resolution and rank.** As $n_x, n_y$ grow, more boundary cells can carry structure, and the spectrum deepens.

Four built-in exercises walk students through these four stages.

## How to use it

Open the live URL above, or clone and open `index.html` directly in a browser.

The applet is divided into two panels, side by side:

- The **left panel** ("Shape & lattice") shows the polygon $P$ drawn on the unit square with the lattice overlaid, plus the controls that modify the shape and the lattice resolution.
- The **right panel** ("SVD decomposition") shows the rank-1 components of the coverage matrix as heatmaps, the singular value spectrum as a scree plot, and the collapsible exercises and numeric-details sections.

### In the left panel

- **Drag** the three blue vertices to reshape the polygon. The origin (black) is fixed. Vertices snap to the nearest lattice joint on release. Invalid configurations (coincident vertices, all four collinear) are rejected with a status message shown just below the sliders.
- **Sliders** $n_x$ and $n_y$ (below the drawing) control the lattice resolution from 2 to 10 independently.
- **Reset** (bottom row, next to the status message) returns the vertices and sliders to the default parallelogram.
- A **legend** at the bottom of the panel names the three cell categories (fully covered, partially covered, uncovered).

### In the right panel

- At the **top**, a horizontal **colorbar** shows the diverging blue-gray-orange color scale used by all heatmaps, and a **"Normalize each component independently"** checkbox rescales each heatmap to its own range (useful for inspecting the internal structure of small components); off by default so magnitude comparisons across components stay honest.
- Below the colorbar, a **2 × 5 grid of heatmap slots** displays the rank-1 components $\sigma_k \mathbf{u}_k \mathbf{v}_k^\top$ for $k = 0, \ldots, 9$. Slots whose $k$ exceeds $\min(n_x, n_y)$ are shown as "absent." Each slot has a colored top border that matches its bar in the scree plot below.
- Below the heatmap grid sits the **scree plot** of $\sigma_k$ versus $k$. A **"Switch to log scale / linear scale"** button in its header toggles the y-axis mode.
- At the **bottom**, two collapsible sections: the **Exercises** panel contains four guided activities with hints; the **Numeric details** panel shows the coverage matrix $A$, the singular values, and the full matrices $U$ and $V$ for inspection.

### Interactions that span both panels

- **Hover** any heatmap cell or scree bar for a tooltip with exact values. Dragging a vertex in the left panel updates the right panel live, in real time.


## Implementation

A single `index.html` file, pure HTML/CSS/JavaScript with three CDN-loaded dependencies:

- [D3.js](https://d3js.org/) — SVG rendering, scales, axes, data joins.
- [polygon-clipping](https://github.com/mfogel/polygon-clipping) — robust polygon-polygon intersection for computing partial-cell coverage.
- [ml-matrix](https://github.com/mljs/matrix) — singular value decomposition.

There is no build step, no server, no toolchain. The whole applet runs client-side in any modern browser.

### Project structure

```
index.html     # the applet
README.md      # this file
```

That's it.

### Font-size scale

The CSS exposes four font-size variables at `:root`:

```css
--fs-title: 15px;  /* panel titles */
--fs-body:  14px;  /* exercise prompts */
--fs-meta:  13px;  /* slider labels, buttons, section headings */
--fs-small: 12px;  /* tooltips, axis ticks, table cells */
```

Change these to rescale the whole applet uniformly — useful when projecting in a classroom.

## Running locally

Clone the repo and open `index.html` in a browser. No server required.

```bash
git clone https://github.com/andreplima/shape-rank-svd.git
cd shape-rank-svd
open index.html   # or xdg-open index.html on Linux
```

The applet needs network access for the three CDN dependencies. For a fully offline version, download the three library files from the CDN URLs in the `<script>` tags at the top of `index.html` and host them alongside `index.html`, then update the paths.

## License

This applet is released under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/) (CC BY-NC-SA 4.0). See the [LICENSE](LICENSE) file for the full legal text.

In plain terms, you are free to:

- **Share** — copy and redistribute the applet in any medium or format;
- **Adapt** — remix, transform, and build on it (fix bugs, add features, translate the exercises, repurpose it for your own classroom);

under the following terms:

- **Attribution** — you must give appropriate credit, link back to this repository, and indicate if changes were made;
- **NonCommercial** — you may not use the applet for primarily commercial purposes;
- **ShareAlike** — if you adapt the applet, you must distribute your version under the same license.

Issues and pull requests are welcome.
