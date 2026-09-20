---
name: orf-prediction-and-blast-annotation
description: Use when assembled contigs (from SPAdes/metaSPAdes/MEGAHIT inside virMine,
  or supplied via -A) must be gene-predicted and BLAST-annotated against viral and
  nonviral protein FASTA databases to classify contigs as viral.
when_to_use_negative:
- Input is still raw short-read FASTQ and no assembly has been run — this skill operates
  on assembled contigs or supplied long-read/contig FASTA
- No viral or nonviral protein databases are available — -v and -nv are required parameters
  and the gene-origin scoring cannot proceed without them
- The goal is coverage statistics or read QC rather than contig classification — those
  are handled by earlier virMine stages (BBMap 38.94 coverage computation, read quality
  control)
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0085
tools:
- name: virMine 2.0
  role: Runs gene prediction on contigs, BLAST-annotates predicted ORFs against the
    viral/nonviral protein databases, and scores contigs by gene-content origin
  repo: https://github.com/putonti/virmine
- name: Python 3.9
  role: Runtime for virMine.py, which implements the ORF prediction, BLAST calls,
    and contig scoring
  repo: https://github.com/putonti/virmine
- name: Docker
  role: Container in which virMine and all annotation dependencies are executed; Docker
    Hub image genevievej16/virmine
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:36666b63f62852d57a7c8ded3897d64ddaa39c2bcc40c6cdf23ee016729c9a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BBMap
  - Docker
  - MEGAHIT
  - Python
  - Python 3.9
  - SPAdes
  - metaSPAdes
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# orf-prediction-and-blast-annotation

## Summary

virMine 2.0's gene-prediction and BLAST annotation stage: predicted ORFs on assembled contigs are searched against user-supplied viral (viral_aa.fasta) and nonviral (nonviral_aa.fasta) protein databases so each contig can be scored by gene-content origin (cellular, viral, or unknown) and classified as viral.

## When to use

Use when assembled contigs (from SPAdes/metaSPAdes/MEGAHIT inside virMine, or supplied via -A) must be gene-predicted and BLAST-annotated against viral and nonviral protein FASTA databases to classify contigs as viral.

## When NOT to use

- Input is still raw short-read FASTQ and no assembly has been run — this skill operates on assembled contigs or supplied long-read/contig FASTA
- No viral or nonviral protein databases are available — -v and -nv are required parameters and the gene-origin scoring cannot proceed without them
- The goal is coverage statistics or read QC rather than contig classification — those are handled by earlier virMine stages (BBMap 38.94 coverage computation, read quality control)

## Inputs

- Assembled contigs or long-read FASTA (virMine-assembled final_contigs.fasta or a file passed via -A)
- Viral protein database FASTA (viral_aa.fasta), required via -v
- Nonviral protein database FASTA (nonviral_aa.fasta), required via -nv
- Optional: contig filters (-m minimum contig size, -M maximum contig size, -c minimum coverage) and BLAST option (-b blastx|blastp)

## Outputs

- final_contigs.fasta (all assembled/supplied contigs)
- viral_contigs.fasta (contigs classified as viral by gene-content scoring)
- All predicted ORFs output file
- Viral ORFs-only output file
- BLAST output files against the viral and nonviral databases

## How to apply

After assembly (or directly on a user-supplied contig/long-read FASTA passed with -A), run virMine.py, which performs gene prediction on each contig and BLASTs the predicted ORFs against the required viral (-v) and nonviral (-nv) protein databases. Choose the BLAST mode with -b ('blastx' or 'blastp'; 'blastp' is faster) and, if needed, restrict candidate contigs beforehand with the -m/-M/-c filters. The pipeline then scores each contig by the origin of its gene content — cellular, viral, or unknown — rather than relying solely on homology. Contigs scored as viral are written to viral_contigs.fasta alongside final_contigs.fasta; the ORF and BLAST tables are emitted as separate output files for inspection. Rationale: scoring by gene-content origin lets virMine detect viral elements whose ORFs are homologous to known viruses and flag contigs whose genes are cellular or of unknown origin.

## Related tools

- **virMine 2.0** (Runs gene prediction on contigs, BLAST-annotates predicted ORFs against the viral/nonviral protein databases, and scores contigs by gene-content origin) — https://github.com/putonti/virmine
- **Python 3.9** (Runtime for virMine.py, which implements the ORF prediction, BLAST calls, and contig scoring) — https://github.com/putonti/virmine
- **Docker** (Container in which virMine and all annotation dependencies are executed; Docker Hub image genevievej16/virmine) — https://hub.docker.com/repository/docker/genevievej16/virmine

## Examples

```
python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- On the E. coli K12 + phage Lambda test data with -a all3, final_contigs.fasta contains exactly 3 contigs and viral_contigs.fasta contains exactly 1 viral-classified contig
- Output folder contains the expected ORF files (all predicted ORFs and viral ORFs only) and BLAST output files in addition to the contig FASTAs
- Every contig receives a gene-origin label restricted to cellular, viral, or unknown
- BLAST outputs cover both required databases — hits against viral_aa.fasta and nonviral_aa.fasta are both present for the predicted ORF set
- Only contigs passing any applied -m/-M/-c filters enter the scoring step

## Limitations

- Classification quality depends on the currency and completeness of the user-supplied viral and nonviral protein databases; virMine 2.0 mitigates sole dependency on homology by combining an exclusion technique with gene-content scoring, but genes of unknown origin remain unclassified
- Choice of BLAST mode affects runtime — the README notes 'blastp' is faster than 'blastx'
- Docker resource limits can cause failures; the README advises checking available memory if any section errors
- Versions are pinned in the pipeline (Python v.3.9, BBMap v.38.94, SPAdes v.3.15.3), so results may differ with other dependency versions

## Evidence

- [abstract] Next, virMine preforms gene prediction.: "Next, virMine preforms gene prediction."
- [abstract] Contigs are scored based on their gene content's origin, i.e., cellular, viral, or unknown.: "Contigs are scored based on their gene content’s origin, i.e., cellular, viral, or unknown."
- [readme] -b : blast option ['blastx' or 'blastp'] ('blastp' is faster): "-b : blast option ['blastx' or 'blastp'] ('blastp' is faster)"
- [readme] Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral"
- [intro] These assembled short-reads or supplied long-reads/contigs are then annotated: "These assembled short-reads or supplied long-reads/contigs are then annotated"
