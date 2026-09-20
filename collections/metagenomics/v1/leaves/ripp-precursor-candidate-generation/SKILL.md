---
name: ripp-precursor-candidate-generation
description: Use when you have microbial (or metagenomic, assembled into contigs)
  nucleotide FASTA and need to enumerate putative RiPP precursor ORFs — either a small,
  high-confidence set (motif-ORF) or an exhaustive set (all-ORF) — for downstream
  peptide-spectrum matching.
when_to_use_negative:
- When searching spectra from a producer whose genome is unavailable — as with the
  lacticin producer, search spectra against its previously described biosynthetic
  gene clusters instead of generating candidates from a genome
- When the analysis target is non-RiPP compound classes (e.g., many GNPS spectral
  families belonging to non-ribosomal peptides); use Dereplicator+ structure-based
  search instead
- 'When candidate sensitivity matters more than search space size: motif-ORF is not
  sensitive and misses peptides without sequence similarity to known RiPPs'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0080
tools:
- name: MetaMiner
  role: Pipeline implementing 6-frame translation, HMMer enzyme detection, 10 kbp
    BGC window construction, and motif-ORF/all-ORF candidate generation; available
    as a command line tool and GNPS workflow
  repo: https://github.com/mohimanilab/MetaMiner
- name: HMMer
  role: Detects RiPP modification enzymes on six-frame translated sequences to anchor
    putative BGC windows
- name: antiSMASH
  role: Detects biosynthetic gene clusters and classifies RiPP-encoding clusters in
    input genomes (e.g., 1,140 BGCs, 168 RiPP-encoding, across 36 ACTI genomes)
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.cels.2019.09.004
    title: 'MetaMiner: A Scalable Peptidogenomics Approach for Discovery of Ribosomal
      Peptide Natural Products with Blind Modifications from Microbial Communities'
  source_content_hash: sha256:9bcdbfa6639df072b42e14e7e1a47661330dbab5c2772ee54d75ad619baede3e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mohimanilab/MetaMiner
  tools:
  - HMMer
  - MetaMiner
  - antiSMASH
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ripp-precursor-candidate-generation

## Summary

Generates candidate RiPP precursor peptide ORFs from microbial genomes using MetaMiner's two strategies: motif-ORF (motif/HMMer-enzyme anchored) and all-ORF (exhaustive 6-frame translation within putative BGC windows). This candidate database is the search space that MetaMiner/Dereplicator-based scoring queries with mass spectra.

## When to use

Use when you have microbial (or metagenomic, assembled into contigs) nucleotide FASTA and need to enumerate putative RiPP precursor ORFs — either a small, high-confidence set (motif-ORF) or an exhaustive set (all-ORF) — for downstream peptide-spectrum matching.

## When NOT to use

- When searching spectra from a producer whose genome is unavailable — as with the lacticin producer, search spectra against its previously described biosynthetic gene clusters instead of generating candidates from a genome
- When the analysis target is non-RiPP compound classes (e.g., many GNPS spectral families belonging to non-ribosomal peptides); use Dereplicator+ structure-based search instead
- When candidate sensitivity matters more than search space size: motif-ORF is not sensitive and misses peptides without sequence similarity to known RiPPs

## Inputs

- Assembled nucleotide contigs or strain genomes in nucleotide FASTA format (e.g., 36 ACTI RefSeq genomes)
- Raw metagenomic/metatranscriptomic short reads (assembled into contigs by MetaMiner first)
- RiPP modification-enzyme profile HMMs for HMMer (motif-ORF mode)
- Known RiPP sequence motifs (motif-ORF mode)

## Outputs

- Candidate RiPP precursor peptide database (e.g., all_orf_candidates.fasta) with ORFs annotated by parent 10 kbp putative BGC window
- Per-genome counts of short ORF candidates under motif-ORF and all-ORF modes (e.g., acti_bgc_orf_counts.csv)
- Decoy database of randomly shuffled precursor peptides for FDR estimation
- 10 kbp putative biosynthetic gene cluster windows centered on HMMer-detected modification enzymes

## How to apply

First prepare the genome: if input is short reads, assemble them into contigs; otherwise use assembled nucleotide FASTA directly. Translate each contig into protein sequence in all six frames, then run HMMer (Eddy, 2011) with RiPP modification-enzyme profile HMMs against the translated sequences to detect modification enzymes; construct a 10 kbp window centered on each hit, labeling it a putative biosynthetic gene cluster. In all-ORF mode, extract every ORF within each 10 kbp window shorter than the pre-defined length threshold (default 200 amino acids), which exhaustively covers candidates without requiring sequence similarity to known RiPPs at the cost of a much larger search space. In motif-ORF mode, restrict candidates to ORFs with sequence similarity to known RiPP motifs, dramatically shrinking the candidate set (e.g., 67 vs 565,138 short ORFs across 36 ACTI genomes, a ~4-order-of-magnitude reduction). Optionally construct a decoy database by randomly shuffling each target precursor peptide for downstream target-decoy FDR estimation. Write candidate ORFs (e.g., to all_orf_candidates.fasta) annotated with their parent putative BGC window for spectral searching via MetaMiner's modified Dereplicator with p-values computed by MS-DPR and a 1% FDR threshold.

## Related tools

- **MetaMiner** (Pipeline implementing 6-frame translation, HMMer enzyme detection, 10 kbp BGC window construction, and motif-ORF/all-ORF candidate generation; available as a command line tool and GNPS workflow) — https://github.com/mohimanilab/MetaMiner
- **HMMer** (Detects RiPP modification enzymes on six-frame translated sequences to anchor putative BGC windows)
- **antiSMASH** (Detects biosynthetic gene clusters and classifies RiPP-encoding clusters in input genomes (e.g., 1,140 BGCs, 168 RiPP-encoding, across 36 ACTI genomes))

## Examples

```
python -c "from Bio.Seq import Seq; recs=[r for r in SeqIO.parse('genome.fna','fasta')]; [print('>orf|%s|%d\n%s'%(r.id,frame,Seq(str(r.seq)[off:].translate(table=11,to_stop=False)).replace('*','X'))) for r in recs for frame in range(6) for off in range(3)]"
```

## Evaluation signals

- all_orf_candidates.fasta exists and is valid FASTA; every reported ORF sequence length is below the 200 amino acid threshold
- Each candidate ORF maps within a 10 kbp window centered on an HMMer-detected modification enzyme
- Aggregate counts reproduce known benchmarks: 67 motif-ORF vs 565,138 all-ORF short ORFs across 36 ACTI genomes (2 vs 14,694 for Streptomyces roseosporus NRRL 11379)
- antiSMASH cluster tally matches expected totals (1,140 BGCs including 168 RiPP-encoding clusters on the ACTI genomes)
- Decoy database contains one shuffled peptide per target precursor peptide, enabling target-decoy 1% FDR estimation downstream

## Limitations

- Motif-ORF strategy is not sensitive: peptides without sequence similarity to known RiPPs are missed
- All-ORF mode yields a much larger search space (e.g., 14,694 short ORFs across six BGCs in one genome), increasing computational cost
- Only ~1% of GNPS spectra are accompanied by genomics/metagenomics data, limiting how often this candidate-generation step can be paired with spectra
- 99% of GNPS spectra cannot be searched against genomes, so candidates generated here only support a small fraction of public spectral data

## Evidence

- [readme] MetaMiner is available as a workflow at the Global Natural Products Social Molecular Networking (GNPS).: [evidence span withheld — anchor status: not_found]
