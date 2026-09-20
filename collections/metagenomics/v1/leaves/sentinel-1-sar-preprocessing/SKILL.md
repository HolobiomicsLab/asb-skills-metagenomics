---
name: sentinel-1-sar-preprocessing
description: Use when raw Sentinel-1 GRDH IW products (uncalibrated digital numbers
  in slant-range coordinates) must be converted to calibrated, terrain-flattened,
  orthorectified gamma_0 backscatter at 10 m pixel spacing.
when_to_use_negative:
- Input is Single Look Complex (SLC) data — the study used only backscatter coefficients,
  so deriving GRDH from SLC was deemed unnecessary and this chain targets GRDH only
- The analysis requires spatial speckle filtering or per-scene spatial smoothing —
  this chain deliberately skips speckle filtering and relies on temporal median compositing
  instead
- The task needs preprocessed data before precise orbits become available (~2 weeks
  after data take), since the workflow depends on precise orbit state vectors
tools:
- name: SNAP toolbox
  role: Open-source ESA software executing the Sentinel-1 SAR preprocessing via the
    provided batch processing configuration file
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of SNAP used to run the Sentinel1_GRDH_to_gamma0.xml
    processing chain with placeholder substitution flags
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:27cc3e2d5571dcf10787cfde95479c1d5fc1b087bf528973e21d7075e7eed7b1
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
yes_rate: 1.0
n_claims: 9
---

# sentinel-1-sar-preprocessing

## Summary

Reproduce the Sentinel-1 GRDH-to-gamma_0 preprocessing chain of Bae et al. (2019) using ESA's open-source SNAP toolbox via its Graph Processing Tools (GPT) command-line interface and the provided Sentinel1_GRDH_to_gamma0.xml batch file.

## When to use

Use when raw Sentinel-1 GRDH IW products (uncalibrated digital numbers in slant-range coordinates) must be converted to calibrated, terrain-flattened, orthorectified gamma_0 backscatter at 10 m pixel spacing.

## When NOT to use

- Input is Single Look Complex (SLC) data — the study used only backscatter coefficients, so deriving GRDH from SLC was deemed unnecessary and this chain targets GRDH only
- The analysis requires spatial speckle filtering or per-scene spatial smoothing — this chain deliberately skips speckle filtering and relies on temporal median compositing instead
- The task needs preprocessed data before precise orbits become available (~2 weeks after data take), since the workflow depends on precise orbit state vectors

## Inputs

- Sentinel-1 GRDH IW product (uncalibrated digital numbers, slant-range coordinates) from the ESA Sentinel Data Hub
- Sentinel1_GRDH_to_gamma0.xml batch processing configuration file (with input/output placeholders)
- SRTM 1 sec DEM (used by terrain flattening and range-Doppler terrain correction)

## Outputs

- gamma_0 backscatter raster in geographic coordinates at 10 m pixel spacing
- Intermediate beta_0 (radar brightness) output after radiometric calibration, comparable across acquisitions and sensors

## How to apply

Download the Sentinel-1 GRDH IW product (uncalibrated DN in slant-range coordinates) from the ESA Copernicus data portal, then run the provided batch graph (Sentinel1_GRDH_to_gamma0.xml) through GPT, replacing the input/output placeholders via command-line flags. The graph applies six ordered steps: (1) apply precise orbit state vectors — a metadata-only step using orbits available ~2 weeks after acquisition; (2) remove border noise within a search distance of 700 pixels from each swath margin; (3) remove thermal detector noise, keeping data in DN; (4) radiometric calibration converting DN to beta_0 radar brightness, which is the required input level for terrain flattening; (5) radiometric terrain flattening using the SRTM 1 sec DEM to convert beta_0 to gamma_0 in slant-range coordinates; (6) range-Doppler terrain correction (orthorectification) with the same DEM and precise orbits. No spatial speckle filter is applied because the study computes temporal medians over multiple acquisitions, suppressing speckle in time rather than space without further loss of geometric detail.

## Related tools

- **SNAP toolbox** (Open-source ESA software executing the Sentinel-1 SAR preprocessing via the provided batch processing configuration file) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of SNAP used to run the Sentinel1_GRDH_to_gamma0.xml processing chain with placeholder substitution flags)

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -Pinput=<S1_GRDH_scene.zip> -Poutput=<output_dir>
```

## Evaluation signals

- Final output raster is gamma_0 expressed in geographic coordinates (not slant-range/image coordinates) with 10 m pixel spacing
- After radiometric calibration the data are in beta_0 (radar brightness), the required input data-level for radiometric terrain flattening
- Backscatter intensities are calibrated and comparable across acquisitions and sensors
- The apply-precise-orbit step changes only metadata (adds orbit state vectors) and does not modify the input data
- Border-noise removal operates within a search distance of 700 pixels from each margin, and DN is retained until calibration

## Limitations

- Precise orbit state vectors are only available approximately two weeks after data acquisition
- Sentinel-1 GRDH data exhibit strong artefacts at swath borders, which motivates the 700-pixel border-noise removal step
- The workflow intentionally omits spatial speckle filtering; speckle suppression relies on temporal median compositing over multiple acquisitions downstream, so single-scene speckle remains
- Chain targets GRDH IW high-level products; SLC-to-backscatter conversion is not covered

## Evidence

- [readme] GPT runs the xml batch graph: "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox"
- [methods] Article points to the SNAP batch configuration file: "Processing of Sentinel-1 SAR data as utilized in this manuscript can be replicated by applying the provided batch processing configuration file for the SNAP toolbox software."
- [readme] beta_0 is the calibration output and required input for terrain flattening: "Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening."
- [readme] Final output specification: "The output of this module is gamma_0 in geographic coordinates with 10 m pixel spacing."
- [readme] Speckle handled temporally, not spatially: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in the space"
