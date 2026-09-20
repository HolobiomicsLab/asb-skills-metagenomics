---
name: reference-database-otu-sequence-retrieval
description: Use when PM-CNN input is an OTU abundance table lacking a phylogeny and
  you must obtain representative marker-gene (amplicon) sequences for each OTU before
  multiple sequence alignment and FastTree tree inference.
when_to_use_negative:
- The phylogenetic tree is already available in NEWICK format (e.g. for shotgun data)
  — PM-CNN Step 1 is optional and the retrieval/alignment/tree-building chain can
  be skipped
- The input feature table is a species table with taxonomy-derived phylogeny rather
  than OTU identifiers requiring GreenGenes lookup
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_0084
tools:
- name: PM-CNN
  role: Pipeline whose Step 1 requires the retrieved GreenGenes representative OTU
    sequences as the starting input for tree construction
  repo: https://github.com/qdu-bioinfo/PM_CNN
- name: Mafft
  role: Consumes the retrieved representative-sequence FASTA in the next step ('mafft
    --auto') to produce the multiple sequence alignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae013
    title: 'PM-CNN: microbiome status recognition and disease detection model based
      on phylogeny and multi-path neural network'
  source_content_hash: sha256:d74e2246c50c97bdcb78dadfeac09db438c6fd5329c5ff7d4d569441a5836f1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qdu-bioinfo/PM_CNN
  tools:
  - FastTree
  - Mafft
  - PM-CNN
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# reference-database-otu-sequence-retrieval

## Summary

Retrieve the representative nucleotide sequences of all OTUs from the GreenGenes reference database so they can be aligned with Mafft and used to build the phylogenetic tree consumed by PM-CNN's downstream cophenetic-distance step.

## When to use

Use when PM-CNN input is an OTU abundance table lacking a phylogeny and you must obtain representative marker-gene (amplicon) sequences for each OTU before multiple sequence alignment and FastTree tree inference.

## When NOT to use

- The phylogenetic tree is already available in NEWICK format (e.g. for shotgun data) — PM-CNN Step 1 is optional and the retrieval/alignment/tree-building chain can be skipped
- The input feature table is a species table with taxonomy-derived phylogeny rather than OTU identifiers requiring GreenGenes lookup

## Inputs

- OTU abundance table (microbial feature table, e.g. example_abundance_table.csv) listing OTU identifiers
- GreenGenes reference database (source of representative OTU nucleotide sequences)

## Outputs

- Representative-sequence FASTA file for all OTUs (e.g. example_respresent_seqs.txt), one nucleotide sequence per OTU, ready for 'mafft --auto'

## How to apply

Before running Mafft, look up the representative nucleotide sequence for every OTU appearing in the input microbial feature table (OTU table) in the GreenGenes reference database, as prescribed by the PM-CNN workflow. Collect the retrieved sequences into a single FASTA-format file (e.g. example_respresent_seqs.txt) containing one representative sequence per OTU, with identifiers matching the OTU names in the abundance table so the downstream NEWICK tree tips correspond to the dataset's OTUs. This retrieval step is the prerequisite for Step 1 of the PM-CNN pipeline ('mafft --auto' alignment followed by 'FastTree -nt -gtr'), and it applies to amplicon/marker-gene data only — for shotgun data the tree is instead supplied directly as a NEWICK file.

## Related tools

- **PM-CNN** (Pipeline whose Step 1 requires the retrieved GreenGenes representative OTU sequences as the starting input for tree construction) — https://github.com/qdu-bioinfo/PM_CNN
- **Mafft** (Consumes the retrieved representative-sequence FASTA in the next step ('mafft --auto') to produce the multiple sequence alignment)

## Evaluation signals

- The FASTA file contains exactly one representative nucleotide sequence for each OTU present in the input abundance table (no missing or extra OTUs)
- Retrieved sequences originate from the GreenGenes database, matching the article's stated prerequisite
- Downstream 'mafft --auto' alignment succeeds on the file, producing output_aligned.fasta without unmatched identifiers
- The resulting NEWICK tree tips correspond to the OTUs in the input dataset, satisfying PM-CNN's validation requirement for Step 1 output

## Limitations

- Sequence retrieval depends on OTU identifiers being mappable to GreenGenes; the README gives no direct download command and says the related software must be obtained from official websites or by contacting the authors
- Only applicable to marker-gene/amplicon data; for shotgun studies the tree must be user-provided in NEWICK format instead
- Step 1 of PM-CNN (and hence this retrieval) is explicitly optional

## Evidence

- [readme] Before performing multiple sequence alignment, the representative sequences of all OTUs need to be found from the GreenGenes database.: "Before performing multiple sequence alignment, the representative sequences of all OTUs need to be found from the GreenGenes database."
- [other] Retrieve the representative nucleotide sequences of all OTUs from the GreenGenes reference database.: [evidence span withheld — anchor status: not_found]
- [readme] the phylogeny tree can either be constructed from representative sequences (e.g. marker gene or amplicon), or provided by users from a NEWICK format file (e.g. for shotgun): "the phylogeny tree can either be constructed from representative sequences (e.g. marker gene or amplicon), or provided by users from a NEWICK format file (e.g. for shotgun)"
