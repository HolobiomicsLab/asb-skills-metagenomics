---
name: blast-database-construction
description: Use when a pipeline needs a searchable nucleotide reference from a large
  multi-genome FASTA (e.g. the GeneMark-HM pan-genome) for similarity-based contig-to-pan-genome
  assignment or gene-to-reference matching.
when_to_use_negative:
- Input sequences are proteins rather than nucleotides — the pipeline builds a -dbtype
  nucl database for blastn/MegaBLAST searches
- The task needs de novo gene prediction without any reference similarity step — pan-genome
  assignment is irrelevant for unmatched short contigs handled by MetaGeneMark or
  GeneMarkS-2 fallback
- The FASTA is a single small genome; a full pan-genome BLAST database build and the
  150-rGenomes-per-SGB capping step are unnecessary overhead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3365
tools:
- name: makeblastdb
  role: Formats pan.fasta into the searchable 'pan' nucleotide BLAST database with
    parsed sequence IDs
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: blastn
  role: Similarity search of MetaGeneMark-predicted genes against the 'pan' database
    for contig-to-pan-genome assignment
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark-HM
  role: Pipeline (gm_hm.pl) that consumes the BLAST database output to select species-specific
    models from the refmod library
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:e7f562c30169d4c89e5b27ff31c07e3bd5f668c85b11b1efc3db3d9120d4f038
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
  - NCBI BLAST (blastn)
  - blastn
  - compp
  - makeblastdb
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# blast-database-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a searchable NCBI BLAST nucleotide database from the ~20 Gb human-microbiome pan-genome FASTA (pan.fasta) so that MetaGeneMark-predicted genes from metagenomic contigs can be used as blastn queries to assign contigs to species-level pan-genomes and select GeneMark-HM models.

## When to use

Use when a pipeline needs a searchable nucleotide reference from a large multi-genome FASTA (e.g. the GeneMark-HM pan-genome) for similarity-based contig-to-pan-genome assignment or gene-to-reference matching.

## When NOT to use

- Input sequences are proteins rather than nucleotides — the pipeline builds a -dbtype nucl database for blastn/MegaBLAST searches
- The task needs de novo gene prediction without any reference similarity step — pan-genome assignment is irrelevant for unmatched short contigs handled by MetaGeneMark or GeneMarkS-2 fallback
- The FASTA is a single small genome; a full pan-genome BLAST database build and the 150-rGenomes-per-SGB capping step are unnecessary overhead

## Inputs

- ~20 Gb pan-genome nucleotide FASTA (pan.fasta) covering 5082 species-level pan-genomes for the human microbiome
- NCBI blastn/makeblastdb executables available in PATH
- (optionally) gene predictions in GTF format (e.g. genemark.gtf from MetaGeneMark initial pass) to use as query sequences

## Outputs

- BLAST nucleotide database named 'pan' (with parsed sequence IDs from -parse_seqids)
- blastn hit tables used for pan-genome assignment and model selection
- Validated pan.fasta reference retained in the refdb folder

## How to apply

Download the ~20 Gb pan-genome FASTA (pan.fasta, hosted on Amazon S3 for GeneMark-HM) and the refmod model library, then format pan.fasta as a nucleotide BLAST database with makeblastdb using '-in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan' so sequence identifiers are parsed for downstream reporting. Keep the resulting 'pan' database in a location where blastn is resolvable in PATH (or hard-code the binary path via 'my $blastn' in gm_hm.pl). At run time, genes initially predicted in each contig (from MetaGeneMark initial predictions, e.g. ../genemark.gtf) serve as blastn/MegaBLAST queries against this database; for pan-genome construction itself, query genes are admitted only when no target reaches >95% ANI with >90% query coverage, and for contig assignment only hits with E-value <1e-5, >90 nt aligned query length, and ANI >95% are retained. Validate by confirming pan.fasta parses as nucleotide FASTA and that the 'pan' database is queryable via blastn.

## Related tools

- **makeblastdb** (Formats pan.fasta into the searchable 'pan' nucleotide BLAST database with parsed sequence IDs) — https://github.com/GeneProbe/GeneMark-HM
- **blastn** (Similarity search of MetaGeneMark-predicted genes against the 'pan' database for contig-to-pan-genome assignment) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark-HM** (Pipeline (gm_hm.pl) that consumes the BLAST database output to select species-specific models from the refmod library) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan
```

## Evaluation signals

- makeblastdb completes without errors and the 'pan' database files exist and are queryable via blastn
- pan.fasta parses as nucleotide FASTA and the pan-genome set covers the reported 5082 species
- Downstream filtering retains only hits with E-value <1e-5, alignment length >90 nt, and ANI >95%; contig assignment goes to the pan-genome holding the majority of its matched genes
- Running the bundled test (../../bin/gm_hm.pl --seq ../sequence.fasta --cores 4 --out output.gtf --clean) reproduces the reference genemark.gtf when compared with ../../bin/compp -a ../genemark.gtf -b output.gtf
- Contig-to-pan-genome assignment rate is consistent with reported coverage (e.g. up to 99.4% for human gut metagenomes)

## Limitations

- The pan-genome file is ~20 Gb and download from Amazon S3 can take considerable time and disk space
- GeneMark-HM accuracy depends on correct assignment of a metagenomic contig to a reference pan-genome; mismatches degrade predictions
- Only the 150 longest rGenomes per SGB/tCluster were used in pan-genome construction, so very recently diverged strain content may be underrepresented; oral and skin metagenomes show lower assignment rates (89.0% and 79.4%) possibly due to environmental contamination

## Evidence

- [methods] Build a BLAST nucleotide database from the pan-genome sequences using makeblastdb with parameters '-in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan'.: [evidence span withheld — anchor status: not_found]
- [methods] Filter hits to retain only those with E-value <1e-5, alignment length >90 nt, and average nucleotide identity >95%.: [evidence span withheld — anchor status: not_found]
- [readme] pan-genome sequences are located on Amazon S3 storage: [evidence span withheld — anchor status: external_doc]
- [methods] Pan-genome file is ~20 Gb in size and download can take some time.: "Pan-genome file is ~20 Gb in size and download can take some time."
- [intro] The genes initially predicted in each metagenomic contig were used for a fast similarity search against the database of genes of pan-genomes: "The genes initially predicted in each metagenomic contig were used for a fast similarity search against the database of genes of pan-genomes"
