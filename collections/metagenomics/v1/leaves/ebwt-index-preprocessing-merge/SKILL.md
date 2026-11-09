---
name: ebwt-index-preprocessing-merge
description: Use when preparing ebwt/lcp/da data structures for LiME classification,
  i.e. when a FASTA collection of paired-end reads (e.g. setA2/setB2, SRR1804065)
  and a reference genome set (e.g. Refs.fasta, 930 genomes) must be merged into ebwt(S),
  lcp(S), da(S).
when_to_use_negative:
- When the classification target tool is not LiME-style eBWT-based (e.g. Kraken 2,
  Centrifuge or Magic-BLAST build their own index/database structures)
- When a single combined FASTA of reads+genomes already exists and building ebwt/lcp/da
  from it directly with BCR/eGSA is simpler than the split-and-merge route
- When finding the best/optimal construction algorithm for ebwt/da/lcp is the goal
  — the paper explicitly excludes this from its aims
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Downstream consumer of the merged ebwt/lcp/da files; Preprocessing.sh and
    Install_tools_preprocessing.sh ship with it
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds the multi-string eBWT (and lcp/da) incrementally from FASTA collections;
    can update G's structures with R's symbols
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGSA
  role: External-memory construction of generalized suffix array, LCP array and optional
    BWT in a single .gesa file, converted via EGSAtoBCR
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges the ebwts of R and G, computing lcp and da via --lcp and --da (optionally
    --trlcp k)
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:1d3c546c1d6c63dcb8339dc1f434986b90df927c4893fe7839feab70ef4119f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGAP
  - eGSA
  - eGap
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ebwt-index-preprocessing-merge

## Summary

Preprocessing workflow for the LiME metagenomic classifier: build the extended Burrows-Wheeler transform (ebwt), LCP array and document array (da) for reads and genomes with BCR or eGSA, then merge the read-set and genome-set structures into collections for S=R∪G using eGap with --lcp and --da.

## When to use

Use when preparing ebwt/lcp/da data structures for LiME classification, i.e. when a FASTA collection of paired-end reads (e.g. setA2/setB2, SRR1804065) and a reference genome set (e.g. Refs.fasta, 930 genomes) must be merged into ebwt(S), lcp(S), da(S).

## When NOT to use

- When the classification target tool is not LiME-style eBWT-based (e.g. Kraken 2, Centrifuge or Magic-BLAST build their own index/database structures)
- When a single combined FASTA of reads+genomes already exists and building ebwt/lcp/da from it directly with BCR/eGSA is simpler than the split-and-merge route
- When finding the best/optimal construction algorithm for ebwt/da/lcp is the goal — the paper explicitly excludes this from its aims

## Inputs

- FASTA file(s) of the read set R (paired-end or single-end, e.g. setB2_1.fasta / setB2_2.fasta, 100 bp reads)
- FASTA file of the reference genome set G (e.g. Refs.fasta, 930 genomes)
- Precomputed .ebwt, .lcp, .da files for R and G when using the merge route
- Available internal memory setting (e.g. INTMEM=4096 MB)

## Outputs

- Merged ebwt, lcp, da files for each full collection S (e.g. Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs per dataset)
- Per-set .ebwt, .lcp, .da files for R and G from BCR/eGSA (+ EGSAtoBCR-converted files when starting from .gesa)

## How to apply

Build ebwt, lcp and da for the genome set G and each read set (forward and reverse complement of each mate of a paired-end collection) using BCR (BCR_LCP_GSA) or eGSA; eGSA emits fastaFile.K.gesa which must be converted with EGSAtoBCR into fileFasta.ebwt, fileFasta.lcp and fileFasta.da. Then merge the ebwt of reads R with that of genomes G using eGap with both --lcp and --da (optionally --trlcp k to cap computed LCP values at k and speed up merging), producing the four collections Reads1F+Refs, Reads1RC+Refs, Reads2F+Refs, Reads2RC+Refs per dataset. Building G only once and reusing it across experiments is the key efficiency rationale. Memory for egsa/eGap can be bounded via the INTMEM parameter (e.g. 4096 MB). The classification result is invariant to which construction algorithm is used, so BCR, eGSA, or incremental BCR updates of G's structures are interchangeable.

## Related tools

- **LiME** (Downstream consumer of the merged ebwt/lcp/da files; Preprocessing.sh and Install_tools_preprocessing.sh ship with it) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds the multi-string eBWT (and lcp/da) incrementally from FASTA collections; can update G's structures with R's symbols) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGSA** (External-memory construction of generalized suffix array, LCP array and optional BWT in a single .gesa file, converted via EGSAtoBCR) — https://github.com/felipelouza/egsa
- **eGap** (Merges the ebwts of R and G, computing lcp and da via --lcp and --da (optionally --trlcp k)) — https://github.com/felipelouza/egap

## Examples

```
./eGap --lcp --da -m 4096 Reads1F+Refs.fasta ; EGSAtoBCR Refs.fasta 930
```

## Evaluation signals

- All four per-dataset merged collections exist (forward and reverse-complement of both mates, each merged with Refs) with .ebwt, .lcp and .da present
- Classify downstream yields exactly one status label (C/U/A/H) per read pair, confirming structures are well formed
- Swapping the construction algorithm (BCR vs eGSA) does not change the classification output, per the paper's invariance statement
- Using --trlcp k replaces all lcp values greater than k with k; verify downstream classification is unaffected within tolerance
- LiME's ClusterLCP/ClusterBWT_DA run without errors on the merged files, i.e. ClusterLCP finds both fileFasta.lcp and fileFasta.da

## Limitations

- Merged lcp values can be truncated via --trlcp k; values above k are replaced by k
- Building data structures for G is expensive and should be done once and reused across experiments
- Finding the best construction method for ebwt/da/lcp is explicitly out of the paper's scope
- Internal-memory usage of egsa and eGap must be bounded explicitly (scripts set INTMEM to 4096 MB)

## Evidence

- [methods] To merge the two ebwts associated with the sets R and G, one could use eGap: "To merge the two ebwts associated with the sets R and G, one could use eGap"
- [methods] classification invariant to construction algorithm: [evidence span withheld — anchor status: not_found]
- [readme] eGap trlcp option: "by using the option *--trlcp k*, as an altenative to *--lcp*, eGap stops merging the two ebwts when the lcp value calculated is *k*"
- [readme] EGSAtoBCR conversion: "The executable file EGSAtoBCR is to convert fastaFile.K.gesa into fileFasta.ebwt, fileFasta.lcp, and fileFasta.da -- use command EGSAtoBCR filefasta K"
- [readme] advantage of building G once: "The advantage of the latter choice lies on building the data structures of the set *G* of genomes only once if *G* is the same for each experiment."
