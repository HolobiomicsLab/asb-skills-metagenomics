---
name: nucleotide-sequence-clustering-redundancy-removal
description: Use when you have per-sample FASTA sequences of putative elements (e.g.
  insertion sequences from many metagenomes) and need a non-redundant catalogue by
  clustering at ~95% nucleotide identity.
when_to_use_negative:
- Input sequences are proteins or need clustering at identities far from nucleotide
  level (use CD-HIT protein mode instead of CD-HIT-EST).
- The target insertion sequences lack inverted terminal repeats (>=25 nt), since upstream
  Palidis detection — and hence the pooled input — cannot recover ITR-less ISs.
- 'You need to annotate against the released ISC v1.0.0 rather than build a catalogue:
  use it as a reference database instead of re-clustering.'
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: Palidis
  role: Detects ISs as palindromic ITRs (>=25 nt) flanking 500-3000 bp regions via
    maximal exact matching, producing the per-sample FASTA sequences that are pooled
    before clustering
  repo: https://github.com/blue-moon22/palidis
- name: CD-HIT-EST
  role: Clusters pooled nucleotide IS sequences at -c 0.95 identity to remove redundancy
    and emit the non-redundant catalogue
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:fa10b51ddf1838d2871767913ceeb665e294379c5f7dabcacd3307fc9f010f96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - CD-HIT-EST
  - Palidis
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# nucleotide-sequence-clustering-redundancy-removal

## Summary

Pool nucleotide sequences identified per-sample (e.g. insertion sequences detected by Palidis in 264 HMP metagenomes) and cluster them with CD-HIT-EST at a 0.95 sequence identity threshold to emit a non-redundant catalogue (ISC v1.0.0: 879 unique ISs, 87 unique transposases).

## When to use

Use when you have per-sample FASTA sequences of putative elements (e.g. insertion sequences from many metagenomes) and need a non-redundant catalogue by clustering at ~95% nucleotide identity.

## When NOT to use

- Input sequences are proteins or need clustering at identities far from nucleotide level (use CD-HIT protein mode instead of CD-HIT-EST).
- The target insertion sequences lack inverted terminal repeats (>=25 nt), since upstream Palidis detection — and hence the pooled input — cannot recover ITR-less ISs.
- You need to annotate against the released ISC v1.0.0 rather than build a catalogue: use it as a reference database instead of re-clustering.

## Inputs

- Pooled FASTA file of insertion sequences detected per-sample by Palidis (2517 ISs across 218 of 264 HMP samples)
- Per-sample IS information table (IS_name, sample_id, contig, is_start, is_end, description)
- Raw paired-end short-read metagenomes from the Human Microbiome Project (download links in Data S1)

## Outputs

- Non-redundant Insertion Sequence Catalogue v1.0.0 (FASTA) containing 879 unique insertion sequences
- Catalogue annotation of 87 unique transposases (InterPro/PANTHER accessions in IS names)

## How to apply

First run the per-sample detection (Palidis v3.1.0 default parameters: >=25 nt inverted terminal repeats flanking regions in the 500-3000 bp length range, found via maximal exact matching) over all 264 Human Microbiome Project metagenomes, yielding 2517 ISs on 1837 contigs across 218 samples. Pool the FASTA sequences of all reported insertion sequences across samples. Cluster the pooled nucleotide sequences with CD-HIT-EST using a sequence identity threshold of -c 0.95 to remove redundant near-identical ISs detected in multiple samples or contigs. Emit one representative sequence per cluster as the non-redundant Insertion Sequence Catalogue (ISC v1.0.0) and verify it contains 879 unique ISs with 87 unique transposases, matching the release at github.com/blue-moon22/ISC. Rationale: multiple samples harbour the same IS, so per-sample detections must be deduplicated before the catalogue can serve as an annotation reference.

## Related tools

- **Palidis** (Detects ISs as palindromic ITRs (>=25 nt) flanking 500-3000 bp regions via maximal exact matching, producing the per-sample FASTA sequences that are pooled before clustering) — https://github.com/blue-moon22/palidis
- **CD-HIT-EST** (Clusters pooled nucleotide IS sequences at -c 0.95 identity to remove redundancy and emit the non-redundant catalogue)

## Examples

```
cd-hit-est -i pooled_is_sequences.fasta -o ISC_v1.0.0.fasta -c 0.95
```

## Evaluation signals

- Clustered catalogue contains exactly 879 unique insertion sequences (matches released ISC v1.0.0).
- Catalogue contains 87 unique transposases.
- No two catalogue sequences share >=95% nucleotide identity (cluster representatives are non-redundant).
- Upstream detections total 2517 ISs on 1837 contigs across 218 of 264 samples before pooling.
- Output FASTA IS lengths fall within the 500-3000 bp range enforced by Palidis defaults.

## Limitations

- ISC v1.0.0 only contains common DDE-type ISs with ITRs; Palidis cannot discover ISs lacking ITRs.
- Only ISs with ITRs of >=25 nt are captured; ITRs as short as ten nucleotides are missed at default settings, and lowering thresholds or k-mer length is more computationally intensive.
- ISs lengthened by nested insertion events beyond the 500-3000 bp proximity range are missed; the maximum IS length may need increasing in future ISC iterations.
- ISC v1.0.0 was built only from healthy human oral and stool metagenomes, limiting its use as an annotation reference to those sources.

## Evidence

- [abstract] Palidis applied to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized.: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [intro] These ISs can then be pooled and clustered to create a non-redundant catalogue of ISs.: "These ISs can then be pooled and clustered to create a non-redundant catalogue of ISs."
- [discussion] we have released the first version of ISC, a catalogue containing 879 ISs: "we have released the first version of ISC, a catalogue containing 879 ISs"
- [readme] Cluster candidate ITRs using CD-HIT-EST: "5. Cluster candidate ITRs using CD-HIT-EST"
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length"
