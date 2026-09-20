---
name: nucleotide-diversity-birth-mode-comparison
description: Use when you have coReads .bam files mapped to representative sub-species
  reference genomes and need to compare per-genome nucleotide diversity (microdiversity)
  across metadata groups such as birth mode.
when_to_use_negative:
- Reads are not yet mapped to sub-species reference genomes — no .bam input exists
  for inStrain profile
- A sub-species falls below the >= 0.5 unmaskedBreadth detection threshold or is flagged
  'likely contaminant' (negative control or campaign-enriched), so its diversity estimate
  is not trustworthy
- The question is strain sharing between samples rather than within-sample microdiversity
  — use inStrain compare (ConANI/popANI) instead
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: inStrain
  role: Profiles .bam files to compute per-scaffold and genome-level nucleotide diversity
    and SNVs; genome_wide aggregates scaffold results
  repo: github.com/MrOlm/instrain
- name: Samtools
  role: Underlying read interaction with .bam/.sam files for coverage and read filtering
    inside the inStrain workflow
- name: Pandas
  role: Filter detected sub-species by unmaskedBreadth >= 0.5, exclude likely contaminants,
    and group diversity values by birth mode
- name: SciPy
  role: Statistical test of the nucleotide diversity difference between cesarean-
    and vaginally-born infants
- name: Matplotlib
  role: Render the Klebsiella diversity-by-birth-mode comparison figure
- name: Seaborn
  role: Render the Klebsiella diversity-by-birth-mode comparison figure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:a7ac71a31f35298883b92cf5a5af7920b3d24861a1b01e406e1da621e49619fa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Matplotlib
  - Numpy
  - Pandas
  - Samtools
  - SciPy
  - Seaborn
  - inStrain
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# nucleotide-diversity-birth-mode-comparison

## Summary

Profiles SNV-based nucleotide diversity of sub-species populations (e.g., Klebsiella) from metagenomic read mappings with inStrain, then tests whether microdiversity differs by metadata group — here, birth mode (cesarean vs. vaginal) in premature infants.

## When to use

Use when you have coReads .bam files mapped to representative sub-species reference genomes and need to compare per-genome nucleotide diversity (microdiversity) across metadata groups such as birth mode.

## When NOT to use

- Reads are not yet mapped to sub-species reference genomes — no .bam input exists for inStrain profile
- A sub-species falls below the >= 0.5 unmaskedBreadth detection threshold or is flagged 'likely contaminant' (negative control or campaign-enriched), so its diversity estimate is not trustworthy
- The question is strain sharing between samples rather than within-sample microdiversity — use inStrain compare (ConANI/popANI) instead

## Inputs

- coReads .bam files (paired reads mapped with Bowtie 2 to concatenated representative sub-species .fasta)
- Concatenated representative sub-species .fasta reference file
- Scaffold-to-bin mapping table or list of individual .fasta files for inStrain genome_wide
- Metadata table with infant birth mode (cesarean vs. vaginal) and sampling campaign labels
- Negative extraction control sample profile for contaminant flagging

## Outputs

- inStrain profile folders with per-scaffold coverage, clonality, and SNV-based nucleotide diversity
- Genome-level (sub-species-level) microdiversity table filtered to unmaskedBreadth >= 0.5 and de-contaminated
- Birth-mode comparison plot of Klebsiella nucleotide diversity (matplotlib/seaborn) with SciPy test result

## How to apply

Run `inStrain profile` on each coReads .bam against the concatenated representative sub-species .fasta with default read filters (min read-pair ANI 0.95, min 5 reads per SNV position, >5% variant base frequency, SNP FDR 1e-6, un-paired reads removed by default). Aggregate scaffold-level results into genome-level metrics with `inStrain genome_wide` using a scaffold-to-bin table. In pandas, keep only sub-species detected at >= 0.5 unmaskedBreadth and exclude those detected in the negative extraction control or enriched in a single sampling campaign (e.g., Sloan2 or NIH4) as 'likely contaminant'. Extract nucleotide diversity values for the Klebsiella sub-species, group by birth mode (cesarean vs. vaginal), test with a SciPy statistical test, and plot with matplotlib/seaborn. Correct judgment requires reproducing the direction of the reported result: higher Klebsiella nucleotide diversity in C-section-born infants (Figure 4 / Supplemental Figure S6).

## Related tools

- **inStrain** (Profiles .bam files to compute per-scaffold and genome-level nucleotide diversity and SNVs; genome_wide aggregates scaffold results) — github.com/MrOlm/instrain
- **Samtools** (Underlying read interaction with .bam/.sam files for coverage and read filtering inside the inStrain workflow)
- **Pandas** (Filter detected sub-species by unmaskedBreadth >= 0.5, exclude likely contaminants, and group diversity values by birth mode)
- **SciPy** (Statistical test of the nucleotide diversity difference between cesarean- and vaginally-born infants)
- **Matplotlib** (Render the Klebsiella diversity-by-birth-mode comparison figure)
- **Seaborn** (Render the Klebsiella diversity-by-birth-mode comparison figure)

## Examples

```
inStrain profile coReads.bam representatives.fasta -o inStrain_profile1 && inStrain genome_wide -i inStrain_profile1 -s scaffold_to_bin.tsv
```

## Evaluation signals

- Resulting figure shows Klebsiella with higher nucleotide diversity in C-section-born infants than in vaginally born infants, reproducing the Figure 4 / Supplemental Figure S6 result
- Every sub-species included in the test has unmaskedBreadth >= 0.5 in the sample
- inStrain profile used default SNV filters: minimum 5 reads per position, >5% variant base frequency, and SNP FDR 1e-6
- Sub-species detected in the negative extraction control or enriched in one sampling campaign (e.g., Sloan2, NIH4) are excluded before testing
- The association remains significant when samples from individual campaigns (e.g., Sloan2 and NIH4) are excluded

## Limitations

- Variant sites near the ends of genome fragments can be missed with inStrain
- Genome de-replication at 98% ANI is needed to reduce multi-mapped reads, otherwise diversity estimates can be distorted
- It is difficult to distinguish microbiome diversity that evolved in situ from diversity introduced by immigration, so the birth-mode difference may reflect acquisition rather than within-infant evolution

## Evidence
