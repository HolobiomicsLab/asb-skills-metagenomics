---
name: genome-scale-sequence-screening
description: Use when you must detect and type Dsr-dependent dissimilatory sulphur
  metabolism proteins across thousands of complete genomes or (meta)genomic protein
  records on a personal computer, without reconstructing phylogenies or running blast-based
  marker searches.
when_to_use_negative:
- The input is nucleotide sequence rather than protein FASTA — DiSCo.pl expects protein
  sequences ('takes user-supplied protein sequences').
- The goal is a phylogenetic classification of DsrAB sequences or manual distinction
  of paralogous families where small marker proteins (~100 aa, e.g. DsrEFH) are involved
  — DiSCo's HMM best-hit assignment is not a phylogeny replacement in those cases.
- 'The records are already classified: re-running the screen adds no information if
  a validated DiSCo hitlist already exists.'
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0623
- http://edamontology.org/topic_0091
tools:
- name: DiSCo 1.0
  role: Standalone Perl 5 pipeline that runs the HMM screen, applies the filter script
    with best-hit strategy, and emits the classified tab-separated hitlist
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 engine invoked as DiSCo.pl's first step to search the input protein
    FASTA against the DiSCo HMM library (default single thread, -n 1)
  repo: http://hmmer.org/
- name: Perl 5
  role: Runtime language in which DiSCo.pl and its filter script are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:92c237239ba3a82686580d3a68e2dfca9b0897ed0746aabdb8595ca041eba543
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0623
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# genome-scale-sequence-screening

## Summary

Screens large collections of protein FASTA records (thousands of genomes or metagenomes) for Dsr-dependent dissimilatory sulphur metabolism proteins by running them against the curated DiSCo HMM library with HMMER3 hmmsearch, then classifies each hit by predicted enzyme type. It replaces slow phylogeny-based or blast-based marker screening with a fast, laptop-scale HMM workflow.

## When to use

Use when you must detect and type Dsr-dependent dissimilatory sulphur metabolism proteins across thousands of complete genomes or (meta)genomic protein records on a personal computer, without reconstructing phylogenies or running blast-based marker searches.

## When NOT to use

- The input is nucleotide sequence rather than protein FASTA — DiSCo.pl expects protein sequences ('takes user-supplied protein sequences').
- The goal is a phylogenetic classification of DsrAB sequences or manual distinction of paralogous families where small marker proteins (~100 aa, e.g. DsrEFH) are involved — DiSCo's HMM best-hit assignment is not a phylogeny replacement in those cases.
- The records are already classified: re-running the screen adds no information if a validated DiSCo hitlist already exists.

## Inputs

- Protein sequence files in FASTA format (complete-genome proteomes or metagenomic protein sets)
- DiSCo HMM library (default: current directory, or path given via -m)
- Optional user-defined cut-off table (tab-separated: model, score, E-value) via -t

## Outputs

- Filtered DiSCo hitlist, tab-separated by default (-s 0), with 20 columns per hit (sequence identifier, HMM name, predicted enzyme type, sequence/HMM lengths, score, bias, E-value, accuracy, domain metrics, domain start/end, annotation, DsrC-type note)
- Optional FASTA file of DiSCo hit sequences (via -o) for downstream analyses

## How to apply

Download DiSCo (v1.0, Perl 5) and ensure hmmsearch (HMMER 3.1b2) is in $PATH; supply each genome or metagenome as a protein FASTA file to DiSCo.pl (-i), which runs hmmsearch against the DiSCo HMM library as its first step, keeping the default single hmmsearch thread (-n 1). Retain hits that fulfil the hmmer-specific conditional and independent E-value cutoff of ≤10−10 and accuracy ≥0.5, with bias one order of magnitude lower than score. The filter script then applies a best-hit strategy, comparing all hits to the same input sequence to assign the protein family and predicted enzyme type. Collect the filtered DiSCo hitlist in the default tab-separated format (20 columns: sequence identifier, HMM name, predicted enzyme type, scores, E-values, domain coordinates, DsrC cysteine-type note), optionally adding -o to export hit sequences as FASTA for downstream analyses. When scaling up, wrap invocations with process-level monitoring; the paper's benchmark on 5100 complete genomes averaged ~27 MB RAM and ~3.4 s per genome in single-thread mode.

## Related tools

- **DiSCo 1.0** (Standalone Perl 5 pipeline that runs the HMM screen, applies the filter script with best-hit strategy, and emits the classified tab-separated hitlist) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 engine invoked as DiSCo.pl's first step to search the input protein FASTA against the DiSCo HMM library (default single thread, -n 1)) — http://hmmer.org/
- **Perl 5** (Runtime language in which DiSCo.pl and its filter script are implemented)

## Examples

```
perl DiSCo.pl -i genome.faa -p genome_disco -s 0 -n 1
```

## Evaluation signals

- Every reported hit fulfils the hmmer-specific conditional and independent E-value cutoff of ≤10−10 and accuracy ≥0.5, with bias one order of magnitude lower than score.
- The filtered hitlist matches the README's 20-column schema, tab-separated under default -s 0 settings.
- Benchmark on the 5100 complete genomes (4825 bacteria, 253 archaea) reproduces ~27 MB peak RAM and ~3.4 s per genome in single-thread mode.
- On complete-genome benchmarks, classification accuracy is at least 0.98 (0.93 for metagenomic records).
- Sanity-check per-genome costs: a 5100-genome run should complete within hours on a standard laptop, not require high-performance servers.

## Limitations

- DiSCo.pl requires hmmsearch to be present in $PATH; the run fails or must be redirected otherwise.
- Distinction between paralogous families can be hampered by the small size (~100 amino acids) of some marker proteins traditionally used as classifiers.
- Accuracy drops from 0.98 (complete genomes) to 0.93 for metagenomic records, so metagenomic assignments are less certain.
- hmmsearch defaults to a single thread (-n 1); larger parallelism must be requested explicitly via -n and was not part of the paper's performance benchmark.

## Evidence

- [intro] hmmsearch of FASTA input against HMM library: [evidence span withheld — anchor status: not_found]
- [intro] hit filtering cutoffs: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [abstract] scale claim on personal computers: "DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers."
- [readme] dependency on HMMER3 and protein FASTA input: "All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH."
- [readme] thread count option: "-n <number> 			#number of threads for hmmsearch [optional, default = 1]"
