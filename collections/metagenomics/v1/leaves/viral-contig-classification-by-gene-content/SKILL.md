---
name: viral-contig-classification-by-gene-content
description: Use when assembled metagenomic contigs (SPAdes/metaSPAdes/MEGAHIT assemblies
  or supplied long-read/contig FASTA via -A) must be classified as viral without relying
  solely on direct homology to known viral sequences.
when_to_use_negative:
- Input is raw short reads and the goal is assembly itself — this skill is the downstream
  scoring step; run QC/assembly (SPAdes, metaSPAdes, or MEGAHIT via -a) first or use
  -A with preassembled contigs
- The analysis targets taxonomy/annotation of known viral genomes by homology alone
  — this skill's exclusion-based scoring is designed to avoid sole dependency on homology
  identification, not to replicate it
- A strict BLAST-based taxonomy assignment to a reference viral genome is required
  — contigs here are scored as cellular/viral/unknown by gene-content origin, not
  assigned to a specific virus
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: virMine 2.0
  role: Implements gene prediction, exclusion-based BLAST search against viral/nonviral
    databases, and gene-content-origin scoring of contigs
  repo: https://github.com/putonti/virmine
- name: BLAST
  role: Searches predicted ORF proteins against the viral (-v) and nonviral (-nv)
    databases; blastp or blastx selectable via -b, with blastp faster
- name: Docker
  role: Only prerequisite for running virMine; all dependencies handled by the Dockerfile
  repo: https://hub.docker.com/repository/docker/genevievej16/virmine
- name: BBMap 38.94
  role: Computes coverage statistics used for contig filtering/scoring in the pipeline
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1128/mra.00107-22
    title: 'virMine 2.0: Identifying Viral Sequences in Microbial Communities'
  source_content_hash: sha256:7056584b17b7bd40b23dcdbc6ef47549ff20480b0b1487213758a83176205088
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/putonti/virmine
  tools:
  - BBMap
  - BBMap 38.94
  - BLAST
  - Docker
  - MEGAHIT
  - Python
  - SPAdes
  - metaSPAdes
  - virMine 2.0
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# viral-contig-classification-by-gene-content

## Summary

Scores assembled contigs for viral origin by predicting ORFs, BLAST-searching predicted proteins against user-supplied viral (-v) and nonviral (-nv) databases, and classifying each contig by the origin of its gene content as cellular, viral, or unknown using an exclusion technique that avoids sole dependency on homology to known viruses.

## When to use

Use when assembled metagenomic contigs (SPAdes/metaSPAdes/MEGAHIT assemblies or supplied long-read/contig FASTA via -A) must be classified as viral without relying solely on direct homology to known viral sequences.

## When NOT to use

- Input is raw short reads and the goal is assembly itself — this skill is the downstream scoring step; run QC/assembly (SPAdes, metaSPAdes, or MEGAHIT via -a) first or use -A with preassembled contigs
- The analysis targets taxonomy/annotation of known viral genomes by homology alone — this skill's exclusion-based scoring is designed to avoid sole dependency on homology identification, not to replicate it
- A strict BLAST-based taxonomy assignment to a reference viral genome is required — contigs here are scored as cellular/viral/unknown by gene-content origin, not assigned to a specific virus

## Inputs

- Assembled contigs or long-read FASTA file (e.g., inputFiles/assembled_contigs.fasta, supplied via -A) OR raw paired-end short-read FASTQ files for in-pipeline assembly
- Viral protein database (FASTA, e.g., viral_aa.fasta), user-supplied or generated from RefSeq eukaryotic viral and phage genomes
- Nonviral protein database (FASTA, e.g., nonviral_aa.fasta), user-supplied
- Optional filter parameters: minimum/maximum contig size (-m/-M), minimum coverage (-c), minimum SPAdes coverage (-cov)

## Outputs

- viral_contigs.fasta containing contigs classified as viral
- final_contigs.fasta with all scored contigs
- Predicted ORF files (all predicted ORFs and viral ORFs only)
- BLAST output files for ORF searches against the viral and nonviral databases

## How to apply

Load the supplied contig assembly via virMine's -A option (or let virMine assemble short reads first), then perform gene prediction on each contig to produce the set of all predicted ORFs. Search predicted ORF proteins with BLAST against the user-supplied viral database (-v), using blastp or blastx selectable via the -b flag (blastp is the faster option). Apply the exclusion technique by first removing nonviral matches with the user-supplied nonviral database (-nv), so scoring is not limited to known-virus homology. Score each contig by the origin of its gene content — cellular, viral, or unknown — and classify contigs whose gene content is viral in likelihood, writing them to viral_contigs.fasta while retaining the predicted ORF and BLAST output files. On the test data (E. coli K12 + Escherichia phage Lambda artificial reads), correctness is judged when 1 of the 3 contigs in final_contigs.fasta is classified as viral.

## Related tools

- **virMine 2.0** (Implements gene prediction, exclusion-based BLAST search against viral/nonviral databases, and gene-content-origin scoring of contigs) — https://github.com/putonti/virmine
- **BLAST** (Searches predicted ORF proteins against the viral (-v) and nonviral (-nv) databases; blastp or blastx selectable via -b, with blastp faster)
- **Docker** (Only prerequisite for running virMine; all dependencies handled by the Dockerfile) — https://hub.docker.com/repository/docker/genevievej16/virmine
- **BBMap 38.94** (Computes coverage statistics used for contig filtering/scoring in the pipeline)

## Examples

```
python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o virmineDockerOutputFolder/output
```

## Evaluation signals

- On the published test data (artificial reads of E. coli K12 NC_000913.3 and Escherichia phage Lambda J02459.1) run with -a all3, final_contigs.fasta contains exactly 3 contigs and viral_contigs.fasta contains exactly 1 viral-classified contig
- BLAST output files exist for searches against both the viral and nonviral databases
- Predicted ORF files are present (all ORFs plus viral ORFs only)
- A contig classified viral can be traced to viral-database BLAST hits surviving the nonviral exclusion filter
- Using -A on preassembled contigs bypasses assembly and still yields ORF, BLAST, and viral_contigs.fasta outputs

## Limitations

- Scoring depends on the quality and currency of the user-supplied viral and nonviral protein databases; the default viral database derives from the latest RefSeq eukaryotic viral and phage genomes
- blastx is slower than blastp; the -b blastp choice trades translated-search sensitivity for speed
- Classification is by gene-content origin (cellular, viral, unknown) rather than taxonomic assignment — 'unknown' genes are not resolved
- Docker resource limits can cause pipeline failures; check that you have enough memory in your Docker resources if a section errors

## Evidence

- [abstract] virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements, eliminating the sole dependency on homology: "virMine 2.0 uses an exclusion technique to remove nonviral data from sequencing reads and scores the remaining data based on relatedness to viral elements, eliminating the sole dependency on homology"
- [abstract] Contigs are scored based on their gene content's origin, i.e., cellular, viral, or unknown.: "Contigs are scored based on their gene content’s origin, i.e., cellular, viral, or unknown."
- [methods] python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta -o output: "python3 virMine.py -A inputFiles/assembled_contigs.fasta -v inputFiles/viral_aa.fasta -nv inputFiles/nonviral_aa.fasta"
- [readme] Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral: "Running the test data using the option of all 3 assembly methods should result in 3 contigs (final_contigs.fasta), with 1 of those contigs being classified as viral (viral_contigs.fasta)."
- [abstract] generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz): "generates a viral database from the latest collection of RefSeq eukaryotic viral and phage genomes (ftp://ftp.ncbi.nlm.nih.gov/genomes/Viruses/all.faa.tar.gz)"
