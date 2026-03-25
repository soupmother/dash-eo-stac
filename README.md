# ESA CDSE STAC API Demo

## Translation:

ESA: [European Space Agency](https://www.esa.int/)  
CDSE: [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/)  
STAC: [SpatioTemporal Asset Catalogs](https://stacspec.org/en)  
API: [Application Programming Interface](https://en.wikipedia.org/wiki/API)

This repository contains example material demonstrating how to access Earth observation data using the European Space Agency's Copernicus Data Space Ecosystem (CDSE) SpatioTemporal Asset Catalog (STAC) API (https://stac.dataspace.copernicus.eu/v1).

>_The STAC specification is a common language to describe geospatial information, so it can more easily be worked with, indexed, and discovered._ - stacspec.org/en

## Contents

- **notebooks/**
  - `stac_query_sentinel-2` — Databricks notebook that guides users through authenticating with CDSE, defining an area of interest, performing STAC searches, and constructing a datacube of assets for further processing or analysis.
- **README.md** — Overview and usage instructions.

## Requirements

- CDSE account with active S3 credentials  
- Access to Databricks Secrets for securely storing access keys

## Getting Started

1. [Clone the repository](https://learn.microsoft.com/en-us/azure/databricks/repos/git-operations-with-repos) into Databricks as a "Git folder".  
2. Configure your secret scope containing:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
3. Run the notebook and follow in-notebook instructions to draw an AOI and retrieve data.

## Further Development

Future examples may include:
- Loading data into xarray with `stackstac`  
- Generating temporal composites or animations or training data for machine learning models
- notebooks to demonstrate working with other types of data like synthetic aperture radar

