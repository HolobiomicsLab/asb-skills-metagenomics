---
name: command-line-pipeline-execution
description: Use when a large FASTA protein set (thousands to >190,000 (meta)genomic
  records) must be screened from the command line against an HMM library, with configurable
  output separator, prefixes, threads, and sequence extraction.
when_to_use_negative:
- The input is already a classified protein family or hitlist rather than raw FASTA
  protein sequences to be scanned against the HMM library
- The task requires phylogenetic tree-based classification of DsrAB sequences instead
  of fast HMM-based prediction (DiSCo was designed to avoid time-consuming phylogenetic
  reconstructions)
- Only a single marker protein such as DsrD is available, which the article warns
  may lead to incorrect assignments of dissimilatory sulphur metabolism
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: DiSCo
  role: Standalone Perl command-line pipeline that runs hmmsearch of FASTA input against
    the DiSCo HMM library, applies the filter script with best-hit strategy, and writes
    the filtered hitlist and optional FASTA sequences
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: Perl 5
  role: Implementation language in which DiSCo.pl and the filter script are written
    and executed from the shell
- name: hmmsearch 3.1b2
  role: HMMER3 search engine invoked by DiSCo.pl against the DiSCo HMM library; thread
    count controlled via the -n option and must be in $PATH
  repo: http://hmmer.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:8729db003754165cd1c937b981ee10d42942459294d9eadd4c051bfb0b7d0871
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# command-line-pipeline-execution

## Summary

Execute a standalone Perl/HMMER3 command-line tool (DiSCo.pl) with configurable flags to scan FASTA protein sets against a curated HMM library, producing filtered hitlists and FASTA sequence outputs for large-scale (meta)genomic screening.

## When to use

Use when a large FASTA protein set (thousands to >190,000 (meta)genomic records) must be screened from the command line against an HMM library, with configurable output separator, prefixes, threads, and sequence extraction.

## When NOT to use

- The input is already a classified protein family or hitlist rather than raw FASTA protein sequences to be scanned against the HMM library
- The task requires phylogenetic tree-based classification of DsrAB sequences instead of fast HMM-based prediction (DiSCo was designed to avoid time-consuming phylogenetic reconstructions)
- Only a single marker protein such as DsrD is available, which the article warns may lead to incorrect assignments of dissimilatory sulphur metabolism

## Inputs

- Protein sequences in FASTA format (user-supplied, via -i <filename>)
- DiSCo HMM library (default current dir ./, override with -m)
- Filter Perl script (default current dir ./, override with -f)
- Optional user-defined cutoffs table: tab-separated, column 1 model, column 2 score, column 3 E-value (via -t)

## Outputs

- Filtered DiSCo hitlist (disco_hitlist.tsv) with configurable separator (TAB, comma, semicolon, or space), containing sequence identifier, HMM name, predicted enzyme type, scores, E-values, accuracy, domain coordinates, annotation, and DsrC-type notes
- Optional FASTA file of DiSCo hit sequences (with -o) for downstream analyses

## How to apply

Assemble input protein sequences in FASTA format, then invoke the pipeline via 'perl DiSCo.pl -i <file>' under default settings, relying on the HMM library and filter script located in the current directory (or overridden with -m and -f). Set a filename prefix for the filtered hitlist with -p, choose the hitlist separator with -s (e.g. -s 1 for comma), and enable -o so DiSCo hits are also emitted as FASTA sequences for downstream analyses. For large datasets like the 193,978 assemblies plus 1900 literature-derived (meta)genomes, allocate multiple hmmsearch threads via -n to reduce runtime, since DiSCo can analyse thousands of genomes in hours on a personal computer. Verify hits fulfil the hmmer-specific cutoffs — conditional and independent E-value ≤ 1e-10, accuracy ≥ 0.5, and bias one order of magnitude below score — with the best-hit strategy applied per input sequence.

## Related tools

- **DiSCo** (Standalone Perl command-line pipeline that runs hmmsearch of FASTA input against the DiSCo HMM library, applies the filter script with best-hit strategy, and writes the filtered hitlist and optional FASTA sequences) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **Perl 5** (Implementation language in which DiSCo.pl and the filter script are written and executed from the shell)
- **hmmsearch 3.1b2** (HMMER3 search engine invoked by DiSCo.pl against the DiSCo HMM library; thread count controlled via the -n option and must be in $PATH) — http://hmmer.org/

## Examples

```
perl DiSCo.pl -i proteomes.faa -p disco -s 1 -o -n 8
```

## Evaluation signals

- The filtered hitlist file (with the chosen -p prefix and -s separator) exists and parses as the expected delimited table with protein family and predicted type columns per hit
- Every reported hit fulfils the DiSCo cutoffs: conditional and independent E-value ≤ 1e-10, accuracy ≥ 0.5, and bias one order of magnitude lower than score
- The best-hit strategy was applied per input sequence (all hits to the same input sequence compared, single best assignment retained)
- Lineage/phylum aggregation of DsrA/DsrB hits matches expected scale (e.g. 1798 lineages across 66 phyla when run on the full 195,878-record dataset)
- If -o was set, the FASTA output contains sequences corresponding one-to-one with hitlist entries

## Limitations

- Requires hmmsearch (HMMER3) to be present in $PATH; only Perl 5 and HMMER3 dependencies are documented
- User-defined cut-offs via -t expect a specific tab-separated schema (model, score, E-value); deviations will break threshold application
- No software version number is stated for DiSCo in some sections, complicating reproducibility pinning

## Evidence

- [abstract] DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers.: "DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers."
- [intro] The first step of DiSCo.pl consists of an hmmsearch of a given protein sequences set in FASTA format against the DiSCo HMM library: [evidence span withheld — anchor status: not_found]
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [readme] perl DiSCo.pl -i example.faa -p example -s 1 -o #runs DiSCo with given filename prefix, specifies outfile separator, and produces sequence outfile: "perl DiSCo.pl -i example.faa -p example -s 1 -o #runs DiSCo with given filename prefix, specifies outfile separator, and produces sequence outfile"
- [readme] All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH.: "All programs are written in perl (version 5) and dependent on HMMER3. 
The script requires hmmsearch to be in your $PATH."
