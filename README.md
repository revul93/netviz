# netviz — Network Diagram Automation & Documentation

netviz is a single-file, offline-capable web application for documenting network
topologies. You feed it link/node data (CSV, JSON, or YAML), it renders an
interactive, richly encoded topology graph, lets you edit everything in-app, and
exports images, data files, and a full HTML documentation report.

There is **no backend and no build step**. `netviz.html` opens directly from your
file system and works offline after the first load (CDN libraries are cached by the
browser).

---

## Quick start

1. Open `netviz.html` in a modern browser (Chrome, Edge, Firefox). Double-clicking the
   file (`file://`) works, or serve it — see **Deployment** below.
2. You start with an empty project. Click **Import CSV** and select `samples/nodes.csv`
   and `samples/links.csv` (you can select both at once). A mapping wizard appears —
   the columns auto-map; pick a merge mode and click **Import & Validate**.
3. Or click **Open** and load `samples/demo-project.json` for a ready-made, laid-out
   diagram.

> First load needs an internet connection so the browser can fetch the CDN libraries.
> After that they are cached and the app works fully offline.

---

## Features

- **Rich visual encoding** — line color/style by media, dotted colored lines per tunnel
  type, line width by speed, STP-blocked and link-down badges, planned-link opacity,
  original inline-SVG icons for all 17 device types, per-node accent ring on select.
- **Import** — CSV with a column-mapping wizard + validation report; JSON/YAML project
  import; merge modes (replace / merge-add / update-matching).
- **Full editing** — click any node/link for an editable slide-in panel; add/delete
  nodes and links; draw-link mode; right-click context menu; box-select bulk edit;
  drag with position persistence; 50-step undo/redo.
- **Layouts** — force-directed (fCoSE), hierarchical by layer, circular-per-site, grid,
  and manual. Group by site into collapsible compound containers.
- **Search & filters** — search matches name / mgmt IP / VLAN / subnet / circuit ID and
  zooms to hits; composable filters (site, layer, link type, VLAN, media, status) that
  AND across categories and OR within a category.
- **Overlays** — physical-only, tunnels-only, or combined; VLAN coloring; show/hide
  decommissioned links.
- **Export** — PNG (1×/2×/4×, transparent or themed background, optional baked-in title
  block + legend), JPEG, project JSON/YAML, CSV round-trip (nodes + links), and a
  standalone HTML documentation report (embedded diagram image, link inventory table,
  per-site node inventory).
- **Settings** — per-type icon mapping, upload custom icons (embedded as base64 so the
  project file stays portable), theme, project metadata.
- **Validation panel** — never blocks; warns about duplicate link IDs, duplicate
  interface pairs, unknown enum values (coerced + flagged), malformed subnet/VLAN
  strings, self-loops, STP-blocked routed links, and tunnels with no underlying
  physical path.
- **Dark theme** by default with a light-theme toggle.

---

## Keyboard shortcuts

| Key                                        | Action                               |
| ------------------------------------------ | ------------------------------------ |
| `Ctrl/Cmd + Z`                             | Undo                                 |
| `Ctrl/Cmd + Y` (or `Ctrl/Cmd + Shift + Z`) | Redo                                 |
| `Ctrl/Cmd + S`                             | Save project (download JSON)         |
| `Del` / `Backspace`                        | Delete selected node(s) / link       |
| `F`                                        | Fit graph to screen                  |
| `/`                                        | Focus the search box                 |
| `Esc`                                      | Cancel draw-link, close panels/menus |

---

## CSV schema reference

Import is column-order independent — the wizard maps your headers to netviz fields, and
auto-matches columns whose names line up. Referenced nodes that are not explicitly
defined are auto-created with `type: unknown`.

### Nodes

| Field             | Notes                                                                                                                                                                |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | **required**; used as `id` if `id` omitted                                                                                                                           |
| `id`              | optional unique id (auto-derived from name)                                                                                                                          |
| `type`            | `router, l3-switch, l2-switch, firewall, server, hypervisor, access-point, wlc, load-balancer, storage, camera, printer, workstation, cloud, internet, isp, unknown` |
| `mgmt_ip`         | management IP                                                                                                                                                        |
| `site`            | site name (used for grouping and per-site inventory)                                                                                                                 |
| `layer`           | `core, distribution, access, edge, dmz, wan`                                                                                                                         |
| `vendor`, `model` | free text                                                                                                                                                            |
| `icon`            | icon key or `custom:<id>` (usually left blank)                                                                                                                       |
| `notes`           | free text                                                                                                                                                            |
| `x`, `y`          | manual coordinates (blank = auto-layout)                                                                                                                             |

### Links

| Field                                                          | Notes                                                                |
| -------------------------------------------------------------- | -------------------------------------------------------------------- |
| `source`, `target`                                             | **required**; node names or ids                                      |
| `source_if`, `target_if`                                       | interface names                                                      |
| `speed`                                                        | `10M, 100M, 1G, 10G, 25G, 40G, 100G`                                 |
| `media`                                                        | `copper, fiber-sm, fiber-mm, dac, wireless, serial, virtual`         |
| `link_type`                                                    | `physical, lag, ipsec, gre, mpls, sdwan, vpn-client, vxlan, logical` |
| `stp_state`                                                    | `forwarding, blocked, n/a`                                           |
| `vlan`                                                         | single (`30`), list (`"10,20,30"`), or range (`1-4094`)              |
| `subnet`                                                       | e.g. `10.0.0.0/30`                                                   |
| `status`                                                       | `up, down, planned, decommissioned`                                  |
| `bandwidth_committed`, `provider`, `circuit_id`, `description` | free text                                                            |

Unknown enum values are coerced to a safe default (`unknown` / `copper` / `physical` /
`up` / `n/a`) and reported in the Issues panel — import never fails.

### Visual encoding legend

- **Media** → copper `#9aa0a6` solid · fiber-sm `#fbbc04` solid · fiber-mm `#ff8a00`
  solid · dac `#00bfa5` solid · wireless `#4d9fff` dashed · serial `#b39ddb` dash-dot ·
  virtual dotted (colored by tunnel type).
- **Tunnels (dotted)** → ipsec `#ab47bc` · gre `#8d6e63` · mpls `#66bb6a` ·
  sdwan `#26c6da` · vpn-client `#ec407a` · vxlan `#7e57c2` · logical `#78909c`.
- **Speed → width** → 10M/100M 1.5px · 1G 2.5px · 10G 4px · 25G 5px · 40G 6px · 100G 8px.
- **STP blocked** → `⛔` badge at midpoint, line desaturated.
- **Status** → down `#ef5350` with `✕` badge · planned 40% opacity · decommissioned
  hidden unless the "Show decommissioned" overlay is on.
- **LAG** → thicker line plus a `⇉` bundle badge at the midpoint. _(Cytoscape does not
  render true parallel multi-edges cleanly, so LAG is shown as a thick line with a
  bundle badge rather than a doubled line.)_

---

## Project file format

Saved projects are JSON with this shape:

```json
{
  "meta": {
    "name": "",
    "schema_version": 1,
    "author": "",
    "created": "",
    "modified": ""
  },
  "settings": {
    "theme": "dark",
    "layout": "fcose",
    "icon_map": {},
    "custom_icons": {}
  },
  "nodes": [],
  "links": [],
  "groups": []
}
```

`custom_icons` holds base64 data URIs, so a project with uploaded icons is fully
self-contained and portable. Use **Save** to download the JSON and **Open** to reload it.

---

## Deployment

"Deploy" is just the file itself.

- **(a) Local / file share** — copy `netviz.html` anywhere (network share, USB, your
  Desktop) and open it. It runs from `file://`. Keep the `samples/` folder next to it if
  you want the sample data handy.
- **(b) Static host / team** — serve the folder over HTTP so a team can reach it:

  ```bash
  cd netviz
  python -m http.server 8000
  # then browse to http://localhost:8000/netviz.html
  ```

  Any static host works (Nginx, GitHub Pages, S3, an internal web server). No server-side
  code is required.

---

## Dependencies

Loaded from CDN at runtime (cached for offline use afterward):

| Library                                       | Source       | Purpose                     |
| --------------------------------------------- | ------------ | --------------------------- |
| Cytoscape.js 3.30                             | cdnjs        | Graph rendering             |
| PapaParse 5.4                                 | cdnjs        | CSV parsing                 |
| js-yaml 4.1                                   | cdnjs        | YAML import/export          |
| layout-base / cose-base / cytoscape-fcose 2.2 | **jsDelivr** | fCoSE force-directed layout |
| cytoscape-expand-collapse 4.1                 | **jsDelivr** | Site-group collapse/expand  |

> **Note on CDN choice:** the build spec asked for cdnjs. cdnjs publishes Cytoscape,
> PapaParse, js-yaml, and layout-base, but it does **not** host `cose-base`,
> `cytoscape-fcose`, or `cytoscape-expand-collapse`. Those three are therefore loaded
> from jsDelivr (the packages' documented distribution CDN). If fCoSE fails to load for
> any reason, netviz automatically falls back to Cytoscape's built-in `cose` layout.

**SVG image export** is intentionally omitted: `cytoscape-svg` is not published on cdnjs,
and per the build spec SVG export is dropped when the plugin is unavailable. PNG (up to
4×) and JPEG cover the raster export needs, and the HTML report embeds a PNG.

---

## Roadmap (out of scope for this build)

- Project versioning and visual diff between two saved states.
- CDP/LLDP neighbor-discovery import (auto-build topology from device output).
- True parallel-edge rendering for LAG bundles.
