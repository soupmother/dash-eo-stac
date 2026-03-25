# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

A growing collection of Databricks notebooks and Python utilities for searching and streaming Earth Observation (EO) data from the ESA Copernicus Data Space Ecosystem (CDSE) STAC API. The focus is on cloud-native streaming (no full downloads) using COG windowed reads. Spark-based distributed compute is a near-future target.

## Setup

```bash
pip install -r requirements.txt
# or, inside the notebook:
%pip install --upgrade stackstac rioxarray && pip install folium pystac_client geogif s3fs stacmap shapely rasterio
```

For local development, copy `.env.example` to `.env` and populate with CDSE S3 credentials from [dataspace.copernicus.eu](https://dataspace.copernicus.eu/).

## Running Notebooks

Notebooks are designed for **Databricks**. Credentials are read from Databricks Secrets (scope `tomkdefra_cdse`, keys `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`).

Run cells sequentially — the AOI widget cell requires manual interaction (draw polygon → Export → copy link → paste in the next cell).

## Key Architecture

### STAC API endpoints

| Endpoint | Notes |
|---|---|
| `https://stac.dataspace.copernicus.eu/v1` | **Use this.** Returns per-band GeoTIFF assets with direct `s3://eodata/...` hrefs. Collection: `sentinel-2-l2a`. |
| `https://catalogue.dataspace.copernicus.eu/stac/` | Older API. Returns whole SAFE products, not per-band. Avoid for streaming. |

### S3 access

CDSE S3 is a non-AWS endpoint requiring path-style access:

```python
# For s3fs (listing, existence checks)
fs = s3fs.S3FileSystem(
    anon=False,
    key=os.environ["AWS_ACCESS_KEY_ID"],
    secret=os.environ["AWS_SECRET_ACCESS_KEY"],
    client_kwargs={
        "endpoint_url": "https://eodata.dataspace.copernicus.eu",
        "verify": False,
    },
)

# For GDAL/rioxarray (COG windowed reads) — convert href and use /vsis3/
vsi_path = href.replace("s3://", "/vsis3/")
band = rioxarray.open_rasterio(vsi_path, chunks={"x": 1024, "y": 1024}, lock=False)
```

The GDAL env vars (set on the driver) are sufficient for single-node. For distributed workers (Spark/Dask), credentials must be passed explicitly via `stackstac.DEFAULT_GDAL_ENV.updated(always={...})`.

### Streaming pattern (single-node)

1. STAC search → list of `pystac.Item` objects with per-band `s3://` hrefs
2. Convert href `s3://` → `/vsis3/` for GDAL
3. `rioxarray.open_rasterio(vsi_path, chunks=..., lock=False)` → lazy dask array
4. `rio.clip_box(...)` after reprojecting AOI bounds from EPSG:4326 to the COG's native CRS
5. `.compute()` to stream only the clipped window from S3

### stackstac (multi-item datacube)

`stackstac.stack()` builds a lazy `(time, band, y, x)` dask array across all search results. Always pass `gdal_env`:

```python
gdal_env = stackstac.DEFAULT_GDAL_ENV.updated(always={
    "AWS_S3_ENDPOINT": "eodata.dataspace.copernicus.eu",
    "AWS_ACCESS_KEY_ID": os.environ["AWS_ACCESS_KEY_ID"],
    "AWS_SECRET_ACCESS_KEY": os.environ["AWS_SECRET_ACCESS_KEY"],
    "AWS_HTTPS": "YES",
    "AWS_VIRTUAL_HOSTING": "FALSE",
    "GDAL_HTTP_UNSAFESSL": "YES",
    "GDAL_HTTP_TCP_KEEPALIVE": "YES",
})
cube = stackstac.stack(items, assets=[...], gdal_env=gdal_env, ...)
```

### Related repos

- `../cdse-stac-api` — working proof that `s3fs` + `rioxarray.open_rasterio(file_handle)` streams COGs from CDSE; also shows OData/HTTPS download via bearer token
- `../cdse-stac-examples` — STAC item/collection JSON samples showing storage and auth scheme extensions
