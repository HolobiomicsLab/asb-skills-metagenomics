---
name: gc-content-and-uncalled-base-interpretation
description: 'Use when comparing de novo assemblies (e.g. ABySS, Velvet, SPAdes, MEGAHIT
  contigs in FASTA) and QUAST''s report.tsv G+C content or # N''s per 100 kbp columns
  are needed to detect contamination, divergence from the expected organismal G+C
  range, or uncalled-base problems.'
when_to_use_negative:
- A reference genome is available and reference-based metrics (mismatches, indels,
  genome fraction, duplication ratio, NGA50) are the relevant quality signals
- Input is not assembled contigs but raw reads or a single already-validated assembly
  with no comparison baseline
- Metagenomic datasets, where a single expected G+C range does not apply and MetaQUAST-style
  per-reference evaluation is needed instead
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: 'Computes the reference-free quality metrics including G+C content and # N''s
    per 100 kbp; report.tsv is parsed for these columns'
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web-server interface to QUAST that runs the same reference-free evaluation
    (30 metrics, three assessment plots) used for this interpretation
  repo: https://www.ccb.uni-saarland.de/quast/
- name: BUSCO
  role: Complementary completeness check (98.7% complete across all four E. coli assemblies)
    used alongside G+C/N-content signals
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:a7fe9e330751347bdce1c59400c37a7b47f8dd501bb3cfbddf808d1bd4f5df54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - BUSCO
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# GC-content-and-uncalled-base-interpretation

## Summary

Interprets the G+C content (%) and # N's per 100 kbp columns from QUAST/WebQUAST reference-free evaluation reports to judge assembly purity and base-calling quality, using organism-expected G+C ranges and per-assembler N-content comparisons as calibration (e.g. E. coli K-12 MG1655 expected 50.4–50.8% G+C, observed ~50.7% across all four assemblers).

## When to use

Use when comparing de novo assemblies (e.g. ABySS, Velvet, SPAdes, MEGAHIT contigs in FASTA) and QUAST's report.tsv G+C content or # N's per 100 kbp columns are needed to detect contamination, divergence from the expected organismal G+C range, or uncalled-base problems.

## When NOT to use

- A reference genome is available and reference-based metrics (mismatches, indels, genome fraction, duplication ratio, NGA50) are the relevant quality signals
- Input is not assembled contigs but raw reads or a single already-validated assembly with no comparison baseline
- Metagenomic datasets, where a single expected G+C range does not apply and MetaQUAST-style per-reference evaluation is needed instead

## Inputs

- Assembly contig FASTA files from multiple assemblers (e.g. ABySS, Velvet, SPAdes, MEGAHIT outputs for E. coli K-12 MG1655)
- QUAST report.tsv (tab-separated summary table with G+C content and # N's per 100 kbp rows)
- Expected organismal G+C content range for the species under evaluation

## Outputs

- Extracted G+C content (%) and # N's per 100 kbp values per assembly
- A cross-checked metrics table (report.tsv) with the four assemblies as columns and values validated against expected ranges
- An assessment of which assembly deviates in base composition or uncalled-base content

## How to apply

Run quast.py on all assembler contig FASTA files together without a reference (-o quast_output) so the 30 reference-free quality metrics are computed, then parse the generated report.tsv and extract the average G+C content and # N's per 100 kbp rows. Compare the observed G+C against the expected range for the organism (for E. coli K-12 MG1655 this is 50.4–50.8%; the four test assemblies averaged 50.7%, confirming consistency with the expected range). Inspect # N's per 100 kbp for assembler-specific base-calling artifacts: in the Use Case 1 example, Velvet reported ~94 uncalled bases per 100 kbp while MEGAHIT had none, which flags run-on ambiguity codes rather than genuine fragmentation. Combine these two metrics with contig count and N50/N90/auN (ABySS 176 contigs vs 90–95, SPAdes best N50/N90/auN with a 285 kb largest contig) so that G+C and N-content interpretation is not made in isolation from contiguity metrics.

## Related tools

- **QUAST** (Computes the reference-free quality metrics including G+C content and # N's per 100 kbp; report.tsv is parsed for these columns) — https://github.com/ablab/quast
- **WebQUAST** (Web-server interface to QUAST that runs the same reference-free evaluation (30 metrics, three assessment plots) used for this interpretation) — https://www.ccb.uni-saarland.de/quast/
- **BUSCO** (Complementary completeness check (98.7% complete across all four E. coli assemblies) used alongside G+C/N-content signals)

## Examples

```
./quast.py abyss_contigs.fasta velvet_contigs.fasta spades_contigs.fasta megahit_contigs.fasta -o quast_output
```

## Evaluation signals

- report.tsv exists, is tab-separated, and contains the four assemblies as columns
- Average G+C content falls within the expected E. coli range of 50.4–50.8% (observed ~50.7%)
- # N's per 100 kbp is ~94 for Velvet and 0 for MEGAHIT, matching the reported uncalled-base pattern
- Contig counts match the reported ranges: 176 for ABySS versus 90–95 for Velvet, SPAdes and MEGAHIT
- Total lengths are ~4.8 Mb (ABySS) versus ~4.6 Mb (others) and SPAdes has the largest contig at 285 kb

## Limitations

- G+C content and # N's per 100 kbp are computed only as reference-free metrics; they say nothing about misassemblies, which require a reference genome
- QUAST compiles its sub-parts on first use, so a bare Python3 + GCC 4.7+ environment with zlib dev files is needed before metrics appear
- G+C interpretation depends on an external expected range for the organism; without one, deviation cannot be judged (no quantitative benchmark discussed for such cases)

## Evidence

- [results] ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT): "ABySS produced the largest (4.8 Mb versus 4.6 Mb) but also the most fragmented assembly (176 contigs versus 90–95 for Velvet, SPAdes and MEGAHIT)"
- [results] reference-free metric count: "When a reference genome is unavailable, WebQUAST computes 30 quality metrics and draws three assessment plots"
- [readme] report.tsv purpose: "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
- [full_text] expected G+C range calibration from the source card finding: [evidence span withheld — anchor status: not_found]
