# CDSE STAC API — Exploration and Streaming Examples

A collection of Databricks notebooks demonstrating how to search for Earth Observation data via the [Copernicus Data Space Ecosystem (CDSE)](https://dataspace.copernicus.eu/) [STAC API](https://stac.dataspace.copernicus.eu/v1) and stream assets directly from S3 for further processing.

> _The STAC specification is a common language to describe geospatial information, so it can more easily be worked with, indexed, and discovered._ — stacspec.org

## Notebooks

- **`stac_query_sentinel-2`** — Walks through connecting to the CDSE STAC catalogue, exploring available collections, searching for Sentinel-2 L2A scenes by area of interest and cloud cover, inspecting search results and asset metadata, and streaming band data directly from S3 into a lazy `xarray.DataArray` ready for processing.

## Requirements

- CDSE account with active S3 credentials ([generate here](https://documentation.dataspace.copernicus.eu/APIs/S3.html))
- Databricks workspace with a secret scope containing `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

## Getting Started

1. [Clone into Databricks](https://learn.microsoft.com/en-us/azure/databricks/repos/git-operations-with-repos) as a Git folder.
2. Configure your Databricks secret scope with your CDSE S3 credentials.
3. Run the notebook top to bottom, drawing an AOI on the map widget when prompted.
