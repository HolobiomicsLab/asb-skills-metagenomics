---
name: genome-quality-filtering
description: Use when curating a set of reference genomes (e.g., NCBI/JGI assemblies)
  prior to extracting marker genes, to discard assemblies that are too fragmented
  or too incomplete to yield reliable single-copy markers.
when_to_use_negative:
- Input is already a curated, de-redundant reference database (e.g., an existing marker-gene
  catalog like the FunOMIC-T FASTA) — re-filtering at the genome level is redundant
- Analysis targets short-read metagenomic samples for taxonomic/functional profiling
  rather than reference genome curation
- The goal is functional protein database construction, which uses a separate 95%
  CD-HIT clustering workflow rather than genome contiguity cutoffs
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: BUSCO 5.0.0
  role: Identifies single-copy marker genes using Fungi OrthoDB v10.1 (758 ortholog
    HMMs) with the AUGUSTUS gene predictor; genomes with <30 marker genes are discarded
- name: AUGUSTUS
  role: Gene predictor used by BUSCO in default mode to call marker genes during genome
    completeness assessment
- name: FunOMIC
  role: Pipeline whose reference-genome curation step (collection of 9,401 fungal
    genomes from NCBI and JGI MycoCosm, then quality filtering) this skill implements
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:31f7ddcd095f99eeea54327c8f000c1e41f8ac47f0a87312f0966b51dea597a9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - AUGUSTUS
  - BUSCO
  - BUSCO 5.0.0
  - CD-HIT
  - FunOMIC
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# genome-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control filtering of reference fungal genomes before marker-gene database construction, using assembly contiguity cutoffs (>500 contigs, N50 < 10 kbp) and downstream BUSCO single-copy marker gene counts (<30 marker genes) to remove low-quality or fragmented assemblies.

## When to use

Use when curating a set of reference genomes (e.g., NCBI/JGI assemblies) prior to extracting marker genes, to discard assemblies that are too fragmented or too incomplete to yield reliable single-copy markers.

## When NOT to use

- Input is already a curated, de-redundant reference database (e.g., an existing marker-gene catalog like the FunOMIC-T FASTA) — re-filtering at the genome level is redundant
- Analysis targets short-read metagenomic samples for taxonomic/functional profiling rather than reference genome curation
- The goal is functional protein database construction, which uses a separate 95% CD-HIT clustering workflow rather than genome contiguity cutoffs

## Inputs

- Fungal genome assemblies (FASTA, high-quality genomes and draft genomes downloaded from NCBI and JGI MycoCosm)
- Assembly statistics per genome (number of contigs, N50)
- BUSCO 5.0.0 single-copy marker gene counts per genome (Fungi OrthoDB v10.1, 758 orthologs)

## Outputs

- Curated set of 4,839 reference fungal genomes passing the >500 contigs / N50 >= 10 kbp cutoff
- Final filtered set of 4,816 genomes with >=30 identified single-copy marker genes (per Supplementary Table S8 accessions)
- Per-genome BUSCO marker gene counts used as filtering evidence

## How to apply

Download candidate genomes (in the FunOMIC workflow, 9,401 strain-level fungal genomes/draft genomes from NCBI and JGI MycoCosm) and compute assembly statistics for each. Apply a contiguity filter: discard any genome with more than 500 contigs and/or N50 below 10 kbp, which yields the curated pool of 4,839 fungal genomes. Then run BUSCO 5.0.0 with Fungi OrthoDB version 10.1 (758 single-copy ortholog HMMs) using the AUGUSTUS gene predictor, and discard any genome yielding fewer than 30 identified single-copy marker genes, retaining a final set of 4,816 genomes. The rationale is that fragmented or incomplete assemblies produce unreliable marker-gene calls, which would propagate noise into the marker-gene catalog later clustered at 99% identity with CD-HIT.

## Related tools

- **BUSCO 5.0.0** (Identifies single-copy marker genes using Fungi OrthoDB v10.1 (758 ortholog HMMs) with the AUGUSTUS gene predictor; genomes with <30 marker genes are discarded)
- **AUGUSTUS** (Gene predictor used by BUSCO in default mode to call marker genes during genome completeness assessment)
- **FunOMIC** (Pipeline whose reference-genome curation step (collection of 9,401 fungal genomes from NCBI and JGI MycoCosm, then quality filtering) this skill implements) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
busco -i genome.fasta -m genome --lineage_dataset fungi_odb10 --augustus ; awk 'NR>1 && ($2>500 || $4<10000) {print $1}' assembly_stats.tsv > low_quality_genomes.txt
```

## Evaluation signals

- Genome count after contiguity filtering matches ~4,839 genomes from the initial 9,401 downloaded assemblies
- Genome count after BUSCO filtering is ~4,816 (i.e., fewer than 23 genomes discarded at the <30 marker-gene cutoff)
- Every retained genome has >=30 single-copy marker genes identified by BUSCO against the 758 Fungi OrthoDB v10.1 orthologs
- No retained genome violates the assembly cutoffs (contigs <= 500 and N50 >= 10 kbp)
- Downstream specificity check: fewer than 0.01% of marker genes extracted from the filtered genomes map to the prokaryotic UHGG catalog

## Limitations

- Extraction and quality of single-copy marker genes depend on the completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon databases
- Quality thresholds (500 contigs, N50 10 kbp, 30 marker genes) were tuned for fungal reference genome curation; applying them to other kingdoms or very small draft genomes may be inappropriate
- Filtering is based on assembly contiguity and BUSCO completeness only; no explicit contamination screening is applied at this stage

## Evidence

- [methods] BUSCO ... was used to identify marker genes using Fungi OrthoDB version 10.1 in the pool of 4,839 fungal genomes: [evidence span withheld — anchor status: not_found]
