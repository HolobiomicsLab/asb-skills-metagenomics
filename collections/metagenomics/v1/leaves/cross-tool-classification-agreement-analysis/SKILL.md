---
name: cross-tool-classification-agreement-analysis
description: Use when you have per-read classification files from several metagenomic
  classifiers run on the same read set and need to measure pairwise or reference-based
  agreement (e.g. Jaccard-based rates) rather than accuracy against ground truth.
when_to_use_negative:
- On simulated metagenomes with known true composition, where sensitivity/precision/F1/specificity
  against ground truth (not inter-tool agreement) is the appropriate evaluation
- When tools were run on differently filtered read sets or different reference databases,
  since agreement rates would conflate preprocessing differences with classifier behavior
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: LiME
  role: Classifier under evaluation; produces per-read classification file (C/U/A/H,IdSeqRead,TaxID,maxSim)
    compared against Magic-BLAST
  repo: https://github.com/veronicaguerrini/LiME
- name: MagicBlast
  role: Reference comparator whose classified read set anchors the agreement computation
- name: BLAST
  role: Filters the real metagenome at 98% sequence identity and provides taxonomic
    information for the customized reference database
- name: CLARK-S
  role: State-of-the-art classifier run on the same filtered metagenome for agreement
    comparison
- name: Centrifuge
  role: Mapping-based classifier run on the same filtered metagenome for agreement
    comparison
- name: Kraken 2
  role: k-mer-based classifier run on the same filtered metagenome for agreement comparison
- name: TaxMaps
  role: Mapping-based classifier run on the same filtered metagenome for agreement
    comparison
- name: eGSA
  role: Builds ebwt/lcp/da data structures for read and genome sets prior to LiME
    classification
  repo: https://github.com/felipelouza/egsa
- name: eGap
  role: Merges read and genome ebwt data structures with --lcp and --da options
  repo: https://github.com/felipelouza/egap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:aeede388e09ec2aa004fd6ddf867e4146d02e74bc6c1a2f4a9f3aa6ee8844898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BLAST
  - CLARK-S
  - Centrifuge
  - Kraken 2
  - LiME
  - MagicBlast
  - TaxMaps
  - eGSA
  - eGap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-tool-classification-agreement-analysis

## Summary

Quantify concordance between per-read taxonomic classification outputs of multiple metagenomic classifiers on the same (filtered) real metagenome, using Jaccard-based agreement rates with one tool (Magic-BLAST) as the reference comparator. In the LiME paper this established that LiMEeBWT agrees with Magic-BLAST on ~95.2% of classified reads on HMP sample SRR1804065.

## When to use

Use when you have per-read classification files from several metagenomic classifiers run on the same read set and need to measure pairwise or reference-based agreement (e.g. Jaccard-based rates) rather than accuracy against ground truth.

## When NOT to use

- On simulated metagenomes with known true composition, where sensitivity/precision/F1/specificity against ground truth (not inter-tool agreement) is the appropriate evaluation
- When tools were run on differently filtered read sets or different reference databases, since agreement rates would conflate preprocessing differences with classifier behavior

## Inputs

- BLAST-filtered real metagenome read set (e.g. SRR1804065 paired-end 100 bp reads, filtered at 98% sequence identity against the nt database)
- Customized reference database built from BLAST-derived taxonomic information
- Per-read classification output files from each classifier (LiME: comma-separated C/U/A/H,IdSeqRead,TaxID,maxSim; plus Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps outputs)

## Outputs

- Jaccard-based agreement rates (rid, ras) between each tool's classified read set and Magic-BLAST's classified read set
- Tallies of reads classified identically to Magic-BLAST and reads classified by Magic-BLAST but missed by each tool (reproducing Table S5)
- Parsed per-read classification tables (read ID and assigned TaxID per tool)

## How to apply

First place all classifiers on the same footing: filter the real metagenome (e.g. HMP sample SRR1804065) with BLAST by mapping reads against the whole nt database at 98% sequence identity and build the customized reference database from the BLAST-derived taxonomic information. Run each classifier on the same filtered metagenome and database — for LiME, build ebwt/lcp/da with eGSA/eGap (eGap with --lcp and --da) and run LiME_paired.sh on the four collections (forward and reverse-complement of both mates) with alpha=16, beta=0.25, TaxRank=1 for 100 bp paired-end reads. Parse each tool's per-read output (LiME format: C/U/A/H,IdSeqRead,TaxID,maxSim) into classified read sets. Compute Jaccard-based agreement rates (rid, ras) between the classified read sets of each tool and the reference comparator (Magic-BLAST), tallying reads classified identically by both, and reads classified by Magic-BLAST but missed by each other tool. Validate by reproducing the article's Table S5 values: 95.2% of classified reads agreed identically, 93.6% assigned to the same taxon, and LiME missed 42,788 Magic-BLAST-classified reads.

## Related tools

- **LiME** (Classifier under evaluation; produces per-read classification file (C/U/A/H,IdSeqRead,TaxID,maxSim) compared against Magic-BLAST) — https://github.com/veronicaguerrini/LiME
- **MagicBlast** (Reference comparator whose classified read set anchors the agreement computation)
- **BLAST** (Filters the real metagenome at 98% sequence identity and provides taxonomic information for the customized reference database)
- **CLARK-S** (State-of-the-art classifier run on the same filtered metagenome for agreement comparison)
- **Centrifuge** (Mapping-based classifier run on the same filtered metagenome for agreement comparison)
- **Kraken 2** (k-mer-based classifier run on the same filtered metagenome for agreement comparison)
- **TaxMaps** (Mapping-based classifier run on the same filtered metagenome for agreement comparison)
- **eGSA** (Builds ebwt/lcp/da data structures for read and genome sets prior to LiME classification) — https://github.com/felipelouza/egsa
- **eGap** (Merges read and genome ebwt data structures with --lcp and --da options) — https://github.com/felipelouza/egap

## Examples

```
./ClusterLCP ReadsF+Refs.fasta numReads numRefs 16 threads; ./ClusterBWT_DA ReadsF+Refs.fasta 100 0.25 threads; ./Classify N Input1 ... InputN numReads numRefs OutputFile LineageFile 1 threads
```

## Evaluation signals

- Agreement table matches Table S5: ~95.2% of classified reads classified identically by LiME and Magic-BLAST, and 93.6% assigned to the same taxon
- LiME's missed-read count versus Magic-BLAST reproduces the reported 42,788 Magic-BLAST-classified reads missed, with per-tool missed counts for CLARK-S, Centrifuge, Kraken 2 and TaxMaps within tolerance
- Each tool's output parses into a complete per-read table; LiME lines conform to C/U/A/H,IdSeqRead,TaxID,maxSim
- All tools consumed the identical BLAST-filtered metagenome (98% sequence identity) and the same customized reference database
- Jaccard-based rates (rid, ras) computed on the classified read sets only, excluding unclassified reads consistently across tools

## Limitations

- Version numbers of the compared tools (Magic-BLAST, CLARK-S, Centrifuge, Kraken 2, TaxMaps) are not stated in the paper
- The real metagenome must be BLAST-filtered before comparison; agreement on unfiltered data is not established
- Agreement with Magic-BLAST is a relative concordance measure, not accuracy against known truth — two tools can agree and both be wrong

## Evidence

- [results] On the real metagenome SRR1804065, LiMEeBWT shows the strongest agreement with Magic-BLAST: 95.2% of all classified reads: [evidence span withheld — anchor status: not_found]
- [results] Jaccard similarity on classified read sets: "we compute the similarity between the two sets of classified reads by using the Jaccard similarity coefficient"
- [results] BLAST filtering at 98% identity: "we first filtered this dataset by using BLAST: we mapped reads against the whole nucleotide sequence database with a sequence identity of 98%"
- [readme] LiME output format for parsed classification files: "OutputFile is the comma-separated file where the classification results will be stored according to the format:"
- [abstract] Comparable results to MagicBlast (abstract claim being validated): "we show that LiME is able to deliver classification results comparable to that of MagicBlast"
