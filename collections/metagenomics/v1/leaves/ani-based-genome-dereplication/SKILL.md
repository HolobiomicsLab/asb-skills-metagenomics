---
name: ani-based-genome-dereplication
description: Use when constructing a de novo source-associated genome database (sourceapp_build.py)
  where redundant reference genomes and MAGs must be collapsed to 95% ANI species
  representatives per fecal source category before metagenomic source tracking.
when_to_use_negative:
- Input is raw shotgun FASTQ reads or assemblies rather than curated genome FASTA
  files — this dereplication operates on genomes, not reads
- Strain-level source discrimination is required — 95% ANI collapses strains into
  species representatives, and ≥95% ANI species shared between source categories are
  explicitly not resolvable (cross-reactive)
- Genomes have not passed the aggregate quality cutoff of 50 (completeness − 5 × redundancy)
  — low-quality MAGs are discarded upstream and would distort clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: SourceApp
  role: sourceapp_build.py orchestrates quality filtering, 95% ANI clustering within/across
    source categories, cross-reactive flagging, and emission of the '<name>_SourceAppdb/'
    directory
  repo: https://github.com/bglindner/SourceApp
- name: CheckM2
  role: computes completeness/redundancy for the aggregate quality cutoff of 50 (quality
    = completeness − 5 × redundancy) applied before dereplication
- name: dRep
  role: ANI-based dereplication engine (fastANI-based S_algorithm) used to cluster
    quality-passing genomes at 95% ANI and retain species-cluster representatives
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:09b98a406dffc05233b1f46515e5abea6dba914f69c3ce1284ecf1ed50711683
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CheckM2
  - Python
  - SourceApp
  - dRep
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ani-based-genome-dereplication

## Summary

Dereplicates source-associated genome databases at 95% ANI so each fecal source category (e.g., cow, pig, dog, cat, chicken, goat, sewage, septage) is represented by species-cluster representative genomes, with cross-category pairs at ANI ≥ 95% flagged as cross-reactive for SourceApp metagenomic source tracking.

## When to use

Use when constructing a de novo source-associated genome database (sourceapp_build.py) where redundant reference genomes and MAGs must be collapsed to 95% ANI species representatives per fecal source category before metagenomic source tracking.

## When NOT to use

- Input is raw shotgun FASTQ reads or assemblies rather than curated genome FASTA files — this dereplication operates on genomes, not reads
- Strain-level source discrimination is required — 95% ANI collapses strains into species representatives, and ≥95% ANI species shared between source categories are explicitly not resolvable (cross-reactive)
- Genomes have not passed the aggregate quality cutoff of 50 (completeness − 5 × redundancy) — low-quality MAGs are discarded upstream and would distort clustering

## Inputs

- Directory of genome FASTA files (e.g., *.fna) — publicly available reference genomes plus short-/long-read MAGs from fecal spike-in slurries
- Headerless TSV source association table mapping genome file name (col1) to fecal source category (col2)
- CheckM2 database (.dmnd) for computing genome quality before clustering
- ANI clustering threshold parameter (e.g., -a at 95% ANI)

## Outputs

- Dereplicated species-cluster representative genomes per source category
- Cross-reactive flags for genome pairs in separate source categories with ANI ≥ 95%
- Formatted '<name>_SourceAppdb/' database directory consumable by sourceapp.py via -d

## How to apply

Collect input genome FASTAs (publicly available reference genomes plus MAGs from short- and long-read sequencing of fecal spike-ins, NCBI BioProject PRJNA1092107) with a headerless TSV mapping each genome file name to its source category (cow, pig, dog, cat, chicken, goat, sewage, septage). Run quality filtering first via CheckM2 (sourceapp_build.py -d pointing at the .dmnd database), discarding genomes with aggregate quality below 50, where quality = completeness − 5 × redundancy. Then dereplicate survivors within and across source categories by clustering at the ANI threshold set with -a (95% ANI, fastANI-based S_algorithm), retaining species-cluster representatives. Separately record any genome pair from different source categories with ANI ≥ 95% as cross-reactive in the database structure, since such shared species cannot uniquely attribute reads to one source. The build succeeds only if a correctly formatted '<name>_SourceAppdb/' directory is created that can be supplied via -d to sourceapp.py.

## Related tools

- **SourceApp** (sourceapp_build.py orchestrates quality filtering, 95% ANI clustering within/across source categories, cross-reactive flagging, and emission of the '<name>_SourceAppdb/' directory) — https://github.com/bglindner/SourceApp
- **CheckM2** (computes completeness/redundancy for the aggregate quality cutoff of 50 (quality = completeness − 5 × redundancy) applied before dereplication)
- **dRep** (ANI-based dereplication engine (fastANI-based S_algorithm) used to cluster quality-passing genomes at 95% ANI and retain species-cluster representatives)

## Examples

```
python sourceapp_build.py -i genomes/ -o mydb -s source_associations.tsv -d checkm2.dmnd -a 0.95
```

## Evaluation signals

- Genomes with aggregate quality (completeness − 5 × redundancy) below 50 do not appear in the dereplicated database
- No two representative genomes retained within the same source category share ≥ 95% ANI (all intra-category duplicates collapsed)
- Every genome pair from different source categories with ANI ≥ 95% is recorded as cross-reactive in the database structure
- The build succeeds only if a correctly formatted '<name>_SourceAppdb/' output directory is created and accepted by sourceapp.py via -d
- Each retained representative is traceable to exactly one fecal source category in the association TSV

## Limitations

- Cross-reactive genome pairs (ANI ≥ 95% between different source categories) are flagged, not removed — species shared across sources limit attribution specificity
- Cow fecal microbiomes were undersampled, so representative coverage after dereplication is biased and SourceApp performance varied substantially by source (cows were underestimated)
- Heavily fragmented MAGs below the aggregate quality score of 50 never reach dereplication, so poorly recovered taxa are absent from the database

## Evidence

- [methods] All medium and high-quality MAGs were dereplicated (at 95% ANI) within their source category: "All medium and high-quality MAGs were dereplicated (at 95% ANI) within their source category"
- [methods] Genome pairs in separate source categories were flagged as cross-reactive when ANI values greater than or equal to 95% were observed: "Genome pairs in separate source categories were flagged as cross-reactive when ANI values greater than or equal to 95% were observed"
- [methods] genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded: "genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded"
- [readme] De novo databases can be constructed by the user with `sourceapp_build.py` and then supplied in the call to `sourceapp.py` itself in place of the default database.: "De novo databases can be constructed by the user with `sourceapp_build.py` and then supplied in the call to `sourceapp.py` itself in place of the default database."
