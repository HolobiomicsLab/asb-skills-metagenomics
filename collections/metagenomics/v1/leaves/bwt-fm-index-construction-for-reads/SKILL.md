---
name: bwt-fm-index-construction-for-reads
description: Use when a SAM seed alignment and a FASTA read set must be searched for
  read–read overlaps via an FM-index, e.g. to expand a small seed set before de novo
  haplotype assembly with TAR-VIR/PEHaplo.
when_to_use_negative:
- The target virus shares no conservation with any available reference genome — TAR-VIR's
  read classification stage cannot detect such a virus, so no seed SAM can be produced
- The task is de novo haplotype assembly rather than read recruitment — use PEHaplo
  directly instead of building the overlap-extension index
- The read set has not been error-corrected (e.g. with karect) — uncorrected errors
  undermine overlap-based recruitment
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0797
tools:
- name: TAR-VIR (Overlap_extension)
  role: Provides the 'build' binary that constructs the BWT/FM-index (generalized
    suffix array over concatenated reads) and the 'overlap' binary that queries it
    for iterative read recruitment
  repo: https://github.com/chjiao/TAR-VIR
- name: Bowtie2
  role: Produces the seed SAM alignment against a partial reference using the allowed
    error function 'L,0,-0.6' before index-based recruitment
- name: PEHaplo
  role: Downstream consumer of the recruited-read FASTA for de novo viral haplotype
    reconstruction
  repo: https://github.com/chjiao/PEHaplo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-2878-2
    title: 'TAR-VIR: a pipeline for TARgeted VIRal strain reconstruction from metagenomic
      data'
  source_content_hash: sha256:44c7fc67eb401033f1f4a20b25bf12f5928395a4f4bd5e91ab19a53662b7b2ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chjiao/TAR-VIR
  tools:
  - Bowtie2
  - PEHaplo
  - TAR-VIR
  - TAR-VIR (Overlap_extension)
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# BWT/FM-index construction for reads

## Summary

Builds a BWT/FM-index (a generalized suffix array over the concatenated reads) with TAR-VIR's Overlap_extension 'build' binary so that overlap detection between seed queries and the full read set is efficient during overlap-extension read recruitment.

## When to use

Use when a SAM seed alignment and a FASTA read set must be searched for read–read overlaps via an FM-index, e.g. to expand a small seed set before de novo haplotype assembly with TAR-VIR/PEHaplo.

## When NOT to use

- The target virus shares no conservation with any available reference genome — TAR-VIR's read classification stage cannot detect such a virus, so no seed SAM can be produced
- The task is de novo haplotype assembly rather than read recruitment — use PEHaplo directly instead of building the overlap-extension index
- The read set has not been error-corrected (e.g. with karect) — uncorrected errors undermine overlap-based recruitment

## Inputs

- Read set in FASTA format (e.g. error-corrected viral metagenomic reads, test_data/virus.fa)
- SAM file of seed reads aligned to a partial or remotely related reference (e.g. test_data/HIV.sam with 287 HIV-1 reads)
- Overlap cutoff parameter (set longer than half the read length, e.g. 180 for 250 bp reads)

## Outputs

- BWT/FM-index files under a user-specified prefix (e.g. 'virus')
- FASTA of recruited reads (e.g. virus_recruit.fa) containing reads beyond the seed set

## How to apply

Compile the Overlap_extension module with C++11 ('make' inside TAR-VIR/Overlap_extension), producing the 'build' and 'overlap' binaries. Run './build -f reads.fa -o prefix' on the (error-corrected) read set to construct the BWT/FM-index used for efficient overlap detection. Then run './overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff -o recruited_reads.fa', where align.sam holds the seed reads aligned by Bowtie2 (allowed error function 'L,0,-0.6') and the overlap cutoff should be set longer than half the read size (e.g. 180 for 250 bp reads) to prevent recruiting chimeric reads. The tool iteratively uses newly recruited reads as new queries until no additional reads can be recruited. Correct application is verified by counting reads in the output FASTA against known example values (e.g. 8,008 from 287 HIV-1 seed reads in the repository example).

## Related tools

- **TAR-VIR (Overlap_extension)** (Provides the 'build' binary that constructs the BWT/FM-index (generalized suffix array over concatenated reads) and the 'overlap' binary that queries it for iterative read recruitment) — https://github.com/chjiao/TAR-VIR
- **Bowtie2** (Produces the seed SAM alignment against a partial reference using the allowed error function 'L,0,-0.6' before index-based recruitment)
- **PEHaplo** (Downstream consumer of the recruited-read FASTA for de novo viral haplotype reconstruction) — https://github.com/chjiao/PEHaplo

## Examples

```
./build -f test_data/virus.fa -o virus; ./overlap -S test_data/HIV.sam -x virus -f test_data/virus.fa -c 180 -o virus_recruit.fa
```

## Evaluation signals

- The 'build' binary completes and produces index files under the requested prefix without C++11 compilation errors
- The recruited-read FASTA is produced and, for the repository's exact example (287 HIV-1 seed reads, cutoff 180), its read count is exactly 8,008
- Recruitment greatly expands the small seed set (in a related run, 19,714 seed reads against a partial HPgV CDS recruited 118,339 reads at cutoff 150), confirming iterative querying terminated on convergence
- The chosen overlap cutoff is longer than half the read size (e.g. 180 for 250 bp reads), per the article's guidance to avoid recruiting chimeric reads
- Recruited reads are specific to the target virus (no contamination from off-target viruses), as shown for SARS-CoV reads in the simulated benchmark

## Limitations

- A completely new virus sharing no conservation with any reference genome will not be detected, so no seed alignment (and hence no index query) can be produced
- No specific recommended overlap threshold value is stated in the article's discussion; users must pick cutoffs based on common string sizes between species
- No quantitative criteria are given for when local-mode screening via alignment length and score should accept or reject reads
- No software version numbers are provided for TAR-VIR or its components in the article
- Chimeric reads may be recruited if the overlap cutoff is not longer than half the read size

## Evidence

- [readme] build reads index
./build -f reads.fa -o prefix: "(1) build reads index    ./build -f reads.fa -o prefix   (2) recruite reads    ./overlap -S align.sam -x prefix -f reads.fa -c overlap_cutoff"
- [methods] we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads: "we set the overlap threshold longer than half of the read size to prevent recruiting these chimeric reads"
- [readme] This SAM file contains a small subset of aligned HIV-1 reads.: "This SAM file contains a small subset of aligned HIV-1 reads. With these aligned (287) reads, more HIV-1 reads can be recruited"
- [readme] If everything is good, the recruited reads number should be 8008.: "If everything is good, the recruited reads number should be 8008."
- [results] from only 19,714 seed reads aligned to a partial HPgV CDS reference, the overlap-extension step with an overlap cutoff of 150 bp recruited 118,339 reads: [evidence span withheld — anchor status: not_found]
