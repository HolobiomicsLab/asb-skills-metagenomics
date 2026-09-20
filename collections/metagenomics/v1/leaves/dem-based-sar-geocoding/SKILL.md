---
name: dem-based-sar-geocoding
description: Reference for reproducing the Sentinel-1 GRDH-to-gamma0 chain when input
  is uncalibrated DN in slant-range coordinates and output must be calibrated, terrain-normalized
  gamma_0 at 10 m pixel spacing.
when_to_use_negative:
- Data are already gamma_0 in geographic coordinates (e.g. temporal gamma_0 stacks
  like ALB_gamma0_VH.tif used downstream in R) — only pixel/neighborhood statistics
  are needed
- A spatial speckle filter is required before single-scene analysis — this chain deliberately
  omits spatial speckle filtering because median compositing over acquisitions suppresses
  speckle implicitly
- Working from SLC products requiring full SLC-level processing, since the study used
  only backscatter coefficients available in GRDH
tools:
- name: SNAP toolbox
  role: Open-source ESA software hosting the Sentinel-1 pre-processing operators;
    the batch processing configuration file is applied within it
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of SNAP used to execute the Sentinel1_GRDH_to_gamma0.xml
    processing chain with input/output placeholders replaced by command-line flags
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:fc5c7bde48634ee114034a414f51ad271344b539961d75b4738b46fefdfd8833
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# DEM-based SAR geocoding

## Summary

Converts Sentinel-1 GRDH backscatter from slant-range image coordinates to terrain-flattened, orthorectified gamma_0 in geographic coordinates by combining radiometric terrain flattening and range-Doppler terrain correction with the SRTM 1 sec DEM and precise orbits in SNAP's GPT.

## When to use

Reference for reproducing the Sentinel-1 GRDH-to-gamma0 chain when input is uncalibrated DN in slant-range coordinates and output must be calibrated, terrain-normalized gamma_0 at 10 m pixel spacing.

## When NOT to use

- Data are already gamma_0 in geographic coordinates (e.g. temporal gamma_0 stacks like ALB_gamma0_VH.tif used downstream in R) — only pixel/neighborhood statistics are needed
- A spatial speckle filter is required before single-scene analysis — this chain deliberately omits spatial speckle filtering because median compositing over acquisitions suppresses speckle implicitly
- Working from SLC products requiring full SLC-level processing, since the study used only backscatter coefficients available in GRDH

## Inputs

- Sentinel-1 GRDH IW product (uncalibrated digital numbers in slant-range coordinates) from the ESA Copernicus data portal
- Precise orbit state vectors (downloaded automatically by the Apply precise orbit step)
- SRTM 1 sec DEM
- Sentinel1_GRDH_to_gamma0.xml batch processing configuration file for SNAP/GPT

## Outputs

- gamma_0 backscatter raster in geographic coordinates with 10 m pixel spacing
- Per-scene gamma_0 layers suitable for building temporal VH/VV stacks (e.g. GeoTIFF) for median compositing and summary statistics

## How to apply

Run the provided batch graph (Sentinel1_GRDH_to_gamma0.xml) through GPT, the command-line interface of the open-source SNAP toolbox, filling input/output placeholders via the command-line flags. First apply precise orbit state vectors (metadata-only, available ~2 weeks after acquisition), then remove border noise within a 700-pixel search distance from each swath margin and thermal detector noise, keeping data in DN. Radiometrically calibrate DN to beta_0 radar brightness — the required input level for terrain flattening — then apply radiometric terrain flattening using the SRTM 1 sec DEM to convert beta_0 to gamma_0 in slant-range coordinates. Finally run range-Doppler terrain correction (orthorectification) with the same DEM plus the precise orbits to yield gamma_0 in geographic coordinates at 10 m pixel spacing. Skip spatial speckle filtering, since the downstream temporal median over multiple acquisitions suppresses speckle in time rather than space, preserving geometric detail.

## Related tools

- **SNAP toolbox** (Open-source ESA software hosting the Sentinel-1 pre-processing operators; the batch processing configuration file is applied within it) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of SNAP used to execute the Sentinel1_GRDH_to_gamma0.xml processing chain with input/output placeholders replaced by command-line flags)

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -Pinput=<scene_GRDH.zip> -Poutput=<gamma0_out.tif>
```

## Evaluation signals

- Output raster is gamma_0 in geographic coordinates with 10 m pixel spacing
- Backscatter intensities are calibrated (DN → beta_0 → gamma_0) and comparable across acquisitions and sensors
- Signal intensity is normalized across terrain differences because flattening used the SRTM 1 sec DEM
- Each pixel is exactly geolocated after range-Doppler terrain correction (which relies on the DEM plus precise orbits from step 1)
- No spatial speckle filter was applied; speckle suppression is deferred to the temporal median over multiple acquisitions

## Limitations

- Precise orbit state vectors are only available approximately two weeks after data take, delaying the metadata-only Apply-precise-orbit step
- The GRDH product carries geometric and radiometric distortions and noise as uncalibrated DN, so every calibration/noise step is mandatory before intensities are comparable
- Radiometric terrain flattening and terrain correction both depend on DEM quality; the study specifically used the SRTM 1 sec DEM
- Skipping spatial speckle filtering means single-scene outputs remain speckled; smoothing only occurs via temporal median compositing downstream

## Evidence

- [methods] Processing of Sentinel-1 SAR data as utilized in this manuscript can be replicated by applying the provided batch processing configuration file for the SNAP toolbox software.: "Processing of Sentinel-1 SAR data as utilized in this manuscript can be replicated by applying the provided batch processing configuration file for the SNAP toolbox software."
- [methods] All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox: "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox"
- [readme] Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening.: "Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening."
- [readme] it uses a digital elevation model (DEM), which in this study was the SRTM 1 sec DEM.: "The aim of radiometric terrain flattening is to normalize the signal intensity across differences in underlying terrain. To this end, it uses a digital elevation model (DEM), which in this study was"
- [readme] In order to exactly geolocate each pixel, and correct for geometric differences due to the local terrain, on needs to apply this geometric terrain correction: "In order to exactly geolocate each pixel, and correct for geometric differences due to the local terrain, on needs to apply this geometric terrain correction or orthorectification."
- [readme] The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing.: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
