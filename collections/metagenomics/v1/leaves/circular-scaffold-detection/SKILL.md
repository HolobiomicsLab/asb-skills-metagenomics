---
name: circular-scaffold-detection
description: Use when preparing viral scaffold FASTAs (e.g., VIBRANT/VirSorter predictions)
  as input to vRhyme binning, to flag complete circular viral genomes before binning
  and dereplication.
when_to_use_negative:
- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme performance
  on whole metagenomes has not been fully evaluated and it is not meant to bin microbes
- Goal is to bin fragmented non-circular scaffolds into vMAGs — circular scaffolds
  are removed, not binned
- Scaffolds lack terminal repeats (linear viral fragments) — no circularity signal
  exists to detect
edam_operation: http://edamontology.org/operation_0328
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: vRhyme
  role: Provides the flag_circular.py auxiliary script used to identify circular scaffolds
    pre-binning; circular scaffold removal was added in v1.1.0
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Nucmer
  role: Used with Mash for optional scaffold dereplication after circular scaffold
    removal
  repo: https://mummer4.github.io/install/install.html
- name: Mash
  role: Optional dereplication of remaining input scaffolds (--derep_id 0.99 composite
    / 0.97 longest)
  repo: https://github.com/marbl/mash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:292dd09ad9102cdd2b492cb29ca58ccd455eae6050144f1e0613c20b349e6a49
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - Mash
  - Mmseqs2
  - Nucmer
  - Prodigal
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0328
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# circular-scaffold-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-binning filter step that identifies circular scaffolds (complete viral genomes) by checking for terminal repeats of at least 20bp with up to 2 mismatches, so complete genomes are excluded from vRhyme binning input and validated independently.

## When to use

Use when preparing viral scaffold FASTAs (e.g., VIBRANT/VirSorter predictions) as input to vRhyme binning, to flag complete circular viral genomes before binning and dereplication.

## When NOT to use

- Input is a whole metagenome rather than predicted viral scaffolds — vRhyme performance on whole metagenomes has not been fully evaluated and it is not meant to bin microbes
- Goal is to bin fragmented non-circular scaffolds into vMAGs — circular scaffolds are removed, not binned
- Scaffolds lack terminal repeats (linear viral fragments) — no circularity signal exists to detect

## Inputs

- Viral scaffold FASTA from virus prediction (e.g., VIBRANT or VirSorter output)
- Existing vRhyme bins (flag_circular.py can scan within bins)

## Outputs

- Flagged list of circular (complete viral genome) scaffolds removed from binning input
- Cleaned scaffold FASTA (length-filtered, minimum 2000bp) for vRhyme -i

## How to apply

Run vRhyme's flag_circular.py auxiliary script on the input scaffold FASTA (or within existing bins) to detect circular scaffolds; scaffolds are called circular when terminal repeats of at least 20bp are found with up to 2 mismatches. Remove flagged circular scaffolds pre-binning, since complete viral genomes do not need to be binned into vMAGs, and optionally dereplicate remaining scaffolds with Mash and Nucmer (--method longest or composite, -m 1000bp, --derep_id 0.99/0.97, --frac 0.20/0.70) before passing the length-filtered FASTA to vRhyme. Confirm flagged circular scaffolds with CheckV completeness — e.g., the 22-scaffold Herelleviridae vMAG in the human skin cohort reaches ~100% completeness post-binning, the expected outcome for near-complete genomes.

## Related tools

- **vRhyme** (Provides the flag_circular.py auxiliary script used to identify circular scaffolds pre-binning; circular scaffold removal was added in v1.1.0) — https://github.com/AnantharamanLab/vRhyme
- **Nucmer** (Used with Mash for optional scaffold dereplication after circular scaffold removal) — https://mummer4.github.io/install/install.html
- **Mash** (Optional dereplication of remaining input scaffolds (--derep_id 0.99 composite / 0.97 longest)) — https://github.com/marbl/mash

## Examples

```
python flag_circular.py -i viral_scaffolds.fasta
```

## Evaluation signals

- flag_circular.py output lists scaffolds with terminal repeats >= 20bp and <= 2 mismatches as circular
- No circular/complete viral genomes remain in the scaffold FASTA passed to vRhyme -i
- Binning of the remaining skin-cohort scaffolds yields 849 vMAGs from 10,601 viral scaffolds (762 with low protein redundancy), as reported
- CheckV confirms near-complete genomes, e.g., the Herelleviridae vMAG of 22 scaffolds reaches ~100% completeness

## Limitations

- Only complete circular genomes are removed pre-binning; fragmented linear viral scaffolds still require binning
- vRhyme is built for viral sequences; performance on whole metagenomes is not fully evaluated and non-viral genome binning is an anticipated limitation
- Circular-scaffold removal and coverage outlier removal are v1.1.0 features; older versions do not flag circular scaffolds

## Evidence

- [methods] Identify and remove circular scaffolds (complete viral genomes) pre-binning by checking terminal repeats of at least 20bp with up to 2 mismatches: [evidence span withheld — anchor status: not_found]
- [methods] flag_circular.py identifies circular scaffolds from input fasta or within bins: "flag_circular.py*: Identify scaffolds that are circular (i.e., complete viral genomes) from an input fasta file or within bins."
- [readme] v1.1.0 release notes describe circular scaffold removal pre-binning: "Added circular scaffold (i.e., complete viral genome) removal pre-binning"
- [abstract] Herelleviridae vMAG validation with CheckV: "highlighted by identification of a Herelleviridae vMAG comprised of 22 scaffolds"
