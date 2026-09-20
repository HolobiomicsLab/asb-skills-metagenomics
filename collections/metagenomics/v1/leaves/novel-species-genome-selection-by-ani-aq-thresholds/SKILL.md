---
name: novel-species-genome-selection-by-ani-aq-thresholds
description: 'Use when building an absent-species false-positive benchmark: you have
  a genome set (e.g. PRJNA348753 human gut MAGs) and need to pick genomes whose closest
  RefSeq/training relative fails novelty cutoffs (AQ<60%, ANI<95%).'
when_to_use_negative:
- Your goal is to benchmark precision/recall on species that ARE present in reference
  databases — this filter deliberately excludes any genome with a close relative (ANI>=95%
  or AQ>=60%)
- You need all genomes regardless of novelty; this skill discards genomes whose species
  are already represented in RefSeq or training data
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Mash
  role: Sketches the reference database and provides genome distance screening inside
    mashdiff.sh before whole-genome alignment
- name: MUMmer (dnadiff)
  role: Performs whole-genome alignment of each candidate genome to its best reference
    hit to compute ANI and % Query aligned used for the AQ/ANI filters
- name: CheckM
  role: Quality assessment (lineage_wf + tree_qa) of candidate MAGs via checkm_assessment.sh
    prior to benchmark inclusion
- name: DeepMicrobes-data
  role: Hosts Table S6 (false positives on the 121 novel-species genomes) and benchmark
    scripts for the downstream classification comparison
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:ef24b20a6be9283f5ef3e5968b2368e21a90ef481d55b10624705be9322b64e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes-data
  tools:
  - CLARK
  - CLARK-S
  - CheckM
  - DeepMicrobes
  - DeepMicrobes-data
  - Kraken
  - Kraken 2
  - MUMmer (dnadiff)
  - Mash
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# novel-species-genome-selection-by-ani-aq-thresholds

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies genomes whose species are absent from all reference databases by screening each candidate genome's closest database relative with Mash/dnadiff and keeping only those with alignment query coverage (AQ) below 60% and ANI below 95%, yielding a truly novel-species benchmark set (121 genomes in the DeepMicrobes study).

## When to use

Use when building an absent-species false-positive benchmark: you have a genome set (e.g. PRJNA348753 human gut MAGs) and need to pick genomes whose closest RefSeq/training relative fails novelty cutoffs (AQ<60%, ANI<95%).

## When NOT to use

- Your goal is to benchmark precision/recall on species that ARE present in reference databases — this filter deliberately excludes any genome with a close relative (ANI>=95% or AQ>=60%)
- You need all genomes regardless of novelty; this skill discards genomes whose species are already represented in RefSeq or training data

## Inputs

- Folder of candidate genomes in FASTA format with .fa extension (e.g. MAGs from PRJNA348753)
- Reference sketch file (.msh) generated with `mash sketch` from the reference database (e.g. RefSeq + training genomes)
- Supplementary Table S2 metadata of 3,269 high quality MAGs used for benchmarking (DeepMicrobes-data)

## Outputs

- *dbname_parsed.tab files per query genome (Query name / Reference name / Ref length / % Ref covered / Query length / % Query aligned / ANI / Mash distance)
- Filtered set of novel-species genomes (121 in the paper) with AQ<60% and ANI<95% relative to all databases
- 1x coverage simulated short reads per selected genome for classifier benchmarking (downstream)

## How to apply

Sketch the reference database (e.g. RefSeq plus training genomes) with `mash sketch`, then run mashdiff.sh from Finn-Lab/MGS-gut to compare each candidate genome against it, combining Mash distance with MUMmer dnadiff whole-genome alignment. Each query yields a `*dbname_parsed.tab` file whose columns include % Ref covered, % Query aligned, and ANI. Apply the novelty filters: a genome is 'novel' if its closest relative has query aligned fraction (AQ) below 60% (paper: 10-60% band) and ANI below 95%, so no classifier's database contains the true source species. Optionally confirm genome quality with checkm_assessment.sh (CheckM lineage_wf + tree_qa) before inclusion. The selected genomes (121 in the paper, from PRJNA348753) are then used to simulate 1x coverage short reads and benchmark false positives of DeepMicrobes against Kraken, Kraken 2, CLARK and CLARK-S.

## Related tools

- **Mash** (Sketches the reference database and provides genome distance screening inside mashdiff.sh before whole-genome alignment)
- **MUMmer (dnadiff)** (Performs whole-genome alignment of each candidate genome to its best reference hit to compute ANI and % Query aligned used for the AQ/ANI filters)
- **CheckM** (Quality assessment (lineage_wf + tree_qa) of candidate MAGs via checkm_assessment.sh prior to benchmark inclusion)
- **DeepMicrobes-data** (Hosts Table S6 (false positives on the 121 novel-species genomes) and benchmark scripts for the downstream classification comparison) — https://github.com/MicrobeLab/DeepMicrobes-data

## Examples

```
mashdiff.sh -i genome_folder/ -r reference.msh -s refseq -p gut
```

## Evaluation signals

- Every selected genome's best reference hit in the parsed tab file has % Query aligned within the 10-60% band and ANI < 95%
- The filtered set size matches the published benchmark scale (121 genomes from PRJNA348753)
- Simulated reads from the selected genomes are misclassified by all baselines (source species absent from every tool's database), so false positives can be tallied per tool
- ANI values in the parsed .tab output are consistent between Mash distance pre-screen and dnadiff whole-genome alignment

## Limitations

- AQ/ANI cutoffs (60% query aligned, 95% ANI) are tuned to human gut MAGs vs RefSeq; other clades or databases may need recalibration
- Genomes below both cutoffs may still have partial representation in databases, so 'novel' means absent at species level, not absent at sequence level
- The ANI/AQ screening pipeline is coded for LSF cluster environments and Python 2.7 (per the MGS-gut README)

## Evidence

- [readme] False positive species classification of different tools on data sets generated by simulating 1x coverage reads using 121 genomes from species not included in RefSeq database: [evidence span withheld — anchor status: external_doc]
- [readme] The output for each query genome is a `*dbname_parsed.tab` file containing the dnadiff and Mash results.: [evidence span withheld — anchor status: not_found]
- [other] select the 121 genomes whose closest database relatives fall below the novelty cutoffs (AQ<60% and ANI<95%) so that no tool's database contains the true source species: [evidence span withheld — anchor status: not_found]
