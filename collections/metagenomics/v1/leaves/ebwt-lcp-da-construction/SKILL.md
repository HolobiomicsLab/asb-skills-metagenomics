---
name: ebwt-lcp-da-construction
description: 'Use when preparing inputs for LiME''s ClusterLCP/ClusterBWT_DA steps:
  you need ebwt(S), lcp(S), and da(S) for a fasta collection of paired-end reads plus
  reference genomes.'
when_to_use_negative:
- If only the document array is needed downstream (EBWT=0 variant of ClusterBWT_DA
  uses only fileFasta.da), the full ebwt construction step may be unnecessary
- If an ad-hoc in-memory indexed reference database is preferred (e.g. Kraken-style),
  the sequential-scan eBWT preprocessing approach is not the right fit
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: LiME
  role: Framework consuming the ebwt/lcp/da files in downstream ClusterLCP, ClusterBWT_DA,
    and Classify steps
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds ebwt, lcp, and da from scratch from a single fasta, or incrementally
    updates genome-only structures with reads
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: egsa
  role: External-memory construction of generalized enhanced suffix arrays (BWT, LCP,
    GSA) output as fastaFile.K.gesa
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges ebwt structures of sets R and G with --lcp and --da options (optionally
    --trlcp k)
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:5e68ad5296eef1b673da0b3cb4b6da52282f221ea37ab69fd8ae6bc73b24467d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - eGAP
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ebwt-lcp-da-construction

## Summary

Preprocessing step for LiME's metagenomic classification: build the extended Burrows-Wheeler transform (ebwt), LCP array, and document array (da) for the collection S containing both reads R and reference genomes G, either from a single fasta or by building separately and merging.

## When to use

Use when preparing inputs for LiME's ClusterLCP/ClusterBWT_DA steps: you need ebwt(S), lcp(S), and da(S) for a fasta collection of paired-end reads plus reference genomes.

## When NOT to use

- If only the document array is needed downstream (EBWT=0 variant of ClusterBWT_DA uses only fileFasta.da), the full ebwt construction step may be unnecessary
- If an ad-hoc in-memory indexed reference database is preferred (e.g. Kraken-style), the sequential-scan eBWT preprocessing approach is not the right fit

## Inputs

- FASTA file containing both reads R (paired-end, e.g. setB2_1.fasta/setB2_2.fasta, 100 bp reads) and reference genomes G (e.g. Refs.fasta with 930 genomes)
- Optionally, separate FASTA files for reads and genomes to be built individually then merged

## Outputs

- fileFasta.ebwt (multi-string BWT of collection S)
- fileFasta.lcp (LCP array of collection S)
- fileFasta.da (document array of collection S)
- Alternatively fastaFile.K.gesa from eGSA, converted to .ebwt/.lcp/.da with EGSAtoBCR

## How to apply

Either build ebwt, lcp, and da directly from a single fasta file containing both reads and genomes using BCR (github.com/giovannarosone/BCR_LCP_GSA) or eGSA (github.com/felipelouza/egsa, which returns fastaFile.K.gesa convertible via EGSAtoBCR filefasta K), or build the data structures separately for R and G and merge them with eGap (github.com/felipelouza/egap), setting both --lcp and --da options. The merge option's advantage is that genome data structures are built only once when G is fixed across experiments; use eGap's --trlcp k option to cap lcp computation at k values for faster merging. BCR can also be used incrementally to update the genome-only structures with read symbols without constructing the eBWT and lcp for R from scratch. Note the classification result does not depend on which construction algorithm is used, and internal memory for egsa/eGap can be bounded (e.g. 4096 MB via INTMEM in the provided Preprocessing.sh script).

## Related tools

- **LiME** (Framework consuming the ebwt/lcp/da files in downstream ClusterLCP, ClusterBWT_DA, and Classify steps) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds ebwt, lcp, and da from scratch from a single fasta, or incrementally updates genome-only structures with reads) — https://github.com/giovannarosone/BCR_LCP_GSA
- **egsa** (External-memory construction of generalized enhanced suffix arrays (BWT, LCP, GSA) output as fastaFile.K.gesa) — https://github.com/felipelouza/egsa
- **eGap** (Merges ebwt structures of sets R and G with --lcp and --da options (optionally --trlcp k)) — https://github.com/felipelouza/egap

## Examples

```
./Install_tools_preprocessing.sh && ./Preprocessing.sh
```

## Evaluation signals

- All three files fileFasta.ebwt, fileFasta.lcp, and fileFasta.da exist for the combined fasta after preprocessing (or the .gesa file was converted via EGSAtoBCR)
- If merging separately built structures with eGap, both --lcp and --da options were set so lcp and da are emitted alongside the merged ebwt
- If eGSA was used, EGSAtoBCR was run with the correct sequence count K so downstream LiME tools find .ebwt/.lcp/.da with the expected naming
- Downstream ClusterLCP runs without errors on the merged structures, indicating da(S) and lcp(S) are consistent with ebwt(S)
- Classification results are unchanged when a different construction algorithm is used (the paper states the construction method does not affect classification)

## Limitations

- eGap's --trlcp k replaces all lcp values greater than k with k, truncating lcp information in exchange for faster merging
- Finding the best method for building the data structures is explicitly out of scope of the paper
- Internal memory of egsa and eGap must be bounded explicitly (e.g. 4096 MB via INTMEM parameters in the scripts)

## Evidence

- [methods] There are mainly two options to obtain the required data structures (ebwt, lcp, da) for S: "There are mainly two options to obtain the required data structures (ebwt, lcp, da) for S"
- [methods] one could use BCR [https://github.com/giovannarosone/BCR_LCP_GSA], or egsa: "one could use BCR [https://github.com/giovannarosone/BCR_LCP_GSA], or egsa [https://github.com/felipelouza/egsa] for instance"
- [methods] one could use eGap and set both options --lcp and --da: "one could use eGap [https://github.com/felipelouza/egap] and set both options --lcp and --da"
- [methods] the used method does not affect the classification, so one can use other algorithms to construct ebwt(S), da(S) and lcp(S): [evidence span withheld — anchor status: not_found]
- [readme] by using the option --trlcp k, as an altenative to --lcp, eGap stops merging the two ebwts when the lcp value calculated is k: "by using the option *--trlcp k*, as an altenative to *--lcp*, eGap stops merging the two ebwts when the lcp value calculated is *k*"
