---
name: classification-sensitivity-f1-evaluation
description: Use when a simulated metagenome benchmark (e.g. setA2, setB2 with known
  read provenance) has been classified by LiME and you must quantify classification
  accuracy via sensitivity, precision and F1, or reproduce reported values like SEN
  95.55%/F1 97.66% on setA2.
when_to_use_negative:
- Classifying a real metagenome without known read provenance (e.g. SRR1804065) —
  use Jaccard-based agreement rates rid/ras for tool concordance instead of sensitivity/F1
- The reference database or classifier has not yet produced an OutputFile; metric
  computation requires the classification results first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: LiME
  role: Runs the three-step classification pipeline (ClusterLCP, ClusterBWT_DA, Classify)
    that produces the OutputFile records scored by sensitivity, precision and F1
  repo: https://github.com/veronicaguerrini/LiME
- name: BCR
  role: Builds ebwt, lcp and da data structures for the reference genome set G and
    read set R before classification
  repo: https://github.com/giovannarosone/BCR_LCP_GSA
- name: eGap
  role: Merges the eBWT data structures of reads and genomes with --lcp and --da options
    to yield the per-dataset collections used in classification
  repo: https://github.com/felipelouza/egap
- name: egsa
  role: Alternative external-memory builder of ebwt/lcp/da data structures for the
    collection S
  repo: https://github.com/felipelouza/egsa
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03628-w
    title: Metagenomic analysis through the extended Burrows-Wheeler transform
  source_content_hash: sha256:3121c85033b98d14c476816a8f55d8054102287fcc5f7acf22d5702bbe5657f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/veronicaguerrini/LiME
  tools:
  - BCR
  - LiME
  - eGap
  - egsa
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# classification-sensitivity-f1-evaluation

## Summary

Compute sensitivity, precision and F1 score for LiME's metagenomic read classification by parsing the comma-separated OutputFile records (C/U/A/H,IdSeqRead,TaxID,maxSim) against known simulated provenances, optionally at any taxonomic rank from species to phylum.

## When to use

Use when a simulated metagenome benchmark (e.g. setA2, setB2 with known read provenance) has been classified by LiME and you must quantify classification accuracy via sensitivity, precision and F1, or reproduce reported values like SEN 95.55%/F1 97.66% on setA2.

## When NOT to use

- Classifying a real metagenome without known read provenance (e.g. SRR1804065) — use Jaccard-based agreement rates rid/ras for tool concordance instead of sensitivity/F1
- The reference database or classifier has not yet produced an OutputFile; metric computation requires the classification results first

## Inputs

- LiME classification OutputFile (comma-separated records 'C/U/A/H,IdSeqRead,TaxID,maxSim')
- Simulated metagenome read sets with known provenance (e.g. setA2, setB2 paired-end 100 bp reads)
- Taxonomic lineage file Reference_database.csv (semicolon-separated genome taxonomy)
- Parameter settings (alpha, beta, TaxRank, EBWT variant)

## Outputs

- Sensitivity, precision and F1 score per dataset (e.g. SEN 95.55%, F1 97.66% for setA2; SEN 95.95%, F1 97.91% for setB2)
- alpha x beta x variant sweep table of SEN/PREC/F1 configurations
- Phylum-level relative abundance estimates (aggregated reads per phylum TaxID)

## How to apply

Run the three-step LiME pipeline (ClusterLCP with alpha=16, ClusterBWT_DA with beta=0.25, Classify) at the desired TaxRank (1 for species, 6 for phylum), passing the taxonomic lineage file Reference_database.csv, to produce the OutputFile with records 'C/U/A/H,IdSeqRead,TaxID,maxSim'. Parse the OutputFile, counting classified (C), unclassified (U), ambiguous (A) and higher-rank (H) reads per dataset. From the correctly assigned reads per taxon compute sensitivity, precision and F1 against the known simulated provenances; for rank-level analyses aggregate reads by TaxID to estimate relative abundances. For the parameter sweep, repeat across alpha in [16,22] and beta in [0.15,0.40] for both EBWT=1 and EBWT=0 variants and check the trade-off (lower beta raises sensitivity, lowers precision).

## Related tools

- **LiME** (Runs the three-step classification pipeline (ClusterLCP, ClusterBWT_DA, Classify) that produces the OutputFile records scored by sensitivity, precision and F1) — https://github.com/veronicaguerrini/LiME
- **BCR** (Builds ebwt, lcp and da data structures for the reference genome set G and read set R before classification) — https://github.com/giovannarosone/BCR_LCP_GSA
- **eGap** (Merges the eBWT data structures of reads and genomes with --lcp and --da options to yield the per-dataset collections used in classification) — https://github.com/felipelouza/egap
- **egsa** (Alternative external-memory builder of ebwt/lcp/da data structures for the collection S) — https://github.com/felipelouza/egsa

## Examples

```
Classify 4 setB2_1+Refs.fasta setB2_1_RC+Refs.fasta setB2_2+Refs.fasta setB2_2_RC+Refs.fasta 20249373 930 OutputFile Reference_database.csv 6 8
```

## Evaluation signals

- Sensitivity and F1 match reported values: 95.55% / 97.66% for setA2 and 95.95% / 97.91% for setB2
- Sweep covers all alpha in [16,22] and beta in [0.15,0.40] for both EBWT=1 and EBWT=0 variants
- Sweep reproduces the monotone trend: decreasing beta raises sensitivity and lowers precision; alpha=16 with beta=0.25 among best F1 configurations per Table S2
- OutputFile parses cleanly into C/U/A/H records with fields IdSeqRead, TaxID, maxSim for every read
- Estimated phylum abundances are concordant with the phylum composition designed in the Lindgreen et al. benchmarking datasets

## Limitations

- Full parameter sweep results are only in supplementary Additional file 1 Table S2, not in the main text
- Classification quality depends on parameter choice: increasing beta decreases sensitivity while increasing precision
- Metric evaluation requires known ground-truth provenance, so it only applies to simulated datasets such as setA2/setB2

## Evidence

- [readme] OutputFile is the comma-separated file where the classification results will be stored according to the format: *C/U/A/H,IdSeqRead,TaxID,maxSim*: "OutputFile is the comma-separated file where the classification results will be stored according to the format:"
- [results] quality metrics definition: "from which we can calculate the quality metrics: sensitivity SEN, precision PREC, and the F1 score"
- [results] parameter trade-off finding: [evidence span withheld — anchor status: not_found]
- [methods] dataset description: "the dataset setB2 is a sample of 20,249,373 paired-end short reads (100 bps)"
- [readme] class counting: "where C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1)"
