---
name: gpt-batch-graph-configuration
description: Use when raw Sentinel-1 GRDH IW products (uncalibrated DN in slant-range
  coordinates) must be converted to comparable gamma0 backscatter rasters, and a reproducible
  SNAP batch configuration file is provided.
when_to_use_negative:
- Input is already a geocoded, calibrated gamma0 product — the graph is designed for
  uncalibrated GRDH DN in slant-range coordinates
- You need SLC-level processing or derive products beyond backscatter coefficients;
  the README notes GRDH was used because 'only backscatter coefficients were utilized
  in this manuscript' and SLC derivation 'was not necessary'
- A spatial speckle filter is explicitly required in the output product rather than
  temporal smoothing via downstream median compositing
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: SNAP toolbox
  role: Open-source ESA software whose provided batch processing configuration file
    replicates the Sentinel-1 SAR pre-processing used in the manuscript
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of the SNAP toolbox used to execute the Sentinel1_GRDH_to_gamma0.xml
    processing chain
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:473a0382d53d2a27cf00cc339ac1234bc055e4a0eded189c8a09cc375ac6e9d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Graph Processing Tools (GPT)
  - SNAP toolbox
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gpt-batch-graph-configuration

## Summary

Reproduce Sentinel-1 GRDH-to-gamma0 pre-processing by executing the provided SNAP batch graph (Sentinel1_GRDH_to_gamma0.xml) via the Graph Processing Tools (GPT) command-line interface, converting uncalibrated digital numbers into orthorectified, terrain-flattened gamma0 backscatter.

## When to use

Use when raw Sentinel-1 GRDH IW products (uncalibrated DN in slant-range coordinates) must be converted to comparable gamma0 backscatter rasters, and a reproducible SNAP batch configuration file is provided.

## When NOT to use

- Input is already a geocoded, calibrated gamma0 product — the graph is designed for uncalibrated GRDH DN in slant-range coordinates
- You need SLC-level processing or derive products beyond backscatter coefficients; the README notes GRDH was used because 'only backscatter coefficients were utilized in this manuscript' and SLC derivation 'was not necessary'
- A spatial speckle filter is explicitly required in the output product rather than temporal smoothing via downstream median compositing

## Inputs

- Sentinel-1 GRDH IW product (uncalibrated digital numbers in slant-range coordinates) from the ESA Copernicus data portal
- Batch processing configuration file Sentinel1_GRDH_to_gamma0.xml with input/output placeholders
- SRTM 1 sec DEM (accessed by SNAP operators)
- Precise orbit state vectors (downloaded automatically ~2 weeks after data take)

## Outputs

- gamma0 backscatter raster in geographic coordinates with 10 m pixel spacing
- Calibrated backscatter intensities comparable across acquisitions and sensors

## How to apply

Run the provided batch file Sentinel1_GRDH_to_gamma0.xml through SNAP's GPT command-line interface, replacing the input/output placeholders with the target scene and output paths. The graph chains six operators in order: (1) Apply precise orbit (metadata-only, available ~2 weeks post-acquisition), (2) Remove border noise within a search distance of 700 pixels from each margin, (3) Remove thermal noise, (4) Radiometric calibration to beta0 radar brightness, (5) Radiometric terrain flattening using the SRTM 1 sec DEM to yield gamma0 in slant-range coordinates, and (6) Range-Doppler terrain correction with the same DEM and precise orbits for orthorectification. Skip spatial speckle filtering because temporal median compositing over multiple acquisitions suppresses speckle implicitly without further loss of geometric detail.

## Related tools

- **SNAP toolbox** (Open-source ESA software whose provided batch processing configuration file replicates the Sentinel-1 SAR pre-processing used in the manuscript) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of the SNAP toolbox used to execute the Sentinel1_GRDH_to_gamma0.xml processing chain)

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -Pinput=S1A_GRDH_scene.zip -Poutput=scene_gamma0.tif
```

## Evaluation signals

- Output raster is gamma0 in geographic coordinates with 10 m pixel spacing
- Backscatter intensities are calibrated and comparable across acquisitions and sensors (radiometric calibration step completed)
- Border noise removed within 700-pixel search distance from each margin and thermal noise removed while data remain DN before calibration
- Terrain flattening used the SRTM 1 sec DEM and terrain correction used the DEM combined with the precise orbits applied in step 1
- No spatial speckle filter applied; speckle suppression expected via temporal median over multiple acquisitions downstream

## Limitations

- Precise orbit data only become available approximately two weeks after data take, so recent acquisitions cannot use the precise orbit step immediately
- Border-noise removal is a heuristic that only 'attempts' to remove artefacts within 700 pixels of each swath margin
- The workflow skips spatial speckle filtering; if the downstream analysis lacks temporal median compositing over multiple acquisitions, speckle remains unsmoothed
- SNAP is 64-bit only (Windows, Mac OS X, Unix)

## Evidence

- [readme] All processing steps can be reproduced by means of the Graph Processing Tools (GPT): "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox"
- [methods] GPT applied to batch file Sentinel1_GRDH_to_gamma0.xml: "by applying the provided processing chain batch file (Sentinel1_GRDH_to_gamma0.xml). The batch file contains placeholders for input and output files"
- [readme] Border noise removal within 700 pixels: "This module attempts to remove these artefacts within a search distance of 700 pixels from each margin."
- [methods] Radiometric calibration outputs beta0 for terrain flattening: "Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening."
- [readme] Final output gamma0 in geographic coordinates at 10 m: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
