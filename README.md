# Venezuela Earthquake Building Damage Map

This project processes and visualizes building damage assessment data for areas affected by the June 2026 earthquake in Venezuela.

The workflow focuses on building-level damage predictions produced by the Microsoft AI for Good Lab for:

* Catia La Mar
* Catia La Mar East
* Caraballeda
* La Guaira

The final output is a cleaned and classified GeoJSON dataset prepared for visualization in ArcGIS Online or other GIS platforms.

---

## Project Objective

The main objective is to create a reproducible geospatial workflow that identifies buildings with possible earthquake damage and organizes them into practical damage categories.

The project is intended for:

* exploratory humanitarian mapping;
* preliminary identification of affected areas;
* prioritization of visual inspection;
* communication of spatial damage patterns;
* portfolio and educational purposes.

The results must not be interpreted as confirmed structural damage without field verification.

---

## Data Sources

### Microsoft AI for Good Lab

Building damage assessment datasets were downloaded from the Humanitarian Data Exchange.

The datasets include building footprints and automated damage indicators derived from post-event satellite imagery.

Main attributes used:

| Field            | Description                                                           |
| ---------------- | --------------------------------------------------------------------- |
| `id`             | Local feature identifier within each source dataset                   |
| `damage_pct_0m`  | Proportion of damage-classified pixels inside the building footprint  |
| `damage_pct_10m` | Damage proportion calculated using a 10 m surrounding area            |
| `damage_pct_20m` | Damage proportion calculated using a 20 m surrounding area            |
| `damaged`        | Binary indicator activated when at least one damage pixel is detected |
| `unknown_pct`    | Proportion of pixels classified as unknown                            |

The original note accompanying the dataset explains that `damaged = 1` may be assigned when only one pixel inside a building is classified as damaged. For that reason, this project does not use the binary `damaged` field as the main classification criterion.

---

## Damage Classification

The project classifies buildings using `damage_pct_0m`.

| Category           | Rule                           |
| ------------------ | ------------------------------ |
| High probability   | `damage_pct_0m >= 0.80`        |
| Medium probability | `0.40 <= damage_pct_0m < 0.80` |
| Possible damage    | `0 < damage_pct_0m < 0.40`     |
| No damage detected | `damage_pct_0m == 0`           |

These categories represent the proportion of the building footprint covered by pixels classified as damage.

They should be interpreted as automated indicators, not verified engineering assessments.

---

## Overlap Between Catia La Mar Datasets

The Catia La Mar and Catia La Mar East datasets have a substantial overlapping area.

The `id` field could not be used to remove duplicates because it is a local sequential identifier that restarts in each dataset.

A geometric comparison showed that many footprints intersected, but their geometries were not always identical due to differences in footprint generation and image alignment.

To avoid duplicating buildings, the following approach was used:

1. Load the valid-area mask for Catia La Mar East.
2. Reproject the mask to the same coordinate reference system as the building datasets.
3. Identify Catia La Mar buildings whose centroids fall inside the Catia La Mar East coverage area.
4. Remove those buildings from the original Catia La Mar dataset.
5. Retain the Catia La Mar East buildings inside the newer coverage area.
6. Combine the remaining Catia La Mar buildings with the complete Catia La Mar East dataset.

This created a cleaned Catia La Mar dataset containing:

* 17,723 buildings from Catia La Mar outside the East coverage;
* 24,732 buildings from Catia La Mar East;
* 42,455 buildings in the combined Catia La Mar area.

---

## Final Dataset

After integrating Catia La Mar, Catia La Mar East, Caraballeda, and La Guaira, the final dataset contains:

| Damage category    |  Buildings |
| ------------------ | ---------: |
| No damage detected |     51,259 |
| Possible damage    |      3,208 |
| Medium probability |      1,363 |
| High probability   |      2,428 |
| **Total**          | **58,258** |

---

## Output Files

### Full classified dataset

```text
03_ARCGIS_UPLOAD/VEN_EQ2026_building_damage_all_categories.geojson
```

Contains all evaluated buildings and all four damage categories.

### Damage-detected dataset

```text
03_ARCGIS_UPLOAD/VEN_EQ2026_building_damage_detected.geojson
```

Contains only buildings classified as:

* possible damage;
* medium probability;
* high probability.

This lighter file is recommended for web mapping and ArcGIS Online.

---

## Project Structure

```text
Venezuela_Earthquake_Humanitarian_Map/
│
├── 01_RAW_DATA/
│   └── Microsoft_AI_Damage/
│       ├── Caraballeda/
│       ├── Catia_La_Mar/
│       ├── Catia_La_Mar_East/
│       └── La_Guaira/
│
├── 03_ARCGIS_UPLOAD/
│   ├── VEN_EQ2026_building_damage_all_categories.geojson
│   └── VEN_EQ2026_building_damage_detected.geojson
│
├── building_damage_processing.ipynb
├── README.md
└── .gitignore
```

The raw datasets are not included in the repository because of file size and data duplication. They should be downloaded directly from the original Humanitarian Data Exchange sources.

---

## Software and Libraries

The workflow was developed in Jupyter Notebook inside Visual Studio Code.

Main Python libraries:

```text
geopandas
pandas
numpy
matplotlib
shapely
pyogrio
```

Install the required packages with:

```bash
python -m pip install geopandas pandas numpy matplotlib shapely pyogrio jupyter
```

---

## Coordinate Reference Systems

The source building datasets use:

```text
EPSG:32619
```

This projected coordinate system was used for:

* geometric intersections;
* area calculations;
* centroid-based coverage analysis;
* overlap assessment.

The final GeoJSON files were exported in:

```text
EPSG:4326
```

for compatibility with ArcGIS Online and other web mapping applications.

---

## Running the Workflow

1. Download the source GeoPackage and valid-area mask files.
2. Place them inside the corresponding folders under:

```text
01_RAW_DATA/Microsoft_AI_Damage/
```

3. Open the Jupyter notebook in Visual Studio Code.
4. Run the notebook cells in order.
5. Review the data summaries and maps.
6. Export the final GeoJSON files to:

```text
03_ARCGIS_UPLOAD/
```

---

## Important Limitations

This project uses automated damage predictions derived from satellite imagery.

Possible limitations include:

* false positives caused by shadows, debris, vegetation, or image misalignment;
* differences between satellite acquisition dates;
* differences between models used for each geographic area;
* footprint displacement caused by image orthorectification;
* buildings outside the valid assessment coverage;
* buildings incorrectly classified due to partial pixel overlap;
* lack of field validation.

A building classified as high probability should not automatically be described as destroyed, unsafe, or structurally compromised.

Recommended terminology:

> Building with high probability of damage detected by automated satellite-image analysis.

---

## Humanitarian Use Disclaimer

This dataset may support exploratory mapping, situational awareness, and preliminary prioritization.

It should not be used as the sole basis for:

* evacuation decisions;
* structural safety declarations;
* rescue operations;
* compensation decisions;
* official damage statistics;
* identification of safe shelters.

All results require visual review, local knowledge, and field verification.

---

## Suggested ArcGIS Online Symbology

| Damage category    | Suggested color |
| ------------------ | --------------- |
| High probability   | Dark red        |
| Medium probability | Orange          |
| Possible damage    | Yellow          |
| No damage detected | Light gray      |

Recommended layer title:

```text
Venezuela Earthquake 2026 — AI-Detected Building Damage
```

Recommended map note:

> Damage classifications were generated through automated satellite-image analysis and require field verification.

---

## Author

**Ada Peña**
Architect and Computational Designer
Draai Design Studio

This project was developed as a geospatial data-processing and humanitarian mapping exercise using Python, GeoPandas, Jupyter Notebook, and ArcGIS Online.
