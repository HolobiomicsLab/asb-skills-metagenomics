---
name: cas-operon-annotation-verification
description: Use when assembled contigs from targeted (CasCollect) or complete assembly
  runs must be validated to confirm all known cas genes are recovered and cas operons/CRISPR
  arrays are correctly assembled and annotated.
when_to_use_negative:
- Input is raw unassembled reads rather than assembled contigs — run CasCollect's
  assembly/annotation steps first
- The goal is de novo discovery of novel cas gene families not represented in the
  120 default HMMs; user-supplied HMMs would need to be configured instead
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: CRISPRCasFinder
  role: Annotates targeted-assembly contigs with cas genes and CRISPR arrays
- name: HMMER
  role: Searches predicted ORFs against the default 120 Cas protein family profile
    HMMs to identify cas genes
- name: CasCollect
  role: Pipeline whose annotation step automates HMMER and CRISPRCasFinder annotation
    of assembled contigs
  repo: https://github.com/sandialabs/CasCollect
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:aa7b2bd69205de1f0705ed96f2ee64477b144a6718b4dfda010003b84b8f558e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - PRICE
  - Seqtk
  - bbduk
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cas-operon-annotation-verification

## Summary

Verification skill for confirming that assembled contigs (from CasCollect or full assembly pipelines) recover all known cas genes and yield correctly assembled, annotated cas operons and CRISPR arrays, using HMMER-based Cas protein HMM searches and CRISPRCasFinder annotation.

## When to use

Use when assembled contigs from targeted (CasCollect) or complete assembly runs must be validated to confirm all known cas genes are recovered and cas operons/CRISPR arrays are correctly assembled and annotated.

## When NOT to use

- Input is raw unassembled reads rather than assembled contigs — run CasCollect's assembly/annotation steps first
- The goal is de novo discovery of novel cas gene families not represented in the 120 default HMMs; user-supplied HMMs would need to be configured instead

## Inputs

- Assembled contigs in FASTA format from CasCollect targeted assembly, metaSPAdes complete assembly, or PRICE
- Default set of 120 Cas protein family profile HMMs shipped with CasCollect
- 105 CRISPR repeat sequences provided as individual FASTA files with CasCollect
- List of previously known cas genes/operons from the source isolate genomes for comparison

## Outputs

- Cas gene and CRISPR array annotations per contig (CRISPRCasFinder output)
- HMMER hit tables for the 120 Cas protein HMMs against predicted ORFs
- Verification table (e.g. cascollect_metagenome_benchmark.csv) mapping each pipeline's contigs to recovered cas genes and operon completeness

## How to apply

Collect contigs from each assembly run (CasCollect targeted assembly, complete metaSPAdes assembly, or PRICE baseline), apply the 500 bp contig size cutoff used by CasCollect's annotation script to reduce the number of contigs searched, then annotate with CRISPRCasFinder while searching the default set of 120 Cas protein family profile HMMs with HMMER as query. Compare the recovered cas gene set against the previously known cas genes from the source genomes (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39) to confirm nothing was missed, and inspect that cas operons and CRISPR arrays are assembled as intact loci rather than fragmented contigs. Tabulate per-run results (contig counts at 500 bp cutoff, cas genes detected, operon completeness) so runtime and contig-output differences between pipelines can be judged alongside annotation quality.

## Related tools

- **CRISPRCasFinder** (Annotates targeted-assembly contigs with cas genes and CRISPR arrays)
- **HMMER** (Searches predicted ORFs against the default 120 Cas protein family profile HMMs to identify cas genes)
- **CasCollect** (Pipeline whose annotation step automates HMMER and CRISPRCasFinder annotation of assembled contigs) — https://github.com/sandialabs/CasCollect

## Examples

```
CasCollect --protein --meta --trim pooled_metagenome_R1.fq pooled_metagenome_R2.fq
```

## Evaluation signals

- All previously known cas genes from each source isolate genome (e.g. E. coli KLY, P. aeruginosa VA-134, S. pyogenes M39) are identified in the annotation output
- Cas operons and CRISPR arrays are assembled and annotated as complete loci rather than fragmented across contigs
- Contigs are filtered with the 500 bp size cutoff before annotation, matching the pipeline's stated cutoff
- CasCollect and complete assembly pipelines agree on the set of cas genes detected, as reported for the three bacterial isolates

## Limitations

- Annotation relies on the default 120 Cas protein family HMMs; cas genes outside these families will not be detected unless user-supplied HMMs are provided
- The 500 bp contig size cutoff excludes shorter contigs, so truncated or partially assembled operon fragments may be silently dropped from annotation
- No software version numbers are given for HMMER, VSEARCH, SPAdes or CRISPRCasFinder, so exact reproduction of annotation results may vary across tool versions

## Evidence

- [results] The targeted assembly contigs are searched with the CRISPRCasFinder program (13).: "The targeted assembly contigs are searched with the CRISPRCasFinder program (13)."
- [results] Our Perl script for automating the annotation had set a size cutoff of 500 bp: "Our Perl script for automating the annotation had set a size cutoff of 500 bp, which reduced the total number of contigs analyzed for the complete assembly to an average of 161K contigs"
- [results] by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query: "by HMMER (21) with a default set of 120 Cas protein family profile HMMs (13) as query"
- [results] Both CasCollect and complete assembly pipelines were able to identify all known previously identified cas genes: "Both CasCollect and complete assembly pipelines were able to identify all known previously identified cas genes within each of the three bacterial isolate genomes."
- [results] We have included 105 CRISPR repeat sequences (13) as individual Fasta files with CasCollect: "We have included 105 CRISPR repeat sequences (13) as individual Fasta files with CasCollect"
