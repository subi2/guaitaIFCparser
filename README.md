# guaitaIFCparser

<sup>GUAITA · IFC PARSER</sup>

## Compare IFC engines *side by side*

guaitaIFCparser opens an IFC file and displays it at the same time in up to four independent viewers. In each viewer you choose which *parser* reads it and which *renderer* draws it, and compare geometry, element count and processing time. Everything happens inside the browser, with the WASM engine embedded and nothing sent anywhere.

## Contents

1. [What it is](#01-what-it-is)
2. [Getting started](#02-getting-started)
3. [The interface](#03-the-interface)
4. [Parsers: what we compare](#04-parsers-what-we-compare)
5. [WASM engines](#05-wasm-engines)
6. [Renderers and modes](#06-renderers-and-modes)
7. [Views and navigation](#07-views-and-navigation)
8. [Troubleshooting](#08-troubleshooting)
9. [Licenses and credits](#09-licenses-and-credits)

---

## 01. What it is

**guaitaIFCparser — multi-parser comparator**

It's a standalone single-HTML-file tool for opening an **IFC** model and comparing, at the same time, how different *parsers* read it and how different *renderers* draw it. The window is split into 1, 2 or 4 viewers and each viewer can use a different combination, so you see the differences in geometry and performance at a glance.

| Feature | Description |
|---|---|
| **Multi-parser** | Reads the same IFC with four independent engines: web-ifc (WASM), Guaita's native STEP parser, BIMrocket (CSG) and ifc-lite (Rust/WASM), which also opens IFC5/IFCX. |
| **Multi-renderer** | Draws each model with Three.js or with a raw WebGL2 engine using its own shaders, to see the rendering differences. |
| **Up to 4 viewers** | Splits the window into 1, 2 or 4 panels and puts each parser/renderer combination in a different panel. |
| **Live metrics** | Each viewer shows schema, elements, meshes, vertices, processing time and filtered meshes, to compare performance and coverage. |

> [!NOTE]
> **Privacy** — All processing happens on your computer, inside the browser. The four engines are embedded in the file (no CDN) and files are never sent anywhere.

## 02. Getting started

**Open your first IFC**

1. Open `guaitaIFCParser.html` with a modern browser (updated Chrome, Edge or Firefox). No installation or connection needed: the engine is already inside the file.
2. Click **Open IFC** and pick a `.ifc` or `.ifcx` file, or drag and drop it onto the window (the frame **Drop the IFC here** appears).
3. Before anything is processed, the **pre-load report** shows up, with the file's contents and a per-engine memory estimate: you choose how many viewers you want, which engine goes in each one and the orientation, then confirm or cancel. On confirming, the viewers process the model. The filename and size in MB appear at the top; in each panel, the status bar at the bottom shows the schema and the metrics.

> [!NOTE]
> **Out of the box** — The viewers come preset with a different engine each — **web-ifc, native STEP, BIMrocket and ifc-lite** — and the pre-load report recommends how many to open based on the model's weight. This way you compare engines from the very first moment without locking up the browser.

## 03. The interface

**The window's areas**

| Area | Content |
|---|---|
| **Header** (top) | Open IFC · background color · category filter · 1 / 2 / 4 layout · Sync · Console |
| **Viewer selectors** (left) | In each panel: parser → renderer, mode, projection, standard view and Z-up / Y-up orientation. |
| **3D area** (center) | The model as drawn, with the axis triad at the bottom left. Orbit, pan and zoom with the mouse directly over the canvas. |
| **Quick buttons** (right) | ⤢ frame · ⟲ reset to opening view. |
| **Status bar** (bottom) | schema · elements · meshes · vertices · time · filtered meshes · warnings |

The top bar is shared by all viewers. The controls you'll find there are:

| Control | What it does |
|---|---|
| **Open IFC** | Loads a `.ifc` or `.ifcx` file from disk. You can also drag it onto the window. Before anything is processed, a **pre-load report** is shown. |
| **Background color** | Four color swatches for the viewers' background: Black, Technical blue, Dark gray and Light. |
| **1 / 2 / 4** | Number of viewers visible at once (one, two or four panels). |
| **Category filter** | Shows or hides **Grid, Annotations, Site, Openings and Spaces**. Applied equally to all engines, so the comparison stays fair. Only spaces are shown by default. |
| **⟳ Sync** | Locks the camera of all visible viewers so they move together. |
| **Console** | Real-time log with timestamps: scans, each engine's time, warnings and errors. Useful for diagnostics. |

*Each panel has its own selector header and its own independent status bar, so you can have different combinations and metrics in each pane.*

## 04. Parsers: what we compare

**What we compare: the engine that reads the IFC**

The *parser* is the component that reads the IFC file and extracts its geometry (triangle meshes with their colors and transforms). It's the **first dropdown** in each viewer's header. Switching it lets you see how the same model is interpreted by different engines.

All four deliver the result in a **common normalized format** — positions, normals, indices, matrix and color per mesh — so any parser can be paired with any renderer, and the comparison stays fair.

### web-ifc <sub>C++ → WebAssembly</sub>

Engine from the ThatOpen project (formerly IFC.js), the de facto standard for web IFC viewers. Full geometric kernel: resolves extrusions, revolutions, B-reps, **boolean operations** and `IfcMappedItem`. Delivers already-tessellated geometry via *streaming*, with interleaved position and normal buffers, transform matrix and color per geometry. The WASM is embedded in the file (no CDN).

> [!NOTE]
> **Role** — The practical reference for comparison: fast and complete, but a black box — if it tessellates a piece wrong, you can't see why.

### Native STEP (Guaita) <sub>Pure JavaScript</sub>

Written from scratch for this tool, with no dependencies or WASM. Tokenizes the STEP file's `DATA` section, builds an entity index and resolves placements (chains of `IfcLocalPlacement`, `IfcAxis2Placement3D/2D`). Covers `IfcExtrudedAreaSolid` with rectangular and arbitrary profiles, `IfcMappedItem`/`IfcRepresentationMap` with `IfcCartesianTransformationOperator3D`, and `IfcFacetedBrep`/`IfcClosedShell`. Triangulates by *ear-clipping* and generates flat normals per triangle.

> [!NOTE]
> **Deliberate limits** — Doesn't do booleans, voids, *half-spaces* or curves. That's why it gives much less geometry: it's the readable common minimum. The gap with the others tells you which part of the model depends on the geometric kernel.

### BIMrocket <sub>Pure JavaScript · CSG kernel</sub>

Engine from the Ajuntament de Sant Feliu de Llobregat (Catalonia). Carries its own STEP tokenizer and, above all, a **geometric kernel with real CSG**: `Solid`, `Profile`, `Extruder`, `Revolver` and `BooleanOperator` classes, plus constructors for standard profiles (I, L, T, U, Z, circle…). Builds solids parametrically and then tessellates them.

It's the only one that also rebuilds the **full BIM tree**: each node carries its IFC class and the `IfcProject › IfcSite › IfcBuilding › IfcBuildingStorey › product` hierarchy. It breaks down each element into faces and edges as separate objects, hence the far higher mesh count. It's the one that best resolves the voids and booleans the native parser skips, at the cost of more geometry and time.

> [!NOTE]
> **Visibility** — BIMrocket internally marks part of what it generates as not visible (auxiliary representations, openings). The tool **respects this**: what the engine itself doesn't paint isn't drawn. The console reports how many nodes were skipped.

### ifc-lite <sub>Rust → WebAssembly</sub>

Engine from LTplus AG, with an **exact-arithmetic** CSG kernel validated element by element against IfcOpenShell. The only one of the four that reads **IFC5/IFCX** in addition to IFC2X3, IFC4 and IFC4X3. It works through two separate pipelines depending on the file:

| Format | How it's processed |
|---|---|
| **IFC STEP** `.ifc` | WASM compiled from Rust; geometry is extracted already tessellated and converted to the common format. |
| **IFC5 / IFCX** `.ifcx` | Pure **JavaScript component, no WASM**: composes the layered JSON graph (USD-style) and returns already-tessellated meshes with IFC class and colors with transparency. |

> [!NOTE]
> **Vertical axis** — ifc-lite exports in **Y-up** (USD convention). The tool automatically applies the rotation to Z-up so it lines up with the other three engines and the comparison can be overlaid.

### Why they diverge

On the same test IFC (116 elements with geometry), the four engines give very different results. This divergence *is* the information you're looking for:

| Parser | Meshes | Vertices | Booleans and voids |
|---|---|---|---|
| **Native STEP** | 115 | 5,820 | no |
| **web-ifc** | 119 | 30,258 | yes |
| **ifc-lite** | 116 | 36,030 | yes (exact) |
| **BIMrocket** | 239 | 121,896 | yes (CSG) |

*When a piece looks right in one panel and wrong in another, the comparison tells you whether the problem lies in the IFC file or in that particular engine.*

> [!WARNING]
> **IFC5 / IFCX** — If you load a `.ifcx`, only **ifc-lite** can read it: the tool automatically switches every viewer to this engine. If you force another one, the panel will tell you explicitly instead of failing silently.

> [!NOTE]
> **Performance** — Each parser processes the file only **once**: the result is cached in memory, so switching renderer doesn't re-parse the IFC. When an engine is no longer used by any viewer, its geometry is freed from memory.

## 05. WASM engines

**What a WASM engine is and what it implies**

**WebAssembly** (WASM) is a binary instruction format that browsers know how to run. It's not meant to be written by hand, but as a *compilation target*: code written in C, C++ or Rust is compiled to it so it runs inside the browser at near-native speed.

The reason it exists is that JavaScript is good for logic and interface, but for intensive computation — tessellating geometry, solving boolean operations, processing B-reps — it's slower and less predictable. With WASM, a geometric kernel written in C++ or Rust that has existed for years can be ported to the browser with practically no rewriting.

### How it works

The browser has a WASM virtual machine alongside the JavaScript engine. Each module lives inside its own **linear memory**: a contiguous block of bytes it can't escape. A WASM module has no access to the DOM, the GPU, or the disk. Everything it needs from the outside has to be provided by JavaScript, and everything it produces comes out through that same shared memory.

That's why, besides the binary, each engine carries an auto-generated JavaScript *glue* layer that initializes memory, translates calls and converts data between the two worlds.

### The tool's two WASM engines

| Engine | Source language | Toolchain | Binary size |
|---|---|---|---|
| **web-ifc** | C++ | Emscripten | 1.3 MB |
| **ifc-lite** | Rust | wasm-bindgen | 3.7 MB |

*The other two parsers — **native STEP** and **BIMrocket** — are pure JavaScript. So the tool's comparison isn't only about geometric coverage: it also contrasts two quite different execution strategies.*

### Practical consequences

| Effect | Why |
|---|---|
| APIs have an unusual shape | Functions like `GetVertexArray(pointer, size)` don't return a list: they receive an **address and a length** inside the module's linear memory, and JavaScript creates a view over it. For the same reason, geometry has to be freed explicitly: that memory isn't managed by JavaScript's garbage collector. |
| The app file is fairly heavy | The `.wasm` is an independent binary normally downloaded separately. Since the tool is a single file with no CDN, it's embedded in base64, which increases its size by 33%. |
| Bytes are injected, not downloaded | By default, these engines *download* the binary and compile it in *streaming*. Inside restricted-environment viewers (sandbox, CSP) that download is blocked, so the tool delivers the bytes already loaded in memory and avoids any network access. |
| Runs on a single thread | WASM supports threads, but they require `SharedArrayBuffer`, which in turn requires cross-origin isolation HTTP headers. A local file doesn't have those, so single-thread mode is forced. |

> [!NOTE]
> **WASM doesn't always mean faster** — Crossing the boundary between JavaScript and WASM has a cost, and so does copying large buffers. With small models, a pure-JavaScript parser can be faster. The processing time each viewer shows in the status bar lets you check this against real models.

> [!NOTE]
> **Privacy** — WASM modules run inside the browser's own sandbox: no access to disk or network. IFC files never leave the computer.

## 06. Renderers and modes

**The engine that draws and the display modes**

The *renderer* is what paints the geometry produced by the parser onto the screen. It's the viewer's **second dropdown** (after the → arrow). Parser and renderer are independent: any combination is valid, and both share the same camera.

### Three.js <sub>MIT · r160</sub>

Full 3D library. Builds a scene with lighting (hemispheric + directional) and PBR materials, and has two cameras — perspective and orthographic — that switch depending on the chosen projection. Supports **all** display modes, since it generates derived geometry (wireframe and edges) for the line-based modes.

### Raw WebGL2 <sub>Own GLSL</sub>

Direct implementation on top of WebGL2, with no library in between: GLSL *shaders* written for the tool (one vertex, one fragment), vertex array objects (VAO), interleaved position/normal buffers, and 32-bit indices. For wireframe mode it builds a line buffer from the triangles. Supports four modes.

Being low-level code, it's useful for seeing the geometry **with no intermediate layer**: what it draws is exactly what the parser delivered.

| Renderer | Modes | Projections |
|---|---|---|
| **Three.js** | All seven | Perspective and orthographic |
| **Raw WebGL2** | Solid, Wireframe, Vertices, Normals | Perspective and orthographic |

> [!NOTE]
> **If a panel fails** — Renderer errors are never silent: if a *shader* fails to compile, the panel shows the driver's message and the GLSL code with the offending line marked; if the browser releases the GPU, the tool warns you and recovers on its own. Hovering over the renderer selector shows the real GPU and GLSL version.

### Display modes

The third dropdown chooses how the model is drawn. The list adapts to the renderer: with raw WebGL2 only the first four are available.

| Mode | Result |
|---|---|
| **Solid** | Lit surfaces. This is the default mode. |
| **Wireframe** | The edges of every triangle. |
| **Vertices** | Point cloud of the mesh's vertices. |
| **Normals** | Color based on each face's normal direction. |
| **Flat** <sub>(Three.js only)</sub> | Flat shading, no smoothing between faces. |
| **Edges** <sub>(Three.js only)</sub> | Only marked edges (sharp-angle corners). |
| **X-Ray** <sub>(Three.js only)</sub> | Semi-transparent model to see its interior. |

> [!NOTE]
> **Background** — The background color is chosen in the top bar and applies to all viewers at once.

## 07. Views and navigation

**Moving the camera, projection and standard views**

Each viewer is orbited directly with the mouse over the 3D area. There are no keyboard shortcuts: navigation is by mouse and by the panel header's selectors.

| Mouse | Action |
|---|---|
| Drag (left button) | Orbit around the model. |
| <kbd>Shift</kbd> + drag · right button + drag | Pan the view. |
| Mouse wheel | Zoom in / out. |

### Viewer selectors and buttons

| Control | Function |
|---|---|
| **Projection** | Perspective (conic) or Axonom. (orthogonal). |
| **View** | Standard views: Iso, Top, Front, Back, Left, Right, Bottom. |
| <kbd>⤢</kbd> | Frames the model to the panel's current window. |
| <kbd>⟲</kbd> | Resets the camera to how it was when the model was opened. |
| **Orientation** | **Z-up / Y-up** selector: manually choose which axis is vertical, depending on how the IFC was exported. Orbit rotation stays consistent with both options. |

> [!NOTE]
> **Axis triad** — At the bottom left of each viewer are the axes, which rotate with the camera: **X** in red, and the other two labels depend on orientation — with Z-up you see **X, Y, UP**; with Y-up, **X, Z, UP**. It lets you always know which way you're looking.

> [!NOTE]
> **Syncing viewers** — With **⟳ Sync** active, moving one viewer's camera moves all visible viewers' cameras at the same time. It's the way to see exactly the same view with different parsers or renderers. Click the button again to turn it off.

## 08. Troubleshooting

**Troubleshooting and frequently asked questions**

<details>
<summary>A panel shows up empty or with less geometry than another</summary>

That's exactly what you're comparing. The native STEP parser only covers a subset of explicit geometry (extrusions, faceted breps, mapped elements), while web-ifc, ifc-lite and BIMrocket resolve booleans and voids. Differences between panels are expected and useful: they show how far each engine goes.
</details>

<details>
<summary>A panel says the IFCX format isn't supported</summary>

`.ifcx` (IFC5) files are only read by **ifc-lite**. When you load one, the tool automatically switches all viewers to this engine; if you force another one, the panel will tell you.
</details>

<details>
<summary>Setting-out axes or topography are covering the model</summary>

Use the top bar's **category filter**: Grid, Annotations, Site and Openings are hidden by default. The status bar shows how many meshes were filtered and the breakdown by category.
</details>

<details>
<summary>A heavy IFC uses too much memory</summary>

When you open the file, the **pre-load report** scans it without processing it and estimates vertices and RAM per engine, with a traffic-light indicator and a recommendation on how many viewers to open. You can adjust it or cancel before any engine is invoked. Reducing the number of viewers frees the memory of engines that are no longer used.
</details>

<details>
<summary>You get "WebGL2 not available" or "renderer not available"</summary>

Your browser or hardware doesn't support WebGL2 at that moment. Switch that panel's renderer to Three.js in the second dropdown. The panel always shows the specific reason, and the **console** keeps a log of it.
</details>

<details>
<summary>The model looks tipped over or with the vertical axis rotated</summary>

It depends on how the IFC was exported. Switch the panel's **orientation** selector to Z-up or Y-up and then press <kbd>⤢</kbd> to reframe. The viewer's axis triad confirms which axis is vertical.
</details>

<details>
<summary>I want to go back to the initial view</summary>

<kbd>⟲</kbd> returns the camera to how it was right when the model was opened; <kbd>⤢</kbd> frames the model to the window's current size. For a specific view, use the views dropdown (Iso, Top, Front…).
</details>

<details>
<summary>I switched renderer and don't want to wait for it to re-parse</summary>

No need: each parser's result is cached in memory per file. Switching renderer reuses the geometry already processed; it's only re-parsed if you change parser or load a new file.
</details>

## 09. Licenses and credits

**Component licenses and credits**

**guaitaIFCparser** is distributed under **GNU AGPL-3.0-or-later**. It incorporates third-party components under the following licenses, verified against the versions actually included in the file. The same table can be consulted inside the app, via the **licenses and credits** link in the footer.

| Component | Version | License | Holder |
|---|---|---|---|
| **guaitaIFCparser** <br><sub>app · native STEP parser · WebGL2 renderer</sub> | — | AGPL-3.0-or-later | © 2026 SBS BIM Consulting |
| **web-ifc** <br><sub>parser · C++ → WASM</sub> | 0.0.77 | MPL-2.0 | ThatOpen / web-ifc |
| **BIMrocket** <br><sub>parser · CSG kernel in JS</sub> | — | EUPL-1.1+ *or* LGPLv3+ | © 2021-2026 Ajuntament de Sant Feliu de Llobregat |
| **ifc-lite** <br><sub>parser · Rust → WASM + IFCX in pure JS</sub> | @ifc-lite/wasm · /ifcx | MPL-2.0 | LTplus AG |
| **three.js** <br><sub>renderer</sub> | r160 | MIT | © 2010-2023 three.js authors |

### Compatibility

MIT and MPL-2.0 are compatible with AGPL-3.0. BIMrocket is **dual-licensed** and offers the **LGPLv3+** option, compatible with AGPL-3.0. The original third-party files' license headers are preserved, and the app's HTML file header contains the full notice.

### Source code

The source code of the MPL-2.0 components (web-ifc, ifc-lite) is available in their respective public repositories. The tool's own code is published under AGPL-3.0 in the Guaita Apps repository.

> [!WARNING]
> **Disclaimer** — This information is indicative and does not constitute legal advice. If you redistribute the tool or incorporate it into another project, review the terms of each license.

---

<p align="center"><b>GUAITA</b><sub>IFC PARSER</sub></p>
<p align="center">Multi-parser and multi-renderer IFC comparator · SBS BIM Consulting 2026</p>

<sub>Guaita Apps is distributed "as is", with no warranty. The tools are designed from research and as technical and educational support; verification of results and any liability arising from their use rest solely with the end user.</sub>
