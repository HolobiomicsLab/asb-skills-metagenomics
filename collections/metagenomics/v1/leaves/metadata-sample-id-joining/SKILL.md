---
name: metadata-sample-id-joining
description: Use when a header-present CSV metadata table must be aligned to a sample-by-sample
  SNP distance matrix (or selected subset of samples) before visualization or outbreak
  analysis, ensuring each genomic sample resolves to its patient metadata.
when_to_use_negative:
- The metadata table has no sample-ID field shared with the distance matrix sample
  keys, so no reliable join key exists
- The goal is de novo WGS clustering or SNP distance computation itself — CATHAI consumes
  already-computed distance matrices rather than producing them
- Only raw sequence data are available; metadata coupling presupposes a precomputed
  sample-keyed distance matrix
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: CATHAI
  role: Interactive web-based platform whose processing step ingests the distance
    matrix and metadata and keeps them coupled during visualization and analysis
  repo: https://github.com/FordeGenomics/cathai
- name: Docker
  role: Runs the CATHAI 'process' command with the host data directory bind-mounted
    at /opt/cathai/shiny/data so coupled records land in the shared data directory
  repo: https://github.com/FordeGenomics/cathai
- name: Python 3.9
  role: Loads the CSV metadata and distance matrix and performs the sample-ID join
    with join-completeness QC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac040
    title: 'CATHAI: cluster analysis tool for healthcare-associated infections'
  source_content_hash: sha256:08bb8c3b73adec5c95333a8fc1af185984d5b5c891d30d4789da395d07862125
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metadata-sample-id-joining

## Summary

Couples clinical/epidemiological patient metadata to a sample-keyed WGS SNP distance matrix by joining on the sample-ID field, keeping metadata rows in sync with the genomic samples selected for clustering in CATHAI so results can be interpreted in a clinically meaningful way.

## When to use

Use when a header-present CSV metadata table must be aligned to a sample-by-sample SNP distance matrix (or selected subset of samples) before visualization or outbreak analysis, ensuring each genomic sample resolves to its patient metadata.

## When NOT to use

- The metadata table has no sample-ID field shared with the distance matrix sample keys, so no reliable join key exists
- The goal is de novo WGS clustering or SNP distance computation itself — CATHAI consumes already-computed distance matrices rather than producing them
- Only raw sequence data are available; metadata coupling presupposes a precomputed sample-keyed distance matrix

## Inputs

- Header-present CSV patient metadata file (clinical and epidemiological fields)
- Sample-keyed WGS SNP distance matrix (samples as IDs on rows/columns)
- Set of sample IDs selected in the distance matrix / CATHAI clustering

## Outputs

- coupled_metadata_records.csv — coupled genomic-plus-metadata record table in the shared data directory, with one metadata record per distance-matrix sample ID

## How to apply

First build and run the CATHAI processing step inside Docker so the data land in the shared bind-mounted directory: build the image with 'docker image build --file ./docker/Dockerfile --tag cathai ./docker', then run the 'process' command with port mapping 8080:5000 and UID/GID 1000, bind-mounting ./shiny/data/ at /opt/cathai/shiny/data. Load the header-present CSV metadata file and the sample-keyed WGS distance matrix from that mounted data directory using Python 3.9. Join metadata rows to distance-matrix entries via the matching sample-ID field, restricting (inner-joining) metadata to exactly the samples present in the matrix so the coupled table never contains orphan metadata or missing records. Write the coupled genomic-plus-metadata table as coupled_metadata_records.csv back into the data directory, then perform join-completeness QC: confirm every sample ID present in the distance matrix resolves to exactly one metadata record in the emitted table.

## Related tools

- **CATHAI** (Interactive web-based platform whose processing step ingests the distance matrix and metadata and keeps them coupled during visualization and analysis) — https://github.com/FordeGenomics/cathai
- **Docker** (Runs the CATHAI 'process' command with the host data directory bind-mounted at /opt/cathai/shiny/data so coupled records land in the shared data directory) — https://github.com/FordeGenomics/cathai
- **Python 3.9** (Loads the CSV metadata and distance matrix and performs the sample-ID join with join-completeness QC)

## Examples

```
docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process
```

## Evaluation signals

- Every sample ID present in the distance matrix resolves to exactly one metadata record in the emitted coupled_metadata_records.csv (join completeness QC, no duplicates)
- No rows in coupled_metadata_records.csv correspond to sample IDs absent from the distance matrix (no orphan metadata)
- coupled_metadata_records.csv is written into the shared bind-mounted data directory and is readable by the CATHAI webapp after the 'run' command on port 8080:5000
- Metadata column values for a known sample match the source CSV exactly after the join (no value corruption or reordering)

## Limitations

- Metadata and distance matrix must share a matching sample-ID field; identifiers must be written consistently on both sides or the join will silently drop samples
- Samples in the metadata file that are not selected in the distance matrix are deliberately excluded — the coupled table covers only selected genomic samples
- The CATHAI shiny server should be bound only on 127.0.0.1 or firewalled, since the authentication portal proxies the R shiny apps and exposed ports would leak coupled patient data

## Evidence

- [abstract] CATHAI (cluster analysis tool for healthcare-associated infections) is an interactive web-based visualization platform that couples WGS informed clustering with associated metadata: "CATHAI (cluster analysis tool for healthcare-associated infections) is an interactive web-based visualization platform that couples WGS informed clustering with associated metadata"
- [methods] docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process: "docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process"
- [methods] conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0: "conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0"
- [readme] docker image build --file ./docker/Dockerfile --tag cathai ./docker: "docker image build --file ./docker/Dockerfile --tag cathai ./docker"
