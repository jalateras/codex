# Idea 7: Geospatial & Remote Sensing Scripting Assistant

## Table of Contents
- [Concept Overview](#concept-overview)
- [Target Domain(s)](#target-domains)
- [Detailed Description](#detailed-description)
  - [Core Functionality](#core-functionality)
  - [Example Interactions](#example-interactions)
  - [Supported Libraries & Tools](#supported-libraries--tools)
  - [Common Operations](#common-operations)
  - [Integration with Public Data APIs](#integration-with-public-data-apis)
- [How Codex CLI/Agent Features Are Leveraged](#how-codex-cliagent-features-are-leveraged)
- [Potential Benefits](#potential-benefits)
- [Considerations & Challenges](#considerations--challenges)

## Concept Overview
Codex assists users in generating scripts (primarily Python) for common geospatial and remote sensing tasks. This includes operations on vector and raster data, interacting with geospatial libraries, and fetching data from public remote sensing or GIS APIs.

## Target Domain(s)
-   Ag-Tech (Precision Agriculture, Crop Monitoring)
-   Environmental Science & Monitoring
-   Urban Planning
-   Geography & GIS
-   Forestry
-   Disaster Management

## Detailed Description

### Core Functionality
1.  **Interpret Geospatial Task**: Codex parses user prompts to identify:
    -   The type of geospatial data involved (raster, vector).
    -   Input data sources (file paths, URLs, API parameters).
    -   Desired geospatial operation (e.g., calculate NDVI, reproject, buffer, zonal statistics).
    -   Relevant parameters for the operation (e.g., band numbers for NDVI, buffer distance, target CRS).
    -   Output requirements (e.g., save to new file, display basic info).
2.  **Script Generation**:
    -   Generates Python scripts using standard geospatial libraries.
    -   Scripts will include:
        -   Importing necessary libraries.
        -   Loading input data.
        -   Performing the specified geospatial operations.
        -   Saving or displaying results.
3.  **API Interaction**:
    -   For tasks involving public data APIs (e.g., Landsat, Sentinel, Open-Meteo), Codex can generate scripts to construct and execute API requests, then download or process the returned data.
4.  **Execution and Output**:
    -   Saves the generated script.
    -   With user approval, executes the script.
    -   Informs the user about output files or displays textual results.

### Example Interactions

**Example 1 (NDVI Calculation):**
*User Prompt*: `"codex, calculate NDVI for 'landsat_scene.tif' using band 4 for Red and band 5 for NIR. Save the output as 'ndvi_output.tif'."`

*Codex Actions*:
1.  Identifies: Task (NDVI), input file, Red band (4), NIR band (5), output file.
2.  Generates `calculate_ndvi.py`:
    ```python
    import rasterio
    import numpy as np

    input_raster = 'landsat_scene.tif'
    output_raster = 'ndvi_output.tif'
    # Assuming bands are 1-indexed in user prompt, adjust to 0-indexed for rasterio
    red_band_idx = 4 - 1
    nir_band_idx = 5 - 1

    with rasterio.open(input_raster) as src:
        red = src.read(red_band_idx + 1).astype(np.float32) # Read specific band
        nir = src.read(nir_band_idx + 1).astype(np.float32) # Read specific band

        # Suppress division by zero warnings for the NDVI calculation
        np.seterr(divide='ignore', invalid='ignore')
        ndvi = (nir - red) / (nir + red)
        ndvi[np.isnan(ndvi)] = 0  # Handle NaN values if any, e.g. by setting to 0

        # Get metadata from source and update for output
        profile = src.profile
        profile.update(dtype=rasterio.float32, count=1, compress='lzw')

        with rasterio.open(output_raster, 'w', **profile) as dst:
            dst.write(ndvi.astype(rasterio.float32), 1)
    print(f"NDVI calculated and saved to {output_raster}")
    ```
3.  Suggests running `python calculate_ndvi.py`.

**Example 2 (Reprojecting a Shapefile):**
*User Prompt*: `"codex, reproject 'farms.shp' from EPSG:4326 to EPSG:32632. Output to 'farms_utm.shp'."`

*Codex Actions*:
1.  Identifies: Task (reproject), input file, source CRS (EPSG:4326), target CRS (EPSG:32632), output file.
2.  Generates `reproject_shapefile.py` using `geopandas`:
    ```python
    import geopandas as gpd

    input_shp = 'farms.shp'
    output_shp = 'farms_utm.shp'
    source_crs = 'EPSG:4326'
    target_crs = 'EPSG:32632'

    gdf = gpd.read_file(input_shp)
    if gdf.crs is None:
        print(f"Warning: Input shapefile {input_shp} has no CRS defined. Assuming {source_crs}.")
        gdf.set_crs(source_crs, inplace=True)
    elif gdf.crs.to_string().upper() != source_crs:
        print(f"Warning: Input shapefile CRS ({gdf.crs}) differs from provided source CRS ({source_crs}). Using input file's CRS for reprojection if different.")
        # Or, strictly use source_crs: gdf.set_crs(source_crs, inplace=True, allow_override=True)


    gdf_reprojected = gdf.to_crs(target_crs)
    gdf_reprojected.to_file(output_shp)
    print(f"Shapefile reprojected from {gdf.crs} to {target_crs} and saved to {output_shp}")
    ```
3.  Suggests running the script (and potentially installing `geopandas` if not available).

### Supported Libraries & Tools
-   **Python Libraries**:
    -   `Rasterio` and `GDAL` (often via Rasterio) for raster data.
    -   `Geopandas`, `Fiona`, `Shapely`, `PyProj` for vector data.
    -   `NumPy` for numerical operations.
    -   `Requests` for API interactions.
-   **Command-Line Tools (Advanced)**: Could potentially wrap GDAL/OGR command-line tools for users more familiar with them, similar to Idea 2 (Smart CLI Wrapper).

### Common Operations
-   **Raster**: NDVI, EVI, other band math; reprojection; resampling; clipping; zonal statistics.
-   **Vector**: Reprojection; buffering; overlays (intersection, union); simplifying geometries; reading/writing various formats (Shapefile, GeoJSON, KML).
-   **Data Conversion**: Between different raster and vector formats.

### Integration with Public Data APIs
-   **Open-Meteo**: Fetching weather forecast or historical weather data.
-   **NASA CMR / USGS EarthExplorer (via `pystac-client` or direct API calls)**: Searching and downloading satellite imagery (Landsat, Sentinel, MODIS).
-   **Planetary Computer APIs**: Accessing large-scale geospatial datasets.
-   Codex could generate scripts to formulate queries, download metadata or preview images, and then download actual data files.

## How Codex CLI/Agent Features Are Leveraged
-   **Natural Language Processing**: To understand geospatial terms, operations, file names, CRS codes, and API parameters.
-   **Code Generation**: Creating Python scripts that use the appropriate geospatial libraries.
-   **File System Interaction**: Reading input geospatial files, writing output files (rasters, vectors, tables).
-   **Command Execution**: Running the generated Python scripts.
-   **`AGENTS.md` Customization**:
    -   Store preferred Coordinate Reference Systems (CRS).
    -   Define common band combinations for specific satellites/sensors.
    -   API keys or endpoint URLs for private geospatial services.
    -   Snippets for complex or custom geospatial algorithms.

## Potential Benefits
-   **Accessibility**: Simplifies complex geospatial operations for users not deeply familiar with specific libraries or GIS software.
-   **Automation**: Automates repetitive geoprocessing tasks.
-   **Reproducibility**: Generated scripts provide a clear record of operations performed.
-   **Data Integration**: Facilitates fetching and preparing geospatial data for use in Ag-Tech, environmental models, or data science workflows.
-   **Rapid Prototyping**: Quickly test geospatial ideas or process sample datasets.

## Considerations & Challenges
-   **Complexity of Geospatial Concepts**: Users might need some understanding of CRS, raster/vector models, etc. Codex might need to offer explanations or make sensible defaults.
-   **Library Installation**: Geospatial Python libraries (especially GDAL) can sometimes be challenging to install due to system dependencies. Codex might need to guide users on this or assume a pre-configured environment (e.g., a GIS-focused Conda environment).
-   **Data Size and Processing Time**: Geospatial datasets can be very large. Scripts generated by Codex would run locally and might be slow for extensive datasets. For very large scale processing, these scripts might serve as prototypes for later deployment on cloud GIS platforms.
-   **API Quotas and Authentication**: Accessing public data APIs often involves managing API keys, usage quotas, and specific query syntaxes.
-   **Error Handling**: Geospatial operations can fail for many reasons (invalid geometries, mismatched CRS, file format issues). Generated scripts need good error handling.
-   **Visualization**: While this idea focuses on processing, a natural extension would be to integrate with visualization capabilities (Idea 4) to plot results (e.g., show the NDVI map).

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
