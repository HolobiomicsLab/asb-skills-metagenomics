---
name: genome-fraction-and-duplication-ratio-computation
description: Use when one or more genome assemblies (FASTA contigs) must be scored
  against a known reference genome to quantify how much of the reference is covered
  and whether contigs redundantly cover the same regions.
when_to_use_negative:
- No reference genome (even a close one) is available — QUAST is 'much more informative
  if at least a close reference genome is provided'; reference-free mode will not
  yield genome fraction or duplication ratio
- Large genomes (e.g., mammalian) better suited to the QUAST-LG extension rather than
  the standard E. coli-scale workflow
- Goal is functional completeness assessment via conserved genes — BUSCO/CEGMA-style
  gene-based evaluation is a different skill
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: QUAST
  role: Aligns contigs to the supplied reference (-r) and computes genome fraction
    %, duplication ratio, NGA50, and mismatch/indel counts, exporting them as report.tsv
  repo: https://github.com/ablab/quast
- name: WebQUAST
  role: Web server providing the same QUAST reference-based evaluation with >60 quality
    metrics, eight assessment plots and two Icarus viewers through a graphical interface
  repo: https://www.ccb.uni-saarland.de/quast/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:4c8813a9d8d4eab0ac7f1525b410d2860f100f51b990fef9a6e1cf6a8cca6bae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - WebQUAST
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# genome-fraction-and-duplication-ratio-computation

## Summary

Compute reference-based completeness and redundancy metrics — genome fraction %, duplication ratio, and NGA50 — by aligning contigs to a reference genome with QUAST/WebQUAST, as demonstrated on four E. coli K-12 MG1655 assemblies.

## When to use

Use when one or more genome assemblies (FASTA contigs) must be scored against a known reference genome to quantify how much of the reference is covered and whether contigs redundantly cover the same regions.

## When NOT to use

- No reference genome (even a close one) is available — QUAST is 'much more informative if at least a close reference genome is provided'; reference-free mode will not yield genome fraction or duplication ratio
- Large genomes (e.g., mammalian) better suited to the QUAST-LG extension rather than the standard E. coli-scale workflow
- Goal is functional completeness assessment via conserved genes — BUSCO/CEGMA-style gene-based evaluation is a different skill

## Inputs

- FASTA contig files for each assembly (e.g., MEGAHIT, SPAdes, Velvet, ABySS outputs)
- Reference genome FASTA (e.g., E. coli K-12 MG1655 NC_000913.3), optionally gzipped
- Optional paired-end reads FASTQ (e.g., SRA ERR008613) supplied as -1/-2
- Optional gene annotation (e.g., GCF_000005845.2) supplied via -g

## Outputs

- report.tsv with genome fraction %, duplication ratio, NGA50, mismatches/indels per 100 kb
- report.txt/report.html combined summary tables and assessment plots
- contigs_reports/misassemblies_report and unaligned_report when a reference is given
- icarus.html interactive misassembly and unaligned-contig viewers

## How to apply

Run quast.py on the assemblies with the reference genome supplied via -r (e.g., E. coli K-12 MG1655, accession NC_000913.3 / GCF_000005845.2) and read pairs via -1/-2 to also enable read-pair-based misassembly detection. QUAST aligns contigs to the reference, splits alignments at misassembly breakpoints, and reports genome fraction % (assembled part of the reference), duplication ratio (total aligned bases in the assembly divided by those in the reference), NGA50, and mismatches/indels per 100 kb. Export the tab-separated summary as report.tsv for parsing. In the WebQUAST use case on four E. coli assemblies, ABySS showed genome fraction 98.7% versus 98.0–98.4% for MEGAHIT, SPAdes, and Velvet, while MEGAHIT and SPAdes had zero extensive misassemblies versus four each for Velvet and ABySS; verify output by checking report.tsv against such expected values.

## Related tools

- **QUAST** (Aligns contigs to the supplied reference (-r) and computes genome fraction %, duplication ratio, NGA50, and mismatch/indel counts, exporting them as report.tsv) — https://github.com/ablab/quast
- **WebQUAST** (Web server providing the same QUAST reference-based evaluation with >60 quality metrics, eight assessment plots and two Icarus viewers through a graphical interface) — https://www.ccb.uni-saarland.de/quast/

## Examples

```
quast.py megahit.fasta spades.fasta velvet.fasta abyss.fasta -r mg1655_NC_000913.3.fasta.gz -1 reads1.fastq.gz -2 reads2.fastq.gz -o quast_out
```

## Evaluation signals

- report.tsv contains genome fraction % and duplication ratio rows populated only when -r was supplied; their absence indicates the reference was not correctly passed
- Duplication ratio near 1.0 indicates contigs do not redundantly cover reference regions; values significantly exceeding 1 indicate overestimated repeat multiplicities or contig overlaps
- Cross-check reported values against Use Case 2 expectations: 0 extensive misassemblies for MEGAHIT and SPAdes versus 4 each for Velvet and ABySS, ABySS genome fraction 98.7% versus 98.0–98.4% for the others
- SPAdes per-base accuracy approximately 1.0 mismatches and 0.3 indels per 100 kb should reproduce in the report
- contigs_reports/ directory must exist with misassemblies_report and unaligned_report when a reference genome is provided

## Limitations

- Genome fraction and duplication ratio require a reference; with no reference QUAST only computes contig-based metrics such as N50 and number of large contigs (>500 bp)
- A high duplication ratio can arise from multiple causes, 'including overestimating repeat multiplicities and overlaps between contigs', so it cannot pinpoint a single error type
- A close but non-identical reference (e.g., W3110 for MG1655) changes the interpretation of structural errors relative to the true reference

## Evidence

- [readme] Genome fraction %, assembled part of the reference. Duplication ratio, the total number of aligned bases in the assembly divided by the total number of those in the reference.: "Genome fraction %, assembled part of the reference. Duplication ratio, the total number of aligned bases in the assembly divided by the total number of those in the reference."
- [results] the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers: "the tool reports >60 quality metrics accompanied by eight assessment plots and two Icarus viewers"
- [results] we evaluated the same four assemblies of the E. coli K-12 MG1655 dataset but selected the reference genome differently: "we evaluated the same four assemblies of the E. coli K-12 MG1655 dataset but selected the reference genome differently"
- [readme] The QUAST package works both with and without reference genomes. However, it is much more informative if at least a close reference genome is provided along with the assemblies.: "The QUAST package works both with and without reference genomes. However, it is much more informative if at least a close reference genome is provided along with the assemblies."
- [readme] report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc): "report.tsv      tab-separated version, for parsing, or for spreadsheets (Google Docs, Excel, etc)"
