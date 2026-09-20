---
name: nucleotide-seed-read-expansion
description: Use when you have raw paired-end Illumina reads from isolate genomes
  or metagenomes and need to recover CRISPR arrays or a repeat-defined locus without
  complete genome assembly, especially when the target repeat sequence is known.
when_to_use_negative:
- When a complete genome assembly is required for other downstream analyses — targeted
  assembly only recovers repeat-adjacent and cas gene regions, not the whole genome
- When no CRISPR repeat or cas gene reference (HMMs or nucleotide seeds) is available
  for the organism of interest, since DNA-mode seeding depends on a user-defined repeat
  FASTA
- For long-read (e.g., PacBio/ONT) datasets — the pipeline is built for short paired-end
  Illumina reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CasCollect
  role: Python/Perl wrapper implementing the seed generation and read subset expansion
    workflow (DNA mode via --nucl, -cycle, -match)
  repo: https://github.com/sandialabs/CasCollect
- name: VSEARCH
  role: Performs identity matching of reads against the user-defined CRISPR repeat
    seed set at the -match threshold (default 95%) during subset expansion
  repo: https://github.com/torognes/vsearch
- name: bbduk
  role: Quality trimming and adapter removal of raw reads prior to seeding (--trim
    flag)
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
- name: SPAdes
  role: Assembles the expanded seed read subsets into targeted contigs (metaSPAdes
    for metagenomes)
- name: HMMER
  role: Protein-mode seed generation against the default 120 Cas protein family profile
    HMMs
- name: CRISPRCasFinder
  role: Annotates the targeted assembly contigs to identify CRISPR arrays and cas
    protein genes (500 bp contig cutoff)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:17451d2898d4765bddc0ac07cc394966816e9fc582c3926f8029c6f9f80024aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CRISPRCasFinder
  - CasCollect
  - HMMER
  - SPAdes
  - VSEARCH
  - bbduk
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Nucleotide seed read expansion

## Summary

CasCollect's DNA-mode strategy for recovering a targeted genomic locus (e.g., cas operons and CRISPR arrays) from unassembled FASTQ reads: seed reads are matched against user-provided CRISPR repeat sequences via VSEARCH identity search, then the read subset is iteratively expanded over multiple cycles before targeted SPAdes assembly.

## When to use

Use when you have raw paired-end Illumina reads from isolate genomes or metagenomes and need to recover CRISPR arrays or a repeat-defined locus without complete genome assembly, especially when the target repeat sequence is known.

## When NOT to use

- When a complete genome assembly is required for other downstream analyses — targeted assembly only recovers repeat-adjacent and cas gene regions, not the whole genome
- When no CRISPR repeat or cas gene reference (HMMs or nucleotide seeds) is available for the organism of interest, since DNA-mode seeding depends on a user-defined repeat FASTA
- For long-read (e.g., PacBio/ONT) datasets — the pipeline is built for short paired-end Illumina reads

## Inputs

- Unassembled paired-end Illumina genomic sequencing reads (FASTQ), e.g., from the 66 CDC & FDA Antibiotic Resistance Isolate Bank P. aeruginosa isolates
- User-defined CRISPR repeat sequence FASTA (CasCollect ships 105 CRISPR repeat sequences as individual Fasta files, e.g., Pseudomonas_aeruginosa_DK2.fas)

## Outputs

- Expanded read subsets corresponding to CRISPR/Cas loci
- Targeted SPAdes assemblies of cas operons and CRISPR arrays (contigs)
- CRISPRCasFinder annotations of CRISPR arrays and cas protein genes per isolate, written to the -out folder

## How to apply

Enable DNA mode seed generation in CasCollect with --nucl and supply a user-defined CRISPR repeat FASTA (e.g., the P. aeruginosa CRISPR repeat for the 66 CDC & FDA Antibiotic Resistance Isolate Bank isolates), keeping the default 120 Cas protein HMMs active for protein-based seed detection via HMMER. Run --trim (bbduk/bbmerge from BBTools) to quality-trim, remove adapters, and merge overlapping reads before seeding. Expand matched seed read subsets iteratively with the -cycle option (default five cycles; the P. aeruginosa case study used -cycle 2), where VSEARCH at the default 95% percent match (-match) retrieves reads overlapping the seed set each cycle; previously matched reads are removed from the database between cycles to eliminate redundant searches. Assemble each expanded subset with SPAdes (metaSPAdes for metagenomes) and annotate contigs with CRISPRCasFinder, using a 500 bp contig cutoff. Rationale: seeding on the conserved repeat plus iterative identity-based expansion captures full CRISPR arrays and flanking cas operons from raw reads at almost two orders of magnitude less runtime than complete assembly.

## Related tools

- **CasCollect** (Python/Perl wrapper implementing the seed generation and read subset expansion workflow (DNA mode via --nucl, -cycle, -match)) — https://github.com/sandialabs/CasCollect
- **VSEARCH** (Performs identity matching of reads against the user-defined CRISPR repeat seed set at the -match threshold (default 95%) during subset expansion) — https://github.com/torognes/vsearch
- **bbduk** (Quality trimming and adapter removal of raw reads prior to seeding (--trim flag)) — https://jgi.doe.gov/data-and-tools/bbtools/
- **SPAdes** (Assembles the expanded seed read subsets into targeted contigs (metaSPAdes for metagenomes))
- **HMMER** (Protein-mode seed generation against the default 120 Cas protein family profile HMMs)
- **CRISPRCasFinder** (Annotates the targeted assembly contigs to identify CRISPR arrays and cas protein genes (500 bp contig cutoff))

## Examples

```
python CasCollect.py -i isolate_R1.fq -I isolate_R2.fq --trim --nucl -query Pseudomonas_aeruginosa_DK2.fas -cycle 2 -out pae_isolate_out
```

## Evaluation signals

- CRISPR arrays and cas operons are successfully detected by CRISPRCasFinder on the targeted contigs across the isolate panel (e.g., all 66 P. aeruginosa isolates), matching results from a complete assembly pipeline
- Seed expansion converges: each cycle retrieves reads overlapping the prior subset, and previously matched reads are removed so redundant searches are eliminated
- Identity matches between reads and the repeat seed set meet or exceed the 95% VSEARCH percent-match threshold (adjustable via -match)
- Targeted assembly runtime is substantially lower than complete assembly plus annotation (benchmark showed ~two orders of magnitude improvement) while detection is retained
- CasCollect pre-run check reports all required software (BBTools, Seqtk, FragGeneScan, HMMER, VSEARCH, SPAdes, CRISPRCasFinder) present in the user's path

## Limitations

- Detection depends on the supplied repeat reference: reads must match the user-defined CRISPR repeat at the chosen -match identity (default 95%) to seed expansion
- Only repeat-adjacent and cas-gene regions are assembled; the rest of the genome is not recovered, and default expansion depth (five cycles) or match stringency may miss divergent or distant flanking regions
- No software version numbers or dataset accession IDs are provided in the article, complicating exact reproducibility

## Evidence

- [results] DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats: "DNA mode implements VSEARCH (22) for identity matching to a user-defined set of CRISPR array repeats"
- [results] default identity match 95% adjustable with -match: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] default five expansion cycles adjustable with -cycle: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
- [results] matched reads removed between cycles: "This process eliminates redundant searches by removing reads from the database set that have been matched in the previous cycle"
- [readme] --trim flag behavior in README: "--trim              runs quality trim, remove adapters, merge overlapping
                      reads"
- [results] The assembly function employs SPAdes on seed/expansion reads: "The assembly function employs the SPAdes assembler (23) using the subset of reads from the seed and expansion steps"
