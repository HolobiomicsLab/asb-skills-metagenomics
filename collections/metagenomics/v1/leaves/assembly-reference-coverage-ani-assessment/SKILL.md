---
name: assembly-reference-coverage-ani-assessment
description: Use when evaluating a de novo (strain-resolved or strain-oblivious) metagenome
  assembly against known reference genomes, needing per-contig reference coverage
  percentages and nucleotide identity to judge strain separation quality.
when_to_use_negative:
- No reference genomes are available for the community — coverage and ANI cannot be
  computed; use completeness/contiguity metrics instead.
- You only need to check that Strainberry ran and produced scaffolds, not quantify
  separation quality against references.
- The community is high-complexity and uncharacterized; Strainberry and this evaluation
  target low-complexity metagenomes with known strains.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Strainberry
  role: Produces the strain-resolved assembly (assembly.scaffolds.fa) whose scaffolds
    are evaluated for reference coverage and ANI
  repo: https://github.com/rvicedomini/strainberry
- name: metaFlye
  role: Generates the strain-oblivious metagenome assembly baseline for coverage/ANI
    comparison
  repo: https://github.com/rvicedomini/strainberry-analyses
- name: minimap2
  role: Aligns scaffolds or reads to references to compute coverage and identity
  repo: https://github.com/rvicedomini/strainberry-analyses
- name: Snakemake
  role: Drives the evaluation workflows that generate evaluation tables and graphs
    per dataset
  repo: https://github.com/rvicedomini/strainberry-analyses
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-24515-9
    title: 'Strainberry: automated strain separation in low-complexity metagenomes
      using long reads'
  source_content_hash: sha256:c144ce8182dd23c0a9fa74a2299ec1420a5367aac4b776c6ddc74541aad5a30f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rvicedomini/strainberry
  tools:
  - Snakemake
  - Strainberry
  - conda
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Assembly reference coverage and ANI assessment

## Summary

Quantifies strain-resolved assembly quality by computing per-scaffold reference coverage and average nucleotide identity (ANI) against known strain references, the metrics Strainberry's Fig. 4 sensitivity curves are built on.

## When to use

Use when evaluating a de novo (strain-resolved or strain-oblivious) metagenome assembly against known reference genomes, needing per-contig reference coverage percentages and nucleotide identity to judge strain separation quality.

## When NOT to use

- No reference genomes are available for the community — coverage and ANI cannot be computed; use completeness/contiguity metrics instead.
- You only need to check that Strainberry ran and produced scaffolds, not quantify separation quality against references.
- The community is high-complexity and uncharacterized; Strainberry and this evaluation target low-complexity metagenomes with known strains.

## Inputs

- Strain-resolved assembly FASTA (e.g. sberry_out/assembly.scaffolds.fa)
- Optional: strain-oblivious metaFlye assembly (flye_out/assembly.fasta) for comparison
- Known strain reference genomes (e.g. simulated community references, E. coli K12 and W)
- Long-read alignment BAM (assembly.scaffolds.bam) optionally for read-level coverage checks

## Outputs

- Per-scaffold reference coverage and ANI table (coverage_ani_table.tsv)
- Aggregate statistics (mean reference coverage %, mean nucleotide identity %) per strain per assembly

## How to apply

Take the final strain-resolved assembly (Strainberry's OUTPUT_DIR/assembly.scaffolds.fa, or a strain-oblivious metaFlye assembly for comparison) and the set of known reference genomes for the community. For each scaffold, align it to the references and compute the fraction of each reference covered and the average nucleotide identity of the aligned region. Write per-scaffold values to a coverage/ANI table (e.g. coverage_ani_table.tsv). Interpret results using the paper's benchmarks: at 60-80x average depth, individual strains of low-complexity metagenomes assemble with almost complete reference coverage and sequence identity exceeding 99.9%; quality degrades as strain number rises (reference coverage dropping from 95% at 3 strains to 75% at 5 strains, ANI from 99.9% to 99.6%), so compare against the expected pattern for the strain count and coverage of your mock community.

## Related tools

- **Strainberry** (Produces the strain-resolved assembly (assembly.scaffolds.fa) whose scaffolds are evaluated for reference coverage and ANI) — https://github.com/rvicedomini/strainberry
- **metaFlye** (Generates the strain-oblivious metagenome assembly baseline for coverage/ANI comparison) — https://github.com/rvicedomini/strainberry-analyses
- **minimap2** (Aligns scaffolds or reads to references to compute coverage and identity) — https://github.com/rvicedomini/strainberry-analyses
- **Snakemake** (Drives the evaluation workflows that generate evaluation tables and graphs per dataset) — https://github.com/rvicedomini/strainberry-analyses

## Examples

```
strainberry -r assembly.fasta -b alignment.sorted.bam -o sberry_out -c 4  # then compute coverage/ANI of sberry_out/assembly.scaffolds.fa vs. references into coverage_ani_table.tsv
```

## Evaluation signals

- coverage_ani_table.tsv contains one row per output scaffold with both reference coverage (%) and ANI (%) columns populated.
- At 60-80x average coverage, expected reference coverage is near-complete with ~99.9% base accuracy/ANI.
- For the two-strain K12/W example, assembly.scaffolds.fa yields two scaffolds, one closer to K12 and one closer to W, each with high ANI to its respective reference.
- Sensitivity trend reproduced: average reference coverage and ANI decrease monotonically as conspecific strain number increases from 3 to 5 (95%→75% coverage; 99.9%→99.6% identity).
- Evaluation tables and graphs appear under results/[dataset]/evaluation after running the Snakemake workflow.

## Limitations

- Strainberry can separate at most a limited number of well-represented strains; coverage and ANI degrade with 5+ conspecific strains (75% coverage, 99.6% identity at 5 E. coli strains).
- Uneven strain abundances break the even-coverage phasing assumption, so low-coverage strains will show poorer reference coverage and ANI.
- No explicit Strainberry version was stated in the article; results may vary across versions (analyses repo pinned version 1.1).

## Evidence

- [abstract] Strainberry produced strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy on benchmarked mock communities.: "We benchmarked Strainberry on mock communities for which it produces strain-resolved assemblies with near-complete reference coverage and 99.9% base accuracy."
- [discussion] Depth of coverage sufficiency for strain assembly quality: "An average depth of coverage of 60–80× suffices to assemble individual strains of low-complexity metagenomes with almost complete coverage and sequence identity exceeding 99.9%."
- [readme] Expected output of a successful Strainberry run on the example dataset: "After a successful run of Strainberry, in the `sberry_out` directory, the file `assembly.scaffolds.fa` should contain two scaffolds (one closer to strain K12, the other closer to strain W)."
- [readme] Evaluation outputs location in the analyses workflows: [evidence span withheld — anchor status: external_doc]
- [discussion] Uneven strain abundances are a known limitation of the phasing approach: "While we showed this is not a limitation for the actual separation of a limited number of well-represented strains, we believe it is an important development worth pursuing."
