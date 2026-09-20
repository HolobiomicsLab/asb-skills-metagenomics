---
name: clustering-sensitivity-runtime-tradeoff-analysis
description: Use when clustering large protein sequence sets at ~50% sequence identity
  and Linclust's default m=20 k-mers per sequence yields more (smaller) clusters than
  a reference tool like UCLUST, prompting a test of whether --kmer-per-seq 80 closes
  the sensitivity gap at acceptable runtime.
when_to_use_negative:
- When the target clustering identity is 90% or 70% rather than 50% — the paper's
  sensitivity gap where Linclust produced 13% more clusters than UCLUST was characterized
  at the 50% identity level
- When the workload cannot absorb a 1.5–2x runtime increase over default Linclust,
  or when pairs are so costly to miscluster that exact quadratic-time pairwise verification
  is mandatory (Linclust's k-mer prefilter is approximate)
- When the input is not redundancy-reducible protein sequence sets but, e.g., already-clustered
  or profile data where k-mer shared-key assumptions do not hold
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MMseqs2/Linclust
  role: 'Runs the clustering under test: linclust at --min-seq-id 0.5, --cov-mode
    1, -c 0.9, varying --kmer-per-seq between 20 (default) and 80 while timing each
    run'
  repo: https://github.com/soedinglab/mmseqs2
- name: UCLUST
  role: Provides the reference clustering at --id 0.5 against which Linclust's recovered
    sensitivity (average cluster size) is compared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:f71d9a3ed5a43563937abcecbad59583b57ebcbe55ff6c851eda8f9c3e78512f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - MMseqs2
  - MMseqs2/Linclust
  - UCLUST
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# clustering-sensitivity-runtime-tradeoff-analysis

## Summary

Benchmarks Linclust's k-mer-per-sequence parameter (m) to quantify the sensitivity/runtime tradeoff at 50% sequence identity: raising m from 20 to 80 recovers cluster sensitivity to a level similar to UCLUST at only 1.5–2x runtime cost.

## When to use

Use when clustering large protein sequence sets at ~50% sequence identity and Linclust's default m=20 k-mers per sequence yields more (smaller) clusters than a reference tool like UCLUST, prompting a test of whether --kmer-per-seq 80 closes the sensitivity gap at acceptable runtime.

## When NOT to use

- When the target clustering identity is 90% or 70% rather than 50% — the paper's sensitivity gap where Linclust produced 13% more clusters than UCLUST was characterized at the 50% identity level
- When the workload cannot absorb a 1.5–2x runtime increase over default Linclust, or when pairs are so costly to miscluster that exact quadratic-time pairwise verification is mandatory (Linclust's k-mer prefilter is approximate)
- When the input is not redundancy-reducible protein sequence sets but, e.g., already-clustered or profile data where k-mer shared-key assumptions do not hold

## Inputs

- FASTA file of UniProt 2016_03 subset protein sequences
- MMseqs2 binary with the linclust module (commit 5e21868 in the paper)
- UCLUST executable for the reference clustering

## Outputs

- linclust_m80_comparison.csv summarizing average cluster sizes, runtimes, sensitivity recovered relative to UCLUST, and runtime overhead of m=80
- Three cluster result sets: Linclust with --kmer-per-seq 80, default Linclust (m=20), and UCLUST --id 0.5
- Runtime measurements for each of the three clustering runs

## How to apply

Load the UniProt 2016_03 subset FASTA and run Linclust via MMseqs2 twice at identical thresholds (--min-seq-id 0.5, --cov-mode 1, -c 0.9): once with --kmer-per-seq 80 and once with the default m=20, recording both runtimes. Run UCLUST with --id 0.5 as the reference clustering and record its runtime. The rationale is that Linclust selects the m k-mers with the lowest hash values per sequence, so larger m raises the chance that co-clustered sequences share a selected k-mer, improving prefilter sensitivity while runtime stays linear O(mN). Compute the average cluster size for each of the three clusterings (larger average clusters indicate fewer clusters, i.e. recovered sensitivity) and the runtime ratio of Linclust-m80 versus default Linclust. Summarize average cluster sizes, runtimes, sensitivity recovered relative to UCLUST, and the m=80 runtime overhead into linclust_m80_comparison.csv. Accept the result if Linclust-m80 shows increased average cluster size toward the UCLUST reference (consistent with the Supplementary Fig. S4 sensitivity trend) while runtime overhead remains moderate (~1.5–2x).

## Related tools

- **MMseqs2/Linclust** (Runs the clustering under test: linclust at --min-seq-id 0.5, --cov-mode 1, -c 0.9, varying --kmer-per-seq between 20 (default) and 80 while timing each run) — https://github.com/soedinglab/mmseqs2
- **UCLUST** (Provides the reference clustering at --id 0.5 against which Linclust's recovered sensitivity (average cluster size) is compared)

## Examples

```
mmseqs linclust uniprot2016_03.db m80_clust tmp --kmer-per-seq 80 --min-seq-id 0.5 --cov-mode 1 -c 0.9
```

## Evaluation signals

- Average cluster size of the Linclust m=80 run is larger than that of the default m=20 run at the same --min-seq-id 0.5 / --cov-mode 1 / -c 0.9 thresholds, indicating fewer clusters and recovered sensitivity
- Average cluster size of Linclust-m80 approaches the UCLUST --id 0.5 reference (paper reports Linclust at m=20 produced 13% more clusters than UCLUST at 50% identity)
- Runtime ratio of Linclust-m80 to default Linclust falls in the expected moderate range of about 1.5–2x
- linclust_m80_comparison.csv contains rows for all three clusterings with average cluster size, runtime, and the derived sensitivity and overhead columns populated
- Sensitivity trend across m values is directionally consistent with the paper's Supplementary Fig. S4

## Limitations

- The tradeoff was established on the UniProt 2016_03 benchmark at 50% sequence identity; sensitivity gains at other identity thresholds or datasets are not demonstrated by this task
- Raising m to 80 brings Linclust's sensitivity to a level 'similar to' UCLUST, not identical, and costs 1.5–2x the default runtime
- Linclust's k-mer prefilter is approximate; when falsely clustered pairs are costly (e.g., redundancy filtering), pairwise links still need verification with the exact similarity measure, which requires quadratic time

## Evidence

- [methods] k-mer selection method: "computing hash values for all k-mers in a sequence and selecting the m k-mers that obtain the lowest hash values"
- [methods] Linclust command-line parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [discussion] linear runtime complexity in m: "Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [readme] MMseqs2 tool description: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
