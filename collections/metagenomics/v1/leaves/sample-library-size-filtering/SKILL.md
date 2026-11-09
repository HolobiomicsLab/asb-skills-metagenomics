---
name: sample-library-size-filtering
description: Use when a phyloseq OTU/feature table with uneven per-sample read depths
  must be QC-filtered before composition or survival analysis, and shallow samples
  (e.g., <3,000 reads) should be removed.
when_to_use_negative:
- The input is already a post-QC phyloseq object (e.g., biom.Rdata from the MiSurv
  Data folder) — re-filtering would drop additional samples and break reproducibility
  against the deposited data.
- The plan is rarefaction to even depth rather than whole-sample removal — rarefaction
  via rarefy_even_depth normalizes depth but does not exclude shallow samples the
  way the library-size QC does.
- Analysis target is raw sequence reads, not a feature/OTU table — read-level filtering
  (e.g., Phred <20 runs) is a separate earlier step.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MiSurv
  role: Web cloud platform whose Data Processing module implements the library-size
    quality control (kingdom, library size, mean proportion, taxonomic name) before
    downstream analysis
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to load the OTU and taxonomy tables, compute sample total library
    sizes, and apply taxa/sample filtering before QC thresholds are applied
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:5b8293582902e460c80c7d4c64a42d589d8ac8efce6072636e2daf9e5b300dcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sample-library-size-filtering

## Summary

Drops microbiome samples whose total sequencing library size falls below a minimum read threshold (MiSurv default: 3,000 reads) during quality control, so downstream survival and composition analyses are not driven by under-sequenced samples.

## When to use

Use when a phyloseq OTU/feature table with uneven per-sample read depths must be QC-filtered before composition or survival analysis, and shallow samples (e.g., <3,000 reads) should be removed.

## When NOT to use

- The input is already a post-QC phyloseq object (e.g., biom.Rdata from the MiSurv Data folder) — re-filtering would drop additional samples and break reproducibility against the deposited data.
- The plan is rarefaction to even depth rather than whole-sample removal — rarefaction via rarefy_even_depth normalizes depth but does not exclude shallow samples the way the library-size QC does.
- Analysis target is raw sequence reads, not a feature/OTU table — read-level filtering (e.g., Phred <20 runs) is a separate earlier step.

## Inputs

- Pre-QC OTU count table (e.g., otu.tab.txt, tab-delimited)
- Taxonomy table (tax.tab.txt) with kingdom-level annotations
- Sample metadata table (sam.dat.txt)
- MiSurv Data Processing module quality control settings (library size threshold)

## Outputs

- Filtered OTU abundance table restricted to samples above the library-size threshold (e.g., post_qc_otu_table.txt)
- Filtered taxonomy table matched to retained samples (e.g., post_qc_tax_table.txt)
- Retained sample set (172 subjects for the T1D example)

## How to apply

After loading the pre-QC OTU count table (otu.tab.txt) and taxonomy table (tax.tab.txt) into a phyloseq object, restrict taxa to kingdom = Bacteria and drop erroneous/missing taxonomic names, then compute each retained sample's total library size and remove any sample whose total falls below the minimum threshold (3,000 reads with MiSurv's default settings). Library-size filtering is one of the four MiSurv quality controls (kingdom, library size, mean proportion, taxonomic name), so it should be applied alongside the mean relative proportion cutoff (0.002%) rather than in isolation. Write the surviving abundance and taxonomy tables to post-QC files for downstream modules. On the example Zhang et al. T1D gut microbiome data, this workflow retained 172 subjects and 294 features (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species).

## Related tools

- **MiSurv** (Web cloud platform whose Data Processing module implements the library-size quality control (kingdom, library size, mean proportion, taxonomic name) before downstream analysis) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to load the OTU and taxonomy tables, compute sample total library sizes, and apply taxa/sample filtering before QC thresholds are applied)

## Examples

```
library(phyloseq); keep <- sample_sums(physeq) >= 3000; physeq_qc <- prune_samples(keep, physeq)
```

## Evaluation signals

- After filtering, the retained sample count matches the expected subject count — 172 subjects for the example T1D dataset under default settings.
- No retained sample has total library size below the 3,000-read threshold; verify by recomputing sample_sums() on the filtered phyloseq object.
- The post-QC tables (post_qc_otu_table.txt / post_qc_tax_table.txt) reproduce the deposited post-QC files (biom.Rdata / otu.tab.txt / tax.tab.txt) from the MiSurv Data folder.
- Feature count after the full QC chain equals 294 features across the stated taxonomic ranks (7 phyla, 14 classes, 18 orders, 21 families, 30 genera, 13 species).
- All retained taxa have kingdom = Bacteria; no samples were lost to missing/erroneous taxonomic names rather than library size.

## Limitations

- The 3,000-read cutoff is a platform default, not a dataset-derived threshold; datasets with systematically lower depth may lose most samples.
- MiSurv's GitHub repository provides no changelog, no R package version numbers (e.g., phyloseq), and no software license, complicating exact reproduction of QC behavior.
- Library-size filtering interacts with the other three QC steps (kingdom, mean proportion 0.002%, taxonomic name); applying only this filter will not yield the deposited 294-feature result.

## Evidence

- [full_text] Default QC threshold and retained result: "quality controls (kingdom, library size, mean proportion, taxonomic name)"
- [full_text] Workflow card: drop low-library-size samples: [evidence span withheld — anchor status: not_found]
- [full_text] Retained counts under default QC: [evidence span withheld — anchor status: not_found]
- [readme] README: QC scope of the Data Processing module: "(3) quality controls (kingdom, library size, mean proportion, taxonomic name)"
- [discussion] Article: example data files deposited as phyloseq object: "in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’) and also as four individual files (see the files named ‘otu.tab.txt’, ‘sam.dat.txt’, ‘tax.tab.txt’, and ‘tree.tre’)"
