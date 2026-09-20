---
name: gene-prediction-accuracy-evaluation
description: Use when you need to quantify and compare gene prediction accuracy across
  gene finders on metagenome assemblies, using a reference annotation to count missed
  genes, start-site errors, or nucleotide Sn/Sp.
when_to_use_negative:
- The goal is gene prediction itself rather than accuracy assessment of predictions
- No reference annotation exists to serve as ground truth for comparison
- Input is raw reads or unassembled sequencing data rather than predicted GTF annotations
  compared on the same contig sets
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: GeneMark-HM
  role: Gene finder under evaluation; predictions compared against reference annotations
    and baselines
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMarkS-2
  role: Self-training baseline tool; also provides native per-genome reference predictions
    for model-transfer accuracy tests
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark.hmm-2
  role: Prediction engine whose externally-supplied-model accuracy is evaluated via
    Sn/Sp against native annotations
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Baseline metagenomic gene finder benchmarked on the same simulated metagenomes
- name: compp
  role: Comparison utility that computes missed genes, translation-start errors, and
    nucleotide-level Sn/Sp between two GTF annotations
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: makeblastdb
  role: Indexes the ~20 Gb pan-genome FASTA (-parse_seqids -dbtype nucl) used in the
    pipeline being benchmarked
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:da14e5295d39d0d356875076a87a51b29914cea9e28241e188ebad937bd692f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/GeneProbe/GeneMark-HM
  repo_url: https://github.com/GeneProbe/GeneMark-HM
  tools:
  - GeneMark-HM
  - GeneMark.hmm-2
  - GeneMarkS-2
  - MetaGeneMark
  - compp
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-prediction-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metagenomic gene finders (GeneMark-HM, MetaGeneMark, MetaProdigal, FragGeneScan, MetaGeneAnnotator) by comparing predicted GTFs against a reference annotation (e.g., PGAP's 1,640,483 genes) with the compp utility, quantifying missed genes, translation-start errors, and nucleotide-level sensitivity/specificity.

## When to use

Use when you need to quantify and compare gene prediction accuracy across gene finders on metagenome assemblies, using a reference annotation to count missed genes, start-site errors, or nucleotide Sn/Sp.

## When NOT to use

- The goal is gene prediction itself rather than accuracy assessment of predictions
- No reference annotation exists to serve as ground truth for comparison
- Input is raw reads or unassembled sequencing data rather than predicted GTF annotations compared on the same contig sets

## Inputs

- Predicted gene annotation files in GTF format from each gene finder
- Reference annotation GTF (e.g., PGAP annotation with 1,640,483 genes)
- Metagenome assembly FASTA (simulated metagenomes built from 625 genomes)
- compp comparison utility from the GeneMark-HM bin folder

## Outputs

- Missed-gene counts and gene start prediction error counts per tool and per D1-D4 database-size variant
- Per-rGenome nucleotide-level sensitivity (Sn) and specificity (Sp) values and (Sn+Sp)/2 averages
- Benchmark tables (e.g., Table 3) comparing GeneMark-HM against FragGeneScan, MetaGeneAnnotator, MetaProdigal, and MetaGeneMark

## How to apply

Run each gene prediction tool (e.g., gm_hm.pl --seq input.fasta --cores 4 --out output.gtf, plus baselines MetaGeneMark, MetaProdigal, FragGeneScan, MetaGeneAnnotator) on identical input FASTAs, including database-size variants D1-D4 when testing reference-database effects. Convert each tool's predictions to GTF and compare them against the PGAP reference annotation (1,640,483 genes) using the compp comparison utility (../../bin/compp -a reference.gtf -b output.gtf) to count missed genes and translation-start errors. For model-transfer tests (e.g., GeneMark.hmm-2 with SGB representative models vs. native GeneMarkS-2 self-training across 1849 SGBs), compute per-rGenome nucleotide-level sensitivity (Sn) and specificity (Sp) with compp and report the (Sn+Sp)/2 average, expecting >98% in most cases. Where relevant, stratify start-error rates by translation-initiation group (e.g., leaderless group C) and check TGA-role determination on contigs with more than 100 genes, and verify lower error rates are not artifacts of gene over-prediction.

## Related tools

- **GeneMark-HM** (Gene finder under evaluation; predictions compared against reference annotations and baselines) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMarkS-2** (Self-training baseline tool; also provides native per-genome reference predictions for model-transfer accuracy tests) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark.hmm-2** (Prediction engine whose externally-supplied-model accuracy is evaluated via Sn/Sp against native annotations) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Baseline metagenomic gene finder benchmarked on the same simulated metagenomes)
- **compp** (Comparison utility that computes missed genes, translation-start errors, and nucleotide-level Sn/Sp between two GTF annotations) — https://github.com/GeneProbe/GeneMark-HM
- **makeblastdb** (Indexes the ~20 Gb pan-genome FASTA (-parse_seqids -dbtype nucl) used in the pipeline being benchmarked) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
../../bin/compp -a reference.gtf -b output.gtf
```

## Evaluation signals

- compp-derived missed-gene and gene-start error counts for every tool and D1-D4 variant reproduce the corresponding entries of Table 3
- (Sn+Sp)/2 nucleotide-level agreement between SGB representative-model and native-model predictions exceeds 98% for most of the 1849 rGenomes
- All tools are compared on identical inputs against the same PGAP reference annotation (1,640,483 genes), so counts are directly comparable
- GeneMark-HM's lower error rate is confirmed not attributable to gene over-prediction
- Lowest start-error rates for GeneMark-HM vs MetaProdigal fall in the leaderless translation initiation group (group C)

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome
- Lower (Sn+Sp)/2 accuracy is observed for highly fragmented rGenomes in representative-model tests
- Pan-genome assignment coverage varies by environment: 99.4% (gut), 89.0% (oral), 79.4% (skin), possibly due to environmental contamination

## Evidence

- [methods] compp against PGAP reference: [evidence span withheld — anchor status: not_found]
- [results] benchmark finding: [evidence span withheld — anchor status: not_found]
- [results] model-transfer accuracy: [evidence span withheld — anchor status: not_found]
- [readme] README database scope: [evidence span withheld — anchor status: external_doc]
