---
name: transposase-integrase-annotation-filtering
description: Use when candidate insertion sequences (putative IS nucleotide contigs
  with ITRs, 500-3000 bp) must be validated as bona fide transposase-family elements
  before inclusion in an IS catalogue or output table.
when_to_use_negative:
- Candidates lack inverted terminal repeats — Palidis is 'currently only equipped
  with discovering ISs with ITRs', so ITR-lacking (non-DDE) elements will not reach
  this annotation step
- Input is already-annotated genomes where IS family membership is known from a reference
  database such as ISfinder — query the catalogue/reference instead of re-annotating
  ORFs
- Sequences fall outside the 500-3000 bp IS length window (e.g. nested insertions
  lengthened beyond max_is_len), which are missed before annotation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0797
tools:
- name: Prodigal
  role: Predicts protein-coding ORFs on the candidate IS nucleotide sequences prior
    to family annotation
- name: InterProScan
  role: Annotates predicted proteins with InterPro and PANTHER protein family signatures
    used for the transposase/integrase-like/Ribonuclease H filter
- name: Palidis
  role: Nextflow pipeline whose runProdigal, installInterproscan, runInterproscan
    and getISInfo steps implement this annotation and filtering stage
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:fdd9939f4353fc0769a6a6727548fae0b297211cbe5326f9e56723381037e3a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - InterProScan
  - Palidis
  - Prodigal
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# transposase-integrase-annotation-filtering

## Summary

Annotates predicted ORFs on candidate insertion sequences with InterPro/PANTHER protein-family signatures (via Prodigal + InterProScan) and retains only ISs whose family descriptions include Transposase, Integrase-like and/or Ribonuclease H, so that the final IS catalogue contains proteins consistent with transposase-family function.

## When to use

Use when candidate insertion sequences (putative IS nucleotide contigs with ITRs, 500-3000 bp) must be validated as bona fide transposase-family elements before inclusion in an IS catalogue or output table.

## When NOT to use

- Candidates lack inverted terminal repeats — Palidis is 'currently only equipped with discovering ISs with ITRs', so ITR-lacking (non-DDE) elements will not reach this annotation step
- Input is already-annotated genomes where IS family membership is known from a reference database such as ISfinder — query the catalogue/reference instead of re-annotating ORFs
- Sequences fall outside the 500-3000 bp IS length window (e.g. nested insertions lengthened beyond max_is_len), which are missed before annotation

## Inputs

- Candidate insertion sequence nucleotide sequences (FASTA), produced by the ITR detection/clustering steps of the Palidis pipeline
- Manifest with sample_id and contig header information for each candidate IS

## Outputs

- FASTA file of insertion sequences
- Tab-delimited information table with columns IS_name, sample_id, contig, is_start, is_end, description, where IS_name encodes IS length, InterPro/PANTHER accessions and transposase positions (e.g. IS_length_1455-IPR013762_1393_1918)

## How to apply

First predict protein-coding ORFs on each candidate IS nucleotide sequence using Prodigal (runProdigal). Then annotate the predicted proteins with InterProScan (in chunks sized by --chunk_size, default 80000 sequences, optionally in LSF cluster mode) to obtain InterPro and PANTHER protein family signatures. Filter the annotated candidates, retaining only ISs whose protein family descriptions contain 'Transposase', 'Integrase-like' and/or 'Ribonuclease H' (e.g. IPR002686:Transposase IS200-like, IPR013762:Integrase-like catalytic domain, PTHR36966). Finally, getISInfo writes each retained IS name encoding the IS nucleotide length plus each detected transposase accession and its amino-acid positions (e.g. IS_length_655-IPR002686_154_418), alongside sample_id, contig, is_start, is_end and the accession descriptions, into a tab-delimited table alongside the FASTA of insertion sequences.

## Related tools

- **Prodigal** (Predicts protein-coding ORFs on the candidate IS nucleotide sequences prior to family annotation)
- **InterProScan** (Annotates predicted proteins with InterPro and PANTHER protein family signatures used for the transposase/integrase-like/Ribonuclease H filter)
- **Palidis** (Nextflow pipeline whose runProdigal, installInterproscan, runInterproscan and getISInfo steps implement this annotation and filtering stage) — https://github.com/blue-moon22/palidis4

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name my_batch -c configs/conf/sanger.config --min_is_len 500 --max_is_len 3000 --chunk_size 80000
```

## Evaluation signals

- Every IS_name in the output table parses as IS_length_<n>-<accession>_<start>_<end> tokens (e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625) and encodes the actual nucleotide length of the corresponding FASTA record
- No retained IS lacks a protein family description containing Transposase, Integrase-like or Ribonuclease H
- Each table row's description field lists definitions for every accession in IS_name (e.g. IPR002686:Transposase IS200-like;PTHR36966:REP-ASSOCIATED TYROSINE TRANSPOSASE)
- sample_id and contig values match those given in the input manifest; is_start/is_end fall within the configured IS length bounds (default 500-3000 bp)
- Regression test ./tests/regression_tests.sh prints 'Test passed. All outputs expected.'

## Limitations

- The filter retains only DDE-type ISs with ITRs; ISs lacking ITRs or with unusual family descriptions are excluded
- ISs whose regions were lengthened by nested insertion events beyond the 500-3000 bp proximity range are missed before annotation
- Default settings miss ITRs shorter than 25 nt, so some short-ITR ISs never reach the annotation step; lowering thresholds is more computationally intensive

## Evidence

- [readme] Prodigal then InterProScan: "7. Get insertion sequences [`runProdigal`, `installInterproscan`, `runInterproscan`, `getISInfo`]"
- [readme] IS_name encodes accessions and positions: "Name assigned by PaliDIS which contains the length, interpro or PANTHER accessions of transposases and their positions"
- [readme] chunk_size parameter: "--chunk_size        Number of protein sequences in each chunk for Interproscan. (Default: 80000)"
- [discussion] scope limitation to ITR-containing ISs: "Palidis is currently only equipped with discovering ISs with ITRs."
- [abstract] catalogue size from pipeline application: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel"
