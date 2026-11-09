---
name: contig-viral-origin-scoring
description: Use when you have assembled contigs (or virMine assembly output) and
  need to classify which contigs are likely viral using gene-content origin scoring
  rather than homology alone.
when_to_use_negative:
- Input is raw short-read sequencing data that has not yet been quality-controlled
  and assembled — use virMine's read QC and assembly steps (-a spades/metaspades/megahit)
  first instead of the -A contig-scoring path.
- You need to identify nonviral (e.g. bacterial) genomic content — this skill explicitly
  excludes nonviral matches and scores only viral likelihood.
- Contigs fail the optional filters (minimum/maximum contig size via -m/-M, minimum
  coverage via -c, minimum SPAdes coverage via -cov) and should be removed before
  scoring.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: virMine 2.0
  role: Primary pipeline performing gene prediction, exclusion-based BLAST against
    viral/nonviral databases, and contig classification into viral_contigs.fasta
  repo: https://github.com/putonti/virmine
- name: BLAST
  role: Searches predicted ORF proteins (blastp or blastx, -b flag, blastp faster)
    against the user-supplied viral and nonviral protein databases
- name: Python 3.9
  role: Language in which virMine 2.0 (including the scoring step) is implemented
    and invoked (python3 virMine.py)
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Container runtime handling all dependencies for running virMine; image pullable
    from Docker Hub
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:0567010a29a0892804bd60a5d7da91791d520c36d6672ff938d8382e2ac97c03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BLAST
  - Docker
  - Python
  - Python 3.9
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# contig-viral-origin-scoring

## Summary

Scores assembled contigs for their likelihood of being viral in origin by predicting genes and classifying each gene's origin as cellular, viral, or unknown via an exclusion-based BLAST strategy against user-supplied viral and nonviral protein databases. This avoids relying solely on direct homology to known viruses, enabling detection of novel viral contigs.

## When to use

Use when you have assembled contigs (or virMine assembly output) and need to classify which contigs are likely viral using gene-content origin scoring rather than homology alone.

## When NOT to use

- Input is raw short-read sequencing data that has not yet been quality-controlled and assembled — use virMine's read QC and assembly steps (-a spades/metaspades/megahit) first instead of the -A contig-scoring path.
- You need to identify nonviral (e.g. bacterial) genomic content — this skill explicitly excludes nonviral matches and scores only viral likelihood.
- Contigs fail the optional filters (minimum/maximum contig size via -m/-M, minimum coverage via -c, minimum SPAdes coverage via -cov) and should be removed before scoring.

## Inputs

- Assembled contigs FASTA file (supplied via virMine -A, e.g. inputFiles/assembled_contigs.fasta)
- Viral protein database FASTA (user-supplied, -v; e.g. RefSeq eukaryotic viral and phage genomes all.faa)
- Nonviral protein database FASTA (user-supplied, -nv; e.g. bacterial COG database 2020 release)

## Outputs

- viral_contigs.fasta containing contigs classified as viral
- Predicted ORF files (all predicted ORFs and viral ORFs only)
- BLAST output files

## How to apply

Load the assembled contigs via virMine 2.0's -A option and run gene prediction to obtain predicted ORFs for each contig. Search the ORF proteins with BLAST against the user-supplied viral database (-v viral_aa.fasta), using blastp or blastx selectable via the -b flag (blastp is faster). Apply the exclusion technique by removing matches to the user-supplied nonviral database (-nv nonviral_aa.fasta), so scoring does not depend solely on direct viral homology. Score each contig based on the origin of its gene content — cellular, viral, or unknown — and classify contigs whose gene content is viral in likelihood as viral. Viral contigs are written to viral_contigs.fasta, and predicted ORF and BLAST output files are retained. Validate with the article's test data (E. coli K12 + Escherichia phage Lambda artificial reads): running with all 3 assembly methods should yield 3 contigs in final_contigs.fasta, 1 of which is classified as viral.

## Related tools

- **virMine 2.0** (Primary pipeline performing gene prediction, exclusion-based BLAST against viral/nonviral databases, and contig classification into viral_contigs.fasta) — https://github.com/putonti/virmine
- **BLAST** (Searches predicted ORF proteins (blastp or blastx, -b flag, blastp faster) against the user-supplied viral and nonviral protein databases)
- **Python 3.9** (Language in which virMine 2.0 (including the scoring step) is implemented and invoked (python3 virMine.py)) — https://github.com/putonti/virmine
- **Docker** (Container runtime handling all dependencies for running virMine; image pullable from Docker Hub) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- Running the article's test data (E. coli K12 NC_000913.3 + Escherichia phage Lambda J02459.1 artificial reads, all3 assembly option) yields 3 contigs in final_contigs.fasta with exactly 1 classified as viral in viral_contigs.fasta
- viral_contigs.fasta exists in the output folder and contains a non-empty set of viral contigs
- All predicted ORF files and all BLAST output files are retained in the output directory
- Genes are partitioned into cellular, viral, or unknown categories, and nonviral database matches were removed before scoring (exclusion technique applied)
- No program errors occur; if sections fail, check that enough memory is allocated in Docker resources

## Limitations

- Scoring depends on user-supplied viral and nonviral databases; the exclusion technique removes nonviral data but contig classification still reflects relatedness to known viral elements in the supplied database
- If any section of the program causes an error or is unable to run, the Docker resources may lack sufficient memory
- When using the Docker Hub image, input files must be accessed from the mounted output folder (virmineDockerOutputFolder) rather than the inputFiles folder

## Evidence

- [abstract] Exclusion-based scoring rationale: "virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements, eliminating the sole dependency on homology"
- [abstract] Gene-content origin classification: "Next, virMine preforms gene prediction. Contigs are scored based on their gene content’s origin, i.e., cellular, viral, or unknown."
- [intro] Scoring workflow positioning: "These assembled short-reads or supplied long-reads/contigs are then annotated, and individual contigs are scored based upon their likelihood of being viral in origin."
- [readme] Command-line invocation for assembled contigs: "python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output"
- [readme] Validation expectation on test data: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral (viral_contigs.fasta)."
- [readme] BLAST option selection: "-b : blast option ['blastx' or 'blastp'] ('blastp' is faster)"
