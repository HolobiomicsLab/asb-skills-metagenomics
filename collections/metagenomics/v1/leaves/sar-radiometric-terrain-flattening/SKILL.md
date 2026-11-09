---
name: sar-radiometric-terrain-flattening
description: Use when uncalibrated Sentinel-1 GRDH (IW) digital numbers have been
  radiometrically calibrated to beta_0 and must be normalized for terrain-induced
  backscatter distortion before orthorectification or time-series compositing.
when_to_use_negative:
- Input data are still raw uncalibrated DN or have not yet been radiometrically calibrated
  to beta_0 — calibration must precede terrain flattening
- Data are already orthorectified gamma_0 in geographic coordinates (e.g. pre-processed
  stacks like ALB_gamma0_VH.tif used for the R summary statistics), where re-flattening
  would be redundant
- The analysis targets speckle reduction rather than terrain normalization — spatial
  speckle filtering was deliberately skipped in this workflow in favor of temporal
  median compositing
tools:
- name: SNAP toolbox
  role: Open-source ESA software in which the Sentinel1_GRDH_to_gamma0.xml batch processing
    configuration is executed for the GRDH-to-gamma0 chain
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of SNAP used to run the batch graph; placeholders for
    input/output files are replaced via command-line flags
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:31f399ca11a5301ccb034dd1474606bb0af4541eb340f89bcd35f01a58e64f57
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sar-radiometric-terrain-flattening

## Summary

Radiometric terrain flattening of Sentinel-1 GRDH SAR data: converts beta_0 radar brightness to terrain-normalized gamma_0 using the SRTM 1 sec DEM, so backscatter intensities are comparable across acquisitions and sensors regardless of underlying terrain.

## When to use

Use when uncalibrated Sentinel-1 GRDH (IW) digital numbers have been radiometrically calibrated to beta_0 and must be normalized for terrain-induced backscatter distortion before orthorectification or time-series compositing.

## When NOT to use

- Input data are still raw uncalibrated DN or have not yet been radiometrically calibrated to beta_0 — calibration must precede terrain flattening
- Data are already orthorectified gamma_0 in geographic coordinates (e.g. pre-processed stacks like ALB_gamma0_VH.tif used for the R summary statistics), where re-flattening would be redundant
- The analysis targets speckle reduction rather than terrain normalization — spatial speckle filtering was deliberately skipped in this workflow in favor of temporal median compositing

## Inputs

- Sentinel-1 GRDH IW product (uncalibrated digital numbers in slant-range coordinates, downloaded from the ESA Sentinel Data Hub)
- Beta_0 radar brightness raster (output of the radiometric calibration step)
- SRTM 1 sec digital elevation model (DEM)
- Precise orbit state vectors (available ~2 weeks after data take, applied as metadata in step 1)

## Outputs

- Gamma_0 backscatter raster in slant-range coordinates (direct output of terrain flattening)
- Gamma_0 backscatter raster in geographic coordinates with 10 m pixel spacing (after the subsequent range-Doppler terrain correction)
- Updated Sentinel1_GRDH_to_gamma0.xml GPT batch graph with input/output placeholders filled

## How to apply

Run step 5 of the Sentinel1_GRDH_to_gamma0.xml batch graph via the SNAP toolbox's Graph Processing Tools (GPT) command-line interface, immediately after radiometric calibration has converted digital numbers (DN) to beta_0 radar brightness — beta_0 is the required input data-level for this module. Supply the SRTM 1 sec DEM: the module uses it to normalize signal intensity across differences in the underlying terrain, producing gamma_0 still in slant-range coordinates. Follow with range-Doppler terrain correction (step 6), which reuses the DEM and the precise orbit state vectors from step 1 to yield gamma_0 in geographic coordinates at 10 m pixel spacing. Do not insert spatial speckle filtering: the downstream median compositing over multiple acquisitions suppresses speckle implicitly in time instead of in space.

## Related tools

- **SNAP toolbox** (Open-source ESA software in which the Sentinel1_GRDH_to_gamma0.xml batch processing configuration is executed for the GRDH-to-gamma0 chain) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of SNAP used to run the batch graph; placeholders for input/output files are replaced via command-line flags)

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -PinFile=S1_GRDH_scene.zip -PoutFile=scene_gamma0.tif
```

## Evaluation signals

- Direct module output is gamma_0 in slant-range coordinates, not yet geographic coordinates
- After the full chain (steps 1–6), the output raster is gamma_0 in geographic coordinates with 10 m pixel spacing
- Backscatter intensities in the output are calibrated and comparable across acquisitions and sensors
- The input to the flattening module is beta_0 (radar brightness coefficient), confirming the calibration step ran first
- Border noise was removed earlier in the chain within a search distance of 700 pixels from each swath margin, so border artefacts do not propagate into gamma_0

## Limitations

- Precise orbit state vectors only become available approximately two weeks after data take, blocking immediate processing of very recent acquisitions
- DEM quality (here SRTM 1 sec) bounds the accuracy of terrain normalization and the subsequent range-Doppler orthorectification
- No spatial speckle filtering is applied; speckle suppression relies entirely on temporal median compositing over multiple acquisitions
- The workflow is specific to Sentinel-1 GRDH IW products provided as uncalibrated DN in slant-range coordinates; SLC-derived processing was not needed since only backscatter coefficients were used

## Evidence

- [readme] Radiometric calibration output is in beta_0: "Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening."
- [readme] DEM used for terrain flattening: "To this end, it uses a digital elevation model (DEM), which in this study was the SRTM 1 sec DEM. The output of this module is gamma_0 in slant-range coordinates."
- [readme] Final output spec: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [methods] Batch workflow definition: "The processing workflow as available in the xml batch file is as follows:"
- [readme] Speckle handling rationale: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in space"
