---
name: twin-cohort-microbiome-reanalysis
description: Use when re-analyzing the 280-sample twin/mother gut V1-V2 amplicon cohort
  (samples with >=1,000 reads) to test whether GCN correction changes enterotype classification
  or phylum-level profiles.
when_to_use_negative:
- Input profiles are already GCN-corrected or expressed as percentages rather than
  raw read counts (CopyRighter requires counts and unaltered taxonomic assignments)
- The OTU table uses a taxonomy other than Greengenes (e.g., Silva), since CopyRighter's
  shipped GCN estimates cover only the Oct 2012 Greengenes taxonomy
- Metagenomic (not amplicon) profiles, unless correcting genome length bias rather
  than 16S rRNA GCN
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CopyRighter
  role: Corrects 16S rRNA gene copy number bias by weighting each species' read counts
    by the inverse of its lineage-estimated GCN, producing corrected relative abundances
    for the twin cohort
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl >= 5.10.0
  role: Runtime environment required to install and run CopyRighter
- name: Bio::Community
  role: Perl dependency of CopyRighter; bc_use_repr_id can replace arbitrary OTU IDs
    by Greengenes IDs for trait lookup
- name: make
  role: Build tool for installing CopyRighter from source (perl Makefile.PL, make,
    make install)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:ca04fb2e291dbfb80843add6016a7b75ff556817880e4a6192b65f0a22310235
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  - make
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# twin-cohort-microbiome-reanalysis

## Summary

Re-analysis protocol for the human gut twin-cohort 16S rRNA amplicon dataset in which CopyRighter's phylogenetic-level 16S rRNA gene copy number (GCN) correction shifts phylum-level abundances, changes the Berger-Parker dominance index, and reclassifies enterotypes from two into three community types.

## When to use

Use when re-analyzing the 280-sample twin/mother gut V1-V2 amplicon cohort (samples with >=1,000 reads) to test whether GCN correction changes enterotype classification or phylum-level profiles.

## When NOT to use

- Input profiles are already GCN-corrected or expressed as percentages rather than raw read counts (CopyRighter requires counts and unaltered taxonomic assignments)
- The OTU table uses a taxonomy other than Greengenes (e.g., Silva), since CopyRighter's shipped GCN estimates cover only the Oct 2012 Greengenes taxonomy
- Metagenomic (not amplicon) profiles, unless correcting genome length bias rather than 16S rRNA GCN

## Inputs

- Twin/mother gut microbiome V1-V2 amplicon OTU table (153 individuals, 280 samples with >=1,000 reads) in QIIME, biom, generic tabular site-by-species, GAAS or Unifrac format with read counts (not percentages) and unaltered Greengenes taxonomic strings
- CopyRighter-shipped phylogenetically-based GCN estimate files for Greengenes OTUs and taxa (tab-delimited ID or taxstring vs 16S rRNA count)

## Outputs

- GCN-corrected per-sample relative abundance profiles (site-by-species file, abundances in percent)
- Phylum-level aggregation comparing median Bacteroidetes and Firmicutes relative abundances before and after correction
- Per-sample Berger-Parker dominance index before and after correction with difference distribution (0-23.3%)
- Genus-level enterotype assignments (PAM clustering) for uncorrected and corrected profiles with reclassification fraction

## How to apply

Install CopyRighter and its dependencies (Perl >= 5.10.0, make, Bio::Community) via 'perl Makefile.PL', 'make', and 'make install', or use local::lib without administrator privileges. Load the twin cohort V1-V2 OTU table (153 individuals, 280 samples with >=1,000 reads; read counts, unaltered Greengenes taxonomic strings) and the phylogenetically-based GCN estimate files for Greengenes OTUs and taxa shipped with CopyRighter. Run copyrighter on every sample using phylogenetic-level rather than taxonomic-level correction, weighting read counts by the inverse lineage-specific GCN (use -l id to match Greengenes OTU IDs, -v to verify trait assignment). Aggregate corrected and uncorrected profiles to phylum level to compare median Bacteroidetes (31.1% -> 21.7%) and Firmicutes (67.1% -> 76.0%), compute per-sample Berger-Parker dominance differences (expected 0-23.3%), and re-run PAM-style genus-level enterotype clustering to verify three rather than two enterotypes with 23.9% of samples misclassified without correction.

## Related tools

- **CopyRighter** (Corrects 16S rRNA gene copy number bias by weighting each species' read counts by the inverse of its lineage-estimated GCN, producing corrected relative abundances for the twin cohort) — github.com/fangly/AmpliCopyrighter
- **Perl >= 5.10.0** (Runtime environment required to install and run CopyRighter)
- **Bio::Community** (Perl dependency of CopyRighter; bc_use_repr_id can replace arbitrary OTU IDs by Greengenes IDs for trait lookup)
- **make** (Build tool for installing CopyRighter from source (perl Makefile.PL, make, make install))

## Examples

```
copyrighter -i twin_gut_otu_table.qiime -l id -v -o twin_gut_copyrighted.generic
```

## Evaluation signals

- Corrected phylum-level medians reproduce Bacteroidetes 21.7% and Firmicutes 76.0% (from 31.1% and 67.1% uncorrected) within reasonable tolerance
- Per-sample Berger-Parker dominance index differences before vs after correction fall within the reported 0-23.3% range
- Enterotype clustering of corrected profiles yields three rather than two enterotypes, with ~23.9% of samples misclassified without correction
- Verbose output (-v) confirms each taxon received the intended phylogenetic-level trait value assignment; input taxa have unaltered Greengenes strings with whitespace after each semicolon
- Output file expresses relative abundance in percent and preserves the input community format

## Limitations

- CopyRighter's shipped GCN estimates cover only the Greengenes taxonomy (IMG 4.0 genomes, Oct 2012); other taxonomies such as Silva are only anticipated future extensions
- GCN estimates for uncultured organisms are inferred from close sequenced relatives, so trait assignment can be uncertain; correction effects are non-trivial to predict for communities with tens to thousands of species
- CopyRighter currently corrects only 16S rRNA GCN (and genome length for metagenomes); GC percent bias, genome length bias beyond metagenomes, and fungal ITS variation would need appropriate pre-computed estimates

## Evidence

- [abstract] Twin cohort enterotype reclassification: "human gut microbiomes from twins were reclassified into three rather than two enterotypes after GCN correction"
- [discussion] Recommended systematic phylogenetic-level correction: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [readme] Inverse GCN weighting rationale: "CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [readme] Input format and count requirement: [evidence span withheld — anchor status: external_doc]
- [methods] Installation steps: "perl Makefile.PL
     make"
- [supplementary] Supplementary twin cohort enterotype figure: "Enterotype classification of human gut microbiomes of a twin cohort at the genus level."
