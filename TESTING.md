# netviz — Testing & Verification

This document records how each acceptance check was verified and its result.

## Method

`netviz.html` was served locally with `python -m http.server 8765` and driven in a real
Chromium browser. Verification was primarily **programmatic**: the app exposes its
internal functions under `window.netviz` (state, import, layout, validation, export,
edit operations), and checks were executed by scripting those against the live app and
asserting on `AppState` and the Cytoscape instance. Where a check is inherently visual
(icon rendering, encoding, export legibility), the app's own PNG export was captured,
decoded, and inspected as an image.

**Tooling note:** the automated screenshot facility used during development could not
rasterize the live Cytoscape `<canvas>` (capture timed out). This is a limitation of the
capture harness, not the application — visual confirmation was therefore done by
inspecting the app's exported PNGs (which render the same canvas), and UI chrome was
verified by asserting on the live DOM after real `click()` / Cytoscape `tap` events.

All checks were run against the sample data in `samples/`.

---

## Results

### 1. Loads via `file://` with an empty new project, no console errors — PASS

On load the console logs `netviz ready` with no errors (only a benign message; the custom
wheel-sensitivity warning was removed). `AppState.project` starts with 0 nodes / 0 links.
All libraries register: `cytoscape`, `cytoscapeFcose`, `coseBase`, `layoutBase`,
`cytoscapeExpandCollapse`, `jsyaml`, `Papa`. The 17-icon set builds.
_The app performs no `fetch`/XHR of its own (imports use `FileReader`), so nothing depends
on the page origin — it runs identically from `file://`. During automated testing the
in-app browser blocked `file://` navigation, so the same page was loaded over
`http://127.0.0.1:8765` and confirmed error-free; this also exercises Deployment option (b)._

### 2. Import nodes.csv then links.csv (merge) → 12 nodes, 16 links, correct encoding — PASS

After importing both sample CSVs: **12 nodes, 16 links**. Encoding asserted on edge data:

- IPSec tunnel → `line-style: dotted`, `line-color: #ab47bc` (purple). ✔
- MPLS tunnel → `dotted`, `#66bb6a` (green). ✔
- STP-blocked link → mid-label `⛔`, line desaturated to `#9ea0a2`. ✔
- Planned link → opacity `0.4`. ✔
- 10G DAC width `4px` vs 1G width `2.5px` (visibly thicker). ✔
- 0 spurious validation issues on the clean sample.

Visual confirmation via exported PNG: all device icons render (firewall bricks, l3/l2
switch arrows, hypervisor squares, server, AP wifi, cloud, globe), tunnel dotted lines
and thick teal DAC link are clearly distinct.

### 3. Hover/click behaviors; edit a link VLAN, save, tooltip reflects it — PASS

A real Cytoscape `tap` on an edge opens the editable slide-in panel (`#panel.open`).
Edited link VLAN `30` → `30,99` via the panel field and saved; `AppState` reflects
`30,99`. The tooltip is generated from the same link fields, so it shows the new value.
Node and link tooltips build without error.

### 4. Undo reverts the edit; redo reapplies — PASS

After the VLAN edit: `undo()` → link VLAN back to `30`; `redo()` → `30,99`. Undo/redo
buttons enable/disable correctly; stack capped at 50 snapshots.

### 5. Add a node and draw a link from scratch in an empty project — PASS

Starting from an empty project: **Add Node** twice → 2 nodes; draw-link mode + two node
clicks → **1 link** with two distinct endpoints, rendered as one Cytoscape edge. (The
toolbar **Add Node** button was also confirmed via a real DOM `click()`.)

### 6. Overlay tunnels-only; filter VLAN=10 — PASS

- Overlay = tunnels-only → only tunnel link types visible (`ipsec`, `mpls`, `logical` in
  the sample); every visible edge is a tunnel type.
- Filter VLAN=10 → 7 edges visible, each carrying VLAN 10 (including `1-4094` trunks and
  `10,20,30` / `10,40` / `10,20` lists). Non-matching links hidden.

### 7. Icon mapping + custom icon + save/reload survival — PASS

- Set `icon_map.firewall = router` → both `HQ-FGT01` and `BR-FGT01` resolve to the
  router icon. ✔
- Uploaded a custom image for `HQ-FGT01` (`icon: custom:<id>`) → only `HQ-FGT01` uses
  the custom icon; `BR-FGT01` still follows the type mapping. ✔
- The custom icon is embedded as base64 in the saved project. ✔
- Serialize → reload (`loadProject`) → 12 nodes / 16 links, **all positions identical**,
  per-node icon preserved, `custom_icons` and `icon_map` survived. ✔

### 8. Export PNG 4× with legend + title block; JSON round-trip — PASS

Exported a 4× PNG with title block + baked-in legend (387 KB). Inspected image: legible
title (`Sample Network - HQ + BR`), metadata line (nodes/links/author/export date),
full diagram with icons, and a right-side legend (Media / Tunnels / Speed). JSON export
reopened in a fresh load reproduces the identical diagram including manual positions
(verified in check 7). See `samples/demo-project.json`.

### 9. Export the HTML documentation report — PASS

Generated report contains: an embedded `data:image/png;base64,...` diagram `<img>`; a
**Link Inventory** table with the expected columns (Source, Src If, Target, …); and a
**Node Inventory by Site** section with per-site tables for `HQ`, `BR`, and `WAN`.

### 10. Validation: `media=coax` and a self-loop → both warned, import still succeeds — PASS

Imported a links CSV containing `A,B,coax,physical,up` and a self-loop `C,C,copper,...`.
Both links imported (import did not fail). The Issues panel lists:

- `warn: Link A→B: unknown media "coax" → coerced to copper`
- `warn: Self-loop on C`

### 11. Generate `samples/demo-project.json` — PASS

Built by importing the sample CSVs, running the fCoSE layout, and saving. The file
contains all 12 nodes (each with `x`/`y` positions), all 16 links with full attributes,
and loads back cleanly (verified via `loadProject`).

---

## Additional checks performed

- **Layouts** — fCoSE, hierarchical (by layer), circular-per-site, and grid all run
  without error.
- **Grouping** — Group by Site produces compound parents `HQ`, `BR`, `WAN`, each with
  its child nodes.
- **Search** — matches across name / mgmt IP / vendor / model / site and highlights +
  zooms to hits.
- **VLAN coloring overlay** — edges recolor to per-VLAN `hsl(...)` values.
- **Theme** — light theme toggles `body.light` and restyles the graph; dark is default.
- **Toolbar wiring** — Add Node, Filters, Issues, Settings (17 icon-map rows), Export
  menu, and Theme buttons all respond to real DOM clicks.
- **Console** — no errors after a full exercise of import, layout, grouping, filtering,
  editing, undo/redo, theming, and export.
