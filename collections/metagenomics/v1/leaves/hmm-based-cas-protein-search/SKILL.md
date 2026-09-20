---
name: hmm-based-cas-protein-search
description: Use when raw paired-end genomic FASTQ reads (isolate or metagenome) should
  be scanned for cas genes/CRISPR arrays without a complete assembly, e.g. phage-therapy
  susceptibility screening or novel CRISPR/Cas discovery.
when_to_use_negative:
- The dataset already has a high-quality complete genome assembly and annotation;
  running the targeted HMM-seeded pipeline adds overhead without benefit
- The target genes lack representative profile HMMs and no reference nucleotide sequences
  are available to reconfigure the pipeline
- Input is not short DNA sequencing reads (e.g. long-read-only data), since the pipeline
  depends on short-read ORF prediction and VSEARCH seed expansion
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: CasCollect
  role: Python wrapper pipeline executing the HMM-based seed generation, read expansion,
    assembly and annotation steps
  repo: https://github.com/sandialabs/CasCollect
- name: HMMER
  role: searches translated reads against the default 120 Cas protein family profile
    HMMs to identify cas gene seed reads
- name: FragGeneScanPlus
  role: open reading frame prediction on short reads prior to HMMER profile search
  repo: https://github.com/hallamlab/FragGeneScanPlus
- name: VSEARCH
  role: read subset expansion from cas gene seeds at 95% identity match across adjustable
    cycles
  repo: https://github.com/torognes/vsearch
- name: SPAdes
  role: assembles only the cas-focused read subset seeded by HMM hits (metaSPAdes
    for metagenomes)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:ae5e801cea8a218c98bf680eace477e39a92bc80c693f91c9c053145cd667417
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - FragGeneScanPlus
  - HMMER
  - SPAdes
  - VSEARCH
  - bbduk
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hmm-based-cas-protein-search

## Summary

Identify CRISPR-associated (cas) protein coding reads in raw Illumina data by searching translated reads against a curated set of 120 Cas protein family profile HMMs with HMMER, producing 'seed' reads that prime targeted assembly. This avoids complete genome assembly when the goal is CRISPR/Cas discovery.

## When to use

Use when raw paired-end genomic FASTQ reads (isolate or metagenome) should be scanned for cas genes/CRISPR arrays without a complete assembly, e.g. phage-therapy susceptibility screening or novel CRISPR/Cas discovery.

## When NOT to use

- The dataset already has a high-quality complete genome assembly and annotation; running the targeted HMM-seeded pipeline adds overhead without benefit
- The target genes lack representative profile HMMs and no reference nucleotide sequences are available to reconfigure the pipeline
- Input is not short DNA sequencing reads (e.g. long-read-only data), since the pipeline depends on short-read ORF prediction and VSEARCH seed expansion

## Inputs

- raw paired-end Illumina genomic DNA sequencing reads (FASTQ, e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39 isolate reads)
- default set of 120 Cas protein family profile HMMs shipped with CasCollect (or user-supplied HMMs)
- optional user-defined contaminant reference FASTA via -ref (default: human genome)

## Outputs

- seed reads matching cas protein HMM profiles
- expanded read subsets around cas loci (VSEARCH 95% identity expansion)
- targeted SPAdes assembly contigs (500 bp annotation cutoff)
- annotated cas genes and CRISPR arrays from CRISPRCasFinder

## How to apply

Run the CasCollect pipeline in default protein mode so reads are ORF-predicted (FragGeneScan) and searched with HMMER against the bundled default set of 120 Cas protein family profile HMMs to yield seed reads matching cas genes. Optionally precede seed generation with the --trim step (quality trim, adapter removal, read merging via bbduk) and cap read subset expansion at -cycle 2 (down from the default five VSEARCH cycles) with 1 CPU and 20 Gb RAM (-cpu/-mem) for reproducible benchmarking. The VSEARCH-based expansion re-queries reads at the default 95% identity match (-match adjustable), removing already-matched reads from the database between cycles to eliminate redundant searches. Only regions around cas loci are assembled with SPAdes (metaSPAdes for metagenomes), and contigs are annotated with CRISPRCasFinder under a 500 bp contig size cutoff. Correctness is judged by recovery of all previously known cas genes in the isolate, as demonstrated for E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39.

## Related tools

- **CasCollect** (Python wrapper pipeline executing the HMM-based seed generation, read expansion, assembly and annotation steps) — https://github.com/sandialabs/CasCollect
- **HMMER** (searches translated reads against the default 120 Cas protein family profile HMMs to identify cas gene seed reads)
- **FragGeneScanPlus** (open reading frame prediction on short reads prior to HMMER profile search) — https://github.com/hallamlab/FragGeneScanPlus
- **VSEARCH** (read subset expansion from cas gene seeds at 95% identity match across adjustable cycles) — https://github.com/torognes/vsearch
- **SPAdes** (assembles only the cas-focused read subset seeded by HMM hits (metaSPAdes for metagenomes))

## Examples

```
python CasCollect.py -fwd R1.fastq -rev R2.fastq -out outdir --trim -cycle 2 -cpu 1 -mem 20
```

## Evaluation signals

- All known previously identified cas genes within each isolate genome are recovered (as for E. coli KLY, P. aeruginosa VA-134 and S. pyogenes M39)
- Seed reads appear only for reads matching the 120 Cas protein profile HMMs; unmatched reads are removed from the database between expansion cycles
- VSEARCH expansion uses the default 95% identity (-match) and the configured number of cycles (-cycle 2 vs default 5)
- Only contigs >= 500 bp are passed to CRISPRCasFinder annotation
- Wall-clock runtime of the targeted pipeline is compared against a complete SPAdes assembly baseline (e.g. 23.3/17.4, 51.2/33.3, 21.6/31.5 min)

## Limitations

- Detection depends on the coverage and sensitivity of the 120 bundled Cas protein HMMs; novel or divergent cas families outside these profiles may be missed unless user-supplied HMMs or reference nucleotide sequences are provided
- Five-cycle default seed expansion can inflate runtime; reducing cycles (e.g. -cycle 2) trades sensitivity of flanking-region recovery for speed
- No software version numbers or accession IDs were reported for tools or datasets, hindering exact reproducibility

## Evidence

- [results] CasCollect identifies reads matching cas HMM profiles: "CasCollect identifies short DNA sequencing reads that match HMM profiles for cas protein genes"
- [results] Default HMM set size: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] Seed-expansion identity default: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [intro] Seed-read rationale: "identifying reads that match cas genes and/or CRISPR arrays and using these ‘seed’ reads to prime the targeted assembly"
- [readme] Pipeline software dependencies: "This pipeline relies on python and perl as well as requiring the installation of several pieces of software within the users path"
