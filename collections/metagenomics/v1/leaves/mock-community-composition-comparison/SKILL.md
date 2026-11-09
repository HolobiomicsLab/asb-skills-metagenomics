---
name: mock-community-composition-comparison
description: Use when a 16S rRNA amplicon profile has been CopyRighter-corrected and
  a known expected mock community composition (equal 16S rRNA gene copies per species)
  is available to quantify accuracy by Hellinger/euclidean distance.
when_to_use_negative:
- No mock community with a known expected composition is available — distances to
  an expected profile cannot be computed and validation is meaningless
- The input table contains relative abundances/percentages instead of raw read counts,
  which the copyrighter -i input explicitly requires
- Taxonomic assignments have been altered from the Greengenes taxonomic string format
  (e.g., k__Bacteria; p__Proteobacteria; ...), preventing GCN trait lookup at the
  description level
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: CopyRighter
  role: Applies lineage-specific 16S rRNA gene copy number correction to the mock
    community amplicon profile before distance comparison
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl
  role: Runtime environment (>= 5.10.0) required to install and run CopyRighter
- name: Bio::Community
  role: CPAN dependency providing community-file handling; bc_use_repr_id can replace
    OTU IDs with Greengenes IDs for trait lookup
  repo: github.com/fangly/AmpliCopyrighter
- name: Greengenes database
  role: Source of phylogenetically-based GCN estimates for all OTUs and taxa used
    to correct the profile
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:da89df078f1215752f5c368a4d025aa19830de9491c0df3e7e941135528fef93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - GNU make
  - Greengenes database
  - Perl
  - make
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mock-community-composition-comparison

## Summary

Validates GCN-corrected 16S rRNA amplicon profiles by computing Hellinger and euclidean distances between uncorrected/corrected profiles and the expected composition of an in vitro mock community (e.g., HMP SRR074387, 22 species). Correction is accepted when it reduces the distance to the known expected profile.

## When to use

Use when a 16S rRNA amplicon profile has been CopyRighter-corrected and a known expected mock community composition (equal 16S rRNA gene copies per species) is available to quantify accuracy by Hellinger/euclidean distance.

## When NOT to use

- No mock community with a known expected composition is available — distances to an expected profile cannot be computed and validation is meaningless
- The input table contains relative abundances/percentages instead of raw read counts, which the copyrighter -i input explicitly requires
- Taxonomic assignments have been altered from the Greengenes taxonomic string format (e.g., k__Bacteria; p__Proteobacteria; ...), preventing GCN trait lookup at the description level

## Inputs

- OTU abundance table with read counts (biom, QIIME, GAAS, Unifrac, or generic tabular site-by-species format) from 16S rRNA amplicon sequencing of an in vitro mock community
- Expected reference composition of the mock community (e.g., 22 species pooled for equal 16S rRNA gene copies)
- CopyRighter GCN trait data file (phylogenetically-based estimates for OTUs/taxa in the Greengenes database, selected via COPYRIGHTER_DB or -d)

## Outputs

- CopyRighter-corrected community profile with relative abundance in percent (same format as input)
- Hellinger and euclidean distances of uncorrected and corrected profiles to the expected mock composition
- Validation results table (e.g., copyrighter_srr074387_results.tsv) with both distances and the corrected profile
- Average trait (16S rRNA GCN) value per community reported on standard output

## How to apply

Preprocess the mock community amplicon reads (e.g., HMP SRR074387 V3-V5, 22-species DNA-based in vitro mock pooled for equal 16S rRNA gene copies) into an OTU abundance table via the QIIME/Acacia/CD-HIT-OTU/BLASTN pipeline with Greengenes-based taxonomic assignment, keeping read counts (not percentages) and unaltered Greengenes taxonomic strings. Convert the OTU table into a CopyRighter-compatible community profile and run the copyrighter script with the phylogenetically-based Greengenes GCN data files, preferring phylogenetic-level over taxonomic-level correction as recommended. Compute Hellinger and euclidean distances of both the uncorrected and corrected relative abundance profiles against the expected 22-species composition. Accept the correction if the corrected profile's distances are smaller than the uncorrected ones, and record both distances plus the corrected profile (e.g., in copyrighter_srr074387_results.tsv).

## Related tools

- **CopyRighter** (Applies lineage-specific 16S rRNA gene copy number correction to the mock community amplicon profile before distance comparison) — github.com/fangly/AmpliCopyrighter
- **Perl** (Runtime environment (>= 5.10.0) required to install and run CopyRighter)
- **Bio::Community** (CPAN dependency providing community-file handling; bc_use_repr_id can replace OTU IDs with Greengenes IDs for trait lookup) — github.com/fangly/AmpliCopyrighter
- **Greengenes database** (Source of phylogenetically-based GCN estimates for all OTUs and taxa used to correct the profile)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Corrected profile has strictly smaller Hellinger and euclidean distances to the expected 22-species composition than the uncorrected profile
- Input community file contains read counts (not percentages) with unaltered Greengenes taxonomic strings, verified with copyrighter -v trait-value assignments
- Output corrected profile is in the same format as the input and expresses relative abundance in percent
- Trait lookup used the phylogenetically-based Greengenes GCN data file (COPYRIGHTER_DB or -d), with phylogenetic-level preferred over taxonomic-level correction
- Average 16S rRNA GCN value for the community is reported on standard output after the run

## Limitations

- CopyRighter version 0.46 covers only 16S rRNA gene GCN (and genome length for metagenomes); GC percent bias and fungal ITS variation correction awaits pre-computed estimates
- GCN estimates are tied to the Oct 2012 Greengenes taxonomy; other taxonomies such as Silva are not yet covered
- Using '-l desc' (default) matches taxa by taxonomic description; OTU IDs must be mapped to Greengenes IDs (e.g., via bc_use_repr_id) for ID-based lookup
- No specific CopyRighter version is stated in the article discussion; effects of correction on communities of tens to thousands of species are non-trivial to predict

## Evidence

- [abstract] Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance: "Software validation with in silico and in vitro mock communities indicated that GCN correction results in more accurate estimates of microbial relative abundance"
- [discussion] prefer phylogenetic-level correction: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [readme] input must contain read counts: [evidence span withheld — anchor status: external_doc]
- [readme] copy number weighting: "CopyRigher corrects microbial relative abundance by applying a weight proportional to the inverse of the estimated copy number to each species."
- [methods] Perl installation steps: "Perl (>= 5.10.0) <http://www.perl.com/download.csp>"
