---
name: viral-taxonomic-classification-pipeline-execution
description: Use when metagenomic paired-end FASTQ data must be classified to viral
  taxa and reconstructed into virus-like super-scaffolds, especially with low-coverage
  or highly divergent viruses that defeat read-only classifiers.
when_to_use_negative:
- Input is already a classified feature table or taxonomy assignments rather than
  raw metagenomic FASTQ reads
- The question only requires individual-read or kmer-based classification without
  reconstruction — VirMAP's mapping assembly and improvement algorithms are designed
  for reconstruction-plus-classification
- RAM/SSD resources cannot accommodate the Genbank-based databases (minimum SSD and
  RAM requirements grow as Genbank expands)
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: 'Core Perl pipeline: dereplication, digital normalization, de-novo and mapping
    assembly, Jaccard filtering, pileup reconstruction, and bits-per-base LCA taxonomic
    classification'
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: De-novo multi-kmer assembler used via --useMegahit to build contigs from subsampled
    metagenomic reads
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Nucleotide mapping aligner (bbnorm for digital normalization, bbmap for read
    alignment to viral nucleotide database)
- name: DIAMOND
  role: Protein aligner (--virDmnd) for translated alignments of reads against the
    viral protein database
  repo: https://github.com/bbuchfink/diamond
- name: khmer
  role: normalize-by-median.py used for digital normalization of input reads
  repo: https://github.com/dib-lab/khmer
- name: pigz
  role: Parallel gzip compression used for FASTQ handling
  repo: https://github.com/madler/pigz
- name: VSEARCH
  role: Dereplication of input reads prior to normalization
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:0c5bc6c7a0bc7fa7113827ca72c2e5ee7099b41c6309878015632a3f9224d63c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - GNU parallel
  - MEGAHIT
  - VSEARCH
  - VirMAP
  - Virmap.pl
  - khmer
  - pigz
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# viral-taxonomic-classification-pipeline-execution

## Summary

Executes the VirMAP Perl pipeline (Virmap.pl) to taxonomically classify viral reconstructions from metagenomic paired-end FASTQ reads, merging nucleotide and protein alignments against Genbank viral/phage databases. It combines de-novo (MEGAHIT) and mapping assembly (BBMap), read assignment with Jaccard-distance filtering, and a bits-per-base LCA scoring system.

## When to use

Use when metagenomic paired-end FASTQ data must be classified to viral taxa and reconstructed into virus-like super-scaffolds, especially with low-coverage or highly divergent viruses that defeat read-only classifiers.

## When NOT to use

- Input is already a classified feature table or taxonomy assignments rather than raw metagenomic FASTQ reads
- The question only requires individual-read or kmer-based classification without reconstruction — VirMAP's mapping assembly and improvement algorithms are designed for reconstruction-plus-classification
- RAM/SSD resources cannot accommodate the Genbank-based databases (minimum SSD and RAM requirements grow as Genbank expands)

## Inputs

- Paired-end FASTQ reads (e.g., SRA accession SRR9875293 from VMC BioProject PRJNA431646)
- Prebuilt VirMAP database: taxa JSON, viral DIAMOND database, viral BBMap nucleotide index, Genbank blastn (gbvrl) and blastx databases
- Genbank viral (gbvrl) and phage (gbphg) divisions as reference sequences

## Outputs

- Taxonomic classification of viral mock community constituents (per-genome read assignments and bits-per-base scores)
- Reconstructed virus-like super-scaffolds / contigs with tiled CDS
- Viral-base retention metric relative to full-depth analysis (e.g., 55.55%, 25.99%, 10.0% at 10/1/0.1% subsampling)
- Run log (stderr redirected to VirmapRun.err)

## How to apply

Retrieve paired-end reads with fasterq-dump (e.g., fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293), then run Virmap.pl with --threads $(nproc), --useMegahit, --useBbnorm, and --sampleName, pointing --taxaJson, --virDmnd, --virBbmap, --gbBlastn, and --gbBlastx at the prebuilt database directory. Internally, the pipeline dereplicates and digitally normalizes reads (khmer normalize-by-median.py), performs de-novo assembly with MEGAHIT and mapping assembly with BBMap, aligns processed reads to viral nucleotide (gbvrl) and protein (gbphg/DIAMOND) databases, and assigns reads to viral genomes filtered by Jaccard distance. Each non-filtered genome is reconstructed by naive pileup assembly, reconstructed CDS are tiled onto originating genomes, and contigs are filtered if their top pass-one taxon is not in the superkingdom virus or if >50% of aligning bases do not resolve to virus. Taxonomic classification uses a bits-per-base scoring system with a rank-wise lowest common ancestor (LCA) algorithm within a dynamically calculated radius per contig; redirect stderr (e.g., VirmapRun.err) to capture run logs.

## Related tools

- **VirMAP** (Core Perl pipeline: dereplication, digital normalization, de-novo and mapping assembly, Jaccard filtering, pileup reconstruction, and bits-per-base LCA taxonomic classification) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (De-novo multi-kmer assembler used via --useMegahit to build contigs from subsampled metagenomic reads) — https://github.com/voutcn/megahit
- **BBMap** (Nucleotide mapping aligner (bbnorm for digital normalization, bbmap for read alignment to viral nucleotide database))
- **DIAMOND** (Protein aligner (--virDmnd) for translated alignments of reads against the viral protein database) — https://github.com/bbuchfink/diamond
- **khmer** (normalize-by-median.py used for digital normalization of input reads) — https://github.com/dib-lab/khmer
- **pigz** (Parallel gzip compression used for FASTQ handling) — https://github.com/madler/pigz
- **VSEARCH** (Dereplication of input reads prior to normalization) — https://github.com/torognes/vsearch

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293 --taxaJson /scratch/VirmapDb --virDmnd /scratch/VirmapDb --virBbmap /scratch/VirmapDb --gbBlastn /scratch/VirmapDb --gbBlastx /scratch/VirmapDb 2> VirmapRun.err
```

## Evaluation signals

- Detection rate: six of the seven VMC constituents are reported in all trials at 10% and 1% subsampling (rotavirus never reported)
- Viral-base retention matches published values: VirMAP 55.55%, 25.99%, and 10.0% at 10%, 1%, and 0.1% subsampling (vs 24.76%, 5.36%, 0.11% for the standard approach)
- 30 independent trials (10 per depth) all complete without fatal errors, with stderr captured in VirmapRun.err
- Filtered contigs: none retained whose top pass-one taxon is outside the superkingdom virus or where >50% of aligning bases fail to resolve to virus
- Reconstructed protein coding sequences are successfully tiled on top of originating genomes in the output super-scaffolds

## Limitations

- No software versions stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole; Genbank database snapshot/version or download date not specified, affecting reproducibility
- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the article text
- Minimum SSD and RAM requirements expand as Genbank expands; Supplementary Data 14 (sensitivity/specificity tuning parameters) is referenced but not provided in the document
- Rotavirus (one of seven VMC constituents) was never reported across subsampling trials

## Evidence

- [methods] Virmap.pl invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [methods] Command-line flags: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [methods] FASTQ retrieval: "fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293"
- [discussion] Assembly strategy: "VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs."
- [discussion] Read preprocessing: "input reads are dereplicated and digitally normalized18 using default parameters"
- [discussion] Jaccard filtering: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] Viral-only contig filter: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [intro] Classification algorithm: "a taxonomic classification algorithm that is centered around a bits-per-base scoring system"
- [intro] Reference databases: "A putative list of viral genomes and protein pseudo-scaffolds are constructed from clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)."
- [discussion] Dataset availability: "Viral Mock Community, BioProject ID PRJNA431646"
- [readme] Software availability: "The paper is published at Nature Communications:
https://doi.org/10.1038/s41467-018-05658-8"
- [discussion] Source code: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
