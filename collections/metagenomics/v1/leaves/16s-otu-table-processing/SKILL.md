---
name: 16s-otu-table-processing
description: Use when marker-gene (16S) abundance data must be converted into a PICRUSt-compatible
  closed-reference OTU table as the first step of metagenome prediction, NSTI calculation,
  or accuracy-vs-NSTI analysis.
when_to_use_negative:
- The data are shotgun metagenome reads rather than 16S marker gene sequences — direct
  functional profiling (e.g., HUMAnN annotation to KOs) is appropriate instead of
  16S-based prediction
- Open-reference or de novo OTU picking is desired — unclassified OTUs without reference
  tree tips cannot be normalized or predicted by PICRUSt's ancestral state reconstruction
- The community contains viral or eukaryotic components whose function should be inferred
  — only bacterial and archaeal 16S sequences are included in this version
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: QIIME 1.5.0-dev
  role: Executes closed-reference OTU picking (pick_reference_otus_through_otu_table.py)
    to produce PICRUSt-compatible OTU tables
- name: uclust
  role: Sequence clustering/search engine used for closed-reference OTU picking against
    Greengenes+IMG
- name: PICRUSt
  role: Consumes the closed-reference OTU table for 16S copy-number normalization,
    NSTI calculation, and metagenome prediction
  repo: http://picrust.github.com
- name: tax2tree
  role: Provided the curated Greengenes 16S reference tree used for all presented
    research
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:5b319cb6a1722099b4b4cfef57de02a4d85ef14ce400de22fe5ccb695a4b9ddb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PICRUSt
  - QIIME 1.5.0-dev
  - tax2tree
  - uclust
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# 16s-otu-table-processing

## Summary

Constructs closed-reference 16S OTU tables (e.g., with QIIME 1.5.0-dev pick_reference_otus_through_otu_table.py using uclust against Greengenes+IMG) in the format required by PICRUSt so that OTU abundances can be normalized by 16S copy number and used for metagenome prediction and NSTI-based accuracy assessment.

## When to use

Use when marker-gene (16S) abundance data must be converted into a PICRUSt-compatible closed-reference OTU table as the first step of metagenome prediction, NSTI calculation, or accuracy-vs-NSTI analysis.

## When NOT to use

- The data are shotgun metagenome reads rather than 16S marker gene sequences — direct functional profiling (e.g., HUMAnN annotation to KOs) is appropriate instead of 16S-based prediction
- Open-reference or de novo OTU picking is desired — unclassified OTUs without reference tree tips cannot be normalized or predicted by PICRUSt's ancestral state reconstruction
- The community contains viral or eukaryotic components whose function should be inferred — only bacterial and archaeal 16S sequences are included in this version

## Inputs

- Raw 16S amplicon data or pre-clustered OTU tables from microbial community samples (e.g., mammal gut, soil, hypersaline mat validation communities)
- Greengenes 16S reference collection (tax2tree version) combined with IMG whole-genome reference data
- 16S marker gene phylogenetic tree containing tips with known (reference genome) and unknown data

## Outputs

- Closed-reference OTU table (PICRUSt-compatible format) mapping OTUs to Greengenes+IMG reference tree tips
- Per-sample NSTI values derived from the OTU table as input to accuracy assessment
- Per-sample metagenome (KO abundance) predictions ready for comparison to sequenced metagenomes

## How to apply

Run closed-reference OTU picking rather than de novo: use QIIME 1.5.0-dev's pick_reference_otus_through_otu_table.py with uclust against the Greengenes+IMG reference (the tax2tree-curated version of Greengenes was used for the presented research). Closed-reference picking is required because PICRUSt needs every OTU mapped to a tip on the 16S marker gene tree so that 16S copy-number normalization, ancestral state reconstruction (e.g., Wagner Parsimony via the COUNT package or APE ACE methods), and exponential distance-weighted gene content prediction can be applied. The resulting OTU table is the input for computing per-sample NSTI (abundance-weighted phylogenetic distance of each OTU to its nearest sequenced reference genome, summed over OTUs to a single value per sample). In the validation workflow, OTU tables were built this way for mammal gut, soil, and hypersaline mat communities so predicted KO abundances could be compared to sequenced metagenomes. Verify the table is PICRUSt-compatible (each OTU maps to a reference tree tip) before proceeding to prediction.

## Related tools

- **QIIME 1.5.0-dev** (Executes closed-reference OTU picking (pick_reference_otus_through_otu_table.py) to produce PICRUSt-compatible OTU tables)
- **uclust** (Sequence clustering/search engine used for closed-reference OTU picking against Greengenes+IMG)
- **PICRUSt** (Consumes the closed-reference OTU table for 16S copy-number normalization, NSTI calculation, and metagenome prediction) — http://picrust.github.com
- **tax2tree** (Provided the curated Greengenes 16S reference tree used for all presented research)

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r greengenes_img_16S.fna -t gg_img_16S.tre -m uclust -o picrust_otus
```

## Evaluation signals

- Every OTU in the output table maps to a reference tip on the Greengenes+IMG 16S tree (closed-reference property) — unclassified OTUs would break downstream NSTI and prediction steps
- Per-sample NSTI values computed from the OTU table fall within the range expected for the habitat (e.g., mammal gut samples restricted to NSTI<0.05 in the validation analysis)
- Spearman correlation between per-sample NSTI and PICRUSt prediction accuracy should show the expected decay (global r=-0.4, p<0.001; per-habitat r=-0.25 to -0.82, p<0.05)
- Predicted KO abundances from the normalized OTU table should recover accuracy comparable to sequenced metagenomes when compared per sample

## Limitations

- Only 16S marker gene sequences corresponding to bacterial and archaeal genomes are included; viral or eukaryotic metagenome components are not inferred
- PICRUSt cannot distinguish strain-level variation if the marker gene sequence is identical among strains
- Some gene gain/loss or laterally transferred genes will be poorly predicted (with broad confidence intervals), so accuracy decreases as NSTI increases
- The reference database has phylogenetic gaps for some environments; additional genome sequences are needed for accurate prediction in poorly covered clades

## Evidence

- [discussion] PICRUSt-compatible OTU tables were constructed using the closed-reference OTU picking protocol in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py): "PICRUSt-compatible OTU tables were constructed using the closed-reference OTU picking protocol in QIIME 1.5.0-dev (pick_reference_otus_through_otu_table.py)"
- [discussion] against Greengenes+IMG using 'uclust': [evidence span withheld — anchor status: not_found]
- [discussion] the 16S 'tax2tree' version of Greengenes45 was downloaded and used for all presented research: [evidence span withheld — anchor status: not_found]
