# netviz — Network Diagram Automation & Documentation

netviz is a single-file, offline-capable web application for documenting network
topologies. You feed it link/node/location data (CSV, JSON, or YAML), it renders an
interactive, richly encoded topology graph, lets you edit everything in-app — including
a physical (site → building → floor → cabinet) view, shapes and text annotations, and a
large device-icon library — and exports images, data files, and a full HTML
documentation report.

There is **no backend and no build step**. `netviz.html` opens directly from your
file system and works offline after the first load (CDN libraries are cached by the
browser).

---

## Quick start

1. Open `netviz.html` in a modern browser (Chrome, Edge, Firefox). Double-clicking the
   file (`file://`) works, or serve it / use the hosted version — see **Deployment**.
2. You start with an empty project. Click **Import CSV** and select `samples/nodes.csv`
   and `samples/links.csv` (both at once). A mapping wizard appears — columns auto-map;
   pick a merge mode and click **Import & Validate**. Add `samples/locations.csv` to
   populate the physical hierarchy.
3. Or click **Open** and load `samples/demo-project.json` for a ready-made, laid-out
   diagram with sites, cabinets, and placed nodes.

> First load needs an internet connection so the browser can fetch the CDN libraries.
> After that they are cached and the app works fully offline.

---

## Interface

- **Toolbar (top)** — New / Open / Save / Import CSV / Export, search, Filters, Overlay,
  and toggles for the Tools panel, Legend, theme, Undo/Redo, Issues, Settings, and the
  keyboard-shortcuts help (`?`).
- **Tools panel (left, dockable & hideable)** — all creation/canvas tools with icons:
  **Add** (Node, Site, Building, Floor, Cabinet, Rectangle, Circle, Text, Manage
  locations…), **Draw Link**, **View** (Logical / Physical), **Layout** (5 options),
  **Group by Site**, and **Fit**. Hide it like the legend; a floating **Tools** button
  brings it back.
- **Detail panel (right)** — click any node, link, location, or annotation to edit it.
- **Legend** (bottom-right) and **status bar**.
- **Responsive** — below 768px the toolbar collapses into a hamburger drawer while a
  compact quick-action bar (Search / Add / Fit) stays visible.

---

## Features

### Views

- **Logical view** — the network topology (nodes + links), with force-directed and other
  layouts and optional per-site compound grouping.
- **Physical view** — a nested **Site › Building › Floor › Cabinet** container layout with
  each device drawn inside its assigned cabinet/floor; unplaced nodes float. Links still
  render across cabinets.

### Locations (physical hierarchy)

- Add **Sites, Buildings, Floors, Cabinets**, each with its own fields (site:
  code/address/city/region/country/lat-long/contact; building: code/address/floors;
  floor: level/elevation; cabinet: rack id/row/position/RU/power; all with notes).
- **Locations manager** — a tree view to add child locations, edit, and delete (cascades
  to descendants; affected nodes become unplaced), plus **Import CSV**.
- **Node placement** — assign a node to a cabinet or floor; its **site is derived
  automatically** from the hierarchy.

### Annotations (shapes & text)

- Add **rectangles, circles, and text boxes**.
- Controls: fill color + opacity, border color + width, text/font color + size, and a
  **z-index** with send-to-back / bring-to-front (draw shapes behind or in front of the
  topology).
- **On-canvas resize handles** (8-point) plus drag-to-move; annotations appear in both
  views and are excluded from auto-layout so they stay where you put them.

### Icons

- **Visual icon picker** on the node panel showing generic type icons, a device-icon
  library, and your custom uploads side by side. **Picking a device icon also sets the
  node's type automatically.**
- **Device-icon library** — Cisco Router / Switch / L3 Switch / Firewall, FortiGate
  Firewall, FortiSwitch, FortiAP, Omada Controller, TP-Link AP / Switch, Hikvision
  Switch / Camera, Wireless Controller (WLC), IP Telephone, Printer, Sensor, Internal
  Camera, Yealink TV, Internet, MPLS Cloud, Windows PC, Windows Server, VMware ESXi,
  Dell Server, HP Server. All are inline SVG (self-contained, no external requests).
- Per-type icon mapping in Settings; upload custom icons (embedded as base64 so the
  project stays portable).

### Editing

- Click any element for an editable panel; add/delete nodes, links, locations, and
  annotations; draw-link mode; right-click context menu; box-select bulk edit; drag with
  position persistence; **50-step undo/redo**.

### Import / export

- **Import** — CSV with a column-mapping wizard + validation report (nodes, links, or
  locations); JSON/YAML project import; merge modes (replace / merge-add /
  update-matching).
- **Export** — PNG (1×/2×/4×, transparent or themed background, optional baked-in title
  block + legend), JPEG, project JSON/YAML, CSV round-trip (nodes + links), and a
  standalone HTML documentation report (embedded diagram, link inventory, per-site node
  inventory).

### Visual encoding, search, overlays

- **Rich link encoding** — color/style by media, dotted colored lines per tunnel type,
  width by speed, STP-blocked and link-down badges, planned-link opacity, LAG bundle
  badge, per-node accent ring on select.
- **Search & filters** — search matches name / mgmt IP / VLAN / subnet / circuit ID and
  zooms to hits; composable filters (site, layer, link type, VLAN, media, status) that
  AND across categories and OR within a category.
- **Overlays** — physical-only, tunnels-only, or combined; VLAN coloring; show/hide
  decommissioned links.

### Validation & theme

- **Validation panel** — never blocks; warns about duplicate link IDs, duplicate
  interface pairs, coerced unknown enum values, malformed subnet/VLAN strings,
  self-loops, STP-blocked routed links, and tunnels with no underlying physical path.
- **Dark theme** by default with a light-theme toggle.

---

## Keyboard shortcuts

Press `?` in the app for the full list.

| Key                                        | Action                                  |
| ------------------------------------------ | --------------------------------------- |
| `N`                                        | Add node                                |
| `D`                                        | Draw-link mode                          |
| `Ctrl/Cmd + D`                             | Duplicate selection                     |
| `Ctrl/Cmd + A`                             | Select all                              |
| `Del` / `Backspace`                        | Delete selection                        |
| `Arrows` (`Shift` = ×10)                   | Nudge selection                         |
| `Ctrl/Cmd + Z` / `Ctrl/Cmd + Y`            | Undo / Redo                             |
| `F` / `0`                                  | Fit graph to screen                     |
| `+` / `-`                                  | Zoom in / out                           |
| `V`                                        | Toggle Logical / Physical view          |
| `G`                                        | Group by site                           |
| `L`                                        | Toggle legend                           |
| `T`                                        | Toggle theme                            |
| `/`                                        | Focus the search box                    |
| `I`                                        | Open validation issues                  |
| `Ctrl/Cmd + S`                             | Save project (download JSON)            |
| `?`                                        | Keyboard-shortcuts help                 |
| `Esc`                                      | Cancel draw-link, close panels/menus    |

---

## CSV schema reference

Import is column-order independent — the wizard maps your headers to netviz fields, and
auto-matches columns whose names line up. Referenced nodes that are not explicitly
defined are auto-created with `type: unknown`.

### Nodes

| Field             | Notes                                                                                                                                                                              |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | **required**; used as `id` if `id` omitted                                                                                                                                        |
| `id`              | optional unique id (auto-derived from name)                                                                                                                                       |
| `type`            | `router, l3-switch, l2-switch, firewall, server, hypervisor, access-point, wlc, load-balancer, storage, camera, printer, workstation, ip-phone, sensor, tv, cloud, internet, isp, unknown` |
| `mgmt_ip`         | management IP                                                                                                                                                                     |
| `site`            | site name (auto-derived from `location` when the node is placed)                                                                                                                  |
| `layer`           | `core, distribution, access, edge, dmz, wan`                                                                                                                                      |
| `vendor`, `model` | free text                                                                                                                                                                         |
| `icon`            | icon key, `vendor:<device>` (device-icon library), or `custom:<id>` (usually left blank)                                                                                          |
| `location`        | id of the cabinet/floor the node lives in (physical view)                                                                                                                         |
| `notes`           | free text                                                                                                                                                                         |
| `x`, `y`          | manual coordinates (blank = auto-layout)                                                                                                                                          |

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

### Locations

A location CSV needs at least **`kind`** and **`name`**. `parent` references the parent
location by name or id (resolved after import).

| Field         | Notes                                                        |
| ------------- | ----------------------------------------------------------- |
| `kind`        | **required**; `site, building, floor, cabinet`             |
| `name`        | **required**                                                |
| `parent`      | parent location by name or id (`building`→site, etc.)      |
| `code`        | site/building code or rack id                               |
| `address`, `city`, `region`, `country` | site fields                       |
| `floors_count` | building field                                             |
| `level`, `elevation`                   | floor fields                      |
| `row`, `position`, `ru`, `power`       | cabinet fields                    |
| `notes`       | free text                                                   |

Unknown enum values are coerced to a safe default (`unknown` / `copper` / `physical` /
`up` / `n/a` / `site`) and reported in the Issues panel — import never fails.

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
- **LAG** → thicker line plus a `⇉` bundle badge at the midpoint.

---

## Project file format

Saved projects are JSON with this shape:

```json
{
  "meta": { "name": "", "schema_version": 1, "author": "", "created": "", "modified": "" },
  "settings": { "theme": "dark", "layout": "fcose", "icon_map": {}, "custom_icons": {} },
  "nodes": [],
  "links": [],
  "locations": [],
  "annotations": [],
  "groups": []
}
```

`custom_icons` holds base64 data URIs, so a project with uploaded icons is fully
self-contained and portable. Use **Save** to download the JSON and **Open** to reload it.

---

## Deployment

`netviz.html` is a self-contained static file — deploy it anywhere.

- **Hosted (GitHub Pages)** — this repo is published at
  **https://revul93.github.io/netviz/** (the root redirects to `netviz.html`).
- **Local / file share** — copy `netviz.html` anywhere (network share, USB, Desktop) and
  open it from `file://`. Keep the `samples/` folder next to it for the sample data.
- **Static host / team** — serve the folder over HTTP:

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
| layout-base / cose-base / cytoscape-fcose 2.2 | jsDelivr     | fCoSE force-directed layout |
| cytoscape-expand-collapse 4.1                 | jsDelivr     | Site-group collapse/expand  |

If fCoSE fails to load for any reason, netviz automatically falls back to Cytoscape's
built-in `cose` layout. Device and generic icons are inline SVG (no external requests).

---

## Samples

- `samples/demo-project.json` — a ready-made project (two sites, cabinets, placed nodes,
  logical + physical layouts).
- `samples/nodes.csv`, `samples/links.csv` — node/link import example.
- `samples/locations.csv` — physical-hierarchy import example.

All sample data is fictional.
