---
name: sequence-similarity-searching
description: Use when you need to quantify how many sequences in a novel catalogue
  (e.g. 879 ISC insertion sequences in FASTA) have known homologs in a curated database
  such as ISfinder, and how those hits split by similarity stringency.
when_to_use_negative:
- The catalogue sequences are already annotated against ISfinder and you only need
  the existing classification
- You are looking for ISs lacking inverted terminal repeats — Palidis/ISC only contains
  common DDE-type ISs with ITRs, so results would not represent such elements
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ISfinder
  role: Curated reference database of known transposable elements queried via its
    online BLAST service to classify catalogue sequences as strict, loose, or novel
    homologs
- name: Palidis
  role: Tool that generated the ISC catalogue of insertion sequences being benchmarked
    against ISfinder
  repo: https://github.com/blue-moon22/palidis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:8db1df643e4ffc5b7e6df4573014040330ddfe620fbb3289205c689ea7a736c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - ISfinder
  - Palidis
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sequence-similarity-searching

## Summary

Benchmark a catalogue of predicted insertion sequences against a curated reference database (ISfinder) by sequence similarity search, classifying each element as a strict homolog, loose homolog, or novel based on e-value thresholds.

## When to use

Use when you need to quantify how many sequences in a novel catalogue (e.g. 879 ISC insertion sequences in FASTA) have known homologs in a curated database such as ISfinder, and how those hits split by similarity stringency.

## When NOT to use

- The catalogue sequences are already annotated against ISfinder and you only need the existing classification
- You are looking for ISs lacking inverted terminal repeats — Palidis/ISC only contains common DDE-type ISs with ITRs, so results would not represent such elements

## Inputs

- FASTA file of 879 insertion sequences from ISC v1.0.0 (github.com/blue-moon22/ISC)
- ISfinder reference database (online BLAST service)

## Outputs

- Per-sequence hit assignments: strict homology (e-value < 1e-50), loose homology (0.01 > e-value >= 1e-50), or novel
- Summary counts table: isfinder_homology_benchmark.csv (360/41.0% hits = 60 strict + 300 loose; 519 novel)

## How to apply

Download the catalogue FASTA (879 ISC v1.0.0 insertion sequences from github.com/blue-moon22/ISC). Query all sequences against ISfinder's online BLAST service with an e-value cutoff of 0.01 and default parameters (as of 7 October 2022). Bin each catalogue sequence into three classes: strict homology (e-value < 1e-50), loose homology (0.01 > e-value >= 1e-50), or novel (no hit at the 0.01 threshold). Tabulate the counts — the expected outcome for ISC v1.0.0 is 360 (41.0%) ISfinder hits comprising 60 strict and 300 loose, with 519 novel — and write the summary to isfinder_homology_benchmark.csv. The rationale is that the 1e-50 cutoff separates near-certain homologs from more distant matches, letting you report both the characterized fraction and the novel fraction of the catalogue.

## Related tools

- **ISfinder** (Curated reference database of known transposable elements queried via its online BLAST service to classify catalogue sequences as strict, loose, or novel homologs)
- **Palidis** (Tool that generated the ISC catalogue of insertion sequences being benchmarked against ISfinder) — https://github.com/blue-moon22/palidis

## Evaluation signals

- Counts reconcile exactly: strict + loose + novel = 879 total catalogue sequences
- Expected benchmark values reproduced: 360 (41.0%) hits = 60 strict + 300 loose, 519 novel
- Every sequence receives exactly one classification under the mutually exclusive e-value bins (<1e-50, [1e-50, 0.01), no hit)
- isfinder_homology_benchmark.csv is produced with the summary counts

## Limitations

- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, so homology coverage is limited to those sources
- The catalogue only contains ISs with ITRs of 25 or more nucleotides (default --min_itr_length 25); ITRs as short as ten nucleotides are missed, so some homologs will be absent from the query set
- The e-value cutoffs (0.01, 1e-50) and default BLAST parameters were fixed as of 7 October 2022; future ISfinder releases may change counts

## Evidence

- [intro] Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder [7]: "Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder [7]"
- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized.: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [discussion] we have released the first version of ISC, a catalogue containing 879 ISs: "we have released the first version of ISC, a catalogue containing 879 ISs"
- [results] 360 (41.0%) of the 879 ISC ISs have hits in ISfinder while 519 are novel; of the hits, 60 show strict homology (e-value <1e-50) and 300 show loose homology (0.01>e-value ≥1e-50).: [evidence span withheld — anchor status: not_found]
