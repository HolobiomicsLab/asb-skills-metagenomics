# read-taxonomic-functional-profiling-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **trim_and_qc** — remove adapters and low-quality sequence and summarize read quality  →  `metagenomic-read-quality-control--333f0c268bb84080`, `fastqc-quality-reporting`, `multiqc-report-aggregation`, `short-read-quality-trimming--534731b241401dae`, `illumina-adapter-trimming-bbduk`
2. **remove_host** — discard reads aligning above threshold to the configured host reference  →  `host-read-filtering`, `short-read-alignment-to-host-reference`, `fastq-read-count-summary-statistics`, `host-read-alignment-against-human-genome`
3. **build_taxonomic_profile** — classify host-filtered reads into sample-level Kraken2 taxonomic profiles  →  `metagenomic-taxonomic-profiling`, `taxonomic-profile-generation`, `metagenomic-taxonomy-profiling-of-rna-seq-reads`, `kraken2-taxonomy-charting`
4. **quantify_genes** — map reads to UniRef90 and derive fractional gene-length-normalized abundances  →  `metagenomic-read-gene-annotation`, `shotgun-metagenome-gene-abundance-estimation`, `metagenomic-hit-summarization`, `alignment-hits-tabular-output-processing`
5. **aggregate_orthologs** — fractionally aggregate gene abundances into conserved KO profiles  →  `kegg-orthology-annotation`, `metagenomic-gene-to-ortholog-mapping`, `many-to-many-mapping-table-parsing`, `fractional-ortholog-abundance-counting`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
