# assembly-mag-quality-taxonomy-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **clean_reads** — remove adapters and low-quality tails and summarize read quality  →  `metagenomic-read-quality-control--333f0c268bb84080`, `short-read-quality-trimming--534731b241401dae`, `adapter-trimming-quality-control`, `fastqc-quality-reporting`
2. **assemble_contigs** — assemble short reads per sample and compare contiguity metrics  →  `metagenome-de-novo-assembly`, `assembled-contig-fasta-handling`, `assembly-contiguity-quality-assessment`, `paired-end-read-assembly--a5035386db5a753c`, `assembled-contig-length-and-count-qc`
3. **bin_contigs** — group sufficiently long contigs into per-sample MAG FASTA bins  →  `metabat2-contig-binning`, `metagenomic-contig-binning--7530fd71afe1cdc5`, `metagenomic-contig-binning--aa6be0a34a35d65a`, `metagenomic-contig-depth-estimation`
4. **assess_mag_quality** — estimate marker-based completeness and contamination and split classifier routes  →  `metagenome-binning-quality-assessment`, `metagenome-assembled-genome-quality-assessment--b4335ce8bdbe4c98`, `marker-gene-completeness-contamination-estimation`, `per-bin-summary-table-generation`
5. **classify_mags** — classify passing bins with GTDB-Tk and remaining bins with CAT/BAT  →  `gtdbtk-genome-taxonomy-assignment`, `mag-taxonomic-classification--8bedab30d38f7e0c`, `catbat-bin-taxonomic-classification`, `mag-taxonomic-classification--6714e56ce1419c81`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
