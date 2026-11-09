---
name: ebwt-data-structure-preprocessing
description: 'Use when preparing inputs for the LiME metagenomic classifier: FASTA
  files of paired-end reads (e.g. setB2_1.fasta/setB2_2.fasta, 100 bp) and a reference
  database (e.g. Refs.fasta, 930 genomes) must be converted into ebwt, lcp and da
  files before ClusterLCP/ClusterBWT_DA/Classify can run.'
when_to_use_negative:
- The goal is classification itself rather than data-structure construction — the
  three LiME steps (ClusterLCP, ClusterBWT_DA, Classify) operate on already-built
  structures.
- Input is a single-end collection where the four merged paired-end/reverse-complement
  collections are unnecessary — build and merge only the forward structures.
- An ad-hoc in-memory index of the reference database is desired (e.g. Kraken/CLARK-style
  database) — LiME's design explicitly avoids keeping reference data structures in
  internal memory.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: eGSA
  role: External-memory construction of generalized suffix array, LCP array and BWT
    for a FASTA collection; output .gesa converted with EGSAtoBCR
  repo: https://github.com/felipelouza/egsa
- name: BCR
  role: Alternative multi-string BWT/LCP builder (BCR_LCP_GSA), can be used incrementally
    to add reads to genome-only structures
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGap
  role: Merges the ebwt structures of read set R and genome set G, computing lcp (--lcp
    or --trlcp k) and document array (--da)
  repo: https://github.com/felipelouza/egap
- name: LiME
  role: Downstream consumer of the ebwt/lcp/da files; ships Install_tools_preprocessing.sh
    and Preprocessing.sh scripts
  repo: https://github.com/veronicaguerrini/LiME
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:a10bb33d3e458cae1d973309c32e21155c8b491591f5bcd3a2ee3209b67fbe3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/felipelouza/egsa
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  - eGap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ebwt-data-structure-preprocessing

## Summary

Builds the extended Burrows-Wheeler transform (ebwt), LCP array, and document array (da) required by LiME for a collection S of reads and reference genomes, using BCR/eGSA for construction and eGap for merging read-set and genome-set structures.

## When to use

Use when preparing inputs for the LiME metagenomic classifier: FASTA files of paired-end reads (e.g. setB2_1.fasta/setB2_2.fasta, 100 bp) and a reference database (e.g. Refs.fasta, 930 genomes) must be converted into ebwt, lcp and da files before ClusterLCP/ClusterBWT_DA/Classify can run.

## When NOT to use

- The goal is classification itself rather than data-structure construction — the three LiME steps (ClusterLCP, ClusterBWT_DA, Classify) operate on already-built structures.
- Input is a single-end collection where the four merged paired-end/reverse-complement collections are unnecessary — build and merge only the forward structures.
- An ad-hoc in-memory index of the reference database is desired (e.g. Kraken/CLARK-style database) — LiME's design explicitly avoids keeping reference data structures in internal memory.

## Inputs

- FASTA file(s) of the metagenomic read set (e.g. setB2_1.fasta, setB2_2.fasta, 100 bp paired-end reads)
- FASTA reference genome database (e.g. Refs.fasta with 930 genomes)
- Reverse-complemented read files (e.g. setB2_1_RC.fasta, setB2_2_RC.fasta) for paired-end strand handling

## Outputs

- Per-collection .ebwt, .lcp, and .da files (or a single FILE.K.gesa from eGSA converted via EGSAtoBCR)
- Merged collections named like ReadsF+Refs.fasta with accompanying ebwt/lcp/da, including reverse-complement variants for paired-end data

## How to apply

Construct ebwt, lcp and da for the read set R and the genome set G separately with BCR (BCR_LCP_GSA) or eGSA, noting that eGSA emits a single fastaFile.K.gesa file that must be converted with EGSAtoBCR filefasta K into .ebwt/.lcp/.da files. Merge the two sets of structures with eGap, setting both --lcp and --da so the merged collection contains all three data structures; optionally cap LCP computation with --trlcp k to shorten eGap running time, and limit internal memory with -m (e.g. 4096 MB as set via INTMEM in the LiME scripts). For paired-end samples, repeat the pipeline for both mates (setB2_1 and setB2_2) and their reverse complements, yielding four merged collections (e.g. setB2_1+Refs.fasta, setB2_1_RC+Refs.fasta, setB2_2+Refs.fasta, setB2_2_RC+Refs.fasta), since reads' strand direction is unknown. Alternatively, BCR can be used incrementally to update the genome-only structures with the reads. The classification result does not depend on which construction algorithm is chosen, so any correct ebwt/lcp/da builder is acceptable.

## Related tools

- **eGSA** (External-memory construction of generalized suffix array, LCP array and BWT for a FASTA collection; output .gesa converted with EGSAtoBCR) — https://github.com/felipelouza/egsa
- **BCR** (Alternative multi-string BWT/LCP builder (BCR_LCP_GSA), can be used incrementally to add reads to genome-only structures) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGap** (Merges the ebwt structures of read set R and genome set G, computing lcp (--lcp or --trlcp k) and document array (--da)) — https://github.com/felipelouza/egap
- **LiME** (Downstream consumer of the ebwt/lcp/da files; ships Install_tools_preprocessing.sh and Preprocessing.sh scripts) — https://github.com/veronicaguerrini/LiME

## Examples

```
./eGap --lcp --da -m 4096 setB2_1+Refs.fasta  # after building read/genome structures with egsa and converting: EGSAtoBCR filefasta K
```

## Evaluation signals

- The merged collection directory contains all three files per collection: .ebwt, .lcp, and .da (or a .gesa successfully converted by EGSAtoBCR into the three BCR-style files).
- Four merged paired-end collections exist for a paired-end sample: forward and _RC variants for both mates (e.g. setB2_1+Refs.fasta and setB2_1_RC+Refs.fasta).
- eGap was invoked with both --lcp and --da (or --trlcp k plus --da), otherwise ClusterLCP cannot run (it needs .lcp and .da) or ClusterBWT_DA with EBWT=1 cannot run (it needs .da and .ebwt).
- Classification output is invariant to the chosen construction algorithm — rebuilding structures with BCR vs eGSA must yield identical classification results.
- Peak internal memory of egsa/eGap stays within the configured budget (INTMEM, e.g. 4096 MB).

## Limitations

- Finding the best method for building the data structures is explicitly out of scope of the paper.
- eGap running time can be reduced with --trlcp k only by replacing all lcp values greater than k with k, trading exact LCP values for speed.
- For paired-end data, Steps 1 and 2 of LiME must be repeated on data structures of both strands (forward and reverse complement), multiplying preprocessing work by four collections.

## Evidence

- [readme] There are mainly two options to obtain the required data structures (ebwt, lcp, da) for S: "There are mainly two options to obtain the required data structures (ebwt, lcp, da) for *S*"
- [readme] To merge the two ebwts associated with the sets R and G, one could use eGap: "To merge the two ebwts associated with the sets *R* and *G*, one could use eGap [https://github.com/felipelouza/egap] and set both options *--lcp* and *--da*"
- [readme] eGSA returns the three data structures in a single file converted by EGSAtoBCR: "Note that egsa tool returns the three data structures in a single file (fastaFile.K.gesa, with K being the number of sequences). The executable file EGSAtoBCR is to convert fastaFile.K.gesa into"
- [methods] both the original sequences and their reverse complements must be considered: "both the original sequences and their reverse complements must be considered in order to keep reads properly oriented"
- [methods] the used method does not affect the classification: [evidence span withheld — anchor status: not_found]
