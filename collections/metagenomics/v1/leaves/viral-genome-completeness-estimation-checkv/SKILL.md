---
name: viral-genome-completeness-estimation-checkv
description: Use when you have recovered draft viral bins (FASTA) from metagenomic
  Hi-C or shotgun binning and need to quantify completeness (e.g., to compare ViralCC
  vs VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) rather than using marker-gene tools like
  CheckM.
when_to_use_negative:
- The goal is to estimate contamination of vMAGs — CheckV is unable to estimate contamination
  because no marker gene set is available for viruses
- Evaluating bacterial/archaeal MAGs via universal single-copy marker genes — use
  CheckM instead
- The dataset has few putative viral genomes, which makes completeness-based benchmarking
  less convincing per the paper's discussion
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: CheckV
  role: Estimates completeness of vMAGs by comparing them to a database curated from
    NCBI GenBank and environmental samples
- name: ViralCC
  role: Produces the draft viral genome bins whose completeness is assessed
  repo: https://github.com/dyxstat/ViralCC
- name: CheckM
  role: Marker-gene-based alternative that the paper contrasts against; unsuitable
    for viral genomes due to lack of universal single-copy viral genes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:ea71ccd30029025c7b10338b0a9eda552caa0040b2bc00a3bff06f22938d06f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BBTools
  - BWA
  - CheckM
  - CheckV
  - CoCoNet
  - HiCBin
  - MEGAHIT
  - MetaTOR
  - TAXAassign
  - ViralCC
  - bin3C
  - samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Viral genome completeness estimation with CheckV

## Summary

Estimate the completeness of draft viral genomes (vMAGs) recovered by metagenomic binning by comparing them to CheckV's curated database from NCBI GenBank and environmental samples, then classify bins into high/medium completion categories to benchmark viral binning methods like ViralCC.

## When to use

Use when you have recovered draft viral bins (FASTA) from metagenomic Hi-C or shotgun binning and need to quantify completeness (e.g., to compare ViralCC vs VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) rather than using marker-gene tools like CheckM.

## When NOT to use

- The goal is to estimate contamination of vMAGs — CheckV is unable to estimate contamination because no marker gene set is available for viruses
- Evaluating bacterial/archaeal MAGs via universal single-copy marker genes — use CheckM instead
- The dataset has few putative viral genomes, which makes completeness-based benchmarking less convincing per the paper's discussion

## Inputs

- Draft viral genome bins (FASTA files) output by binning methods such as ViralCC, MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme, VAMB
- CheckV curated database (NCBI GenBank and environmental samples)

## Outputs

- Per-bin completeness estimates and completion category assignments (high/medium completion)
- Aggregated per-method, per-dataset counts of high/medium-completion vMAGs
- Percentage improvement of ViralCC over each baseline method

## How to apply

Run the CheckV command (v0.7.0 in the ViralCC paper) on the FASTA file(s) of draft viral bins produced by each binning method, against CheckV's curated GenBank/environmental database. CheckV estimates completeness by aligning bins to reference viral genomes since viruses lack universal single-copy marker genes, which is why CheckM is unsuitable. Aggregate per-method counts of high- and medium-completion vMAGs across datasets (human gut: SRR6131122-24; cow fecal: ERX2333418/ERX2548555-56; wastewater: SRR8239392-93) and compute percentage improvements of one method over baselines. As a control, note the paper's caution that random binning should be included as a control experiment because random binning does not reduce CheckV completeness. Judge success by whether reproduced counts and percentage improvements (e.g., ViralCC exceeding VAMB, CoCoNet, vRhyme, bin3C, MetaTOR by 161%, 140%, 66.7%, 93.5%, and 62.1% on cow fecal) match reported figures.

## Related tools

- **CheckV** (Estimates completeness of vMAGs by comparing them to a database curated from NCBI GenBank and environmental samples)
- **ViralCC** (Produces the draft viral genome bins whose completeness is assessed) — https://github.com/dyxstat/ViralCC
- **CheckM** (Marker-gene-based alternative that the paper contrasts against; unsuitable for viral genomes due to lack of universal single-copy viral genes)

## Examples

```
checkv completeness viral_bins.fasta checkv_output
```

## Evaluation signals

- Per-bin CheckV completeness values are produced and each bin is assigned a high- or medium-completion category
- Reproduced per-method high/medium-completion counts match the values reported in Fig. 3 of the ViralCC paper
- Percentage improvements of ViralCC over baselines on the cow fecal dataset match reported values (161% vs VAMB, 140% vs CoCoNet, 66.7% vs vRhyme, 93.5% vs bin3C, 62.1% vs MetaTOR)
- A random binning control is included, since random binning does not reduce CheckV completeness and inflates apparent recovery

## Limitations

- CheckV is unable to estimate contamination of vMAGs since there is no marker gene set available for viruses
- The CheckV assessment is not comprehensive; the paper supplements it with a systematic benchmarking strategy
- Random binning does not reduce CheckV completeness, so a random binning control model is necessary when CheckV completeness is used as the evaluation metric
- Benchmarking based on single-contig-recoverable viral genomes underestimates true viral community diversity, and putative viral genomes tend to be small (size bias)

## Evidence

- [intro] CheckV completeness estimation: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples43"
- [intro] CheckV contamination limitation: "CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses21"
- [discussion] Random binning control caveat: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [other] Reported benchmark result: [evidence span withheld — anchor status: not_found]
- [readme] CheckV version used: [evidence span withheld — anchor status: external_doc]
