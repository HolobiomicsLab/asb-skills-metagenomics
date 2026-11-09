---
name: jaccard-similarity-computation-on-read-sets
description: Use when you need to quantify concordance between two or more read-level
  metagenomic classifiers by comparing their classified-read sets with Jaccard similarity
  and per-tool missed-read tallies.
when_to_use_negative:
- Evaluating classification accuracy against known ground truth on simulated metagenomes
  (e.g. setA2/setB2 from the Lindgreen et al. benchmark), where sensitivity/precision/F1/specificity
  metrics are the appropriate evaluation
- The input is raw unfiltered reads or taxon-abundance profiles rather than per-read
  classification outputs with read identifiers
- Comparing tools run on different reference databases or differently filtered inputs
  — set overlap is then confounded by preprocessing differences
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: LiME
  role: Produces the per-read classification file (C/U/A/H,IdSeqRead,TaxID,maxSim)
    whose classified-read set is compared to Magic-BLAST's via Jaccard similarity
  repo: https://github.com/veronicaguerrini/LiME
- name: MagicBlast
  role: Reference alignment-based classifier whose classified-read set is the comparison
    anchor for the agreement table
- name: CLARK-S
  role: Comparator classifier; its classified-read set is tallied against Magic-BLAST
    and LiME in the agreement table
- name: Centrifuge
  role: Comparator classifier included in the Jaccard-based concordance evaluation
- name: Kraken 2
  role: Comparator classifier included in the Jaccard-based concordance evaluation
- name: TaxMaps
  role: Comparator classifier included in the Jaccard-based concordance evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:6ac405a8ee4834963a23b92fcc9741c2fce32e12e033b26cd28520f387f61b8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Jaccard similarity computation on read sets

## Summary

Computes Jaccard-based agreement rates between the sets of classified reads produced by different metagenomic classifiers (LiME vs Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) on the BLAST-filtered HMP real metagenome SRR1804065, reproducing Table S5-style concordance values.

## When to use

Use when you need to quantify concordance between two or more read-level metagenomic classifiers by comparing their classified-read sets with Jaccard similarity and per-tool missed-read tallies.

## When NOT to use

- Evaluating classification accuracy against known ground truth on simulated metagenomes (e.g. setA2/setB2 from the Lindgreen et al. benchmark), where sensitivity/precision/F1/specificity metrics are the appropriate evaluation
- The input is raw unfiltered reads or taxon-abundance profiles rather than per-read classification outputs with read identifiers
- Comparing tools run on different reference databases or differently filtered inputs — set overlap is then confounded by preprocessing differences

## Inputs

- Per-read classification file from LiME (comma-separated, fields C/U/A/H,IdSeqRead,TaxID,maxSim)
- Per-read classification outputs from Magic-BLAST, CLARK-S, Centrifuge, Kraken 2 and TaxMaps on the same BLAST-filtered metagenome
- BLAST-filtered real metagenome read set SRR1804065 (mapped against the whole nt database at 98% sequence identity)

## Outputs

- Pairwise Jaccard agreement rates (rid, ras) between classified-read sets of each tool pair
- Tally of reads classified identically by both tools and reads assigned to the same taxon
- Per-tool count of Magic-BLAST-classified reads missed by LiME, CLARK-S, Centrifuge, Kraken 2 and TaxMaps (Table S5-style agreement table)

## How to apply

Parse each tool's per-read classification output (for LiME the comma-separated file with fields C/U/A/H,IdSeqRead,TaxID,maxSim, keeping only lines marked C) and extract the set of classified read identifiers plus, for agreement at the taxon level, the TaxID assignments. Compute the Jaccard similarity coefficient between pairs of classified-read sets to obtain the agreement rates (rid and ras) reported by the paper, tallying reads classified identically by both tools, reads assigned to the same taxon, and reads classified by the reference tool (Magic-BLAST) but missed by each comparator. Benchmark against the paper's Table S5 values on the real metagenome SRR1804065: LiME agrees with Magic-BLAST on ~95.2% of classified reads and 93.6% of reads are assigned to the same taxon, with LiME missing 42,788 Magic-BLAST-classified reads. Rationale: since classifiers label individual reads, set-based overlap of classified-read identifiers is a tool-agnostic way to compare outputs regardless of the underlying algorithm (eBWT-based LiME vs alignment- or k-mer-based tools).

## Related tools

- **LiME** (Produces the per-read classification file (C/U/A/H,IdSeqRead,TaxID,maxSim) whose classified-read set is compared to Magic-BLAST's via Jaccard similarity) — https://github.com/veronicaguerrini/LiME
- **MagicBlast** (Reference alignment-based classifier whose classified-read set is the comparison anchor for the agreement table)
- **CLARK-S** (Comparator classifier; its classified-read set is tallied against Magic-BLAST and LiME in the agreement table)
- **Centrifuge** (Comparator classifier included in the Jaccard-based concordance evaluation)
- **Kraken 2** (Comparator classifier included in the Jaccard-based concordance evaluation)
- **TaxMaps** (Comparator classifier included in the Jaccard-based concordance evaluation)

## Examples

```
python -c "A={l.split(',')[1] for l in open('lime.csv') if l.startswith('C')}; B={l.split(',')[1] for l in open('magicblast.csv') if l.startswith('C')}; print(len(A&B)/len(A|B))"
```

## Evaluation signals

- LiME agrees with Magic-BLAST on ~95.2% of all classified reads (identical classification) on SRR1804065
- ~93.6% of reads are assigned to the same taxon by both LiME and Magic-BLAST
- LiME misses 42,788 reads classified by Magic-BLAST; per-tool missed counts for CLARK-S, Centrifuge, Kraken 2 and TaxMaps reproduce Table S5 within tolerance
- Extracted classified-read sets contain only lines/entries marked Classified (C) in each tool's output; LiME output fields parse to (label, IdSeqRead, TaxID, maxSim)
- All Jaccard values fall in [0,1] and are symmetric across tool pairs

## Limitations

- The comparison is run on the real metagenome pre-filtered with BLAST at 98% sequence identity, so agreement rates do not reflect performance on unfiltered raw reads
- Version numbers of the compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) are not stated in the article
- Reads left unclassified by either tool are excluded from the classified-read-set comparison, so the Jaccard rates do not capture differences in classification coverage alone

## Evidence

- [results] Jaccard-based agreement computation on classified read sets: "we compute the similarity between the two sets of classified reads by using the Jaccard similarity coefficient"
- [results] Headline agreement value between LiME and Magic-BLAST: [evidence span withheld — anchor status: not_found]
- [readme] LiME output format used to extract classified-read identifiers: "C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1)."
- [abstract] Motivation for the concordance evaluation on the real metagenome: "we show that LiME is able to deliver classification results comparable to that of MagicBlast"
- [results] Filtering step that defines the shared evaluation dataset: "we first filtered this dataset by using BLAST: we mapped reads against the whole nucleotide sequence database with a sequence identity of 98%"
