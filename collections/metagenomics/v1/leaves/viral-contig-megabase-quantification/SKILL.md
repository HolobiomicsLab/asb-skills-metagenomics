---
name: viral-contig-megabase-quantification
description: Use when comparing viral recovery across tools or pipelines on metagenomic
  (virome) FASTQ samples, e.g. to express recovered viral sequence as contig megabases
  and taxa-per-sample counts under matched filtering.
when_to_use_negative:
- Input is already a filtered taxa ID table or feature table — this skill quantifies
  from raw contig/alignment outputs and would double-filter
- Dataset is not viral metagenomic sequence data (e.g. 16S amplicon or host-only data)
  — contigs mapping outside the viral superkingdom are filtered and metrics would
  be near zero
- No matched filtering thresholds can be applied across pipelines — megabase and taxa
  counts are only comparable at equivalent bits-per-contig/bits-per-taxon cutoffs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0781
tools:
- name: VirMAP
  role: Produces the virus-like super-scaffolds and filtered taxa tables whose contig
    megabases and taxa counts are quantified
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: De-novo assembler invoked by VirMAP (--useMegahit) to generate contigs counted
    as viral megabases
  repo: https://github.com/voutcn/megahit
- name: DIAMOND
  role: Translated/protein alignment engine (virDmnd/gbBlastx) behind VirMAP's nucleotide-plus-protein
    classification feeding the quantification
  repo: https://github.com/bbuchfink/diamond
- name: fasterq-dump
  role: Retrieves the SRA samples (e.g. SRR9875293) that are the input to quantification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:1beca8dffe3c34f0b1a30fe751d04c0dba1527f40889f862ac2bb4178740e055
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - VirMAP
  - fasterq-dump
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# viral-contig-megabase-quantification

## Summary

Quantifies viral sequence content of virome samples as viral contig megabases and related per-sample metrics (filtered taxa IDs, contigs-per-taxon ratios, taxa-per-sample counts) from VirMAP or standard assembly-and-mapping outputs, aggregating them across subsampling levels into a comparison CSV.

## When to use

Use when comparing viral recovery across tools or pipelines on metagenomic (virome) FASTQ samples, e.g. to express recovered viral sequence as contig megabases and taxa-per-sample counts under matched filtering.

## When NOT to use

- Input is already a filtered taxa ID table or feature table — this skill quantifies from raw contig/alignment outputs and would double-filter
- Dataset is not viral metagenomic sequence data (e.g. 16S amplicon or host-only data) — contigs mapping outside the viral superkingdom are filtered and metrics would be near zero
- No matched filtering thresholds can be applied across pipelines — megabase and taxa counts are only comparable at equivalent bits-per-contig/bits-per-taxon cutoffs

## Inputs

- Four-sample Brazil sewage virome FASTQ files from SRA BioProject PRJNA395784
- VirMAP reference database files (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) built from Genbank viral (gbvrl) and phage (gbphg) divisions
- Per-sample VirMAP raw unfiltered output and filtered table output, plus standard-approach contigs/taxa assignments

## Outputs

- Single CSV aggregating per-sample filtered taxa IDs, viral contig megabases, contigs-per-taxon ratios, and taxa-per-sample counts at 10%, 1%, and 0.1% subsampling levels
- Per-sample VirmapRun.err stderr logs and per-sample output directories

## How to apply

Run each sample (e.g. the four Brazil sewage samples from BioProject PRJNA395784 retrieved with fasterq-dump) through Virmap.pl with --useMegahit --useBbnorm against a VirMAP database built by makeVirmapDb.pl (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd), and in parallel through the standard assembly-and-mapping approach. Apply the comparison filters consistently: raw VirMAP output at a 300 bits/contig minimum and filtered table output at a 1000 bits/taxa ID minimum, versus 300 aggregate bits for the standard approach. For each sample and each subsampling level (10%, 1%, 0.1%), compute filtered taxa IDs, viral contig megabases, contigs-per-taxon ratios, and taxa-per-sample counts, then aggregate all per-sample metrics into a single CSV. Rationale: VirMAP merges nucleotide and protein alignments to classify viral reconstructions independent of genome coverage, so megabase and taxa counts reflect information recovery rather than only high-coverage contigs.

## Related tools

- **VirMAP** (Produces the virus-like super-scaffolds and filtered taxa tables whose contig megabases and taxa counts are quantified) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (De-novo assembler invoked by VirMAP (--useMegahit) to generate contigs counted as viral megabases) — https://github.com/voutcn/megahit
- **DIAMOND** (Translated/protein alignment engine (virDmnd/gbBlastx) behind VirMAP's nucleotide-plus-protein classification feeding the quantification) — https://github.com/bbuchfink/diamond
- **fasterq-dump** (Retrieves the SRA samples (e.g. SRR9875293) that are the input to quantification)

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293 --readF /home/$USER/VirmapTest/SRR9875293_1.fastq
```

## Evaluation signals

- Compiled per-sample CSV reproduces the article's reported values: 1.69 vs 4.3 viral contig megabases for VirMAP and 1.04 vs 4.0 for the standard approach
- Filtered taxa ID averages match reported figures: 231 taxa IDs/sample (VirMAP, 1000 bits/taxa ID minimum) vs 236 (standard, 300 aggregate bits)
- Raw unfiltered counts average ~423 taxa IDs/sample for VirMAP (300 bits/contig) and ~480 for the standard approach
- Metrics are present for all four samples at all three subsampling levels (10%, 1%, 0.1%) in the CSV
- Contig filtering applied: contigs mapping outside the viral superkingdom excluded before megabase summation

## Limitations

- Comparison requires building the VirMAP reference database (makeVirmapDb.pl), which can take over an hour, and minimum SSD/RAM requirements grow as Genbank expands
- VirMAP's sensitivity/specificity tradeoff depends on tunable parameters (Supplementary Data 14) not provided in the paper text, and no software versions were stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole
- The Genbank database snapshot/version or download date is unspecified, so megabase and taxa counts may shift when the reference database is rebuilt

## Evidence

- [discussion] VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs.: "VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs."
- [discussion] Viral superkingdom filter: "Contigs that map outside the viral superkingdom are filtered."
- [methods] Virmap.pl command example: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [abstract] Merging nucleotide and protein classification rationale: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [methods] VirMAP database build scale caveat: "As Genbank expands, the minimum SSD and RAM requirements will expand as well."
