---
name: variant-calling-low-frequency-variants
description: Use when calling variants from SARS-CoV-2 wastewater or other mixed viral
  populations where mutations occur at very low allele frequencies (<0.01) and a caller
  comparison or freebayes-based VCF is needed.
when_to_use_negative:
- The input is clinical consensus-level sequencing where standard Pangolin/consensus
  pipelines are appropriate rather than low-frequency (<0.01) variant calling
- Primer trimming and remapping were skipped (e.g., the four Thessaloniki samples
  between 19 March and 15 April 2021), making AF comparisons across samples inconsistent
- The goal is SNP calling at high fixed frequencies only — mpileup reporting every
  position or Mutect2-style calling may suffice and freebayes's sensitivity to sub-1%
  variants is unnecessary
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: freebayes
  role: Primary variant caller invoked with a low mutation frequency parameter of
    0.01 to capture variants below 1% frequency; judged the most productive and informative
    of three assessed callers
- name: Minimap2
  role: Maps cleaned paired-end reads to the SARS-CoV-2 Wuhan reference NC_045512
    with a minimal chaining score of 40
- name: GATK Mutect2
  role: Comparison caller run in cancer-only mode with default parameters on the same
    alignments
- name: mpileup
  role: Comparison caller configured to report every position
- name: lineagespot
  role: R pipeline run on the freebayes VCF output to identify SARS-CoV-2 lineage-characteristic
    mutations
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:7bb1c48a35aeb80256292ca3393bbcf4f702da8cf63c0cd36e234b8f1e49c34b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - GATK Mutect2
  - Minimap2
  - R
  - freebayes
  - lineagespot
  - mpileup
  - outbreak.info
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# variant-calling-low-frequency-variants

## Summary

Reproduces the paper's low-frequency SARS-CoV-2 variant calling workflow for wastewater NGS: map cleaned reads to NC_045512 with Minimap2 (minimal chaining score 40) and call variants with freebayes at a low mutation frequency of 0.01, the setup the authors judged most productive and informative for detecting mutations below 0.01 frequency.

## When to use

Use when calling variants from SARS-CoV-2 wastewater or other mixed viral populations where mutations occur at very low allele frequencies (<0.01) and a caller comparison or freebayes-based VCF is needed.

## When NOT to use

- The input is clinical consensus-level sequencing where standard Pangolin/consensus pipelines are appropriate rather than low-frequency (<0.01) variant calling
- Primer trimming and remapping were skipped (e.g., the four Thessaloniki samples between 19 March and 15 April 2021), making AF comparisons across samples inconsistent
- The goal is SNP calling at high fixed frequencies only — mpileup reporting every position or Mutect2-style calling may suffice and freebayes's sensitivity to sub-1% variants is unnecessary

## Inputs

- Paired-end FASTQ reads of wastewater NGS samples (e.g., ENA study PRJEB44141 / ERP128154)
- SARS-CoV-2 Wuhan reference genome NC_045512 (FASTA)
- Reference annotation for variant effect annotation (SnpEff NC_045512.2 version 5.0 database)

## Outputs

- Per-sample VCF files of low-frequency variants (freebayes at 0.01)
- Combined table of detected nucleotide mutations with amino acid substitutions, allele frequency and overlapping gene
- Lineage-characteristic mutation assignments per lineage (via lineagespot)

## How to apply

Retrieve FASTQ reads (e.g., from ENA study PRJEB44141 / ERP128154) and map cleaned reads to the SARS-CoV-2 Wuhan reference NC_045512 with Minimap2 using a minimal chaining score (matching bases minus log gap penalty) of 40, following the Illumina ARTIC pipeline variation; retain only paired-end sequences and reads of at least 200 nt after trimming with a sliding-window quality threshold of 15. Call variants with freebayes using a low mutation frequency parameter of 0.01 so that variants below 1% frequency are captured; the paper compared this against GATK Mutect2 (cancer-only mode, default parameters) and mpileup (reporting every position) and concluded freebayes was the most productive and informative option. Annotate the resulting VCFs (e.g., SnpEff with the NC_045512.2 version 5.0 database) and merge them into a combined table of nucleotide mutations, amino acid substitutions, allele frequency and overlapping gene, collapsing at the gene/amino-acid level to reduce noise. Downstream, run lineagespot on the VCFs to detect lineage-characteristic mutations (Pangolin rules / outbreak.info definitions). Sensitivity to the low-frequency parameter can be assessed by generating multiple VCF pairs across varying low mutation frequency values.

## Related tools

- **freebayes** (Primary variant caller invoked with a low mutation frequency parameter of 0.01 to capture variants below 1% frequency; judged the most productive and informative of three assessed callers)
- **Minimap2** (Maps cleaned paired-end reads to the SARS-CoV-2 Wuhan reference NC_045512 with a minimal chaining score of 40)
- **GATK Mutect2** (Comparison caller run in cancer-only mode with default parameters on the same alignments)
- **mpileup** (Comparison caller configured to report every position)
- **lineagespot** (R pipeline run on the freebayes VCF output to identify SARS-CoV-2 lineage-characteristic mutations) — https://github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
freebayes -f NC_045512.fa --pooled-continuous -F 0.01 sample.bam > sample.vcf
```

## Evaluation signals

- Reproduced Nt counts per lineage (B.1.177, B.1.1.7, B.1.351) from each caller match the paper's comparison table, supporting freebayes at 0.01 as the most productive caller
- Computed non-zero minimum allele frequency of B.1.1.7-unique mutations matches reported Table 5 values (e.g., 16.67% for 5–11 Feb 2021; 76.71% for 5–11 Mar 2021) within numerical tolerance
- VCF positions with reference read depth equal to zero are removed before downstream AF aggregation
- All called variants carry annotation (amino acid substitution, overlapping gene) after merging into the combined mutation table
- VCF pairs generated across varying low mutation frequency parameters show the expected change in number of mutations as the threshold increases

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were not applied for the four samples between 19 March 2021 and 15 April 2021
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- No software versions reported for Minimap2, iVar, freebayes, or SnpEff, hindering exact reproduction

## Evidence

- [results] three different variant callers were assessed: (1) freebayes14, (2) mpileup15 and (3) GATK Mutect216 (cancer only mode): "three different variant callers were assessed: (1) freebayes14, (2) mpileup15 and (3) GATK Mutect216 (cancer only mode)"
- [results] freebayes as the most productive and informative approach for low-frequency variants: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] Minimap2 mapping with minimal chaining score 40 to NC_045512: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [discussion] Read filtering: paired-end only, min 200 nt after trimming, sliding window quality 15: "a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality"
- [intro] Illumina ARTIC pipeline variation for mutations below 0.01 frequency: "Based on a variation of the Illumina ARTIC pipeline for the identification of mutations at low frequencies (< 0.01)"
- [readme] lineagespot framework for variant-based SARS-CoV-2 mutation identification: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling format]"
