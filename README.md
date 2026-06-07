# NH Corridor Accessibility & Road Network Gap Analysis — Karnataka

## Overview
This analysis maps the accessibility of Karnataka's road network relative to
its National Highway (NH) spine. It identifies how much of the state falls
within defined buffer zones around NHs, quantifies local road density across
the state using a hexagonal grid, and flags areas with above-median road
density that remain disconnected from the national highway network.

---

## Input Layers

  Layer                  Type     Features  CRS         Key Fields
  ---------------------  -------  --------  ----------  ----------------------------
  karnataka              Polygon  1         EPSG:4326   ST_NM
  karnataka_major_roads  Line     42,475    EPSG:4326   fclass, maxspeed, bridge, tunnel
  national_highways      Line     3,119     EPSG:4326   ref, fclass, length_km

All layers reprojected to EPSG:32643 (WGS 84 / UTM Zone 43N) before processing
to ensure accurate distance and area measurements.

---

## Methodology

### Step 1 — NH Corridor Reconstruction (nh_corridors.gpkg)
National highway segments were dissolved by `ref` to reconstruct 83 full
corridor lines. A TOTAL_KM field was computed as $length / 1000. This gives
a per-corridor length summary directly from geometry.

Top corridors by length:
  NH48: 1101.7 km  |  NH50: 783.9 km  |  NH75: 653.1 km
  NH150A: 645.4 km |  NH69: 565.1 km  |  NH275: 497.2 km

### Step 2 — NH Accessibility Buffer Zones
Three concentric buffers (10 km, 25 km, 50 km) were generated around all NH
corridors (dissolved, SEGMENTS=16) and clipped to the Karnataka boundary.
These represent:
  - 10 km: immediate commercial corridor zone
  - 25 km: viable commute/freight access zone
  - 50 km: outer accessibility limit

Key finding: 650 of 652 hexagonal grid cells (99.7%) fall within the 50 km
NH buffer, indicating near-complete national highway coverage of Karnataka.

### Step 3 — Road Density Hexagonal Grid (road_density_hex.gpkg)
A 20 km hexagonal grid was generated over Karnataka (652 cells after clipping).
Roads from karnataka_major_roads were clipped to Karnataka, segment lengths
computed in metres, then summed per cell using a spatial summary join. Density
was calculated as:

  ROAD_DENS = (sum_length_m / 1000) / (cell_area_m2 / 1000000)  [km/km²]

  Min: 0.00 | Median: 0.84 | Max: 9.74 km/km²

Symbolized with a 5-class sequential colour ramp (YlGnBu):
  0–2 / 2–5 / 5–10 / 10–20 / 20+ km/km²

### Step 4 — High-Density Gap Cells (road_density_gap.gpkg)
Hex cells disjoint from the 50 km NH buffer were extracted and filtered to
those with ROAD_DENS above the state median (> 0.84 km/km²). These represent
areas with meaningful local road infrastructure but no NH connectivity.

Result: 2 gap cells identified — consistent with the high NH coverage finding
above. Karnataka's NH network is dense enough that virtually no road-active
area sits outside the 50 km accessibility threshold.

---

## Output Files

  File                                    Description
  --------------------------------------  ----------------------------------------
  nh_corridors.gpkg                       83 NH corridor lines with TOTAL_KM
  nh_buffer_10km.gpkg                     10 km NH accessibility zone
  nh_buffer_25km.gpkg                     25 km NH accessibility zone
  nh_buffer_50km.gpkg                     50 km NH accessibility zone
  nh_gap_zone.gpkg                        State area beyond 50 km NH buffer
  road_density_hex.gpkg                   652 hex cells with ROAD_DENS (km/km²)
  road_density_gap.gpkg                   High-density cells outside NH 50km zone
  NH_Corridor_Accessibility_Karnataka.qgz QGIS project with all layers + symbology

---

## Symbology

  Layer                   Style
  ----------------------  ---------------------------------------------------
  NH Corridors            Red line (#d73027), width 0.8
  NH Buffer 10km          Dark green fill (#1a6338)
  NH Buffer 25km          Medium green fill (#52ac60)
  NH Buffer 50km          Light green fill (#b8e0b0)
  NH Gap Zone             Red fill (#d7191c)
  Road Density Hex        YlGnBu graduated (5 classes, 0–2–5–10–20–max)
  High-Density Gap Cells  Orange-red fill (#f03b20)

---

## Key Finding
Karnataka's national highway network achieves near-complete spatial coverage —
99.7% of the state (by 20 km hex cells) lies within 50 km of an NH. The two
cells outside that threshold still carry above-median local road density,
suggesting they are not isolated but simply lack direct NH linkage. NH48 is
the dominant corridor at 1,101.7 km.

---

## Limitations
- NH corridor dissolve uses the raw `ref` field which contains inconsistent
  tagging in OSM (e.g. 'NH 150A', 'NH150', 'NH150;SH125'). Corridors sharing
  a ref tag but with OSM tagging variants may appear as separate features.
- Hex grid density counts all fclass types equally — motorways and tertiary
  roads contribute the same weight to ROAD_DENS.
- Buffer analysis uses planar (Euclidean) distance in UTM 43N. Actual road
  travel distance will exceed straight-line buffer distance.

---
Analysis date : May 2026
CRS           : EPSG:32643 (WGS 84 / UTM Zone 43N)
Data source   : OpenStreetMap via Karnataka GIS dataset
