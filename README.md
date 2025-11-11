# OSMnx Street & Transit Downloader · Urban Network Analysis
Toolkit (Jupyter Notebook) for downloading, filtering, and exporting OpenStreetMap data using OSMnx, with optional centrality measures and quick visualizations.
The notebook supports:
📥 Downloading street networks for a place, polygon, or bounding box
🧰 Filtering by highway tags (e.g., motorway, primary, residential, track, etc.)
🚇 Fetching other OSM layers (e.g., public transport stops, rail, cycleways)
🧮 Computing basic centrality metrics (degree, betweenness on the primal graph)
💾 Exporting to GeoPackage (.gpkg) and GeoParquet (.parquet)
🗺️ Quick maps (static) to validate data quality and coverage

Notebook: osmnx_vias.ipynb
Recommended audience: urban planners, architecture/urbanism students, transport and spatial analysis researchers.

To cite: Maffini, A. L. (2025). Urban Segment Network - OSM (v.01). Zenodo. https://doi.org/10.5281/zenodo.17583194

## Table of Contents

1. Project Structure
2. Getting Started
2.1. Environment (conda/mamba)
2.2. Environment (pip)
3. Usage

3.1. Configure Area of Interest (AOI)

3.2. Select Street Types

3.3. Download Other Layers (Transit, Rail, Cycleways)

3.4. Exports

3.5. Optional: Centrality

4. Outputs
5. Tips & Troubleshooting
6. Reproducibility

## 1. Project Structure
(```)
.
├── osmnx_vias.ipynb          # Main notebook
├── data/
│   ├── raw/                   # Raw downloads (OSM extracts, cache)
│   ├── processed/             # Cleaned/filtered layers
│   └── exports/               # .gpkg / .parquet outputs
└── README.md
(```)

The notebook will create data/ subfolders automatically if they don’t exist.

## 2. Getting Started
### 2.1. Environment (conda/mamba)

OSMnx and GeoPandas depend on compiled libs. A conda/mamba environment is the most reliable:
(```)
mamba create -n osmnx-env python=3.11 -y
mamba activate osmnx-env
(```)


### Core stack
(```)
mamba install -c conda-forge osmnx geopandas pyproj shapely fiona rtree pyarrow -y
(```)


### Optional analysis/plots
(```)
mamba install -c conda-forge networkx momepy matplotlib -y
(```)


### Jupyter
(```)
mamba install -c conda-forge jupyterlab ipykernel -y
python -m ipykernel install --user --name osmnx-env --display-name "Python (osmnx-env)"
(```)

Open JupyterLab and select the kernel Python (osmnx-env).

### 2.2. Environment (pip)

If you prefer pip, ensure you’re on Python 3.10–3.12 and install wheels from PyPI:
(```)
python -m venv .venv
(```)
# Windows
(```)
.venv\Scripts\activate
(```)
# macOS/Linux
(```)
source .venv/bin/activate
(```)
(```)
pip install --upgrade pip
pip install osmnx geopandas pyproj shapely fiona rtree pyarrow
pip install networkx momepy matplotlib jupyterlab
(```)

On Windows, if rtree fails, try installing via conda or use precompiled wheels.
If saving to Parquet fails, ensure pyarrow is installed and versions are consistent with GeoPandas.

## 3. Usage

Open osmnx_vias.ipynb and run cells top-to-bottom. The notebook is fully commented for teaching/learning.

### 3.1. Configure Area of Interest (AOI)

Choose one method in the notebook:
(```)
Place name (geocoder):
PLACE = "Porto Alegre, Rio Grande do Sul, Brazil"
(```)

Polygon (WKT/GeoJSON) or BBox:
(```)
BBOX = (-51.30, -30.25, -51.00, -30.00)  # (west, south, east, north)
(```)

### 3.2. Select Street Types

Define the highway tags to include. Examples:
(```)
highways = [
    "motorway", "trunk", "primary", "secondary", "tertiary",
    "unclassified", "residential", "service", "living_space",
    "track", "path", "footway", "cycleway"
]
(```)

You can switch between drive, walk, bike, or all graphs depending on the analysis:
(```)
G = ox.graph_from_place(PLACE, network_type="drive")  # or "walk", "bike", "all"
# or use graph_from_polygon(...) / graph_from_bbox(...)
(```)

### 3.3. Download Other Layers (Transit, Rail, Cycleways)

The notebook includes examples using OSM custom filters to fetch other geometries:
Public transport stops/stations (e.g., public_transport=stop_position, railway=station)
Rail infrastructure (railway=*)
Cycle infrastructure (highway=cycleway, cycleway=*)
(```)
custom_filter_pt = '["public_transport"~"stop_position|platform"]'
gdf_pt = ox.geometries_from_place(PLACE, custom_filter_pt)
custom_filter_rail = '["railway"]'
gdf_rail = ox.geometries_from_place(PLACE, custom_filter_rail)
(```)

### 3.4. Exports

The notebook writes clean layers to GeoPackage and GeoParquet:

# GeoPackage
(```)
gdf_edges.to_file("data/exports/network.gpkg", layer="edges", driver="GPKG")
gdf_nodes.to_file("data/exports/network.gpkg", layer="nodes", driver="GPKG")
(```)
# GeoParquet (fast/columnar)
(```)
gdf_edges.to_parquet("data/exports/edges.parquet")
gdf_nodes.to_parquet("data/exports/nodes.parquet")
(```)

CRS defaults to OSMnx’s CRS; you can reproject (e.g., SIRGAS 2000 / UTM 22S) for metric analyses:
(```)
target_crs = "EPSG:31982"  # SIRGAS 2000 / UTM zone 22S
gdf_edges = gdf_edges.to_crs(target_crs)
gdf_nodes = gdf_nodes.to_crs(target_crs)
(```)
### 3.5. Optional: Centrality

For quick network measures on the primal graph:
(```)
import networkx as nx
(```)
# degree centrality (normalized)
(```)
deg = nx.degree_centrality(G)
nx.set_node_attributes(G, deg, "deg_centrality")
(```)
# betweenness (can be slow; consider sampling or k-shortest paths)
(```)btw = nx.betweenness_centrality(G, weight="length", normalized=True)
nx.set_node_attributes(G, btw, "btw_centrality")
(```)
# Convert to GeoDataFrames with attributes
(```)nodes, edges = ox.graph_to_gdfs(G)
(```)

For urban morphology workflows, consider momepy for extra metrics (reach, closeness on primal, etc.).

## 4. Outputs

Depending on your choices, you’ll get:
Street network (nodes/edges) with geometry and attributes (highway, name, length, etc.)
Transit/rail/cycle layers as GeoDataFrames
Centrality fields added to nodes/edges (e.g., deg_centrality, btw_centrality)
Exports in data/exports/:
network.gpkg (layers: nodes, edges)
edges.parquet, nodes.parquet
optional transit.gpkg, rail.gpkg, etc.

## 5. Tips & Troubleshooting

Performance: Betweenness centrality on large graphs is expensive. Clip to AOI, simplify, or test on samples first.
CRS: Reproject to a projected CRS before computing metric distances or area-based analyses.
Parquet errors (pyarrow/ArrowInvalid): Ensure pyarrow is installed and versions align with GeoPandas.
Windows + pip: If rtree or fiona fail, prefer conda-forge installations.
Numpy/Shapely type errors: Keep numpy, shapely, and geopandas versions compatible (conda-forge tends to resolve this).
Caching: OSMnx can cache geocoding & downloads; see the notebook cell to enable caching for reproducibility.

## 6. Reproducibility

The notebook sets a random seed where relevant and logs package versions.
To fully reproduce runs, export your environment:

# conda
(```)
mamba env export -n osmnx-env > environment.yml
(```
# pip
(```)pip freeze > requirements.txt
(```

Include environment.yml or requirements.txt in the repo.
