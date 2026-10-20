# Campus-Wide Extreme-Heat Risk Screening

This project provides a planning-level screening tool for identifying campus locations that may warrant extreme-heat mitigation or field verification.

The workflow combines warm-season Landsat surface-temperature observations with a campus-wide analysis grid and optional campus attributes such as occupancy, sensitivity, shade, and cooling access.

> This is a screening tool, not a direct measurement of indoor temperature, heat index, building energy use, or health risk. Confirm priority locations with field observations, air-temperature measurements, shade surveys, HVAC assessments, and campus occupancy data.

## What the notebook does

The Colab notebook:

1. Uploads a GeoJSON containing campus building footprints.
2. Reprojects the data to NAD83 / UTM Zone 12N, suitable for Prescott, Arizona.
3. Creates a 100-meter grid around the campus and a 150-meter buffer.
4. Searches Microsoft Planetary Computer for warm-season Landsat Collection 2 Level-2 scenes.
5. Removes cloudy, snowy, shadowed, and invalid pixels using Landsat QA data.
6. Calculates surface-temperature statistics for each grid cell.
7. Combines heat hazard with exposure, sensitivity, and protection indicators.
8. Assigns risk classes and confidence flags.
9. Summarizes the grid results back to individual buildings.
10. Exports GIS, CSV, and map outputs.

## Files

- `campus_heat_screening_revised.ipynb` — executable Google Colab notebook.
- `campus_heat_screening_revised.py` — Python source version of the notebook.

## Running the notebook

1. Open `campus_heat_screening_revised.ipynb` in Google Colab.
2. Run the cells from top to bottom.
3. When prompted, upload a campus building GeoJSON.
4. Review the assumptions printed during setup.
5. Inspect the exported risk map and ranked CSV files.

The notebook installs its required Python packages automatically. It requires an internet connection to query satellite imagery.

## Input GeoJSON

The required input is a GeoJSON containing building polygons. The notebook looks for a building identifier in this order:

1. `building_id`
2. `id`
3. `name`
4. An automatically generated identifier

The input should use a valid coordinate reference system. If no CRS is provided, the notebook assumes EPSG:4326.

### Optional attributes

The following fields improve the risk score. Each should be numeric on a 0–100 scale:

| Field | Meaning | Higher value means |
|---|---|---|
| `occupancy_score` | Expected presence or activity | More people or activity |
| `sensitivity_score` | Relative vulnerability of users or facility | Greater sensitivity |
| `protection_score` | Overall protection from heat | More protection |
| `shade_score` | Shade or tree-canopy availability | More shade |
| `cooling_access_score` | Access to cooled indoor space or other cooling resources | Better access |

If a field is absent, the notebook uses a neutral value of 50 and reports the assumption. For decision-quality results, replace these assumptions with campus-specific values.

## Heat-data workflow

The default analysis period is May through September from 2019 through 2026. The notebook searches for Landsat Collection 2 Level-2 scenes over the campus extent and keeps scenes below the configured cloud-cover threshold.

For each usable scene and grid cell, it calculates:

- Mean surface temperature
- Median surface temperature
- 90th-percentile surface temperature
- Maximum observed surface temperature
- Number of valid thermal pixels

The 90th-percentile temperature is used as the primary hazard indicator because it represents repeatedly hot conditions without relying only on one extreme pixel.

## Risk score

All component scores are normalized to a 0–100 scale. The default composite score is:

```text
overall_heat_risk_score =
    0.50 × heat_hazard_score
  + 0.20 × exposure_score
  + 0.15 × sensitivity_score
  + 0.15 × (100 − protection_score)
```

The default weights emphasize observed surface heat while still accounting for who may be exposed and how much protection is available.

### Risk classes

| Score | Class | General interpretation |
|---:|---|---|
| 0–25 | Low | Lower relative priority for heat intervention |
| 25–50 | Moderate | Monitor and address during heat advisories |
| 50–75 | High | Investigate shade, occupancy, sensitivity, and cooling access |
| 75–100 | Very high | Prioritize field verification and mitigation planning |

These classes are relative to the analyzed campus and should not be interpreted as public-health thresholds.

## Confidence flags

Each cell receives a confidence flag based on scene coverage and valid thermal pixels:

- `low` — limited scene or pixel coverage
- `medium` — usable but incomplete coverage
- `high` — stronger scene and pixel support

Low-confidence areas should be field-checked before funding or design decisions are made.

## Outputs

The notebook creates the following files:

| Output | Description |
|---|---|
| `campus_heat_risk_grid.geojson` | GIS-ready grid with temperature, scoring, class, and confidence fields |
| `campus_heat_risk_grid.csv` | Tabular grid results for ranking and analysis |
| `campus_building_heat_risk.geojson` | Building footprints joined to summarized risk results |
| `campus_building_heat_risk.csv` | Building-level ranked risk table |
| `campus_heat_risk_map.png` | Visual campus risk map |

## Interpreting results

The highest-priority locations are areas that are:

- Consistently hot in warm-season satellite observations;
- Heavily occupied or used for outdoor activity;
- Used by more heat-sensitive populations;
- Poorly shaded or poorly connected to cooling resources; and
- Supported by medium- or high-confidence data.

Possible interventions include shade structures, tree planting, cool pavement, solar canopies, drinking-water stations, HVAC improvements, cooling-room designation, and changes to outdoor scheduling.

## Important limitations

- Landsat surface temperature is not the same as air temperature or heat index.
- Landsat thermal pixels are relatively coarse, so small buildings may not be individually resolved.
- The current workflow uses optional campus attributes rather than automatically deriving occupancy or health vulnerability.
- Missing optional attributes are assigned neutral assumptions, not measured values.
- The score is relative to the analyzed campus and does not establish a regulatory or medical threshold.
- Satellite observations may not capture nighttime heat, short-duration heat events, or shaded pedestrian conditions.
- Results should be validated with campus operations staff and field measurements.

