---
name: greengenes-lineage-data-parsing
description: Use when an OTU or taxon table needs lineage-specific 16S rRNA GCN values
  attached from CopyRighter's Greengenes data files, or when Greengenes taxstrings/IDs
  must be parsed to look up trait values.
when_to_use_negative:
- Input taxa use taxonomies other than Greengenes (e.g. Silva) — CopyRighter's pre-computed
  GCN files cover only the Oct 2012 Greengenes taxonomy
- The input file contains relative percentages instead of read counts, which the lookup
  expects
- The goal is de novo GCN estimation rather than parsing the pre-computed Greengenes
  estimates distributed with CopyRighter
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3301
tools:
- name: CopyRighter
  role: Source of the Greengenes GCN data files being parsed; performs the trait lookup
    and correction downstream
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to read community files and provide bc_use_repr_id for mapping
    OTU IDs to Greengenes IDs
- name: Perl >= 5.10.0
  role: Runtime environment in which the parsing and lookup scripts are executed
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:3504e69813affb3a13ea7b0d93fbe7944217f88bdf82c9b7aeb87c7b3ea4242c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Perl >= 5.10.0
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# greengenes-lineage-data-parsing

## Summary

Load and parse the CopyRighter-distributed Greengenes 16S rRNA gene copy number (GCN) estimate files (OTU IDs and full Greengenes taxonomic strings) plus Greengenes community/OTU tables, using Perl with Bio::Community, so lineage-specific trait values can be matched to taxa for correction or phylogenetic analyses.

## When to use

Use when an OTU or taxon table needs lineage-specific 16S rRNA GCN values attached from CopyRighter's Greengenes data files, or when Greengenes taxstrings/IDs must be parsed to look up trait values.

## When NOT to use

- Input taxa use taxonomies other than Greengenes (e.g. Silva) — CopyRighter's pre-computed GCN files cover only the Oct 2012 Greengenes taxonomy
- The input file contains relative percentages instead of read counts, which the lookup expects
- The goal is de novo GCN estimation rather than parsing the pre-computed Greengenes estimates distributed with CopyRighter

## Inputs

- OTU/community table in biom, QIIME, GAAS, Unifrac, or generic tabular format with read counts and unaltered Greengenes taxonomic assignments
- CopyRighter Greengenes GCN estimate data file (tab-delimited, ID or taxstring in col 1, trait value in col 2, e.g. '4\t1.51098055313977')
- COPYRIGHTER_DB environment variable pointing to the default trait data file (optional)

## Outputs

- Parsed community objects (Bio::Community) with per-OTU Greengenes IDs and taxstrings
- Lineage-to-GCN lookup mapping each OTU/taxon to its phylogenetically-inferred 16S rRNA copy number

## How to apply

Load the input community file (biom, QIIME, GAAS, Unifrac, or generic tabular site-by-species format) using Bio::Community readers in Perl, ensuring the file contains read counts rather than percentages and that Greengenes taxonomic assignments are unaltered. Parse CopyRighter's tab-delimited GCN data files, which contain two columns — an OTU ID or full Greengenes taxstring (e.g. 'k__Bacteria; p__Proteobacteria; c__Alphaproteobacteria; ...; s__siberiense' with whitespace after each semicolon) and the trait estimate — ignoring extra columns, empty lines, and comment lines starting with '#'. Match taxa to trait values using the 'desc' lookup mode (taxstring match) by default, or 'id' mode (Greengenes OTU ID), replacing arbitrary OTU IDs with Greengenes IDs via the Bio::Community script bc_use_repr_id when needed. If the -d data file is not supplied explicitly, resolve the trait file path from the COPYRIGHTER_DB environment variable. Verify that each taxon resolves to a sensible GCN (e.g. 1.0 for Crenarchaeum symbiosum, up to ~15 in Bacteria) using verbose mode to inspect trait value assignments.

## Related tools

- **CopyRighter** (Source of the Greengenes GCN data files being parsed; performs the trait lookup and correction downstream) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to read community files and provide bc_use_repr_id for mapping OTU IDs to Greengenes IDs)
- **Perl >= 5.10.0** (Runtime environment in which the parsing and lookup scripts are executed)

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -d 16S_copies.txt -l desc -v
```

## Evaluation signals

- All taxa in the input community resolve to a GCN value in verbose (-v) mode with expected assignments (e.g. Archaea near 1.0-1.57, Bacteria spanning 1 to 15)
- Parsed taxstrings match the Greengenes 2012/10 format with whitespace after each semicolon and rank prefixes k__/p__/c__/o__/f__/g__/s__
- Comment lines starting with '#' and empty lines in the data file are skipped, and only the first two columns (ID/taxstring, trait value) are consumed
- OTUs lacking a taxonomic assignment are handled via the community-average GCN rather than failing or distorting other OTUs' relative abundances

## Limitations

- CopyRighter's distributed trait files are based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy and do not yet cover Silva or other taxonomies
- Arbitrary OTU IDs must first be replaced by their corresponding Greengenes IDs (e.g. with bc_use_repr_id) for 'id' mode lookup to work
- OTUs without any taxonomic assignment default to the community-average GCN, which is an approximation

## Evidence

- [readme] Input community file ... must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments: [evidence span withheld — anchor status: external_doc]
- [readme] tab-delimited two-column trait file format: [evidence span withheld — anchor status: external_doc]
- [discussion] freely available Greengenes GCN files: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [readme] bc_use_repr_id ID replacement: [evidence span withheld — anchor status: external_doc]
- [methods] install dependencies via Makefile.PL: "you will need to install additional CPAN dependencies. See the Makefile.PL file for details"
