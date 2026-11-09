---
name: cds-coordinate-parsing
description: Use when assembling a gene catalog from metagenomic contigs and/or isolate
  genomes and you need per-gene CDS coordinates, lengths, and source provenance after
  running a gene caller such as MetaGeneMark.
when_to_use_negative:
- Input is already an annotated gene catalog or feature table (e.g., existing 0.geneLength.txt
  or GFF) — no gene prediction or coordinate parsing is needed
- Working with metatranscriptomic reads to be mapped against VIRGO — use the runMapping
  modules instead of re-predicting CDSs
- Sequences are non-bacterial (viral/fungal) targets where VIRGO's catalog has limited
  inclusion and MetaGeneMark bacterial gene models may not apply
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaGeneMark
  role: Gene-prediction tool (v3.25, default parameters) run on metagenomic contigs
    and isolate genomes to predict CDSs whose coordinates and lengths are then parsed
- name: VIRGO
  role: Gene catalog framework that stores the parsed gene lengths as 1_VIRGO/0.geneLength.txt
    and uses gene length as column 3 of per-sample mapping outputs
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:824cb5f894729587174bf9dcd42c3071b82b72f674ada8f4a505eb66aad4fdfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaGeneMark
  - VIRGO
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cds-coordinate-parsing

## Summary

Parses gene-prediction output (e.g., MetaGeneMark) to extract predicted CDS coordinates and compute per-gene lengths in bp, attributing each CDS to its source assembly — the step used to build the VIRGO vaginal gene catalog's gene-length annotation table.

## When to use

Use when assembling a gene catalog from metagenomic contigs and/or isolate genomes and you need per-gene CDS coordinates, lengths, and source provenance after running a gene caller such as MetaGeneMark.

## When NOT to use

- Input is already an annotated gene catalog or feature table (e.g., existing 0.geneLength.txt or GFF) — no gene prediction or coordinate parsing is needed
- Working with metatranscriptomic reads to be mapped against VIRGO — use the runMapping modules instead of re-predicting CDSs
- Sequences are non-bacterial (viral/fungal) targets where VIRGO's catalog has limited inclusion and MetaGeneMark bacterial gene models may not apply

## Inputs

- Assembled vaginal metagenome contigs (FASTA)
- Urogenital bacteria isolate genome sequence assemblies (FASTA)
- MetaGeneMark v3.25 gene-prediction output files (CDS coordinates per contig)

## Outputs

- Per-gene length annotation table 0.geneLength.txt (VIRGO gene ID, length in bp)
- CDS-to-source attribution tally (metagenomic assemblies vs. isolate genomes)
- Total predicted CDS count per source (e.g., 5,509,298 total; ~80% metagenomic, ~20% isolate)

## How to apply

Run MetaGeneMark (version 3.25, gene-prediction mode, default parameters) separately on each input assembly — metagenomic contigs and urogenital isolate genome sequences — then parse the prediction output to extract CDS coordinates and compute each predicted gene's length in bp. Tag every CDS with its assembly source (metagenomic assembly vs. isolate genome) so counts can be tallied per source. Write the per-gene length annotation as a tabular file named 0.geneLength.txt, matching the VIRGO database structure where 1_VIRGO/0.geneLength contains gene length in bp. Validate the result by confirming the total predicted CDS count (~5,509,298 for VIRGO) and the source split (~80% metagenomic assemblies, ~20% isolate genomes). These lengths feed downstream mapping outputs where column 3 of each sample's .out file is the gene length.

## Related tools

- **MetaGeneMark** (Gene-prediction tool (v3.25, default parameters) run on metagenomic contigs and isolate genomes to predict CDSs whose coordinates and lengths are then parsed)
- **VIRGO** (Gene catalog framework that stores the parsed gene lengths as 1_VIRGO/0.geneLength.txt and uses gene length as column 3 of per-sample mapping outputs) — github.com/ravel-lab/VIRGO

## Examples

```
gmetapath -m /path/to/metageneemark/MetaGeneMark_v3.25/gmhmm.mod -A -f GFF -o predictions.gff assembled_contigs.fasta; awk '{len=$3-$2+1; print $1"\t"len}' predictions.gff > 0.geneLength.txt
```

## Evaluation signals

- Total predicted CDS count matches ~5,509,298 for the VIRGO catalog build
- Source split of predicted CDSs is roughly 80% from metagenomic assemblies and ~20% from isolate genome sequences
- Every parsed CDS carries both coordinates and a computed length in bp, and a source label (metagenomic vs. isolate)
- 0.geneLength.txt row count equals the number of predicted CDSs, and gene lengths are positive integers consistent with CDS coordinate spans
- Gene lengths in 0.geneLength.txt agree with column 3 of downstream per-sample mapping .out files (e.g., V1593031 → 3663)

## Limitations

- MetaGeneMark prediction with default parameters may miss or mis-bound genes on short or fragmented metagenomic contigs
- VIRGO's catalog is primarily bacterial, with limited inclusion of viral and fungal gene sequences, so CDS predictions for non-bacterial contigs are underrepresented
- The reference-based catalog contains non-redundant genes not observed in isolate genomes for some species, reflecting limited isolate genome availability or systematic bias in growth and recovery of vaginal species

## Evidence

- [methods] CDS prediction was performed on the metagenomic assemblies and genome sequences using MetaGeneMark, yielding 5,509,298 predicted CDSs: [evidence span withheld — anchor status: not_found]
- [readme] 0.geneLength contains gene length in bp: "- ***0.geneLength*** contains gene length in bp"
- [readme] The 1st column is the VIRGO gene ID, 2nd column is the number of reads mapped onto the VIRGO database, the 3rd column is the gene length: "The 1st column is the VIRGO gene ID, 2nd column is the number of reads mapped onto the VIRGO database, the 3rd column is the gene length."
- [methods] Run MetaGeneMark version 3.25 in gene-prediction mode with default parameters on each set of contigs to predict CDSs: [evidence span withheld — anchor status: not_found]
