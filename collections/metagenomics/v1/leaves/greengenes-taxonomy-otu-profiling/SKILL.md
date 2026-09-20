---
name: greengenes-taxonomy-otu-profiling
description: Use when building OTU/taxonomic abundance profiles from 16S amplicon
  data (biom, QIIME, GAAS, Unifrac or generic site-by-species tables of read counts)
  that will subsequently undergo lineage-specific GCN correction with CopyRighter.
when_to_use_negative:
- When the taxonomy was assigned against a non-Greengenes reference (e.g. Silva) —
  the distributed GCN estimate files cover the Greengenes database only, with Silva
  extension only anticipated
- When the input community file contains percentages instead of read counts, or taxa
  with altered taxonomic assignments — CopyRighter requires raw counts and UNALTERED
  Greengenes taxonomic strings
- For shotgun metagenomic profiles where genome length bias (rather than 16S rRNA
  GCN bias) is the target trait
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: CopyRighter
  role: Consumes the Greengenes OTU/taxonomic abundance profile (with read counts
    and unaltered taxonomic strings) and applies inverse-GCN weighting using the pre-computed
    Greengenes GCN estimate files; optionally corrects qPCR total abundance
  repo: github.com/fangly/AmpliCopyrighter
- name: Bio::Community
  role: Perl library used to load and manipulate the Greengenes OTU tree and taxonomy;
    bc_use_repr_id can replace arbitrary OTU IDs with corresponding Greengenes IDs
- name: Perl
  role: Runtime (>= 5.10.0) for CopyRighter and Bio::Community; installed via perl
    Makefile.PL, make, make install or local::lib
  repo: http://www.perl.com/download.csp
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:71b8573348b758e1b5055c4ce9f9b74d08853b63e7990d08bd8595507304789c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - Grinder
  - Perl
  - make
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# greengenes-taxonomy-otu-profiling

## Summary

Assigns 16S rRNA amplicon reads to Greengenes OTUs and taxa (preserving full, unaltered taxonomic strings) so that CopyRighter's pre-computed phylogenetically-based 16S rRNA gene copy number (GCN) estimates can be applied to correct microbial relative abundances and qPCR total abundance.

## When to use

Use when building OTU/taxonomic abundance profiles from 16S amplicon data (biom, QIIME, GAAS, Unifrac or generic site-by-species tables of read counts) that will subsequently undergo lineage-specific GCN correction with CopyRighter.

## When NOT to use

- When the taxonomy was assigned against a non-Greengenes reference (e.g. Silva) — the distributed GCN estimate files cover the Greengenes database only, with Silva extension only anticipated
- When the input community file contains percentages instead of read counts, or taxa with altered taxonomic assignments — CopyRighter requires raw counts and UNALTERED Greengenes taxonomic strings
- For shotgun metagenomic profiles where genome length bias (rather than 16S rRNA GCN bias) is the target trait

## Inputs

- QC-filtered 16S rRNA gene amplicon reads (e.g. SRA accession SRR1145444)
- Greengenes reference database (Oct 2012 taxonomy; 99%-identity OTU tree and taxonomy)
- Community file with read counts (not percentages) in biom, QIIME, GAAS, Unifrac, or generic tabular site-by-species format, with unaltered Greengenes taxonomic strings
- CopyRighter pre-computed phylogenetically-based GCN estimate file (tab-delimited, ID or taxstring in col 1, trait estimate in col 2)
- Optional tab-delimited qPCR total abundance file (community name, total abundance) passed via -t

## Outputs

- Greengenes OTU/taxonomic relative abundance profile (corrected output expressed in percent, in the same format as input)
- Corrected total microbial abundance file (e.g. out_copyrighted_total.tsv)
- Absolute abundance per taxon file (e.g. out_copyrighted_combined.txt)
- Average community 16S rRNA gene copy number reported on standard output

## How to apply

QC-filter the amplicon reads, then cluster and assign them to Greengenes OTUs/taxa (e.g. the 99%-identity OTU tree and taxonomy manipulated via Bio::Community under Perl >= 5.10.0). Input community files must contain read counts rather than percentages and taxa must keep UNALTERED taxonomic assignments — e.g. the Greengenes 2012/10 string format 'k__Bacteria; p__Proteobacteria; ...; s__siberiense' with a whitespace after each semicolon — because CopyRighter looks up trait values by taxonomic description (default -l desc) or by OTU ID matched to Greengenes IDs (e.g. IDs 4 to 24084 in the GCN estimate files). By default CopyRighter uses its bundled GCN data file based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy, located via the COPYRIGHTER_DB environment variable; run with -v to verify trait value assignments. Downstream, the phylogenetic-level correction is recommended over taxonomic-level correction for highest accuracy, and the tool reports the average community GCN on standard output.

## Related tools

- **CopyRighter** (Consumes the Greengenes OTU/taxonomic abundance profile (with read counts and unaltered taxonomic strings) and applies inverse-GCN weighting using the pre-computed Greengenes GCN estimate files; optionally corrects qPCR total abundance) — github.com/fangly/AmpliCopyrighter
- **Bio::Community** (Perl library used to load and manipulate the Greengenes OTU tree and taxonomy; bc_use_repr_id can replace arbitrary OTU IDs with corresponding Greengenes IDs)
- **Perl** (Runtime (>= 5.10.0) for CopyRighter and Bio::Community; installed via perl Makefile.PL, make, make install or local::lib) — http://www.perl.com/download.csp

## Examples

```
copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic -v
```

## Evaluation signals

- Input community file contains read counts (not percentages) and taxonomic strings in unaltered Greengenes 2012/10 format with a whitespace after each semicolon
- Verbose mode (-v) confirms trait value assignments are resolved as intended for each taxon
- Average community GCN falls within the biologically reported range of 1–15 in Bacteria and up to 5 in Archaea (e.g. reported digester values of 2.69 at day 27 and 1.61 at day 362)
- Corrected profiles show higher agreement with expected/mock compositions than uncorrected profiles, with phylogenetic-level correction slightly more accurate than taxonomic-level
- Output relative abundances are expressed in percent and written in the same format as the input

## Limitations

- Pre-computed GCN estimate files cover only the Greengenes database (IMG 4.0 genomes, Oct 2012 taxonomy); other taxonomies such as Silva are not yet supported
- GCN correction is limited to 16S rRNA gene copy number; correction of GC percent bias, genome length bias, and fungal ITS variation requires other pre-computed estimate files
- For communities with tens to thousands of species, the effects of correction are non-trivial to predict
- No versioned DOI/changelog is provided in the discussion; the README identifies copyrighter version 0.46

## Evidence

- [readme] Input community file must contain read counts (not percentages) and taxa must have UNALTERED taxonomic assignments.: [evidence span withheld — anchor status: external_doc]
- [readme] Greengenes 2012/10 taxonomic string format: [evidence span withheld — anchor status: external_doc]
- [discussion] GCN estimates cover all Greengenes OTUs and taxa: "We have produced freely available data files of the phylogenetically-based estimates of GCN for all OTUs and taxa in the Greengenes database."
- [discussion] Recommendation of phylogenetic-level correction: "We recommend running Copyrighter systematically on every microbial sample, if possible using phylogenetic-level rather than taxonomic-level correction"
- [intro] GCN range check for inferred values: "GCN variation spans over an order of magnitude, from 1 to 15 in Bacteria, but only up to 5 in Archaea"
- [readme] Taxonomy mapping via Bio::Community: [evidence span withheld — anchor status: external_doc]
