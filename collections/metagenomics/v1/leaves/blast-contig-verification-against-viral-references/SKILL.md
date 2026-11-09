---
name: blast-contig-verification-against-viral-references
description: Use when extended or seed contigs from viral metagenomic assembly must
  be validated — after ContigExtender extension, to confirm non-chimeric, accurate
  sequence and assign the correct reference virus.
when_to_use_negative:
- Input sequences are not viral contigs or no corresponding NCBI viral reference nucleotide
  sequence exists for comparison
- The goal is de novo assembly or extension itself (use metaSPAdes/ContigExtender)
  rather than post-extension accuracy verification
- Contigs come from metagenomic assemblies of non-viral organisms where viral reference
  assignment is not the validation criterion
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: ContigExtender
  role: Produces the extended contigs (output/contigs.fasta) that are then verified
    by BLAST against viral references
  repo: https://github.com/dengzac/contig-extender
- name: metaSPAdes
  role: Generates the initial seed contigs whose lengths are compared against the
    extended contigs to compute gained length
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-021-04038-2
    title: 'ContigExtender: a new approach to improving de novo sequence assembly
      for viral metagenomics data'
  source_content_hash: sha256:0689087e7a9bcb2c976ed36062d0b0e636f22cdd0ed99a00498f6e789a0b005c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dengzac/contig-extender
  tools:
  - ContigExtender
  - PRINSEQ
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# blast-contig-verification-against-viral-references

## Summary

Verify accuracy of de novo assembled or ContigExtender-extended contigs by nucleotide BLAST alignment against corresponding NCBI viral reference sequences, assigning each contig to its reference virus and quantifying gained length per contig.

## When to use

Use when extended or seed contigs from viral metagenomic assembly must be validated — after ContigExtender extension, to confirm non-chimeric, accurate sequence and assign the correct reference virus.

## When NOT to use

- Input sequences are not viral contigs or no corresponding NCBI viral reference nucleotide sequence exists for comparison
- The goal is de novo assembly or extension itself (use metaSPAdes/ContigExtender) rather than post-extension accuracy verification
- Contigs come from metagenomic assemblies of non-viral organisms where viral reference assignment is not the validation criterion

## Inputs

- Extended contigs in FASTA format (output/contigs.fasta from ContigExtender, entries named reference_N)
- Seed contigs FASTA (e.g. metaSPAdes output) for baseline length comparison
- NCBI viral reference nucleotide sequences (e.g. BKV.fasta for the simulated example)

## Outputs

- BLAST alignment of each extended contig against its viral reference (accuracy verification)
- Assigned reference virus per contig
- Gained length per contig (extended minus seed length) and tables of contigs extended by >0.2 kb

## How to apply

Collect the extended contigs from output/contigs.fasta for each input; output contigs are sorted by length and named reference_N in input order (e.g. >reference_1, >reference_2). Align each extended contig to the corresponding NCBI viral reference nucleotide sequence with nucleotide BLAST, and assign the reference virus based on the best alignment. Verify that extension occurred along the correct path by checking the extended regions match the viral reference, which flags chimeric extensions as inaccurate. Compute gained length per contig as (extended length minus seed contig length) and tabulate all contigs extended by >0.2 kb — in the ContigExtender study this reproduced Tables 4-5, where 45 of 49 combined human/animal contigs extended with high quality and no chimeric extensions, growing seed contigs from an average of 2.8 kb to 4.8 kb (~2 kb gained on average).

## Related tools

- **ContigExtender** (Produces the extended contigs (output/contigs.fasta) that are then verified by BLAST against viral references) — https://github.com/dengzac/contig-extender
- **metaSPAdes** (Generates the initial seed contigs whose lengths are compared against the extended contigs to compute gained length)

## Examples

```
blastn -query output/contigs.fasta -subject BKV.fasta
```

## Evaluation signals

- Each extended contig aligns over its full length to the expected NCBI viral reference nucleotide sequence by nucleotide BLAST, with no chimeric segments
- Gained length per contig (extended minus seed) is >0.2 kb for all tabulated contigs
- Average extended length matches expected ranges (e.g. combined human/animal datasets: 2.8 kb seed to 4.8 kb final, ~2 kb gained)
- High-quality extension rate matches expected results (45 of 49 combined human/animal contigs non-chimeric)
- Output contigs are correctly named reference_N and sorted by length in input order

## Limitations

- Contigs that cannot be extended at all (no gained length) cannot be verified this way; only extendable contigs gain measurable length
- Accuracy depends on availability of an appropriate NCBI viral reference; novel pathogens without strong sequence homology cannot be validated against a reference
- Higher --extend-tolerance values (e.g. 3.0) force extension but risk false extension, which BLAST verification must catch

## Evidence

- [intro] contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST: "contig accuracy, which is measured by similarity to NCBI viral reference nucleotide sequences using nucleotide BLAST"
- [other] Verify accuracy of each extended contig by nucleotide BLAST alignment to the corresponding NCBI viral reference sequence and assign the reference virus.: [evidence span withheld — anchor status: not_found]
- [readme] To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```.: "To verify the accuracy of the extended contig, the reference genome is provided in ```BKV.fasta```."
- [discussion] For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb: "For contigs that are extendable, ContigExtender can accurately increase the contig length by several kb"
