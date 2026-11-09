---
name: mock-community-benchmark-analysis
description: Use when a community of known composition (mock or synthetic gut) with
  sequenced shotgun reads is available and you must quantify which SSU extraction,
  reconstruction, or classification tool recovers the expected variants, taxa and
  runtimes.
when_to_use_negative:
- The input is real environmental metagenomic data with no known community composition
  — there is no ground truth against which to compute precision/recall/F-measure
- The goal is novel taxonomic discovery or novelty detection rather than head-to-head
  tool benchmarking
- Input is already an assembled or amplicon OTU/ASV feature table rather than raw
  shotgun FASTQ reads
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: BBTools
  role: Adapter and quality trimming of mock community reads before SSU extraction
    (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1)
  repo: https://github.com/oschakoory/RiboTaxa
- name: SortMeRNA
  role: rRNA-specialized extraction of 16S/18S reads against indexed SILVA SSU 138.1;
    outperformed BBMap for assemblable reads
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBMap
  role: General-purpose read mapper used as the comparator for SSU read extraction
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE
  role: Reference-targeted reconstruction of full-length SSU sequences (Emirge.py
    and Emirge_amplicon.py; IDENTITY threshold 1, 40 iterations, MIN_COV 0.3)
  repo: https://github.com/oschakoory/RiboTaxa
- name: MetaRib
  role: SSU reconstruction from reads with 1,000,000-read subsampling per iteration;
    combined with Emirge_amplicon.py for best population characterisation
  repo: https://github.com/oschakoory/RiboTaxa
- name: QIIME 2 Sklearn classifier
  role: Classification of reconstructed full-length SSU sequences at confidence 0.7
    within the RiboTaxa benchmark arm
  repo: https://github.com/oschakoory/RiboTaxa
- name: Kraken2
  role: Competitor short-read classifier benchmarked on the same mock and synthetic
    gut datasets
- name: Centrifuge
  role: Competitor short-read classifier benchmarked on the same datasets
- name: SPINGO
  role: Competitor species-level amplicon classifier benchmarked on the same datasets
  repo: https://github.com/GuyAllard/SPINGO
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:2a80bae38e30ce6916cb0f49757b98b2e00d1c08120ff926f6fa2b39c1d15480
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - Centrifuge
  - EMIRGE
  - Kraken2
  - MetaRib
  - QIIME 2 Sklearn classifier
  - SPINGO
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mock-community-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SSU rRNA read-extraction tools (BBMap vs SortMeRNA), reference-targeted assemblers (Emirge.py, Emirge_amplicon.py, MATAM, MetaRib) and taxonomic classifiers (RiboTaxa, Kraken2, Centrifuge, METAXA2, phyloFlash, SPINGO, BLCA, MEGAN6) on a microbial mock community (SRA SRR5381736) and a synthetic human gut community, using known 16S rRNA variants as ground truth for precision/recall/F-measure evaluation.

## When to use

Use when a community of known composition (mock or synthetic gut) with sequenced shotgun reads is available and you must quantify which SSU extraction, reconstruction, or classification tool best recovers the expected variants, taxa and runtimes.

## When NOT to use

- The input is real environmental metagenomic data with no known community composition — there is no ground truth against which to compute precision/recall/F-measure
- The goal is novel taxonomic discovery or novelty detection rather than head-to-head tool benchmarking
- Input is already an assembled or amplicon OTU/ASV feature table rather than raw shotgun FASTQ reads

## Inputs

- Paired-end mock community shotgun FASTQ reads (SRA SRR5381736)
- Simulated human gut metagenomic reads (HG_R1, HG_R2)
- Indexed SILVA SSU 138.1 database (SortMeRNA and Bowtie indices)
- QIIME2-trained SILVA 138 classifier
- Ground-truth list of 41 known mock community 16S rRNA variants / 20 detectable microorganisms and synthetic gut community composition

## Outputs

- Per-combination table of reconstructed sequence counts, length distributions, and detected 16S variants/microorganisms across all 12 extractor/assembler combinations (mirroring Figures 2-3)
- benchmark_performance_table.csv with genus- and species-level precision, recall and F-measure per classifier (mirroring Tables 2-3 and Figure 8)
- benchmark_runtime_table.csv with wall-clock runtime per tool per dataset

## How to apply

Retrieve the mock community paired-end raw reads (SRA SRR5381736) and preprocess with BBTools adapter/quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1). Extract 16S reads twice — once with SortMeRNA and once with BBMap — both against indexed SILVA SSU 138.1, then reconstruct full-length SSU sequences with four assemblers (Emirge.py, Emirge_amplicon.py with IDENTITY/JOIN_THRESHOLD=1, 40 iterations, MIN_COV=0.3; MATAM; MetaRib with 1,000,000 subsampled reads per iteration), yielding 12 extractor/assembler combinations. Align each reconstructed set against the 41 known mock 16S variants to tabulate variants and microorganisms recovered. For classifier benchmarking, run RiboTaxa (BBTools trimming, SortMeRNA filtering, EMIRGE+MetaRib, QIIME2 sklearn at confidence 0.7) and each competitor (Kraken2, Centrifuge, METAXA2, phyloFlash BS/BE/SE, SPINGO, BLCA, MEGAN6) on the same inputs, compute precision, recall and F-measure at genus and species level versus known composition, and record wall-clock runtime per tool per dataset (e.g. benchmark_performance_table.csv and benchmark_runtime_table.csv).

## Related tools

- **BBTools** (Adapter and quality trimming of mock community reads before SSU extraction (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1)) — https://github.com/oschakoory/RiboTaxa
- **SortMeRNA** (rRNA-specialized extraction of 16S/18S reads against indexed SILVA SSU 138.1; outperformed BBMap for assemblable reads) — https://github.com/oschakoory/RiboTaxa
- **BBMap** (General-purpose read mapper used as the comparator for SSU read extraction) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE** (Reference-targeted reconstruction of full-length SSU sequences (Emirge.py and Emirge_amplicon.py; IDENTITY threshold 1, 40 iterations, MIN_COV 0.3)) — https://github.com/oschakoory/RiboTaxa
- **MetaRib** (SSU reconstruction from reads with 1,000,000-read subsampling per iteration; combined with Emirge_amplicon.py for best population characterisation) — https://github.com/oschakoory/RiboTaxa
- **QIIME 2 Sklearn classifier** (Classification of reconstructed full-length SSU sequences at confidence 0.7 within the RiboTaxa benchmark arm) — https://github.com/oschakoory/RiboTaxa
- **Kraken2** (Competitor short-read classifier benchmarked on the same mock and synthetic gut datasets)
- **Centrifuge** (Competitor short-read classifier benchmarked on the same datasets)
- **SPINGO** (Competitor species-level amplicon classifier benchmarked on the same datasets) — https://github.com/GuyAllard/SPINGO

## Examples

```
bbduk.sh in=SRR5381736_R1.fastq out=trimmed_R1.fastq ktrim=r kmer=21 minlength=60 trimq=20 qtrim=rl maxns=1
```

## Evaluation signals

- The 12 extractor/assembler comparison table covers all combinations (2 extractors x 4 assemblers) and reproduces the qualitative outcome that SortMeRNA-extracted reads assemble better than BBMap-extracted reads
- The combined Emirge_amplicon.py (SortMeRNA-filtered reads) + MetaRib (unfiltered reads) pipeline with 97% clustering yields 53 reconstructed sequences recovering all 20 detectable mock microorganisms and the 41 expected 16S variants
- On the synthetic human gut community, RiboTaxa reproduces F-measure 100% at genus level and 83.7% at species level with no false positives
- Per-tool runtimes fall in the reported minute-scale range (RiboTaxa ~18-26 min) and RiboTaxa is slower than Kraken2/Centrifuge
- Direct classification of short Illumina reads shows high false-positive rates while reconstructed full-length sequences are correctly assigned

## Limitations

- RiboTaxa is not as fast as short-read classifiers (Kraken2, Centrifuge)
- Direct classification of short-length Illumina reads yields a high rate of false positives
- High conservation of 16S rRNA sequences cannot discriminate all species (e.g. certain Enterobacteriaceae and Lachnospiraceae genera), so classification may stop at genus level
- A part of the recovered diversity may originate from artificial in silico diversity created during full-length sequence reconstruction; small indel errors can occur but have little effect on classification
- No chimera detection was included, so chimeric database sequences could carry over into SSU reconstructions, and no 16S rRNA gene copy number normalization is applied

## Evidence

- [methods] Quality control using BBTOOLS ... Trim reads to remove bases matching adapter sequences ... Reads shorter than this length (bases) after trimming will be discarded: [evidence span withheld — anchor status: not_found]
- [readme] Simulated human gut metagenomics data (HG_R1 and HG_R2) are available: [evidence span withheld — anchor status: external_doc]
