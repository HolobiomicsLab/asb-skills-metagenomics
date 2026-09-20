---
name: short-read-alignment-parsing
description: Use when WGS FASTQ reads must be aligned to a reference set of known
  MLST allele sequences and converted into a per-read, per-allele mapping-score table
  to feed downstream abundance-estimation ILP solvers.
when_to_use_negative:
- The input is already parsed `sampleX_geneY_paired_reads.txt` files ready for `alleleDiversity.py`
  — mapping and parsing should be skipped
- Reads are simulated alleles rather than mapped WGS reads; the ART simulation path
  (run_sim.py) handles its own alignment and output locations
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Bowtie
  role: Short-read mapper aligning sample WGS reads onto the known MLST allele database
    (v0.12.7, invoked via mapSamples.py)
  repo: https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- name: samtools
  role: Extracts alignment information from the Bowtie .sam output files during parsing
    (v1.3.1, invoked via mapSamples.py)
  repo: https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- name: fastq-dump
  role: Downloads the sample reads from SRA before mapping (v2.8.2, invoked via download_samples.py)
  repo: https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3204-8
    title: Deconvoluting the diversity of within-host pathogen strains in a multi-locus
      sequence typing framework
  source_content_hash: sha256:e07610175c5784df7bb5240648908bee6ede2769cd8400c7e5650e049bb0088d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Bowtie v0.12.7
  - GAML
  - Python 2.7
  - fastq-dump
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# short-read-alignment-parsing

## Summary

Map WGS sample reads onto a database of known MLST allele sequences with Bowtie v0.12.7, then parse the resulting .sam files with samtools into per-sample, per-gene paired-read text files that encode the weighted read-allele incidence matrix needed for the first-stage Allele Diversity ILP.

## When to use

Use when WGS FASTQ reads must be aligned to a reference set of known MLST allele sequences and converted into a per-read, per-allele mapping-score table to feed downstream abundance-estimation ILP solvers.

## When NOT to use

- The input is already parsed `sampleX_geneY_paired_reads.txt` files ready for `alleleDiversity.py` — mapping and parsing should be skipped
- Reads are simulated alleles rather than mapped WGS reads; the ART simulation path (run_sim.py) handles its own alignment and output locations

## Inputs

- Sample WGS reads (FASTQ, downloaded from SRA via fastq-dump, e.g. SRR_Acc_List.txt samples)
- Database of known MLST allele sequences (B. burgdorferi scheme: 8 housekeeping genes, 1726 alleles)
- Bowtie index built from the allele database via bowtie-build

## Outputs

- Per-sample .sam alignment files from Bowtie
- Per-sample, per-gene `sampleX_geneY_paired_reads.txt` files in the `variantsAndProp` folder containing read-allele mapping information extracted from the .sam files
- A `variantsAndProp` folder organized into one subfolder per sample

## How to apply

First download sample reads with fastq-dump (v2.8.2 used) and build a Bowtie index of the known allele database using bowtie-build (for B. burgdorferi MLST this is 8 housekeeping genes with 1726 known alleles). Run Bowtie v0.12.7 to map each sample's short reads onto the allele database, retaining closely matched reads. Then call samtools v1.3.1 from mapSamples.py to extract alignment information from each .sam file. The parsing step writes, for every sample and every gene, a `sampleX_geneY_paired_reads.txt` file into the `variantsAndProp` folder containing exactly the information the first-stage ILP needs: the normalized Phred-score-based mapping costs (scores normalized over the 33-126 range, with non-mapping reads set to infinity). These parsed files are the direct inputs to `alleleDiversity.py`, which builds the weighted bipartite read-allele incidence matrix M before formulating the 0-1 ILP.

## Related tools

- **Bowtie** (Short-read mapper aligning sample WGS reads onto the known MLST allele database (v0.12.7, invoked via mapSamples.py)) — https://sourceforge.net/projects/bowtie-bio/files/bowtie/0.12.7/
- **samtools** (Extracts alignment information from the Bowtie .sam output files during parsing (v1.3.1, invoked via mapSamples.py)) — https://sourceforge.net/projects/samtools/files/samtools/1.3.1/
- **fastq-dump** (Downloads the sample reads from SRA before mapping (v2.8.2, invoked via download_samples.py)) — https://www.ncbi.nlm.nih.gov/sra/docs/toolkitsoft/

## Examples

```
python mapSamples.py -c 2 -b ../bowtieFolder/ -s ../../samtools
```

## Evaluation signals

- mapSamples.py completes and creates a `variantsAndProp` folder containing one subfolder per sample
- Each sample subfolder contains `sampleX_geneY_paired_reads.txt` text files for all 8 MLST loci
- The parsed files are consumable by `alleleDiversity.py`, which then runs to completion producing 8 `geneX_proportions.csv` files per sample
- Downstream ILP sizing matches reference workload (sample SRR2034336 averages ~20,112 variables per gene, ~1.5GB peak RAM)
- Path arguments for bowtie, bowtie-build, and samtools resolve correctly (e.g. `python mapSamples.py -c 2 -b ../bowtieFolder/ -s ../../samtools`)

## Limitations

- The pipeline assumes specific executable locations; bowtie, bowtie-build, and samtools must be in the user's bin folder or passed explicitly via -b and -s arguments to mapSamples.py
- Bowtie v0.12.7 is an older short-read mapper; alignment quality and mapping sensitivity are tied to reads closely matching known alleles, so divergent or novel alleles may be missed
- Non-mapping reads are set to an infinite mapping cost, so reads that do not align to any known allele carry no information into the ILP

## Evidence

- [methods] Bowtie aligns reads and samtools extracts mapping info: "To align the simulated reads to the alleles of the database, we used Bowtie v0.12.7 [19]."
- [readme] README describes variantsAndProp outputs: "The script will create a `variantsAndProp` folder containing a folder for each sample, where each of these folder contains a list of `sampleX_geneY_paired_reads.txt`"
- [readme] README explains sam parsing role: "The python script calls the `samtools` command while extracting information from sam files after mapping."
- [readme] README: parsed files feed the first-stage ILP: "in which the text file contains information needed for our first stage ILP pipeline"
- [methods] Article: reads are mapped onto known allele database: "WGS data for a set of samples that are mapped using a short-read mapper of choice onto the database of known alleles"
