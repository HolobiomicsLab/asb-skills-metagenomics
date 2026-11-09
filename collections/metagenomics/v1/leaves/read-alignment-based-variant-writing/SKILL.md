---
name: read-alignment-based-variant-writing
description: Use when simulating within-strain short-term evolutionary mutations in
  longitudinal (multi-time-point) simulated raw sequencing reads and a ground-truth
  mutation record (VCF) is needed for benchmarking.
when_to_use_negative:
- Input is real metagenomic data rather than simulated reads — the README warns STEMSim
  only works well on species with high depth and well-characterized genomes in that
  case
- The goal is community-level abundance simulation rather than within-strain mutation
  injection — use CAMISIM to generate the input community reads instead
- Longitudinal/within-strain evolutionary signal is not needed; a standard metagenome
  simulator alone suffices
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: STEMSim
  role: Command-line tool that aligns longitudinal reads to reference genomes, samples
    SNV/small InDel mutations across time points, rewrites overlapping reads with
    injected alleles, and outputs the modified reads plus a VCF of true mutations
  repo: https://github.com/BoyanZhou/STEMSim
- name: Bowtie2
  role: Aligns the longitudinal raw reads to each reference genome to establish mutation
    coordinates
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Processes the Bowtie2 alignments together with pysam during mutation sampling
  repo: http://www.htslib.org/doc/
- name: CAMISIM
  role: Generates the simulated microbial community raw reads used as input in camisim
    mode
  repo: https://github.com/CAMI-challenge/CAMISIM
- name: pysam
  role: Python library used alongside samtools to process read alignments during mutation
    injection
- name: SnpEff
  role: Optionally annotates the VCF output of true mutations when the -p snpEff directory
    and ID-matching table are supplied
  repo: http://pcingola.github.io/SnpEff/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad302
    title: 'STEMSIM: a simulator of within-strain short-term evolutionary mutations
      for longitudinal metagenomic data'
  source_content_hash: sha256:0e7706e4ce3f960f9330e9963fd46ff85b21680f47bcb87e3f462ad04dcd772e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BoyanZhou/STEMSim
  tools:
  - Bowtie2
  - CAMISIM
  - Python
  - STEMSim
  - SnpEff
  - numpy
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# read-alignment-based-variant-writing

## Summary

Rewrites simulated longitudinal metagenomic raw reads so that reads overlapping injected SNV/InDel sites carry the new alleles, using Bowtie2 alignments of reads to reference genomes as the coordinate system, and exports the injected mutations as a VCF. This is the core mutation-injection mechanism of STEMSim.

## When to use

Use when simulating within-strain short-term evolutionary mutations in longitudinal (multi-time-point) simulated raw sequencing reads and a ground-truth mutation record (VCF) is needed for benchmarking.

## When NOT to use

- Input is real metagenomic data rather than simulated reads — the README warns STEMSim only works well on species with high depth and well-characterized genomes in that case
- The goal is community-level abundance simulation rather than within-strain mutation injection — use CAMISIM to generate the input community reads instead
- Longitudinal/within-strain evolutionary signal is not needed; a standard metagenome simulator alone suffices

## Inputs

- CAMISIM output directory of simulated longitudinal raw reads (camisim mode)
- genome_to_id.tsv table mapping each genome_id to its absolute FASTA path (camisim mode)
- Raw simulated read files (FASTQ) from another simulator such as ART, with longitudinal samples separated by ':' and paired reads by ',' (reads mode)
- Reference genome FASTA (.fas) (reads mode)
- STEMSim config.txt parameter file
- Optional non-mutated genomic-region file (.gtf, .gff, or .bed)
- Optional SnpEff directory and genomeID-to-snpEff ID matching table for VCF annotation

## Outputs

- Modified longitudinal raw metagenomics read sets with injected SNV/InDel alleles at mutation sites
- VCF file detailing the injected true mutations
- STEMSim log file of the run

## How to apply

First prepare the inputs: a config.txt parameter file, the raw reads (either from a CAMISIM output directory for 'camisim' mode, or from other simulators such as ART for 'reads' mode), and reference FASTA files (for camisim mode, plus the genome_to_id.tsv table mapping each genome_id to its absolute FASTA path). STEMSim (Python 3.6.5, numpy 1.19.5, pysam 0.16.0.1) internally aligns the longitudinal reads to each reference genome with Bowtie2, processes the alignments with samtools/pysam, and samples SNV and small InDel mutations across time points for each genome while respecting any non-mutated genomic regions supplied via a .gtf/.gff/.bed non-mutated-region file. It then rewrites the read sequences so that reads overlapping mutation sites carry the injected alleles, writes the modified read sets to the output directory with a log of the run, and exports the record of injected true mutations as a VCF (optionally annotated with SnpEff 5.1 using the -p snpEff directory and -a ID-matching table).

## Related tools

- **STEMSim** (Command-line tool that aligns longitudinal reads to reference genomes, samples SNV/small InDel mutations across time points, rewrites overlapping reads with injected alleles, and outputs the modified reads plus a VCF of true mutations) — https://github.com/BoyanZhou/STEMSim
- **Bowtie2** (Aligns the longitudinal raw reads to each reference genome to establish mutation coordinates) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Processes the Bowtie2 alignments together with pysam during mutation sampling) — http://www.htslib.org/doc/
- **CAMISIM** (Generates the simulated microbial community raw reads used as input in camisim mode) — https://github.com/CAMI-challenge/CAMISIM
- **pysam** (Python library used alongside samtools to process read alignments during mutation injection)
- **SnpEff** (Optionally annotates the VCF output of true mutations when the -p snpEff directory and ID-matching table are supplied) — http://pcingola.github.io/SnpEff/

## Examples

```
python /path_to_stemsim_directory/stemsim.py -m camisim -f PATH/config.txt -s test_subject1 --input_from_camisim PATH_to_output_directory_of_CAMISIM/ --genome_to_id PATH_to_genome_to_id/genome_to_id.tsv -o PATH_to_output_directory_of_STEMSim/ -l Path/stemsim_test1_log
```

## Evaluation signals

- The run completes without errors and the STEMSim output directory contains the modified read files and a log of the run
- The output directory contains a VCF of the injected mutations
- The mutation record lists SNV/InDel positions consistent with the configured mutation parameters for each longitudinal sample
- Reads overlapping injected mutation sites carry the new alleles when re-examined against the reference FASTA
- If a --nonmutated_list or --nonmutated_file is supplied, no mutations appear within the listed genomic regions

## Limitations

- Although STEMSim can work on real metagenomic data, it only works well on species with high depth and well-characterized genomes; it is designed for longitudinal or concurrent simulated raw sequencing reads
- CAMISIM is only necessary for 'camisim' mode, and the 'reads' mode works on reads from one reference genome at a time
- No changelog found for the repository BoyanZhou/STEMSim, so versioning of behavior changes is not tracked

## Evidence

- [methods] STEMSim aligns the longitudinal reads to each reference genome with Bowtie2, processes the alignments with samtools/pysam, and samples SNV and small InDel mutations across time points: [evidence span withheld — anchor status: not_found]
- [readme] stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data: "Under this mode, stemsim takes the output directory of CAMISIM as input directory, and generate longitudinal mutations on the simulated raw sequencing data."
- [abstract] The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations.: "The output is the modified reads with within-strain evolutionary mutations and the relevant information of these mutations."
- [readme] Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes.: "Although it can also works on real metagenomic data, it only works well on species with high depth and well-characterized genomes."
- [methods] SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations): "SnpEff (tested with 5.1, optional, annotate the vcf output of true mutations)"
