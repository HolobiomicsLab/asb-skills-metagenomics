---
name: pan-genome-similarity-search-execution
description: Use when annotating short human-microbiome metagenomic contigs that cannot
  support de novo self-training, and species-specific parameters must be imported
  via pan-genome model selection.
when_to_use_negative:
- Input is a complete prokaryotic genome long enough for direct GeneMarkS-2 self-training
  rather than short anonymous contigs
- The metagenome is outside the human microbiome scope and its taxa are absent from
  the >5000-taxon gut-derived pan-genome database
- A contig cannot be reliably assigned to any reference pan-genome, in which case
  the fallback MetaGeneMark-2/GeneMarkS-2 path applies instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: GeneMark-HM
  role: Pipeline that orchestrates the initial prediction, pan-genome blastn search,
    model selection, and final GeneMark.hmm-2 annotation (bin/gm_hm.pl)
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: makeblastdb
  role: Builds the nucleotide BLAST database of pan-genome sequences (-parse_seqids
    -dbtype nucl) before similarity search
- name: blastn
  role: Performs the fast similarity search of initially predicted genes against the
    pan-genome database to identify the best-matching taxon
- name: GeneMarkS-2
  role: Built the species-level pan-genome model library and serves as the fallback
    predictor for long unmatched contigs
- name: MetaGeneMark-2
  role: Fallback gene predictor for shorter contigs not assigned to any pan-genome
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:a6ca75484145e7df4a83df20d60a1beab6ad8ff5433476fdf00cf6154b120eab
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
  - MetaGeneMark-2
  - MetaProdigal
  - blastn
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# pan-genome-similarity-search-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns metagenomic contigs to species-level pan-genomes by a fast blastn similarity search that uses initially predicted genes as queries against a ~20 Gb pan-genome database; the best matches select taxa-specific GeneMark.hmm-2 model files for downstream gene prediction.

## When to use

Use when annotating short human-microbiome metagenomic contigs that cannot support de novo self-training, and species-specific parameters must be imported via pan-genome model selection.

## When NOT to use

- Input is a complete prokaryotic genome long enough for direct GeneMarkS-2 self-training rather than short anonymous contigs
- The metagenome is outside the human microbiome scope and its taxa are absent from the >5000-taxon gut-derived pan-genome database
- A contig cannot be reliably assigned to any reference pan-genome, in which case the fallback MetaGeneMark-2/GeneMarkS-2 path applies instead

## Inputs

- Assembled metagenome FASTA of human microbiome contigs
- ~20 Gb pan-genome sequences FASTA (pan) from Amazon S3 storage
- refmod database of pre-computed species-specific model files
- Initial (first-pass) gene predictions per contig used as blastn queries

## Outputs

- Contig-to-pan-genome (taxon) assignments
- Selected GeneMark.hmm-2 model files and translation-start-site sequence models per assigned contig
- output.gtf gene predictions from the GeneMark-HM pipeline run

## How to apply

First build the nucleotide pan-genome BLAST database with makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan (the pan-genome file is ~20 Gb and download can take time). Run an initial gene prediction on each contig, then use the predicted genes as blastn queries against the pan-gene database; the best matches drive assignment of the contig to one of more than five thousand taxa known in the human gut. A reliable pan-genome assignment triggers automatic selection of a suitable model of sequences near translation start sites and the corresponding GeneMark.hmm-2 model from the refmod library. Contigs not matched to pan-genomes fall back to MetaGeneMark-2 (shorter contigs) or GeneMarkS-2 (longer contigs) based on contig length. The rationale is that species-specific parameters outperform general metagenomic parameters but cannot be estimated from short contigs, so they are borrowed from close relatives' sequenced genomes.

## Related tools

- **GeneMark-HM** (Pipeline that orchestrates the initial prediction, pan-genome blastn search, model selection, and final GeneMark.hmm-2 annotation (bin/gm_hm.pl)) — https://github.com/GeneProbe/GeneMark-HM
- **makeblastdb** (Builds the nucleotide BLAST database of pan-genome sequences (-parse_seqids -dbtype nucl) before similarity search)
- **blastn** (Performs the fast similarity search of initially predicted genes against the pan-genome database to identify the best-matching taxon)
- **GeneMarkS-2** (Built the species-level pan-genome model library and serves as the fallback predictor for long unmatched contigs)
- **MetaGeneMark-2** (Fallback gene predictor for shorter contigs not assigned to any pan-genome)

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan
```

## Evaluation signals

- Every contig query returns blastn hits against the pan database built with 'makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan'
- A pan-genome assignment must be followed by selection of a matching GeneMark.hmm-2 model from the refmod library and a suitable translation-start-site sequence model
- Unmatched contigs must be rerouted to MetaGeneMark-2 or GeneMarkS-2 according to contig length, not left unannotated
- On simulated human gut metagenomes, up to 99.4% of contigs should receive pan-genome assignments (89.0% oral, 79.4% skin as lower bounds)
- Final output.gtf predictions should show fewer gene start errors than MetaGeneMark-2 alone, confirming the model selection step was done efficiently

## Limitations

- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome; wrong assignments propagate wrong models
- The pan-genome database covers taxa known from human gut isolates and in silico reconstructed genomes; taxa outside this set cannot be matched
- The ~20 Gb pan-genome download can take considerable time, and BLAST database construction requires proportional disk resources
- Oral and skin metagenomes show substantially lower assignment coverage (89.0% and 79.4%), possibly due to mixing with environmental microbes

## Evidence

- [readme] Database of species specific parameters was built for more than five thousand taxa, which are known to be present in the human gut metagenomes.: [evidence span withheld — anchor status: external_doc]
