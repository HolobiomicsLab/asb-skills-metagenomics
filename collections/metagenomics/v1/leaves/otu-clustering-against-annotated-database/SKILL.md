---
name: otu-clustering-against-annotated-database
description: Use when you have chimera-filtered paired-end 16S reads (never merged)
  plus an annotated 16S FASTA database and need annotated OTUs at species-level resolution
  without merging reads.
when_to_use_negative:
- Input is already an OTU/feature table rather than raw paired-end FASTQ reads plus
  a FASTA reference database
- The workflow requires merged (assembled) reads or denoised amplicon sequence variants
  (e.g. DADA2/deblur) rather than cluster-based OTUs from unmerged pairs
- Reads are so short that 97% clustering cannot resolve species-level groups without
  database fragments matching the read length
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT
  role: cd-hit-est performs the 99% clusterizations; CD-HIT utilities perform the
    final 97% clustering of reads with annotated database fragments
  repo: github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: defines the separate-clustering scheme (including database reformatting and
    fragment cutting) that the 97% annotated-OTU clustering follows
  repo: github.com/ydkondratenko/cdsnake
- name: Snakemake
  role: workflow engine that chains the clustering and annotation steps into a one-command,
    reproducible run
  repo: github.com/ydkondratenko/cdsnake
- name: Trimmomatic
  role: upstream cropping (p=200 bp R1, q=180 bp R2 defaults) and MINLEN 100 quality
    filtering that produce the reads entering the clustering
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:3d3e2e766118a74b21f965abd2786937e8074f2e3aea373d30805203fe28c315
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - CD-HIT-OTU-Miseq
  - Snakemake
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# otu-clustering-against-annotated-database

## Summary

Annotate OTUs by clustering quality-filtered paired-end 16S reads together with fragments of an annotated 16S reference database (Greengenes or SILVA) at 97% similarity, so each OTU inherits a taxonomic annotation from the database sequence it groups with.

## When to use

Use when you have chimera-filtered paired-end 16S reads (never merged) plus an annotated 16S FASTA database and need annotated OTUs at species-level resolution without merging reads.

## When NOT to use

- Input is already an OTU/feature table rather than raw paired-end FASTQ reads plus a FASTA reference database
- The workflow requires merged (assembled) reads or denoised amplicon sequence variants (e.g. DADA2/deblur) rather than cluster-based OTUs from unmerged pairs
- Reads are so short that 97% clustering cannot resolve species-level groups without database fragments matching the read length

## Inputs

- Chimera-filtered paired-end 16S FASTQ reads (R1/R2, cropped/quality-filtered by Trimmomatic, never merged)
- Annotated 16S reference database in FASTA form (e.g. Greengenes or SILVA), reformatted per CD-HIT-OTU-Miseq installation step 4
- Edited Snakefile placed in the input folder (with DATABASE, CDHIT, TRIMMOMATIC, p, q, THREADS, MINLEN parameters set)

## Outputs

- Annotated OTU clusters (read groups at 97% similarity carrying database annotations)
- pooled/OTU.txt — OTU count table for all pooled samples

## How to apply

After the 99% cd-hit-est clusterizations and chimera removal of read pairs whose mates vote for non-matching large clusters, cut the annotated 16S database sequences into fragments matching the read lengths. Cluster the remaining (non-chimeric) reads together with these database fragments at 97% similarity using CD-HIT utilities; at this cutoff, read groups correspond to species or close taxonomic levels, and clusters containing a database fragment inherit its annotation. Launch the workflow with a single 'snakemake' command from the input folder; clustering results for pooled samples are written to the 'pooled' folder, and the OTU count table for all samples is saved as OTU.txt inside that folder. Use a database reformatted per CD-HIT-OTU-Miseq installation step 4, which places sequences with more specific annotations at the beginning of the file.

## Related tools

- **CD-HIT** (cd-hit-est performs the 99% clusterizations; CD-HIT utilities perform the final 97% clustering of reads with annotated database fragments) — github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (defines the separate-clustering scheme (including database reformatting and fragment cutting) that the 97% annotated-OTU clustering follows) — github.com/ydkondratenko/cdsnake
- **Snakemake** (workflow engine that chains the clustering and annotation steps into a one-command, reproducible run) — github.com/ydkondratenko/cdsnake
- **Trimmomatic** (upstream cropping (p=200 bp R1, q=180 bp R2 defaults) and MINLEN 100 quality filtering that produce the reads entering the clustering) — github.com/ydkondratenko/cdsnake

## Examples

```
snakemake
```

## Evaluation signals

- pooled/OTU.txt exists inside the input folder after the run and contains per-sample OTU counts
- On the Balanced mock community (57 bacteria and archaea), reported OTU counts are comparable to the number of strains in the mock community
- OTU clusters were produced at 97% similarity with database fragments included, so OTUs carry taxonomic annotations at species or close taxonomic levels
- Non-chimeric reads from both mates of each pair were carried into the final clustering (no merging step was introduced)
- The run completed with the single 'snakemake' command, yielding reproducible results

## Limitations

- Reads must be unzipped FASTQ with '_R1_'/'_R2_' (or '_1'/'_2') filename patterns; the database must be reformatted as in CD-HIT-OTU-Miseq installation step 4
- The p and q crop lengths must match the reads' good-quality regions (quality ~25+, no sudden drops); poor 3' quality in the overlap region is precisely why merging is avoided
- CDSnake is generally slower than DADA2/deblur pipelines, though it preserves more information from raw data with higher total counts
- Software versions and mock dataset accessions are not stated, complicating exact reproduction of benchmarks

## Evidence

- [methods] 97% species-level clustering: "16S reads clustered with 97% similarity result in read groups corresponding to species or close taxonomic levels."
- [methods] 97% clustering with database fragments: "Remaining reads and fragments from annotated database are then clustered at 97% similarity to yields annotated clusters commonly named Operational Taxonomic Units, or OTU."
- [readme] OTU.txt output location: "Results of clustering for pooled samples will be stored in folder “pooled” in input folder. OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
- [readme] Database reformatting requirement: "database should be formatted as described in 4 step of Installation of CD-HIT-OTU-Miseq (this reformatting places sequences with more specific annotations are at the beginning of the file)"
- [intro] Separate clustering avoids merging: "entirely avoiding reads merging due to separate clustering of paired reads"
