---
name: metavirome-votu-classification
description: Use when dereplicated vOTU FASTA sets from a virome or shotgun metagenome
  need phylum-to-genus taxonomic labels via protein-sharing clustering against curated
  phage references (INPHARED).
when_to_use_negative:
- The goal is read-level taxonomic profiling of raw FASTQs — MetaPhage handles that
  with the Kraken2/MiniKraken2_V1 read-classification step, not the vConTACT2/graphanalyzer
  taxonomy module.
- Input is unassembled reads or individual contigs rather than a dereplicated vOTU
  set of consensus scaffolds — the taxonomy module consumes Prodigal proteins from
  clustered vOTUs.
- A fully resolved taxonomy for every input sequence is required — network-based transfer
  only labels vOTUs that cluster with INPHARED references, so counts apply to the
  classified subset.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: Prodigal
  role: Predicts protein-coding genes on the dereplicated vOTU multifasta sets; predicted
    proteins are the input to the vConTACT2 taxonomy step
- name: vConTACT2
  role: Clusters vOTUs with curated INPHARED bacteriophage references by shared protein
    content in custom-reference modality; does not emit per-vOTU taxonomy on its own
- name: graphanalyzer
  role: Automatically interprets the vConTACT2 protein-sharing network and transfers
    INPHARED taxonomy to vOTU clusters without manual inspection
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: INPHARED
  role: Provides the curated bacteriophage reference genomes and the vConTACT2 input
    split (proteins, gene-to-genome map) used for custom-reference clustering
- name: CheckV
  role: end_to_end genome-quality assessment of vOTUs used to explain inflated vOTU
    counts (partial/low-quality genomes) when interpreting classification tallies
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:e42277024ec1cf447e8e92525c551e2ea6557eac266ae6487bcaf9d1e237cdae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lazzarigioele/graphanalyzer
  tools:
  - CheckV
  - INPHARED
  - MetaPhage
  - Prodigal
  - graphanalyzer
  - vConTACT2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metavirome-votu-classification

## Summary

MetaPhage-style taxonomic classification of vOTUs: Prodigal-predicted proteins from dereplicated vOTU multifastas are clustered with curated INPHARED bacteriophage genomes by vConTACT2, and graphanalyzer automatically transfers phylum-to-genus labels across the resulting protein-sharing network.

## When to use

Use when dereplicated vOTU FASTA sets from a virome or shotgun metagenome need phylum-to-genus taxonomic labels via protein-sharing clustering against curated phage references (INPHARED).

## When NOT to use

- The goal is read-level taxonomic profiling of raw FASTQs — MetaPhage handles that with the Kraken2/MiniKraken2_V1 read-classification step, not the vConTACT2/graphanalyzer taxonomy module.
- Input is unassembled reads or individual contigs rather than a dereplicated vOTU set of consensus scaffolds — the taxonomy module consumes Prodigal proteins from clustered vOTUs.
- A fully resolved taxonomy for every input sequence is required — network-based transfer only labels vOTUs that cluster with INPHARED references, so counts apply to the classified subset.

## Inputs

- Dereplicated vOTU consensus multifasta (FASTA) per dataset, produced by MetaPhage CD-HIT-EST dereplication (0.95 identity, word size 9, 0.85 alignment coverage)
- INPHARED Jan2022 split reference files: vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv
- vConTACT2 protein-sharing network output files

## Outputs

- taxonomy_table.csv with per-vOTU phylum, class, order, family, and genus labels for each dataset
- Per-rank taxon tallies (e.g., 3 phyla / 3 classes / 4 orders / 9 families / 32 genera for the DNA virome set)
- checkv_quality_summary.tsv with per-vOTU CheckV quality calls when genome-quality assessment accompanies classification

## How to apply

Predict protein-coding genes on each dereplicated vOTU multifasta (MetaPhage's CD-HIT-EST output: 0.95 identity, word size 9, 0.85 coverage) with Prodigal and feed the predicted proteins to the viral taxonomy module, following the PRODIGAL --> vConTACT2 flow. Run vConTACT2 in custom-reference modality against the INPHARED Jan2022 split — the vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, and data_excluding_refseq.tsv files — so vOTUs cluster with curated bacteriophage reference genomes by shared protein content. Because vConTACT2 alone leaves classification to extensive manual network inspection, run graphanalyzer.py on the protein-sharing network to automatically transfer taxonomy from INPHARED reference genomes to the vOTU clusters. Aggregate the per-vOTU assignments into a taxonomy_table.csv listing phylum, class, order, family, and genus labels per dataset, then tally distinct taxa per rank; the validation datasets report 3 phyla/3 classes/4 orders/9 families with 32 genera for the DNA virome and 3/3/4/9 with 19 species-level taxa for the shotgun set. When vOTU counts appear inflated relative to the 1,029 phage contigs of Liang et al. (MetaPhage predicted 5,284), run CheckV in end_to_end mode and interpret the classification tallies against its quality tiers (569 vs 397 high-quality/complete; 113 vs 757 partial/low-quality/provirus for VLP vs shotgun).

## Related tools

- **Prodigal** (Predicts protein-coding genes on the dereplicated vOTU multifasta sets; predicted proteins are the input to the vConTACT2 taxonomy step)
- **vConTACT2** (Clusters vOTUs with curated INPHARED bacteriophage references by shared protein content in custom-reference modality; does not emit per-vOTU taxonomy on its own)
- **graphanalyzer** (Automatically interprets the vConTACT2 protein-sharing network and transfers INPHARED taxonomy to vOTU clusters without manual inspection) — https://github.com/lazzarigioele/graphanalyzer
- **INPHARED** (Provides the curated bacteriophage reference genomes and the vConTACT2 input split (proteins, gene-to-genome map) used for custom-reference clustering)
- **CheckV** (end_to_end genome-quality assessment of vOTUs used to explain inflated vOTU counts (partial/low-quality genomes) when interpreting classification tallies)

## Examples

```
git clone https://github.com/MattiaPandolfoVR/MetaPhage
```

## Evaluation signals

- taxonomy_table.csv exists and contains phylum, class, order, family, and genus rank columns for each dataset
- Per-rank tallies match published values within the reproducibility tolerance of the network-based assignment — DNA virome exactly 3/3/4/9 ranks with 9 families and 32 genera; shotgun 3/3/4/9 with 19 species-level taxa
- Every classified vOTU label traces to an INPHARED reference genome in its protein-sharing cluster (taxonomy transferred, not de novo predicted)
- CheckV cross-check reproduces the reported quality tiers: 569 vs 397 high-quality/complete vOTUs and 113 vs 757 partial/low-quality/provirus predictions (VLP vs shotgun), explaining the 5,284-vs-1,029 count gap against Liang et al.
- graphanalyzer was run on the vConTACT2 network output — vConTACT2 run alone should yield no automatic per-vOTU taxonomy table

## Limitations

- The vConTACT2 network is rebuilt every time new data are added, greatly extending runtime
- The all-versus-all protein comparison against large reference databases can require a long time
- vConTACT2 does not classify a vOTU directly — without graphanalyzer the classification depends on extensive manual network inspection
- Only vOTUs clustered with INPHARED references receive labels; reported tallies count taxa among the classified vOTUs
- Users are urged to be mindful of the biases inherent to virome analyses and the recall/precision trade-off in virus prediction

## Evidence

- [intro] clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [discussion] the INPHARED (14) database which provides a collection of curated bacteriophage genomes from GenBank and related metrics: "the INPHARED (14) database which provides a collection of curated bacteriophage genomes from GenBank and related metrics"
- [discussion] vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [discussion] In addition, the network is rebuilt every time new data are added, extending the time required even further.: "In addition, the network is rebuilt every time new data are added, extending the time required even further."
- [readme] This is `graphanalyzer.py`, a script designed to automatically interpret the outputs generated by **vConTACT2** when using the **INPHARED** database.: [evidence span withheld — anchor status: external_doc]
