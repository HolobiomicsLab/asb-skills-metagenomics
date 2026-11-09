---
name: blastn-homology-search-configuration
description: Use when metagenomic contigs from human microbiome assemblies must be
  assigned to species-level reference pan-genomes so that taxa-specific GeneMark.hmm-2
  gene prediction models can be selected.
when_to_use_negative:
- The contig is long enough for de novo self-training with GeneMarkS-2 and no close
  relative's genome is available, so pan-genome matching is unnecessary
- The target metagenome is not from the human microbiome and no pan-genome gene database
  covers the taxa present, since assignment rates drop (89.0% oral, 79.4% skin) and
  unmatched contigs fall back to GC-content-specific models
- You need gene prediction with generic metagenomic parameters only (e.g. running
  MetaGeneMark alone) rather than taxa-specific model selection
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: blastn
  role: Performs the nucleotide similarity search of initially predicted contig genes
    against the pan-genome gene database; expected in PATH or set via 'my $blastn'
    in gm_hm.pl
- name: makeblastdb
  role: Builds the BLAST nucleotide database from pan-genome sequences with '-in pan.fasta
    -parse_seqids -dbtype nucl -out pan -title pan'
- name: MetaGeneMark
  role: Produces the initial gene predictions that serve as blastn queries for contig-to-pan-genome
    assignment
- name: GeneMark-HM
  role: Pipeline (gm_hm.pl) that orchestrates the search, hit filtering, and majority-vote
    pan-genome assignment triggering model selection
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:7b7d84889a21d414d8f1290d04b04291dcecce80af77d1fd5eb06e72d23c7bfa
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
  - MetaGeneMark
  - blastn
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# blastn-homology-search-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure a blastn similarity search of initially predicted metagenomic contig genes against a pan-genome gene database, applying the GeneMark-HM hit filters (E-value < 1e-5, >90 nt aligned query, ANI > 95%) to assign contigs to species-level pan-genomes for gene prediction model selection.

## When to use

Use when metagenomic contigs from human microbiome assemblies must be assigned to species-level reference pan-genomes so that taxa-specific GeneMark.hmm-2 gene prediction models can be selected.

## When NOT to use

- The contig is long enough for de novo self-training with GeneMarkS-2 and no close relative's genome is available, so pan-genome matching is unnecessary
- The target metagenome is not from the human microbiome and no pan-genome gene database covers the taxa present, since assignment rates drop (89.0% oral, 79.4% skin) and unmatched contigs fall back to GC-content-specific models
- You need gene prediction with generic metagenomic parameters only (e.g. running MetaGeneMark alone) rather than taxa-specific model selection

## Inputs

- Metagenomic contig sequences (FASTA, e.g. sequence.fasta from a human microbiome assembly)
- Initial gene predictions per contig from MetaGeneMark (GTF format, e.g. genemark.gtf) used as blastn queries
- Pan-genome sequences database (~20 Gb FASTA, 'pan', hosted on Amazon S3)
- Pre-computed species-specific parameter models (refmod database)

## Outputs

- Filtered blastn hits passing E-value < 1e-5, alignment length > 90 nt, ANI > 95%
- Contig-to-pan-genome assignments by majority vote of matched genes
- Selected GeneMark.hmm-2 parameter model per assigned contig; final gene predictions (output.gtf)

## How to apply

Build a BLAST nucleotide database from pan-genome sequences with makeblastdb ('-in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan'). Use the genes from MetaGeneMark initial predictions in each metagenomic contig (e.g. the ../genemark.gtf of a test run) as blastn query sequences against this database. Filter hits to retain only those with E-value better than 1e-5, at least 90 nt of aligned query sequence, and average nucleotide identity above 95%; the combined restriction was designed to give low false positives with high sensitivity. Assign each contig to the pan-genome holding the majority of its matched genes, which triggers selection of the corresponding pre-computed model from the refmod model library; contigs not matched fall back to MetaGeneMark (shorter contigs) or GeneMarkS-2 (longer ones).

## Related tools

- **blastn** (Performs the nucleotide similarity search of initially predicted contig genes against the pan-genome gene database; expected in PATH or set via 'my $blastn' in gm_hm.pl)
- **makeblastdb** (Builds the BLAST nucleotide database from pan-genome sequences with '-in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan')
- **MetaGeneMark** (Produces the initial gene predictions that serve as blastn queries for contig-to-pan-genome assignment)
- **GeneMark-HM** (Pipeline (gm_hm.pl) that orchestrates the search, hit filtering, and majority-vote pan-genome assignment triggering model selection) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- All retained blastn hits satisfy all three cutoffs simultaneously: E-value < 1e-5, aligned query length > 90 nt, ANI > 95%
- Each contig is assigned to the pan-genome holding the majority of its matched genes (majority-vote invariant)
- Running the bundled test '../../bin/gm_hm.pl --seq ../sequence.fasta --cores 4 --out output.gtf --clean' reproduces output consistent with the reference genemark.gtf when compared via '../../bin/compp -a ../genemark.gtf -b output.gtf'
- The combined filters show the reported low false-positive, high-sensitivity behaviour of the combined restriction
- In verbose mode, the pipeline reports performance of individual steps on STDOUT, allowing confirmation that the similarity-search step ran and produced assignments

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome
- Assignment coverage is not universal: 99.4% for human gut metagenomes but lower for oral (89.0%) and skin (79.4%), possibly due to environmental contamination
- The pan-genome file is ~20 Gb and download can take some time; the database covers more than five thousand taxa known to be present in human gut metagenomes
- No version numbers are stated for the tools, and the pan-genome database's accession/repository location is only given via the project's GitHub/S3 links

## Evidence

- [other] Initial gene predictions are used as BLASTn queries against the pan-genome gene database; hits with E-value better than 1e-5, at least 90 nt of aligned query sequence, and ANI above 95% are retained: [evidence span withheld — anchor status: not_found]
- [intro] The genes initially predicted in each metagenomic contig were used for a fast similarity search against the database of genes of pan-genomes: "The genes initially predicted in each metagenomic contig were used for a fast similarity search against the database of genes of pan-genomes"
- [readme] README: gene database role and fallback algorithms: [evidence span withheld — anchor status: external_doc]
- [readme] README: pipeline checks for contig-to-taxon assignment: [evidence span withheld — anchor status: external_doc]
