---
name: range-doppler-orthorectification
description: 'Reference for geolocating Sentinel-1 GRDH (IW) pixels: input is still
  in slant-range image coordinates after terrain flattening and must be orthorectified
  before analysis or compositing.'
when_to_use_negative:
- Input is already in geographic coordinates (e.g., a previously orthorectified or
  GRD-derived gamma_0 GeoTIFF) — re-running correction adds no value
- You are working with SLC data you need to fully derive into GRDH yourself; this
  skill covers the GRDH-to-gamma_0 chain, not SLC preprocessing
- Analysis does not require geolocation of individual pixels (e.g., quick-look calibration
  checks in slant-range geometry)
tools:
- name: SNAP toolbox
  role: Open-source ESA software providing the Range-doppler terrain correction and
    the other Sentinel-1 GRDH-to-gamma_0 processing nodes
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of SNAP used to execute the provided batch file Sentinel1_GRDH_to_gamma0.xml
    containing the full processing chain
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:7f75013434a2b840ec2f11ada9fedeeca4e93558a50993fcbffd189f2c7bb998
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Graph Processing Tools (GPT)
  - SNAP toolbox
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# range-doppler-orthorectification

## Summary

Geometric orthorectification (Range-doppler terrain correction) of Sentinel-1 GRDH SAR data, converting gamma_0 from slant-range/image coordinates to gamma_0 in geographic coordinates with 10 m pixel spacing, as a terminal step of the SNAP/GPT Sentinel1_GRDH_to_gamma0.xml chain.

## When to use

Reference for geolocating Sentinel-1 GRDH (IW) pixels: input is still in slant-range image coordinates after terrain flattening and must be orthorectified before analysis or compositing.

## When NOT to use

- Input is already in geographic coordinates (e.g., a previously orthorectified or GRD-derived gamma_0 GeoTIFF) — re-running correction adds no value
- You are working with SLC data you need to fully derive into GRDH yourself; this skill covers the GRDH-to-gamma_0 chain, not SLC preprocessing
- Analysis does not require geolocation of individual pixels (e.g., quick-look calibration checks in slant-range geometry)

## Inputs

- Sentinel-1 GRDH IW product (uncalibrated digital numbers in slant-range coordinates) from the ESA Copernicus data portal
- SRTM 1 sec DEM
- Precise orbit state vectors (available ~2 weeks after acquisition; metadata-only)
- Batch graph file Sentinel1_GRDH_to_gamma0.xml with input/output placeholders

## Outputs

- gamma_0 backscatter raster in geographic coordinates with 10 m pixel spacing
- Orthorectified SAR output comparable across acquisitions and sensors

## How to apply

Run Range-doppler terrain correction as the last node of the SNAP batch graph (Sentinel1_GRDH_to_gamma0.xml) after Apply precise orbit, Remove border noise (700 px search distance from each margin), Remove thermal noise, Radiometric calibration to beta_0, and Radiometric terrain flattening to gamma_0. The module relies on a DEM — here the SRTM 1 sec DEM — in combination with the precise orbit state vectors applied in step 1, to exactly geolocate each pixel and correct geometric distortions due to local terrain. Execute the whole chain via the Graph Processing Tools (GPT) command-line interface of the open-source ESA SNAP toolbox, replacing the input/output placeholders in the xml with the target scene and output paths using the corresponding command-line flags. Note that spatial speckle filtering is deliberately skipped because the downstream temporal median compositing over multiple acquisitions suppresses speckle implicitly.

## Related tools

- **SNAP toolbox** (Open-source ESA software providing the Range-doppler terrain correction and the other Sentinel-1 GRDH-to-gamma_0 processing nodes) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of SNAP used to execute the provided batch file Sentinel1_GRDH_to_gamma0.xml containing the full processing chain)

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -Pinput=S1A_IW_GRDH_scene.zip -Poutput=gamma0_geographic.dim
```

## Evaluation signals

- Output raster is gamma_0 in geographic coordinates (no longer slant-range image coordinates)
- Output pixel spacing is 10 m
- Backscatter intensities are calibrated and comparable across acquisitions and sensors
- Every upstream node of the xml batch file ran in order (precise orbit, border noise within 700 px of margins, thermal noise, calibration to beta_0, terrain flattening to gamma_0)
- Scene geolocation matches the SRTM 1 sec DEM and the precise orbit state vectors applied in step 1

## Limitations

- Precise orbit data are only available approximately two weeks after data take, so the chain cannot be fully run immediately after acquisition
- The modules depend on DEM availability (SRTM 1 sec DEM in this study); different regions may require an alternative DEM
- Speckle is not filtered spatially — smoothing relies on downstream temporal median compositing over multiple acquisitions
- At country/EU scale beyond this study's scope, Sentinel-1 limitations include weak permeability of C-band through overstorey layers (relevant to the ecological application, not the correction itself)

## Evidence

- [readme] The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] GPT command-line reproducibility: "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox"
- [readme] Module dependencies on DEM and precise orbits: "As with radiometric terrain flattening, this module relies also on a DEM, in combination with the precise orbit determination applied in step 1."
- [full_text] Terrain correction as step 6 of the workflow card: [evidence span withheld — anchor status: not_found]
