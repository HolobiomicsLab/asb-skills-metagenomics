---
name: wgs-distance-matrix-metadata-coupling
description: Use when a sample-keyed WGS SNP distance matrix must be joined to a header-present
  clinical/epidemiological metadata CSV so downstream cluster visualization and analysis
  retain per-sample metadata.
when_to_use_negative:
- Input is already a coupled genomic-plus-metadata record table produced by a prior
  CATHAI 'process' run
- Sample identifiers in the metadata CSV do not correspond to the IDs in the distance
  matrix (no shared sample-ID join key)
- The task is interactive cluster visualization only — in that case run the CATHAI
  webapp ('run' command) rather than re-deriving the metadata coupling
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: CATHAI
  role: Interactive web-based visualization platform whose 'process' command couples
    WGS-informed clustering (SNP distance matrix) with associated metadata; the coupled
    data are written to the shared shiny/data directory
  repo: https://github.com/FordeGenomics/cathai
- name: Docker
  role: Builds the CATHAI image and runs the 'process' command with bind-mounted data
    directory (source=./shiny/data/, target=/opt/cathai/shiny/data) and port mapping
    8080:5000
- name: Python 3.9
  role: Loads the metadata CSV and distance matrix and performs the sample-ID join
    restricting metadata to selected samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac040
    title: 'CATHAI: cluster analysis tool for healthcare-associated infections'
  source_content_hash: sha256:5cfdf7580ad157906ed3a5d2dfa0ce1e256adbae302b5df8ccde0c5a529322d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FordeGenomics/cathai
  tools:
  - CATHAI
  - Docker
  - Python 3.9
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# wgs-distance-matrix-metadata-coupling

## Summary

Couple patient clinical and epidemiological metadata to a sample-keyed WGS distance matrix by joining on the shared sample-ID field, keeping metadata synchronized with selected genomic samples for clinically meaningful cluster interpretation in CATHAI.

## When to use

Use when a sample-keyed WGS SNP distance matrix must be joined to a header-present clinical/epidemiological metadata CSV so downstream cluster visualization and analysis retain per-sample metadata.

## When NOT to use

- Input is already a coupled genomic-plus-metadata record table produced by a prior CATHAI 'process' run
- Sample identifiers in the metadata CSV do not correspond to the IDs in the distance matrix (no shared sample-ID join key)
- The task is interactive cluster visualization only — in that case run the CATHAI webapp ('run' command) rather than re-deriving the metadata coupling

## Inputs

- Header-present CSV patient metadata file (clinical and epidemiological fields)
- Sample-keyed WGS SNP distance matrix (samples as IDs in rows/columns)
- CATHAI Docker image built from ./docker/Dockerfile with tag 'cathai'

## Outputs

- coupled_metadata_records.csv — coupled genomic-plus-metadata record table written to the shared data directory
- Join completeness QC confirmation that each distance-matrix sample ID maps to exactly one metadata record

## How to apply

Build the CATHAI Docker image from ./docker/Dockerfile with tag 'cathai', then run the CATHAI 'process' command inside Docker with port mapping 8080:5000, UID/GID 1000, and the host ./shiny/data/ directory bind-mounted at /opt/cathai/shiny/data so processed data land in the shared data directory. Load the header-present CSV metadata file and the sample-keyed WGS distance matrix from the mounted data directory using Python 3.9. Join metadata rows to distance-matrix entries via the matching sample-ID field, restricting metadata to only the samples present in the matrix so the two stay in sync. Write the coupled genomic-plus-metadata record table as coupled_metadata_records.csv into the data directory. Validate join completeness by confirming every sample ID in the distance matrix resolves to exactly one metadata record in the emitted coupled table.

## Related tools

- **CATHAI** (Interactive web-based visualization platform whose 'process' command couples WGS-informed clustering (SNP distance matrix) with associated metadata; the coupled data are written to the shared shiny/data directory) — https://github.com/FordeGenomics/cathai
- **Docker** (Builds the CATHAI image and runs the 'process' command with bind-mounted data directory (source=./shiny/data/, target=/opt/cathai/shiny/data) and port mapping 8080:5000)
- **Python 3.9** (Loads the metadata CSV and distance matrix and performs the sample-ID join restricting metadata to selected samples)

## Examples

```
docker image build --file ./docker/Dockerfile --tag cathai ./docker && docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process
```

## Evaluation signals

- Every sample ID present in the distance matrix resolves to exactly one metadata record in coupled_metadata_records.csv (join completeness QC)
- No metadata rows for samples absent from the distance matrix appear in the coupled table (metadata restricted to selected samples)
- coupled_metadata_records.csv is written into the bind-mounted shared data directory (/opt/cathai/shiny/data on the container side)
- The coupled table preserves the clinical and epidemiological fields from the original header-present CSV
- After 'process', the CATHAI webapp launched via the 'run' command on port 8080:5000 renders clusters with the synced metadata

## Limitations

- A user-supplied config.py is required; the sample config.py must be adapted before running CATHAI
- The shiny server should be bound only on 127.0.0.1 or firewalled so shiny server ports are not exposed to outside sources
- Metadata can only be coupled for samples whose IDs are present in the distance matrix; unmatched IDs will fail the join-completeness QC

## Evidence

- [abstract] CATHAI couples WGS clustering with metadata: "CATHAI (cluster analysis tool for healthcare-associated infections) is an interactive web-based visualization platform that couples WGS informed clustering with associated metadata"
- [readme] Docker process command with bind mount: "docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process"
- [methods] Docker build command: "docker image build --file ./docker/Dockerfile --tag cathai ./docker"
- [readme] Python 3.9 environment for CATHAI: "conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0"
- [readme] Shiny server security note: "Make sure that the shiny server is only bound on 127.0.0.1 or that the host has a firewall"
