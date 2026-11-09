---
name: ortholog-hmm-gene-prediction
description: Use when you have assembled fungal genomes (high-quality or draft) and
  need to extract single-copy marker gene sequences to build a taxonomic marker catalog
  or assess genome completeness across a strain collection.
when_to_use_negative:
- Input data is raw sequencing reads rather than assembled genomes — marker prediction
  requires assemblies for gene prediction with AUGUSTUS
- The target is bacterial or archaeal marker extraction — the 758 HMMs here are the
  Fungi OrthoDB v10.1 set, specific to fungal universal orthologs
- You only need functional (protein) profiling rather than taxonomic marker genes
  — use a protein database workflow (e.g., FunOMIC-P with Diamond blastx) instead
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BUSCO 5.0.0
  role: Runs the HMM-based search for 758 fungal single-copy orthologs (Fungi OrthoDB
    v10.1) in each genome to identify marker genes
- name: AUGUSTUS
  role: Gene predictor used by BUSCO (default parameters) to predict gene structures
    during marker identification
- name: CD-HIT
  role: Clusters extracted marker genes at a 99% identity threshold to remove redundancy
    and yield the final marker catalog
- name: FunOMIC
  role: Pipeline into which the resulting FunOMIC-T marker-gene database is integrated
    for read-based taxonomic profiling
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:7968d5e337f1d5351e2ab757a08760e8379b79675bc0f3207a6dac3a7f941371
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - AUGUSTUS
  - BUSCO
  - BUSCO 5.0.0
  - CD-HIT
  - FunOMIC
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ortholog-hmm-gene-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify single-copy marker genes in fungal genomes by running BUSCO with a curated set of universal single-copy ortholog HMMs (Fungi OrthoDB v10.1, 758 orthologs) using the AUGUSTUS gene predictor, then extract marker-gene sequences for database construction. This is the core step FunOMIC used to build its 1.6-million-gene FunOMIC-T taxonomic database.

## When to use

Use when you have assembled fungal genomes (high-quality or draft) and need to extract single-copy marker gene sequences to build a taxonomic marker catalog or assess genome completeness across a strain collection.

## When NOT to use

- Input data is raw sequencing reads rather than assembled genomes — marker prediction requires assemblies for gene prediction with AUGUSTUS
- The target is bacterial or archaeal marker extraction — the 758 HMMs here are the Fungi OrthoDB v10.1 set, specific to fungal universal orthologs
- You only need functional (protein) profiling rather than taxonomic marker genes — use a protein database workflow (e.g., FunOMIC-P with Diamond blastx) instead

## Inputs

- Assembled fungal genome FASTA files (high-quality genomes and draft genomes from NCBI and JGI MycoCosm)
- Fungi OrthoDB version 10.1 set of 758 fungal single-copy ortholog HMM profiles (bundled with BUSCO)
- Quality-filtered genome set (assemblies passing >500-contig and N50 >= 10 kbp screens)

## Outputs

- Per-genome BUSCO single-copy marker gene annotations and counts
- Extracted marker-gene nucleotide sequences (FASTA) per retained genome
- Non-redundant marker-gene catalog after 99% identity CD-HIT clustering (e.g., funomic_t_marker_genes.fasta, ~1.69 million genes)
- Per-taxon marker coverage table spanning genera, species, and strains (e.g., 1,916 species)

## How to apply

Collect reference fungal genomes (e.g., from NCBI and JGI MycoCosm) and pre-filter for assembly quality — FunOMIC removed genomes with more than 500 contigs and N50 < 10 kbp, retaining 4,839 of 9,401 genomes. Run BUSCO 5.0.0 in default mode against the Fungi OrthoDB version 10.1 set of 758 fungal single-copy ortholog HMMs, using the AUGUSTUS gene predictor with default parameters, on each genome individually. Discard genomes yielding fewer than 30 single-copy marker genes (FunOMIC retained 4,816 of 4,839 this way), then extract the marker-gene nucleotide sequences from the retained genomes. Finally, cluster the extracted markers with CD-HIT at a 99% identity threshold to remove redundancy, producing the ~1.69 million-sequence marker catalog (funomic_t_marker_genes.fasta). The rationale is that single-copy universal orthologs provide taxonomically informative, phylogenetically stable markers that generalize across eight fungal phyla.

## Related tools

- **BUSCO 5.0.0** (Runs the HMM-based search for 758 fungal single-copy orthologs (Fungi OrthoDB v10.1) in each genome to identify marker genes)
- **AUGUSTUS** (Gene predictor used by BUSCO (default parameters) to predict gene structures during marker identification)
- **CD-HIT** (Clusters extracted marker genes at a 99% identity threshold to remove redundancy and yield the final marker catalog)
- **FunOMIC** (Pipeline into which the resulting FunOMIC-T marker-gene database is integrated for read-based taxonomic profiling) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
busco -i genome.fasta -l fungi_odb10 --out genome_busco --mode genome
```

## Evaluation signals

- Each retained genome should yield at least 30 single-copy marker genes; genomes below this threshold are discarded (4,839 → 4,816 genomes in the reference run)
- The final non-redundant catalog should contain on the order of 1.6–1.69 million marker genes after 99% identity CD-HIT clustering
- Specificity check: fewer than 0.01% of the catalog's marker genes should map to the prokaryotic UHGG catalogue when aligned with bowtie2
- Mock-community check: a simulated bacterial community (903 genomes, 458 species, ~19.3 million reads) should show <0.01% mapping rate to the marker database (specificity 0.9999 in the mixed mock)
- Taxonomic coverage invariant: the catalog should span the expected taxa count derived from the retained genomes (475 genera, 1,916 species, 4,537 strains across eight fungal phyla in the reference run)

## Limitations

- Extraction and quality of single-copy marker genes depend on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomy compared with amplicon databases
- Genomes with fewer than 30 detected single-copy marker genes must be discarded, which can exclude highly fragmented draft assemblies
- AUGUSTUS gene prediction quality on divergent or low-coverage assemblies can affect the number of single-copy markers recovered per genome

## Evidence
