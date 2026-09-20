---
name: blastn-similarity-search-against-pangenome
description: Use when annotating human microbiome metagenomic contigs where species-specific
  gene-finding parameters cannot be self-trained from short sequence and the contig
  must instead be assigned to a reference pan-genome by sequence similarity.
when_to_use_negative:
- Input is a complete prokaryotic genome long enough for self-training, where GeneMarkS-2
  alone should be used directly
- Metagenomes from environments other than the human microbiome, since the pan-genome
  database was built specifically from genomes of taxa known to be present in human
  microbiomes
- Contigs already have reliable reference annotation and no model selection is needed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: makeblastdb
  role: Builds the nucleotide BLAST database (pan) from the ~20 Gb pan.fasta pan-genome
    file with -parse_seqids -dbtype nucl
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: blastn
  role: Fast similarity search of initially predicted genes in each contig against
    the pan-genome database to select the gene-prediction model; also used as MegaBLAST
    for verification of codon-role assignments
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark-HM (gm_hm.pl)
  role: Pipeline driver that orchestrates initial prediction, pan-genome blastn search,
    model selection, and fallback branches; invoked with --seq, --cores, --out, --clean
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: MetaGeneMark
  role: Provides the initial gene predictions on each contig that serve as the blastn
    queries for the pan-genome search, and is the fallback predictor for short unmatched
    contigs
  repo: https://github.com/GeneProbe/GeneMark-HM
- name: GeneMark.hmm-2
  role: Runs the final gene prediction with the taxon-specific model selected from
    the pan-genome match
  repo: https://github.com/GeneProbe/GeneMark-HM
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqab047
    title: 'GeneMark-HM: improving gene prediction in DNA sequences of human microbiome'
  source_content_hash: sha256:570576c04b289d0c5ba1b5a943ce86eabeb942243c53f256efcfc66c0c584096
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
  - GeneMark-HM (gm_hm.pl)
  - GeneMark.hmm-2
  - GeneMarkS-2
  - MetaGeneMark
  - NCBI blastn
  - Parallel::ForkManager
  - blastn
  - compp
  - makeblastdb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# blastn-similarity-search-against-pangenome

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses initial gene predictions from metagenomic contigs as blastn query sequences against a ~20 Gb species-level pan-genome nucleotide database so that the best matches select a taxon-specific GeneMark.hmm-2 model for final gene prediction in the GeneMark-HM pipeline.

## When to use

Use when annotating human microbiome metagenomic contigs where species-specific gene-finding parameters cannot be self-trained from short sequence and the contig must instead be assigned to a reference pan-genome by sequence similarity.

## When NOT to use

- Input is a complete prokaryotic genome long enough for self-training, where GeneMarkS-2 alone should be used directly
- Metagenomes from environments other than the human microbiome, since the pan-genome database was built specifically from genomes of taxa known to be present in human microbiomes
- Contigs already have reliable reference annotation and no model selection is needed

## Inputs

- Metagenome assembly FASTA of contigs (e.g. human gut, oral, or skin assembly such as GCA_900415335)
- pan.fasta pan-genome nucleotide sequence database (~20 Gb, from Amazon S3)
- refmod.tar database of pre-computed species-specific model files
- Pipeline key installed at ~/.gmhmmp2_key; NCBI blastn on PATH; Perl 5.10 with Parallel::ForkManager

## Outputs

- output.gtf with final gene predictions per contig
- Per-contig assignment of contig to a pan-genome (or record of fallback branch: GeneMarkS-2 or MetaGeneMark)
- Translated protein (aa.fasta) and nucleotide (nt.fasta) prediction files when requested via --aa/--nt

## How to apply

First build a nucleotide BLAST database from the pan-genome sequences with 'makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan' after unpacking the ~20 Gb pan.fasta and the refmod.tar model library into the pipeline directories. Then run the gm_hm.pl driver, which initially predicts genes in each contig with MetaGeneMark and uses those predicted genes as blastn/MegaBLAST queries against the pan database; the best matches lead to selection of the gene-prediction model for GeneMark.hmm-2, and pan-genome assignment additionally triggers automatic selection of a suitable model of sequences near translation start sites. Contigs with no reliable pan-genome match fall back to GeneMarkS-2 (longer contigs) or MetaGeneMark (shorter contigs), with the length threshold deciding between them. Parallelize across cores with the '--cores' option, which relies on Perl 5.10 with Parallel::ForkManager and blastn available on the shell PATH; the pipeline key must be installed at ~/.gmhmmp2_key.

## Related tools

- **makeblastdb** (Builds the nucleotide BLAST database (pan) from the ~20 Gb pan.fasta pan-genome file with -parse_seqids -dbtype nucl) — https://github.com/GeneProbe/GeneMark-HM
- **blastn** (Fast similarity search of initially predicted genes in each contig against the pan-genome database to select the gene-prediction model; also used as MegaBLAST for verification of codon-role assignments) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark-HM (gm_hm.pl)** (Pipeline driver that orchestrates initial prediction, pan-genome blastn search, model selection, and fallback branches; invoked with --seq, --cores, --out, --clean) — https://github.com/GeneProbe/GeneMark-HM
- **MetaGeneMark** (Provides the initial gene predictions on each contig that serve as the blastn queries for the pan-genome search, and is the fallback predictor for short unmatched contigs) — https://github.com/GeneProbe/GeneMark-HM
- **GeneMark.hmm-2** (Runs the final gene prediction with the taxon-specific model selected from the pan-genome match) — https://github.com/GeneProbe/GeneMark-HM

## Examples

```
makeblastdb -in pan.fasta -parse_seqids -dbtype nucl -out pan -title pan; nohup path_to/bin/gm_hm.pl --seq path_to/sequence.fasta --cores 4 --out output.gtf --clean
```

## Evaluation signals

- Reproduced pan-genome assignment fractions on real human metagenomes should match Table 6: up to 99.4% for human gut, 89.0% for oral, and 79.4% for skin metagenomes
- The BLAST database must be built with '-parse_seqids -dbtype nucl' so blastn hits resolve back to specific pan-genome taxa for model selection
- Every contig in output.gtf must be traceable to one of the three branches: pan-genome-matched GeneMark.hmm-2, GeneMarkS-2 (long unmatched), or MetaGeneMark (short unmatched)
- blastn must be found on the shell PATH (or hardcoded in gm_hm.pl) and Parallel::ForkManager must load, otherwise contig-level parallel processing fails
- In verbose mode, per-step performance information on STDOUT should show the similarity-search step completing before model selection

## Limitations

- GeneMark-HM accuracy depends on the correct assignment of a metagenomic contig to a reference pan-genome; mismatches propagate to the gene predictions
- Oral and skin metagenomes show substantially lower pan-genome assignment coverage (89.0% and 79.4% vs 99.4% for gut), possibly because those microbiomes are mixed with environmental microbes
- The pan-genome file is ~20 Gb and the download can take some time; quality of the gene database and model database determines the accuracy gain over the initial MetaGeneMark-2 level
- No taxa-specific parameters can be estimated for contigs without close relatives, forcing the cruder fallback branches

## Evidence

- [abstract] Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database. The best matches led to selection of the model for gene prediction.: "Genes initially predicted in each contig served as queries for a fast similarity search through the pan-genome database. The best matches led to selection of the model for gene prediction."
- [methods] Pan-genome file is ~20 Gb in size and download can take some time.: "Pan-genome file is ~20 Gb in size and download can take some time."
- [readme] Pipeline checks if a contig can be assigned to one of the reference taxa and in case of reliable assignment uses corresponding taxa specific parameters for gene finding in a contig.: [evidence span withheld — anchor status: external_doc]
- [discussion] The highest rate of a contig to a pan-genome assignment, 99.4%, was observed for human gut metagenomes.: "The highest rate of a contig to a pan-genome assignment, 99.4%, was observed for human gut metagenomes."
