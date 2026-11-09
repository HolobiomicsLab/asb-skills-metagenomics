---
name: short-read-mapping-to-allele-database
description: Use when WGS read pairs must be aligned against known MLST allele sequences
  so per-read allele match/mismatch evidence (e.g., Phred-weighted mismatches) can
  be extracted as ILP input for strain deconvolution.
when_to_use_negative:
- Input is de novo assembled contigs or full genomes rather than short reads — reference-based
  strain methods like strainEST or genome-level comparison are more appropriate
- No reference allele database exists for the organism (de novo methods such as DESMAN
  are designed for that case)
- Reads do not come from MLST loci or cannot align to known allele sequences (the
  ILP stage has no allele-candidate space to work with)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Bowtie
  role: Short-read mapper used to align sample/simulated reads onto the 1726-allele
    B. burgdorferi MLST database (bowtie-build indexes alleles, bowtie maps reads)
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools
  role: Extracts per-read mapping information from the Bowtie SAM output, written
    per sample and gene as ILP input files
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: mapSamples.py
  role: Python driver that invokes bowtie, bowtie-build and samtools; exposes -c/-b/-s
    arguments and creates the variantsAndProp output structure
  repo: https://github.com/WGS-TB/MLST
- name: fastq-dump
  role: Downloads the SRA sample reads (v2.8.2) via download_samples.py to produce
    the FASTQ inputs to map
  repo: https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:01b591f159eb03f11bf55dd8bfea88610ddf2d5bf847a94d06cce40df9bbd3f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ART
  - Bowtie
  - Bowtie v0.12.7
  - GAML
  - Python 2.7
  - fastq-dump
  - mapSamples.py
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# short-read-mapping-to-allele-database

## Summary

Maps WGS read pairs onto a curated MLST allele database (the 8-gene B. burgdorferi scheme with 1726 known alleles) using Bowtie v0.12.7, then extracts per-locus mapping information from the SAM files with samtools v1.3.1 to feed the downstream Allele Diversity ILP.

## When to use

Use when WGS read pairs must be aligned against known MLST allele sequences so per-read allele match/mismatch evidence (e.g., Phred-weighted mismatches) can be extracted as ILP input for strain deconvolution.

## When NOT to use

- Input is de novo assembled contigs or full genomes rather than short reads — reference-based strain methods like strainEST or genome-level comparison are more appropriate
- No reference allele database exists for the organism (de novo methods such as DESMAN are designed for that case)
- Reads do not come from MLST loci or cannot align to known allele sequences (the ILP stage has no allele-candidate space to work with)

## Inputs

- Paired-end WGS FASTQ read pairs (2x76 bp) for tick samples, e.g., SRR2034336
- B. burgdorferi MLST allele database: 8 housekeeping genes (clpA, clpX, nifS, pepX, pyrG, recG, rplB, uvrA) totaling 1726 known alleles
- Simulated FASTQ reads from ART (for the ADP simulation path)

## Outputs

- SAM alignment files of reads against the allele database
- variantsAndProp folder with per-sample subfolders containing sampleX_geneY_paired_reads.txt files holding per-locus mapping information extracted from the SAM files (ILP input)

## How to apply

Build a Bowtie index of the allele database with bowtie-build, then map each sample's paired reads (e.g., 2x76 bp tick-sample reads) with Bowtie v0.12.7 using multiple cores if available. Extract mapping information from the resulting SAM files with samtools v1.3.1; this is automated by the mapSamples.py script, which accepts -c (number of Bowtie cores, default 4), -b (path to the bowtie/bowtie-build folder), and -s (path to samtools). The script writes a variantsAndProp folder containing one subfolder per sample, each holding per-locus files named sampleX_geneY_paired_reads.txt that contain the information needed by the first-stage ILP (alleleDiversity.py). The same mapping step is reused in the ADP simulation (run_sim.py) to align ART-simulated reads to the allele database, and downstream allele proportions are computed only from reads that do not map equally well to all selected alleles, so mapping quality and tie information must be preserved.

## Related tools

- **Bowtie** (Short-read mapper used to align sample/simulated reads onto the 1726-allele B. burgdorferi MLST database (bowtie-build indexes alleles, bowtie maps reads)) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools** (Extracts per-read mapping information from the Bowtie SAM output, written per sample and gene as ILP input files) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **mapSamples.py** (Python driver that invokes bowtie, bowtie-build and samtools; exposes -c/-b/-s arguments and creates the variantsAndProp output structure) — https://github.com/WGS-TB/MLST
- **fastq-dump** (Downloads the SRA sample reads (v2.8.2) via download_samples.py to produce the FASTQ inputs to map) — https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/

## Examples

```
python mapSamples.py -c 2 -b ../bowtieFolder/ -s ../../samtools
```

## Evaluation signals

- mapSamples.py creates a variantsAndProp folder with one subfolder per sample, each containing eight sampleX_geneY_paired_reads.txt files (one per MLST locus)
- The extracted per-read records must retain allele identity and Phred-based mismatch weights, since the downstream 0-1 ILP objective minimizes summed normalized Phred mismatch weights
- Running alleleDiversity.py on the mapped output should produce eight geneX_proportions.csv files per sample without ILP infeasibility, indicating the mapping evidence is complete
- On simulated reads (ART, coverages 30/100/300, edit distances 5-25), mapping-derived ILP solutions should yield average precision ~0.98 and recall ~0.96 per locus
- Cross-check that read pairs and locus count match the sample list in SRR_Acc_List.txt and the 8-gene scheme before proceeding to globalStrainDiversity.py

## Limitations

- Pinned legacy versions (Bowtie v0.12.7, samtools v1.3.1, Python 2.7) — newer mappers may change tie-handling among equally scoring alleles, which affects ILP optima and GAML tie-breaking
- The mapper only provides evidence against known alleles; reads from alleles absent from the 1726-allele database appear only as mismatches and novel-allele detection is deferred to the Strain Diversity stage
- Reads mapping equally well to all selected alleles are filtered out before proportion estimation, so highly conserved loci yield less usable proportional signal

## Evidence

- [readme] Bowtie v0.12.7 indexing/mapping role: [evidence span withheld — anchor status: not_found]
- [readme] mapSamples.py arguments and outputs: "The script will create a `variantsAndProp` folder containing a folder for each sample, where each of these folder contains a list of `sampleX_geneY_paired_reads.txt` text files"
- [methods] Article methods on mapping: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [methods] Database scale: "the traditional B. burgdorferi MLST scheme [11] composed of 8 housekeeping genes having a combined total of 1726 known alleles"
