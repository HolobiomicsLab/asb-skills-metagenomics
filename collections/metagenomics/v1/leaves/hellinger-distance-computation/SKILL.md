---
name: hellinger-distance-computation
description: Use when an observed 16S rRNA amplicon profile (uncorrected or CopyRighter-corrected)
  must be quantitatively compared to a mock community of known composition via Hellinger
  and euclidean distances to decide whether GCN correction improved accuracy.
when_to_use_negative:
- The sample is a natural/environmental community with no known expected composition
  — a distance to expectation cannot be computed (this skill is a mock-community validation
  criterion, not a general beta-diversity tool)
- The input is still raw FASTQ reads rather than an OTU/relative abundance profile
  preprocessed with the QIIME/Acacia/CD-HIT-OTU/BLASTN pipeline
- The profile was not produced with Greengenes-based taxonomic assignments, so CopyRighter's
  Greengenes GCN estimates cannot be matched before the distance comparison
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CopyRighter
  role: Generates the trait-corrected (16S rRNA gene copy number) relative abundance
    profile whose Hellinger/euclidean distance to the expected mock composition is
    compared against the uncorrected profile
  repo: github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:13df77c090f3a4ceb41785bf33ed7c01d8ed2f56f5bb83206f585a3d8637d052
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CopyRighter
  - GNU make
  - make
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# hellinger-distance-computation

## Summary

Computes Hellinger (and euclidean) distances between observed 16S rRNA amplicon community profiles and the expected composition of a mock community, providing the quantitative acceptance criterion used to validate that CopyRighter GCN correction improves profile accuracy on the SRR074387 DNA-based in vitro mock.

## When to use

Use when an observed 16S rRNA amplicon profile (uncorrected or CopyRighter-corrected) must be quantitatively compared to a mock community of known composition via Hellinger and euclidean distances to decide whether GCN correction improved accuracy.

## When NOT to use

- The sample is a natural/environmental community with no known expected composition — a distance to expectation cannot be computed (this skill is a mock-community validation criterion, not a general beta-diversity tool)
- The input is still raw FASTQ reads rather than an OTU/relative abundance profile preprocessed with the QIIME/Acacia/CD-HIT-OTU/BLASTN pipeline
- The profile was not produced with Greengenes-based taxonomic assignments, so CopyRighter's Greengenes GCN estimates cannot be matched before the distance comparison

## Inputs

- OTU abundance table with read counts (QIIME/biom format) derived from SRR074387 V3-V5 16S rRNA amplicon reads
- CopyRighter-corrected community profile (relative abundance expressed in percent, e.g. otu_table_copyrighted.generic)
- Expected 22-species composition of the DNA-based in vitro mock community (equal 16S rRNA gene copies per species)

## Outputs

- Hellinger distance of the uncorrected profile to the expected composition
- Hellinger distance of the CopyRighter-corrected profile to the expected composition
- Euclidean distances of both profiles to the expected composition
- Validation record copyrighter_srr074387_results.tsv containing both distances and the corrected profile

## How to apply

From the QIIME/Acacia/CD-HIT-OTU/BLASTN OTU table of the SRR074387 V3-V5 reads and the CopyRighter output (percent relative abundances, e.g. otu_table_copyrighted.generic), place both profiles on the same 22-species axis as the expected mock composition (species pooled for equal 16S rRNA gene copies). Compute the Hellinger distance — the euclidean distance between square-root-transformed relative abundance vectors — and the plain euclidean distance of both the uncorrected and the corrected profiles to the expected composition. Prefer phylogenetic-level over taxonomic-level GCN correction before computing distances, as the paper recommends for highest accuracy. Validation is accepted if the corrected profile's Hellinger/euclidean distance to the expected composition is smaller than the uncorrected distance. Record both distances together with the corrected profile in copyrighter_srr074387_results.tsv.

## Related tools

- **CopyRighter** (Generates the trait-corrected (16S rRNA gene copy number) relative abundance profile whose Hellinger/euclidean distance to the expected mock composition is compared against the uncorrected profile) — github.com/fangly/AmpliCopyrighter

## Examples

```
import numpy as np; p, q = np.asarray(p, float), np.asarray(q, float); H = np.sqrt(0.5 * ((np.sqrt(p / p.sum()) - np.sqrt(q / q.sum())) ** 2).sum())
```

## Evaluation signals

- Corrected profile's Hellinger distance to the expected 22-species composition is smaller than the uncorrected profile's distance
- The same ordering (corrected < uncorrected) holds for the euclidean distance
- Both distances are recorded together with the corrected profile in copyrighter_srr074387_results.tsv
- Corrected profile is expressed in percent relative abundance, matching CopyRighter's -o output format
- CopyRighter verbose mode (-v) confirmed taxa were matched to the intended Greengenes trait values before distances were computed

## Limitations

- Distances to expectation are only meaningful for mock communities with a known target composition; there is no expected profile for environmental samples
- Correction accuracy depends on Greengenes-based phylogenetically-inferred GCN estimates; other taxonomies (e.g., Silva) are not yet covered by CopyRighter
- Effects of GCN correction are non-trivial to predict for communities with tens to thousands of species, so a distance improvement is not guaranteed a priori

## Evidence

- [other] Compute the Hellinger and euclidean distances of both the uncorrected and the CopyRighter-corrected profiles to the expected 22-species composition of the DNA-based mock.: [evidence span withheld — anchor status: not_found]
- [abstract] Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance and improves the agreement between: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance and improves the agreement between"
- [readme] a corrected community file with trait-corrected (16S rRNA gene copy number or genome length) relative abundances is generated: "a corrected community file with trait-corrected (16S rRNA gene copy number or genome length) relative abundances is generated"
