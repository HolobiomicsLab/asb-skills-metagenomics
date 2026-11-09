---
name: minor-allele-variant-detection
description: Use when input is Illumina FASTQ from samples suspected of containing
  low-frequency resistant alleles (mixed infections, heteroresistance, XDR-TB second-line
  drug testing) and Mykrobe predictor's major-clone calls alone miss resistance.
when_to_use_negative:
- The sample is a clean single-clone isolate with no suspected minority resistant
  population — minor-allele mode is unnecessary and the extra threshold may reduce
  specificity
- Mycobacterial samples direct from sputum needing a general mixture model — the current
  implementation models only major/minor clones
- Drugs whose known resistance mutations are poorly predictive (e.g. pyrazinamide),
  which Mykrobe completely excludes from prediction
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0798
tools:
- name: Mykrobe predictor
  role: De Bruijn graph-based predictor run with --format ; re-run with minor-allele
    detection enabled at a uniform 10% allele-frequency threshold to call low-frequency
    resistance determinants
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: mykrobe-parser
  role: R script that parses the M. tuberculosis specific results of Mykrobe JSON
    output, separating mutations into individual genetic regions and removing samples
    where non-tuberculous mycobacteria are detected
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:762b3767d94ad6ddf1a2e87fbbfacd2c87418238c42a564b3f768cfec1dfb2e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  - mykrobe-parser
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# minor-allele-variant-detection

## Summary

Enables Mykrobe predictor to call resistance-determining variants present only as low-frequency (minor) alleles in mixed populations, using a single 10% allele-frequency threshold to boost sensitivity for second-line drug resistance (e.g. capreomycin, amikacin, ofloxacin) in M. tuberculosis and to detect minority resistant populations in simulated S. aureus/CoNS mixtures.

## When to use

Use when input is Illumina FASTQ from samples suspected of containing low-frequency resistant alleles (mixed infections, heteroresistance, XDR-TB second-line drug testing) and Mykrobe predictor's major-clone calls alone miss resistance.

## When NOT to use

- The sample is a clean single-clone isolate with no suspected minority resistant population — minor-allele mode is unnecessary and the extra threshold may reduce specificity
- Mycobacterial samples direct from sputum needing a general mixture model — the current implementation models only major/minor clones
- Drugs whose known resistance mutations are poorly predictive (e.g. pyrazinamide), which Mykrobe completely excludes from prediction

## Inputs

- Illumina sequencing reads (FASTQ, ideally ≥100x mean depth) from M. tuberculosis or S. aureus samples, e.g. MTBC_B second-line-drug validation reads from ENA/SRA
- Phenotypic DST labels (Supplementary Data) for capreomycin, amikacin and ofloxacin to score sensitivity
- Mixture-proportion design (e.g. S. aureus St_B1 / CoNS St_B2 pooled FASTQ at 0.5-20% CoNS frequencies) when validating detection power in silico

## Outputs

- Mykrobe predictor JSON result files per sample with major/minor clone resistance calls
- Per-drug sensitivity under baseline vs minor-allele-enabled settings (e.g. minor_allele_sensitivity_deltas.csv: capreomycin +9.1 pp, amikacin +11.8 pp, ofloxacin +23.0 pp)
- Detection power vs mixture-frequency table (e.g. detection_power_summary.csv)

## How to apply

Run Mykrobe predictor on the sample's sequencing reads twice: once with minor-allele calls disabled to obtain the baseline resistance calls, and once with minor-allele detection enabled using the single 10% allele-frequency threshold applied uniformly to all drugs (per-drug thresholds could not be estimated for M. tuberculosis because phenotyping data was limited). Emit results as JSON ('--format ') so calls and allele frequencies can be parsed programmatically (e.g. with mykrobe-parser or custom JSON parsing of phylo_group/species/lineage and resistance-determinant calls). Compare calls against phenotypic DST labels, computing sensitivity as true positives over all phenotypically resistant samples under both settings; in the MTBC_B validation set this raised sensitivity from 74.5 to 83.6% for capreomycin, 78.0 to 89.8% for amikacin, and 38.5 to 61.5% for ofloxacin (N=13). For in silico S. aureus/CoNS mixtures, detection power reaches 94% for tetracycline and 100% for erythromycin once the minor population reaches 2% frequency.

## Related tools

- **Mykrobe predictor** (De Bruijn graph-based predictor run with --format ; re-run with minor-allele detection enabled at a uniform 10% allele-frequency threshold to call low-frequency resistance determinants) — https://github.com/Mykrobe-tools/mykrobe
- **mykrobe-parser** (R script that parses the M. tuberculosis specific results of Mykrobe JSON output, separating mutations into individual genetic regions and removing samples where non-tuberculous mycobacteria are detected) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- On MTBC_B, enabling minor-allele detection reproduces sensitivity deltas of +9.1 pp (capreomycin 74.5→83.6%), +11.8 pp (amikacin 78.0→89.8%) and +23.0 pp (ofloxacin 38.5→61.5%) at N=13
- In S. aureus/CoNS in silico mixtures, detection power for tetracycline and erythromycin plasmid determinants reaches 94% and 100% at 2% minor population frequency
- All runs emit valid JSON ('--format '); Mykrobe Parser fails on non-JSON Mykrobe files
- Minor-allele mode uses the single 10% allele-frequency threshold for all drugs — verify no per-drug thresholds were silently applied
- QC: samples in which Mykrobe detects non-tuberculous mycobacteria are removed before scoring

## Limitations

- M. tuberculosis data were insufficient to estimate per-drug thresholds, so a single 10% threshold was applied to all drugs despite imperfect phenotyping
- Only 18%/44% of training/validation samples were phenotyped for at least one quinolone or injectable, so the second-line minor-allele finding needs replication with larger data sets
- The implementation models only major/minor clones; a more general mixture model is needed for direct-from-sputum M. tuberculosis analysis
- It remains an open question whether low-frequency alleles affect patient outcome, since they generally fail to cause in vitro resistance

## Evidence

- [abstract] We give evidence that minor alleles improve detection of extremely drug-resistant strains: "We give evidence that minor alleles improve detection of extremely drug-resistant strains, and demonstrate feasibility of the use of emerging single-molecule nanopore"
- [discussion] For M. tuberculosis, we did not have enough data to estimate per-drug thresholds: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [discussion] a more general model of mixtures, rather than simply major/minor clones: "we suspect that incorporating a more general model of mixtures, rather than simply major/minor clones, will be of value when analysing M. tuberculosis samples direct from sputum"
- [discussion] We then made in silico mixtures of these pairs with the CoNS samples at 30 frequencies ranging from 0.5 to 20%: "We then made in silico mixtures of these pairs with the CoNS samples at 30 frequencies ranging from 0.5 to 20%"
- [readme] --format  stores results a . file: "--format 
        * Stores results a . file instead of a text file. Mykrobe Parser only works with  files."
