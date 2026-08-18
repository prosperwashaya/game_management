# Czech hunting-yield hexagonal density dataset

Reproducible workflow for aggregating confidential Czech hunting-ground harvest and
spring-census records (2003–2022) into an open, hexagonal-grid **density** dataset for
six ungulate species (red deer, roe deer, wild boar, fallow deer, sika deer, mouflon).

Ground-level records are confidential; this workflow aggregates them to a hexagonal grid
so that quantitative harvest/census densities can be published without disclosing
individual hunting grounds. Code accompanies the data descriptor (see **Citation**).

## Overview

For each hunting ground, density = animals ÷ ground area (individuals km⁻²). Each hexagon
is assigned the unweighted mean of the densities of all overlapping grounds, computed per
species, sex/age class, metric (harvest, spring census), and year. Reported zeros are kept
distinct from missing data throughout.

## Repository contents

| File | Purpose |
|------|---------|
| `prepare_hunting_join.ipynb` | Clean the raw statistics, keep the wanted metrics, rename columns to English, convert `NULL`→blank, split to per-year CSVs. |
| `aggregate_to_hexgrid_mean.ipynb` | **Core pipeline.** Read the geodatabase, drop non-hunting units, compute density, aggregate to the hexagonal grid (mean-density method), and validate. Produces the GeoPackage. |
| `resolution_sensitivity.ipynb` | Test candidate hexagon sizes (grounds-per-cell, single-ground share, variogram) to justify the 8.5 km resolution. |
| `development_maps.ipynb` | Render the publication maps (per species × harvest/census, fixed per-species scale, 2003–2022) as PDF/PNG. |
| `make_deposit_files.ipynb` | Build the open-repository files: long-format `hex_id`-linked CSV + geometry-only layer (GeoPackage + GeoJSON). |
| `requirements.txt` | Python dependencies. |
| `LICENSE` | Code licence. |
| `CITATION.cff` | How to cite this repository. |

> **Note on filenames:** the final aggregation notebook is `aggregate_to_hexgrid_mean.ipynb`
> (mean-density method, `METHOD = "mean_density"`). An older count-based version is not included.

## Pipeline

```
raw statistics ──prepare_hunting_join──► per-year tables
        │                                     │
        │        (join to hunting-ground polygons in a File Geodatabase, in ArcGIS/QGIS)
        ▼                                     ▼
                        aggregate_to_hexgrid_mean ──► hexgrid_harvest.gpkg
                                          │
             ┌────────────────────────────┼────────────────────────────┐
             ▼                            ▼                             ▼
   resolution_sensitivity        development_maps               make_deposit_files
   (resolution justification)    (figures)                      (deposit CSV + geometry)
```

## Requirements

Python 3.10+ and the packages in `requirements.txt`:

```bash
pip install -r requirements.txt
```

or with conda (recommended for geospatial libraries):

```bash
conda install -c conda-forge geopandas mapclassify matplotlib jupyter pyogrio
```

## Usage

Open each notebook in Jupyter and edit the **config cell** at the top (input paths, output
paths). Run top to bottom. The aggregation and deposit notebooks read a GeoPackage of
per-year hunting-ground layers; see each notebook's header for the expected layer naming.

## Coordinate reference system

All spatial layers use **S-JTSK / Krovák East–North (EPSG:5514)**, the Czech national
projected system; geometry is additionally provided in WGS84 (EPSG:4326). GeoPackage CRS
definitions are written in ESRI WKT1 form so Krovák renders correctly in ArcGIS Pro.

## Known data caveats

- **2010–2012:** the source coded "no harvest" as *missing* rather than *zero* in these
  years (the reverse of all other years); the workflow preserves this, so these years
  appear sparser. Consider excluding them from multi-year summaries.
- **Density spikes:** the hexagon mean is unweighted, so small grounds with high densities
  can elevate a cell; maps therefore use quantile classes rather than the absolute maximum.
- **Sika deer:** Dybowski's sika is unreported across all years; all sika values are
  Japanese sika (*Cervus nippon*).

## Data availability

The aggregated dataset is archived at Zenodo: https://doi.org/10.5281/zenodo.XXXXXXX

## Citation

If you use this code or dataset, please cite the data descriptor:

> [Authors]. A national hexagonal-grid dataset of ungulate harvest and census densities in
> Czechia (2003–2022). *Scientific Data* (in review).

## License

Code released under the MIT License (see `LICENSE`). Data are released separately under
CC BY 4.0 at the Zenodo record above.

## Contact

[Corresponding author name and email]
