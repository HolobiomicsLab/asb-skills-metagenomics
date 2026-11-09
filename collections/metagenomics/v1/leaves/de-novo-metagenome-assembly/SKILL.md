---
name: de-novo-metagenome-assembly
description: Use when input is raw paired-end metagenomic (or spiked-in isolate) Illumina
  FASTQ reads and the goal is to recover cas operons/CRISPR arrays while measuring
  runtime against a complete-assembly baseline.
when_to_use_negative:
- Input is already assembled contigs or a genome — use annotation/detection tools
  directly rather than targeted re-assembly
- The goal is a whole-community taxonomic/functional profile rather than recovery
  of CRISPR/Cas loci — targeted assembly discards the vast majority of the metagenome
- Reads are long-read or non-Illumina data; the pipeline is designed for unassembled
  raw Illumina short reads
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: CasCollect
  role: 'Targeted assembly pipeline: seed generation via HMMER Cas HMMs, VSEARCH expansion,
    assembly, annotation'
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: Identifies seed reads matching the 120 default Cas protein family profile
    HMMs
- name: VSEARCH
  role: 'Read subset expansion: five cycles of identity matching at the default 95%
    (-match) threshold'
  repo: https://github.com/torognes/vsearch
- name: metaSPAdes
  role: Assembler used in place of SPAdes for metagenomes, assembling the seed and
    expanded read subset
- name: CRISPRCasFinder
  role: Annotates assembled contigs for cas genes and CRISPR arrays with a 500 bp
    cutoff
- name: bbduk
  role: BBTools program implementing --trim/--clean quality trimming, adapter removal
    and contaminant read removal
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: Seqtk
  role: Strips/prepares raw reads when pooling isolate and spiked-in reads into the
    simulated metagenome
  repo: https://github.com/lh3/seqtk
- name: PRICE
  role: Baseline assembler benchmarked against CasCollect and complete assembly
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:ad7b80bc2ce94adc2dc241b0cf74868183c4b135f40dc7274c5588ae6cbcd600
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - BBTools
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - Seqtk
  - VSEARCH
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# de-novo-metagenome-assembly

## Summary

Assemble CRISPR/Cas regions from pooled metagenomic FASTQ reads either via targeted assembly (CasCollect: HMMER seed reads against 120 Cas protein HMMs, VSEARCH expansion at 95% identity, metaSPAdes, CRISPRCasFinder) or via complete metaSPAdes assembly, benchmarking runtime and contig output.

## When to use

Use when input is raw paired-end metagenomic (or spiked-in isolate) Illumina FASTQ reads and the goal is to recover cas operons/CRISPR arrays while measuring runtime against a complete-assembly baseline.

## When NOT to use

- Input is already assembled contigs or a genome — use annotation/detection tools directly rather than targeted re-assembly
- The goal is a whole-community taxonomic/functional profile rather than recovery of CRISPR/Cas loci — targeted assembly discards the vast majority of the metagenome
- Reads are long-read or non-Illumina data; the pipeline is designed for unassembled raw Illumina short reads

## Inputs

- Paired-end Illumina FASTQ metagenomic read files (e.g. SRR10598175, ERR472738, SRR11195315, SRR5823773)
- Pooled bacterial isolate reads with spiked-in contaminant reads (e.g. mouse SRR1752459)
- Default 120 Cas protein family profile HMMs (bundled with CasCollect)
- Optional user-defined contaminant reference FASTA for --clean (default human genome)

## Outputs

- Targeted assembly contigs spanning cas operons and CRISPR arrays
- CRISPRCasFinder annotations of cas genes and CRISPR arrays on contigs >= 500 bp
- Runtime and contig-count comparison table (e.g. cascollect_metagenome_benchmark.csv, metagenome_benchmark_summary.csv)

## How to apply

Build the metagenome by pooling paired-end isolate read sets (e.g. SRR1424625, SRR2939129, SRR5280756) and spiking in contaminant reads (e.g. mouse reads SRR1752459), preparing files with Seqtk. Run CasCollect in protein mode with --meta so metaSPAdes replaces SPAdes, using defaults of 120 Cas protein HMMs queried by HMMER, five VSEARCH read-subset expansion cycles at 95% identity, and a 500 bp contig cutoff for annotation by CRISPRCasFinder. Run a complete metaSPAdes assembly of the same reads (plus optional PRICE assembly) as the baseline, recording wall-clock runtimes and contig counts. Compare CasCollect runtime and contigs (target ~1.6 h, 723 contigs vs ~2.5 h, 10,467 contigs; ~36% reduction on the simulated metagenome) and verify all previously known cas genes are recovered and cas operons correctly assembled.

## Related tools

- **CasCollect** (Targeted assembly pipeline: seed generation via HMMER Cas HMMs, VSEARCH expansion, assembly, annotation) — https://github.com/sandialabs/CasCollect
- **HMMER** (Identifies seed reads matching the 120 default Cas protein family profile HMMs)
- **VSEARCH** (Read subset expansion: five cycles of identity matching at the default 95% (-match) threshold) — https://github.com/torognes/vsearch
- **metaSPAdes** (Assembler used in place of SPAdes for metagenomes, assembling the seed and expanded read subset)
- **CRISPRCasFinder** (Annotates assembled contigs for cas genes and CRISPR arrays with a 500 bp cutoff)
- **bbduk** (BBTools program implementing --trim/--clean quality trimming, adapter removal and contaminant read removal) — https://jgi.doe.gov/data-and-tools/bbtools/
- **Seqtk** (Strips/prepares raw reads when pooling isolate and spiked-in reads into the simulated metagenome) — https://github.com/lh3/seqtk
- **PRICE** (Baseline assembler benchmarked against CasCollect and complete assembly)

## Examples

```
CasCollect.py --meta --trim metagenome_R1.fastq metagenome_R2.fastq
```

## Evaluation signals

- CasCollect runtime on the simulated metagenome approximates 1.6 h versus 2.5 h for complete assembly (~36% reduction), with ~723 vs 10,467 contigs
- On real metagenomes, CasCollect averages 1.7 ± 0.3 h versus 50.6 ± 3.0 h for complete assembly plus annotation (~3% of the time, near two-orders-of-magnitude speedup)
- All previously known cas genes from the source isolate genomes (E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39) are identified in the annotations
- Cas operons and CRISPR arrays are correctly assembled and annotated by CRISPRCasFinder on contigs passing the 500 bp cutoff
- CasCollect's Check.py reports all required third-party programs (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) present in the user's path

## Limitations

- Benchmarked only on Illumina short paired-end reads; performance on other platforms is uncharacterized
- Complete assembly produces far more contigs (e.g. ~161K contigs on average after 500 bp cutoff) than targeted assembly, so non-CRISPR genomic context is not recovered
- No software version numbers or accession IDs are given for tools or datasets, complicating exact reproduction
- Defaults (five expansion cycles, 95% identity match) may need adjustment via -cycle and -match for divergent or novel systems

## Evidence

- [results] CasCollect reduced the runtime modestly by 36% relative to the complete assembly, with a CasCollect runtime of 1.6 h versus 2.5 h: [evidence span withheld — anchor status: not_found]
- [results] For metagenomes, metaSPAdes is used in-place-of SPAdes.: "For metagenomes, metaSPAdes is used in-place-of SPAdes."
- [results] This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13): "This pipeline relies on several third-party programs that include: HMMER (21), VSEARCH (22), SPAdes (23) and CRISPRCasFinder (13)"
- [results] The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command.: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [readme] CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads.: "CasCollect is a pipeline for the detection of Cas genes and CRISPR arrays from unassembled raw illumina sequencing reads."
