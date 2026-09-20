---
name: cgmlst-allele-calling
description: Use when you have draft/complete genome FASTA assemblies and a defined
  MLST schema (schema_seed) and need to determine allelic profiles, identify known/novel
  alleles, or scale calling to hundreds/thousands of assemblies.
when_to_use_negative:
- You want to call alleles directly from sequencing reads rather than from assembled
  FASTA genomes — chewBBACA's AlleleCall engine requires draft assemblies or complete
  genomes (MentaLiST is the read-based alternative cited in the article)
- No reference schema exists yet — schema creation (CreateSchema) must be performed
  first before allele calling is possible
- Absolute allele assignments for paralogous loci are required — the 20 loci flagged
  in RepeatedLoci.txt should be removed from profiles due to uncertain allele assignment
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: chewBBACA
  role: AlleleCall module performs the BSR-based allele calling against the schema
    seed, detects novel alleles, and reports paralogous loci
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlying sequence alignment engine used by the BSR-based allele-calling
    algorithm
- name: Python 3.4
  role: Runtime environment in which chewBBACA is implemented and executed on laptops
    or HPC clusters
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:172ac5cb03950c8b70a429d4b056566596c2f2f4897d0aa3cef66898ed83220d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Python 3.4
  - chewBBACA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cgmlst-allele-calling

## Summary

Performs BLAST Score Ratio (BSR)-based allele calling of draft or complete bacterial genome FASTA assemblies against a pre-built cg/wgMLST schema using chewBBACA's AlleleCall module, producing allelic profile matrices and schema updates with novel alleles.

## When to use

Use when you have draft/complete genome FASTA assemblies and a defined MLST schema (schema_seed) and need to determine allelic profiles, identify known/novel alleles, or scale calling to hundreds/thousands of assemblies.

## When NOT to use

- You want to call alleles directly from sequencing reads rather than from assembled FASTA genomes — chewBBACA's AlleleCall engine requires draft assemblies or complete genomes (MentaLiST is the read-based alternative cited in the article)
- No reference schema exists yet — schema creation (CreateSchema) must be performed first before allele calling is possible
- Absolute allele assignments for paralogous loci are required — the 20 loci flagged in RepeatedLoci.txt should be removed from profiles due to uncertain allele assignment

## Inputs

- Directory of draft or complete genome FASTA assemblies (e.g. 100 S. agalactiae assemblies from FigShare, or the 32 complete genomes from the tutorial)
- Pre-built cg/wgMLST schema directory with schema_seed (from CreateSchema or a downloaded Chewie-NS/external schema)
- Optional Prodigal training file for the species (e.g. Streptococcus_agalactiae.trn)
- Optional loci list file (e.g. cgMLSTschema.txt from ExtractCgMLST) passed via --gl

## Outputs

- Allelic profile matrix (results_alleles.tsv) mapping strains to allele identifiers per locus
- Paralogous loci list (RepeatedLoci.txt)
- Updated schema with novel alleles appended (e.g. schema grew from 3,128 to 17,848 alleles in the tutorial)
- FASTA files containing the full allelic diversity of each locus

## How to apply

Run chewBBACA's AlleleCall module with the assemblies directory (-i) and the schema seed directory produced by schema creation (-g), tuning --cpu to the available cores; the underlying engine uses BLAST with a default BSR threshold of 0.6 to assign allele identifiers. For a targeted core-genome analysis, pass the cgMLST loci list (e.g. from ExtractCgMLST) via --gl to restrict calling to core loci. The run identifies novel alleles, assigns them identifiers, and adds them back into the schema (in the tutorial, 14,720 novel alleles raised the schema from 3,128 to 17,848 alleles). Inspect the RepeatedLoci.txt output and remove paralogous loci (e.g. with RemoveGenes) because allele assignment for those loci is uncertain. Runtime scales near-linearly with CPU count up to about 32 CPUs before disk/I/O becomes the bottleneck, with roughly 2.5 s per sample at 32 CPUs on an HPC for 100 S. agalactiae assemblies.

## Related tools

- **chewBBACA** (AlleleCall module performs the BSR-based allele calling against the schema seed, detects novel alleles, and reports paralogous loci) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying sequence alignment engine used by the BSR-based allele-calling algorithm)
- **Python 3.4** (Runtime environment in which chewBBACA is implemented and executed on laptops or HPC clusters)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Allele calling completes within seconds to minutes per strain on a laptop (tutorial baseline: ~32 seconds per genome, 17 minutes for 32 complete genomes on 6 CPU cores)
- Default BSR threshold of 0.6 was used (or explicitly overridden) and recorded
- Output results_alleles.tsv contains an allelic profile per input assembly and one column per schema locus
- Novel allele counts are consistent with schema growth (e.g. schema allele count increases after the call, as 3,128 → 17,848 in the tutorial)
- RepeatedLoci.txt is produced and paralogous loci are filtered from final profiles (tutorial: 20 loci removed, leaving 3,108 loci)

## Limitations

- Scaling benefits plateau beyond ~32 CPUs due to disk/I-O bottleneck; peak observed performance was ~2.5 s per sample on an HPC for 100 S. agalactiae assemblies
- Paralogous loci listed in RepeatedLoci.txt carry allele-assignment uncertainty and should be removed from profiles before downstream core-genome analyses
- Runtime depends strongly on hardware; using a less powerful machine than the tutorial's DELL XPS13 (6-core i7-10710U) can greatly increase analysis duration

## Evidence

- [abstract] The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable for the analysis of large datasets: "The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable for the analysis of large datasets"
- [readme] The allele call step determines the allelic profiles of the analyzed strains, identifying known and novel alleles in the analyzed genomes.: [evidence span withheld — anchor status: external_doc]
- [readme] The allele call used the default BSR threshold of 0.6: [evidence span withheld — anchor status: external_doc]
- [discussion] The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [results] Benchmarking showed near-linear per-sample runtime decrease up to 32 CPUs, after which disk/I/O becomes the bottleneck: [evidence span withheld — anchor status: not_found]
