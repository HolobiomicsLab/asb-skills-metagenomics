---
name: sar-radiometric-calibration-beta0
description: Use when Sentinel-1 GRDH IW data held as uncalibrated digital numbers
  (DN) must be converted to calibrated backscatter (beta_0) comparable across acquisitions
  and sensors, prior to terrain flattening to gamma_0.
when_to_use_negative:
- The scene is SLC data or the analysis requires phase/interferometric information
  — the study only used backscatter coefficients from GRDH and explicitly avoided
  deriving GRDH from SLC.
- The input is already calibrated (beta_0/gamma_0) or already orthorectified gamma_0
  in geographic coordinates — recalibration would be redundant or wrong.
- You intend to apply spatial speckle filtering instead — this chain deliberately
  skips it, relying on temporal median compositing over multiple acquisitions for
  speckle suppression.
tools:
- name: SNAP toolbox
  role: Provides the radiometric calibration operator within the batch processing
    configuration file; open-source, distributed by ESA for Windows, Mac OS X and
    Unix (64bit).
  repo: https://step.esa.int/main/download/snap-download/
- name: Graph Processing Tools (GPT)
  role: Command-line interface of SNAP used to execute the Sentinel1_GRDH_to_gamma0.xml
    batch graph containing the calibration step, with placeholders replaced via command-line
    flags.
  repo: github.com/So-YeonBae/Sentinel1-Biodiversity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12737-x
    title: Radar vision in the mapping of forest biodiversity from space
  source_content_hash: sha256:44faa82fdd3baef8d3b8f08eb2e39c4ad13d74015d8be9e908498164e1537f08
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
n_claims: 8
---

# sar-radiometric-calibration-beta0

## Summary

Radiometric calibration converts uncalibrated Sentinel-1 GRDH IW digital numbers (DN) into beta_0 radar brightness coefficients using SNAP's Calibration operator. It is the required data-level precursor to radiometric terrain flattening that yields gamma_0.

## When to use

Use when Sentinel-1 GRDH IW data held as uncalibrated digital numbers (DN) must be converted to calibrated backscatter (beta_0) comparable across acquisitions and sensors, prior to terrain flattening to gamma_0.

## When NOT to use

- The scene is SLC data or the analysis requires phase/interferometric information — the study only used backscatter coefficients from GRDH and explicitly avoided deriving GRDH from SLC.
- The input is already calibrated (beta_0/gamma_0) or already orthorectified gamma_0 in geographic coordinates — recalibration would be redundant or wrong.
- You intend to apply spatial speckle filtering instead — this chain deliberately skips it, relying on temporal median compositing over multiple acquisitions for speckle suppression.

## Inputs

- Sentinel-1 GRDH IW product as uncalibrated digital numbers (DN) in slant-range coordinates, with border noise and thermal noise removed (output of steps 2–3 of the batch graph)
- Sentinel1_GRDH_to_gamma0.xml SNAP batch processing configuration file with input/output placeholders

## Outputs

- Raster of calibrated beta_0 radar brightness coefficients (slant-range coordinates), suitable as input to radiometric terrain flattening

## How to apply

Run calibration as step 4 of the six-node graph in the provided batch file Sentinel1_GRDH_to_gamma0.xml, executed via GPT (the SNAP command-line interface), after Apply precise orbit, Remove border noise (700-pixel search distance from each margin), and Remove thermal noise — at this point the data are still in DN. Invoke the radiometric calibration operator to convert DN to beta_0, the radar brightness coefficient; the README states beta_0 is the required input data-level for radiometric terrain flattening, so do not feed DN or sigma_0 to that step. Calibration must come before terrain flattening (SRTM 1 sec DEM) and range-Doppler terrain correction, which jointly produce gamma_0 in geographic coordinates at 10 m pixel spacing. Deliberately omit spatial speckle filtering, since the study's downstream temporal median compositing over multiple acquisitions suppresses speckle implicitly without losing geometric detail.

## Related tools

- **SNAP toolbox** (Provides the radiometric calibration operator within the batch processing configuration file; open-source, distributed by ESA for Windows, Mac OS X and Unix (64bit).) — https://step.esa.int/main/download/snap-download/
- **Graph Processing Tools (GPT)** (Command-line interface of SNAP used to execute the Sentinel1_GRDH_to_gamma0.xml batch graph containing the calibration step, with placeholders replaced via command-line flags.) — github.com/So-YeonBae/Sentinel1-Biodiversity

## Examples

```
gpt Sentinel1_GRDH_to_gamma0.xml -Pinput=<scene_GRDH.zip> -Poutput=<gamma0_out.dim>
```

## Evaluation signals

- Output raster values are calibrated backscatter intensities (beta_0 radar brightness), not raw digital numbers (DN).
- Calibrated intensities are comparable across acquisitions and sensors — check consistency between repeat-pass scenes.
- The downstream radiometric terrain flattening node accepts the output (it requires beta_0 as input data-level).
- The full GPT graph (all six nodes) completes and the final product is gamma_0 in geographic coordinates at 10 m pixel spacing.

## Limitations

- The preceding Apply-precise-orbit step depends on precise orbit state vectors, available approximately two weeks after data take.
- Strong artefacts at swath borders must be removed first (within a 700-pixel search distance from each margin), otherwise DN entering calibration contain border noise.
- Speckle suppression is temporal, not spatial: single-acquisition calibrated images retain speckle that is only smoothed by median compositing over multiple acquisitions.
- Calibration output is in slant-range coordinates; geographic coordinates and 10 m pixel spacing are only achieved after range-Doppler terrain correction.

## Evidence

- [readme] Radiometric calibration is needed to convert the SAR data from digital numbers (DNs), to calibrated backscatter intensities, which are comparable across acquisitions and sensors.: "Radiometric calibration is needed to convert the SAR data from digital numbers (DNs), to calibrated backscatter intensities, which are comparable across acquisitions and sensors."
- [readme] beta_0 required input for terrain flattening: "Radiometric calibration output is in beta_0 (radar brightness coefficient), which is the required input data-level for radiometric terrain flattening."
- [methods] GPT as reproduction mechanism: "All processing steps can be reproduced by means of the Graph Processing Tools (GPT), i.e. the command line interface included in the SNAP toolbox"
- [readme] temporal speckle suppression rationale: "calculating median backscatter over multiple acquisitions, smoothing of speckle effects takes place implicitly in the time, instead of in space"
