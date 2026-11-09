---
name: mag-quality-filtering-checkm2
description: Use when building a de novo SourceApp source-associated genome database
  from mixed FASTA inputs (reference genomes plus MAGs) and low-quality or fragmented
  genome drafts must be excluded before ANI dereplication.
when_to_use_negative:
- Input is already a curated, pre-filtered database such as the SourceApp default
  database from Zenodo — no de novo quality filtering is needed
- The analysis goal is read-mapping source attribution with sourceapp.py on existing
  metagenomes, not building a new genome database
- Genomes must be assessed for taxonomic classification only (e.g., GTDB release 220
  assignment) rather than completeness/redundancy quality
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: SourceApp
  role: sourceapp_build.py applies the CheckM2-based aggregate quality threshold of
    50 during de novo source-associated database construction
  repo: https://github.com/bglindner/SourceApp
- name: CheckM2
  role: computes genome completeness and redundancy used for the quality = completeness
    − 5 × redundancy cutoff; supplied as a .dmnd database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:6c664e4e18c1c48d4e35aeed1ce6783628597e466f4cb66625da008e339cdefe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CheckM2
  - Python
  - SourceApp
  - dRep
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# MAG quality filtering with CheckM2

## Summary

Quality-filtering step in SourceApp database construction where CheckM2 scores input reference genomes and MAGs and genomes with aggregate quality (completeness − 5 × redundancy) below 50 are discarded before dereplication.

## When to use

Use when building a de novo SourceApp source-associated genome database from mixed FASTA inputs (reference genomes plus MAGs) and low-quality or fragmented genome drafts must be excluded before ANI dereplication.

## When NOT to use

- Input is already a curated, pre-filtered database such as the SourceApp default database from Zenodo — no de novo quality filtering is needed
- The analysis goal is read-mapping source attribution with sourceapp.py on existing metagenomes, not building a new genome database
- Genomes must be assessed for taxonomic classification only (e.g., GTDB release 220 assignment) rather than completeness/redundancy quality

## Inputs

- Directory of genome FASTA files (*.fna): publicly available reference genomes plus MAGs recovered from short- (Illumina NovaSeq 6000) and long-read (Oxford Nanopore minION) sequencing of fecal spike-ins (NCBI BioProject PRJNA1092107)
- Tab-separated source association table (no header): genome file name (col1) mapped to fecal source category (col2), e.g., cow, pig, dog, cat, chicken, goat, sewage, septage
- CheckM2 database in diamond (.dmnd) format supplied via the -d flag

## Outputs

- Set of genomes passing the aggregate quality threshold of 50 (completeness − 5 × redundancy) retained for 95% ANI dereplication
- SourceApp database output directory appended with '_SourceAppdb/' (e.g., 'SourceAppdb') formatted for use with sourceapp.py via -d

## How to apply

Collect input genomes as FASTA files (e.g., *.fna) in a single directory and run sourceapp_build.py with -d pointing at the CheckM2 database (.dmnd), so CheckM2 estimates completeness and redundancy for every genome. Apply the aggregate quality threshold of 50, computed as quality = completeness − 5 × redundancy, and discard any genome scoring below 50. Only genomes passing this filter proceed to 95% ANI dereplication within/across source categories with fastANI-based clustering, and cross-reactive pairs (ANI ≥ 95%) in separate source categories are flagged. The rationale is that fragmented or contaminated MAGs would otherwise create spurious source attributions in downstream read-mapping-based fecal source tracking.

## Related tools

- **SourceApp** (sourceapp_build.py applies the CheckM2-based aggregate quality threshold of 50 during de novo source-associated database construction) — https://github.com/bglindner/SourceApp
- **CheckM2** (computes genome completeness and redundancy used for the quality = completeness − 5 × redundancy cutoff; supplied as a .dmnd database)

## Examples

```
python sourceapp_build.py -i genomes_dir/ -o SourceAppdb -s source_associations.tsv -d checkm2_db.dmnd -a 0.95
```

## Evaluation signals

- All genomes in the final database have aggregate quality scores of at least 50 (quality = completeness − 5 × redundancy)
- sourceapp_build.py completes and creates the output directory appended with '_SourceAppdb/' (e.g., 'SourceAppdb')
- The resulting database directory is correctly formatted and can be supplied via -d to sourceapp.py without errors
- Genomes below the aggregate quality score of 50 are verifiably absent from the built database
- Retained genomes survive subsequent 95% ANI dereplication as species-cluster representatives

## Limitations

- The aggregate quality threshold of 50 (completeness − 5 × redundancy) discards borderline MAGs that may carry source-informative signal
- CheckM2 quality estimates for highly novel lineages (e.g., poorly sampled cow fecal microbiomes) may be uncertain, contributing to false negatives for cow sources
- Genomes passing quality filtering can still be cross-reactive between source categories (ANI ≥ 95%), requiring flagging rather than removal

## Evidence

- [methods] CheckM2 computes genome quality and the aggregate quality threshold of 50 (quality = completeness - 5 x redundancy) is applied to discard low-quality input genomes: "genomes with aggregate quality scores below 50% as calculated by quality = completeness −5 × redundancy were discarded"
- [readme] sourceapp_build.py takes the CheckM2 database and builds the output database directory: "De novo databases can be constructed by the user with `sourceapp_build.py` and then supplied in the call to `sourceapp.py` itself in place of the default database."
- [abstract] MAGs recovered from short- and long-read sequencing of fecal spike-ins feed the source-associated genome databases: "We built source-associated genome databases using publicly available reference genomes and metagenome assembled genomes (MAGs) recovered from short- and long-read sequencing of the fecal spike-ins"
- [methods] Dereplication follows quality filtering at 95% ANI within source categories: "All medium and high-quality MAGs were dereplicated (at 95% ANI) within their source category"
