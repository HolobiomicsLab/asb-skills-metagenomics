---
name: metagenomic-protein-sequence-screening
description: Use when you need to identify and type-classify proteins (e.g. DsrA/DsrB
  and paralogous families) in thousands of (meta)genomic protein sequences in FASTA
  format, at scale, without phylogenetic reconstruction.
when_to_use_negative:
- Input is nucleotide sequence or raw reads rather than protein sequences in FASTA
  format — DiSCo consumes protein FASTA only
- You require precise phylogenetic placement of DsrAB sequences rather than fast type-level
  classification — the paper notes phylogenetic approaches, though slower, are an
  alternative when tree topology matters
- Only a handful of complete genomes are available and a one-off check against known
  annotations suffices — blastp-based verification may be simpler
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0092
tools:
- name: DiSCo
  role: Standalone classifier that runs hmmsearch of input protein FASTA against its
    HMM library, filters hits via best-hit strategy, and outputs a typed hitlist
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search engine invoked by DiSCo.pl to score input sequences against
    the DiSCo HMM profiles; thread count set via -n
  repo: http://hmmer.org/
- name: Perl 5
  role: Implementation language of DiSCo.pl and its filter script; required on $PATH
    along with hmmsearch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:bb7aa70591341c994c3d2710ed0e91bf3ae67c42a1344ea6633149bdad0f34b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# metagenomic-protein-sequence-screening

## Summary

Screen large (meta)genomic protein datasets with DiSCo, a Perl 5 tool that runs hmmsearch against a curated HMM library to detect Dsr-dependent dissimilatory sulphur metabolism proteins and predict their enzyme type, expanding coverage from 103 complete genomes to 1738 micro-organisms across 1798 lineages.

## When to use

Use when you need to identify and type-classify proteins (e.g. DsrA/DsrB and paralogous families) in thousands of (meta)genomic protein sequences in FASTA format, at scale, without phylogenetic reconstruction.

## When NOT to use

- Input is nucleotide sequence or raw reads rather than protein sequences in FASTA format — DiSCo consumes protein FASTA only
- You require precise phylogenetic placement of DsrAB sequences rather than fast type-level classification — the paper notes phylogenetic approaches, though slower, are an alternative when tree topology matters
- Only a handful of complete genomes are available and a one-off check against known annotations suffices — blastp-based verification may be simpler

## Inputs

- User-supplied protein sequences in FASTA format (e.g. all proteins from 193,978 (meta)genomic assemblies plus 1900 literature-derived (meta)genomes)
- DiSCo HMM library (default location: current directory, overridable with -m)
- Optional user-defined cutoff table (-t: model, score, E-value per line, tab-separated)

## Outputs

- Filtered DiSCo hitlist file (filename via -p prefix) with 20 columns: sequence identifier, HMM name, predicted enzyme type, sequence/HMM lengths, score, bias, E-value, accuracy, alignment details, domain coordinates, input annotation, and DsrC-type notes (1 or 2 cysteines)
- Optional FASTA file of DiSCo hit sequences (with -o) for downstream analyses

## How to apply

Assemble input protein sequences in FASTA format (e.g. 193,978 assemblies plus 1900 literature-derived (meta)genomes), then run `perl DiSCo.pl -i <input.faa>` so hmmsearch 3.1b2 scans the sequences against the DiSCo HMM library in the current directory using the bundled filter script. Apply the cutoffs: conditional and independent E-value <= 1e-10, accuracy >= 0.5, and bias one order of magnitude below the score; retain only hits passing the best-hit strategy where all hits to the same input sequence are compared. Use `-p <prefix>` for the filtered hitlist filename, `-s 1` for comma separation, `-o` to emit hit sequences as FASTA for downstream analyses, and `-n <threads>` to parallelize hmmsearch on large datasets. Aggregate resulting DsrA/DsrB hits by lineage and phylum; in the source study this yielded 1738 micro-organisms, 1798 lineages, and 66 phyla, including new Thaumarchaeota and Spirochaetes lineages.

## Related tools

- **DiSCo** (Standalone classifier that runs hmmsearch of input protein FASTA against its HMM library, filters hits via best-hit strategy, and outputs a typed hitlist) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search engine invoked by DiSCo.pl to score input sequences against the DiSCo HMM profiles; thread count set via -n) — http://hmmer.org/
- **Perl 5** (Implementation language of DiSCo.pl and its filter script; required on $PATH along with hmmsearch)

## Examples

```
perl DiSCo.pl -i metagenome_proteins.faa -p disco -s 1 -o -n 8
```

## Evaluation signals

- Confirm the filtered hitlist file (disco_hitlist.tsv) exists and is delimited as configured (e.g. comma with -s 1)
- Verify every reported hit fulfils the cutoffs: conditional and independent E-value <= 1e-10, accuracy >= 0.5, and bias one order of magnitude below score
- Check that the hitlist contains the expected columns, including protein family and predicted enzyme type per hit
- Confirm the best-hit strategy was applied: for each input sequence only the best qualifying hit is retained
- Compare aggregated lineage counts to expectations from the paper (e.g. 1798 lineages across both prokaryotic domains; benchmark accuracies of 0.98 for complete genomes and 0.93 for metagenomic records)

## Limitations

- HMM-based prediction trades phylogenetic resolution for speed; the paper notes phylogenies can misassign organisms with reductive-type Dsr systems that are dSDM in vivo, and conversely absence of phylogeny limits placement
- Paralogous families of small marker proteins (~100 amino acids, e.g. DsrEFH) are hard to distinguish by other marker-based approaches; users should trust DiSCo's HMM cutoffs rather than single markers like DsrD
- Presence of a reductive-type DsrAB complex alone is not a good proxy for use of oxidized sulphur compounds as electron acceptors — predicted type must be interpreted metabolically
- hmmsearch must be in the user's $PATH; default HMM library and filter script locations assume the current working directory unless -m/-f are given

## Evidence

- [abstract] We ran DiSCo against a dataset composed of over 190 thousand (meta)genomic records and efficiently mapped Dsr-dependent dissimilatory sulphur proteins in 1798 lineages across both prokaryotic domains.: "We ran DiSCo against a dataset composed of over 190 thousand (meta)genomic records and efficiently mapped Dsr-dependent dissimilatory sulphur proteins in 1798 lineages"
- [intro] DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5: "DiSCo hits need to fulfil the cutoff of the hmmer-specific conditional and independent E-value of ≤10−10, an accuracy of ≥0.5"
- [discussion] with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records: "with a minimal accuracy of 0.98 for complete genomes and 0.93 for metagenomic records"
- [readme] perl DiSCo.pl -i example.faa -p example -s 1 -o #runs DiSCo with given filename prefix, specifies outfile separator, and produces sequence outfile: "perl DiSCo.pl -i example.faa -p example -s 1 -o #runs DiSCo with given filename prefix, specifies outfile separator, and produces sequence outfile"
