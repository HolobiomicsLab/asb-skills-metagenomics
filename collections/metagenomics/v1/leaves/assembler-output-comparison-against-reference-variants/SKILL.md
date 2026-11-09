---
name: assembler-output-comparison-against-reference-variants
description: Use when shotgun metagenomic reads from a mock/defined community have
  been processed through several SSU extraction and reference-targeted assembly combinations
  and you must decide which combination recovers the most known 16S rRNA variants.
when_to_use_negative:
- No ground-truth panel of reference 16S variants exists for the input community,
  so recovery cannot be scored
- Input is already amplified 16S amplicon data rather than shotgun metagenomic reads,
  where extractor/assembler benchmarking against full-length reconstruction is not
  the analysis goal
- You only need taxonomic classification of reads, not evaluation of which SSU reconstruction
  combination best recovers variants
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: BBMap
  role: General-purpose read mapper used for one arm of SSU read extraction against
    indexed SILVA SSU 138.1
- name: SortMeRNA
  role: rRNA-specialized tool used for the other arm of SSU read extraction; its reads
    assemble better than BBMap's
- name: EMIRGE (Emirge.py / Emirge_amplicon.py)
  role: Reference-targeted assembler run with IDENTITY 1, 40 iterations and MIN_COV
    0.3 to reconstruct full-length SSU sequences
  repo: https://github.com/oschakoory/RiboTaxa
- name: MATAM
  role: Reference-targeted assembler benchmarked on both extracted read sets
- name: MetaRib
  role: Reference-targeted assembler run with 1,000,000-read subsampling per iteration;
    combined with Emirge_amplicon.py for best population characterisation
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBTools
  role: Adapter and quality trimming of raw paired-end reads before extraction (ktrim=r,
    kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1)
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:571591bb2e7dce91dda236cba6df7f0d7a35dcbc65ecfb12ebdef152e34effe3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - EMIRGE (Emirge.py / Emirge_amplicon.py)
  - MATAM
  - MetaRib
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# assembler-output-comparison-against-reference-variants

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple SSU rRNA read-extraction/assembler combinations by aligning each set of reconstructed full-length 16S sequences against a known panel of reference 16S variants, tabulating recovered variants, microorganisms and sequence length distributions.

## When to use

Use when shotgun metagenomic reads from a mock/defined community have been processed through several SSU extraction and reference-targeted assembly combinations and you must decide which combination recovers the most known 16S rRNA variants.

## When NOT to use

- No ground-truth panel of reference 16S variants exists for the input community, so recovery cannot be scored
- Input is already amplified 16S amplicon data rather than shotgun metagenomic reads, where extractor/assembler benchmarking against full-length reconstruction is not the analysis goal
- You only need taxonomic classification of reads, not evaluation of which SSU reconstruction combination best recovers variants

## Inputs

- Paired-end shotgun metagenomic FASTQ from a mock community with a known set of reference 16S rRNA variants (e.g. SRA SRR5381736)
- Indexed SILVA SSU 138.1 database for SortMeRNA and BBMap read extraction
- Trimmed read sets produced by SortMeRNA and by BBMap SSU extraction
- List of known reference 16S rRNA variants (e.g. 41 mock community variants) for recovery comparison

## Outputs

- Table over all 12 extractor/assembler combinations listing number of reconstructed SSU sequences, their length distribution, and detected reference variants/microorganisms
- Per-combination set of reconstructed full-length 16S rRNA sequences (e.g. 53 sequences from the combined Emirge_amplicon.py + MetaRib strategy at 97% clustering)

## How to apply

Preprocess the paired-end mock community raw reads (e.g. SRA SRR5381736) with BBTools trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1), then extract SSU reads twice — with SortMeRNA and with BBMap, both against the indexed SILVA SSU 138.1 database. Reconstruct full-length SSU sequences with each of four assemblers (Emirge.py, Emirge_amplicon.py, MATAM, MetaRib) per read set using EMIRGE settings IDENTITY threshold 1, 40 iterations, MIN_COV 0.3, and MetaRib subsampling of 1,000,000 reads per iteration, giving 12 extractor/assembler combinations. For each combination, align or compare every reconstructed sequence against the reference panel (e.g. 41 known mock 16S variants) to score which variants and microorganisms are recovered, and tabulate sequence counts and length distributions. Judge success by full coverage of the 12-combination table, near-full-length reconstructions, and reproduction of the qualitative outcome that SortMeRNA-extracted reads assemble better than BBMap-extracted reads and that combining Emirge_amplicon.py with MetaRib yields greater population characterisation.

## Related tools

- **BBMap** (General-purpose read mapper used for one arm of SSU read extraction against indexed SILVA SSU 138.1)
- **SortMeRNA** (rRNA-specialized tool used for the other arm of SSU read extraction; its reads assemble better than BBMap's)
- **EMIRGE (Emirge.py / Emirge_amplicon.py)** (Reference-targeted assembler run with IDENTITY 1, 40 iterations and MIN_COV 0.3 to reconstruct full-length SSU sequences) — https://github.com/oschakoory/RiboTaxa
- **MATAM** (Reference-targeted assembler benchmarked on both extracted read sets)
- **MetaRib** (Reference-targeted assembler run with 1,000,000-read subsampling per iteration; combined with Emirge_amplicon.py for best population characterisation) — https://github.com/oschakoory/RiboTaxa
- **BBTools** (Adapter and quality trimming of raw paired-end reads before extraction (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1)) — https://github.com/oschakoory/RiboTaxa

## Examples

```
bbduk.sh in=SRR5381736_R1.fq in2=SRR5381736_R2.fq out=trimmed_R1.fq out2=trimmed_R2.fq ktrim=r kmer=21 minlength=60 trimq=20 qtrim=rl maxns=1
```

## Evaluation signals

- Benchmark table covers all 12 extractor/assembler combinations (2 extractors x 4 assemblers)
- Best combined strategy (Emirge_amplicon.py on SortMeRNA-filtered reads + MetaRib on unfiltered reads, 97% clustering) reconstructs sequences identifying all 20 detectable mock microorganisms and the 41 expected 16S rRNA variants
- Qualitative outcome reproduced: SortMeRNA-extracted reads assemble better than BBMap-extracted reads across the assemblers
- Reconstructed sequences show near-full-length 16S rRNA length distributions, consistent with full-length SSU reconstruction
- EMIRGE parameter settings applied as reported: IDENTITY threshold 1, 40 iterations, MIN_COV 0.3

## Limitations

- Chimera detection was not included in the 16S rRNA gene reconstruction workflow, and chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of the chimera
- Artificial in silico diversity created during full-length sequence reconstruction cannot be excluded, and occasional small indel errors may occur in reconstructed sequences
- High conservation of 16S rRNA sequence cannot discriminate all species (e.g. Enterobacteriaceae, Lachnospiraceae), so some sequences stop at genus-level assignment

## Evidence

- [discussion] SortMeRNA extracts reads that assemble better: "SortMeRNA performed better than BBMap in extracting reads that could be assembled using different tools such as EMIRGE (Emirge.py and Emirge_amplicon.py), MATAM and MetaRib."
- [discussion] Combining assemblers improves characterisation: "Thus, combining these two tools for SSU reconstruction led a greater population characterisation."
- [methods] EMIRGE merging threshold: "This the JOIN_TRESHOLD parameter of EMIRGE. If two candidate sequences share >= this fractional identity over their bases with mapped reads, then merge the two sequences"
- [methods] MIN_COV coverage threshold: "minimum fraction of the length of a candidate reference sequence that must be covered by mapped reads (Default=0.3)"
- [readme] Pipeline input formats: "RiboTaxa takes as input singled-end or paired-end files which can be in compressed format (fastq.gz) or uncompressed format (.fastq)."
