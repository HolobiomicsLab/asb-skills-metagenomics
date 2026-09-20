---
name: hgt-breakpoint-detection
description: Use when paired-end shotgun metagenomics reads (e.g., HMP IBD data) are
  aligned against a bacterial reference set and the goal is to identify horizontal
  gene transfer breakpoint pairs before local strain reconstruction.
when_to_use_negative:
- The input is RNA-seq or metatranscriptomic data intended for gene-fusion calling
  (use STAR-Fusion for that role instead; it serves here only as orthogonal validation
  within 200 bp of HGT breakpoints)
- Only short single-end reads are available — the method relies on paired junction
  reads and split reads to locate breakpoint pairs
- The reference set has no donors/receptors relevant to the sample, so junction pairs
  mapping to two different references cannot be formed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: LEMON
  role: 'Primary pipeline: get_raw_bkp.py clusters discordant junction pairs with
    DBSCAN; get_acc_bkp refines exact breakpoint positions from split reads'
  repo: https://github.com/lichen2018/LEMON
- name: BWA
  role: Aligns paired-end shotgun reads against the reference genomes to identify
    junction reads (bwa mem -M)
- name: LUMPY
  role: Provides extractSplitReads_BwaMem to extract split reads from the BWA alignment
    as LEMON input
- name: samtools
  role: Filters unique reads (samtools view -q 20) and sorts alignments for the breakpoint-detection
    inputs
  repo: https://github.com/samtools/htslib
- name: DBSCAN
  role: Density-based clustering (via sklearn) of breakpoint-pair candidates using
    Euclidean distance, eps = average insert size, minsample = 1
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:bcc65163c13b7c0afba71784b5a0445880abf7074b3ce137a42a727f5c67e168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - DBSCAN
  - Gurobipy
  - LEMON
  - LUMPY
  - WGSIM
  - iRep
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hgt-breakpoint-detection

## Summary

Detect HGT breakpoints from existing shotgun NGS metagenomics datasets using LEMON's two-stage pipeline: DBSCAN clustering of discordant junction read pairs for raw breakpoints, then split-read refinement for exact positions, as a prerequisite for reconstructing local strains at HGT sites.

## When to use

Use when paired-end shotgun metagenomics reads (e.g., HMP IBD data) are aligned against a bacterial reference set and the goal is to identify horizontal gene transfer breakpoint pairs before local strain reconstruction.

## When NOT to use

- The input is RNA-seq or metatranscriptomic data intended for gene-fusion calling (use STAR-Fusion for that role instead; it serves here only as orthogonal validation within 200 bp of HGT breakpoints)
- Only short single-end reads are available — the method relies on paired junction reads and split reads to locate breakpoint pairs
- The reference set has no donors/receptors relevant to the sample, so junction pairs mapping to two different references cannot be formed

## Inputs

- Paired-end shotgun NGS FASTQ reads (e.g., HMP IBD metagenomic samples)
- Metagenomic reference FASTA (e.g., 16,093 NCBI bacterial species, 1,246,881 scaffolds)
- Unique-mapping BAM file (samtools view -q 20)
- Split-read BAM file (lumpy-sv extractSplitReads_BwaMem)
- Raw breakpoints file produced by get_raw_bkp.py (input to get_acc_bkp)

## Outputs

- raw_breakpoints.txt: candidate breakpoint pairs, one cluster per HGT event, each supported by at least one junction pair
- Accurate breakpoints file: refined breakpoint pair positions from split-read evidence
- Sensitivity and FDR metrics (with 20 bp tolerance) when run on simulated data with known breakpoints

## How to apply

Map paired-end shotgun reads against the metagenomic reference with BWA ('bwa mem -M'), then derive the unique-read BAM ('samtools view -q 20') and split-read BAM ('lumpy-sv extractSplitReads_BwaMem') required by the LEMON workflow. Run 'get_raw_bkp.py' on the unique BAM: it selects junction pairs whose two sides map to two different references, converts each pair's mapped positions to a 2D point, and clusters candidates with DBSCAN using Euclidean distance, eps set to the average insert size and minsample = 1, writing raw breakpoint pairs to a raw breakpoints file. Refine exact breakpoint positions with 'LEMON/getAccBkp/get_acc_bkp', which uses split reads and a log-likelihood quality score of split-read alignments to pick precise breakpoints from each raw cluster. Score detected breakpoints against truth with a 20 bp tolerance (a detection within 20 bp of a true position counts as correct) when evaluating sensitivity (true detected / all true) and FDR (false discovered / all discovered). On simulations (10k bp transferred segments, WGSIM reads at 2X/5X/70X) LEMON achieved 22.39% sensitivity at 2X and 51.19% at 5X with FDR as low as 0.002, outperforming LUMPY's default calls (6.86%/31.12% sensitivity).

## Related tools

- **LEMON** (Primary pipeline: get_raw_bkp.py clusters discordant junction pairs with DBSCAN; get_acc_bkp refines exact breakpoint positions from split reads) — https://github.com/lichen2018/LEMON
- **BWA** (Aligns paired-end shotgun reads against the reference genomes to identify junction reads (bwa mem -M))
- **LUMPY** (Provides extractSplitReads_BwaMem to extract split reads from the BWA alignment as LEMON input)
- **samtools** (Filters unique reads (samtools view -q 20) and sorts alignments for the breakpoint-detection inputs) — https://github.com/samtools/htslib
- **DBSCAN** (Density-based clustering (via sklearn) of breakpoint-pair candidates using Euclidean distance, eps = average insert size, minsample = 1)

## Examples

```
python LEMON/Scripts/get_raw_bkp.py -r ref.fasta -u unique.bam -o raw_breakpoints.txt && LEMON/getAccBkp/get_acc_bkp -r ref.fasta -u unique.bam -s split.bam -b raw_breakpoints.txt -o accurate_breakpoints.txt
```

## Evaluation signals

- Every raw breakpoint cluster is supported by at least one junction pair (minsample = 1) before split-read refinement
- On simulated data (WGSIM at 2X/5X/70X), sensitivity and FDR computed with a 20 bp tolerance approach the reported values: LEMON 22.39%/51.19% sensitivity at 2X/5X, 0.002 FDR at 2X, 0.0034 FDR at 70X
- LEMON shows higher sensitivity and lower FDR than LUMPY defaults (6.86%/31.12% sensitivity, 0.016/0.010 FDR) across all coverage levels
- Accurate breakpoints are consistent between the raw clustered candidates and the split-read-refined positions
- In real data, HGT breakpoints should match STAR-Fusion gene fusion points within 200 bp upstream or downstream

## Limitations

- Sensitivity is low at very low coverage: only 22.39% of true breakpoints detected at 2X coverage in simulation, rising to 51.19% at 5X
- Breakpoint calls depend on junction reads whose two sides map to different references, so multi-mapping reads and repetitive reference regions reduce detectability
- DBSCAN eps must be set to the average insert size; misestimated insert size distribution can fragment or merge breakpoint clusters

## Evidence

- [methods] Select junction pairs whose two sides map to two different references: "Then, we select junction reads, whose two sides are mapped to two different references, from the set of mapped reads."
- [methods] DBSCAN clustering of breakpoint pair candidates: "We then apply the clustering algorithm DBSCAN [23] using Euclidean distance to cluster the breakpoint pair candidates."
- [methods] Split reads give exact breakpoint positions: "Fourthly, we utilize split reads to get the exact positions of HGT breakpoints."
- [abstract] LEMON package takes shotgun NGS datasets: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
- [readme] get_raw_bkp.py usage from README: [evidence span withheld — anchor status: not_found]
- [readme] get_acc_bkp requires unique and split BAMs: "-u FILE unique reads bam file -s FILE split reads bam file -b FILE raw breakpoints file -o FILE accurate reakpoints file"
