---
name: minpath-pathway-reconstruction
description: Use when you have a KO (KEGG Orthology) abundance table from 16S-based
  functional prediction and need MetaCyc pathway presence/abundance inferred by minimum-parsimony
  reconstruction.
when_to_use_negative:
- Input is already a pathway abundance table or another functional profile (e.g. pre-computed
  from Piphillin, PanFP, Tax4Fun, or Tax4Fun2) rather than KO-level abundances requiring
  parsimony-based pathway reconstruction.
- Input sequences have not been processed through the upstream PICRUSt2 functional-prediction
  steps (e.g. DADA2-corrected 16S rDNA sequences and KO prediction), so no KO abundance
  table exists yet.
- You need habitat-specific or pangenome-based functional profiles rather than pathway-level
  reconstruction over KEGG/MetaCyc databases.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: PICRUSt2
  role: Pipeline within which the MinPath pathway-inference step is executed; installation
    instructions are in the PICRUSt2 wiki
  repo: https://github.com/picrust/picrust2
- name: MinPath
  role: Minimum-parsimony pathway-inference component whose source code is bundled
    in the PICRUSt2 repository and distributed under the GNU General Public License
  repo: https://github.com/picrust/picrust2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-0548-6
    title: PICRUSt2 for prediction of metagenome functions
  source_content_hash: sha256:f0948579617703ef12c42684887886e95d97ef5c9b35491426387bf0b7a2562a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust2
  tools:
  - MinPath
  - PICRUSt2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# minpath-pathway-reconstruction

## Summary

Run MinPath minimum-parsimony pathway inference, as bundled within the PICRUSt2 repository (GPL-licensed source), to map KO abundance tables onto MetaCyc/KEGG pathway abundances and write the reconstructed pathway table.

## When to use

Use when you have a KO (KEGG Orthology) abundance table from 16S-based functional prediction and need MetaCyc pathway presence/abundance inferred by minimum-parsimony reconstruction.

## When NOT to use

- Input is already a pathway abundance table or another functional profile (e.g. pre-computed from Piphillin, PanFP, Tax4Fun, or Tax4Fun2) rather than KO-level abundances requiring parsimony-based pathway reconstruction.
- Input sequences have not been processed through the upstream PICRUSt2 functional-prediction steps (e.g. DADA2-corrected 16S rDNA sequences and KO prediction), so no KO abundance table exists yet.
- You need habitat-specific or pangenome-based functional profiles rather than pathway-level reconstruction over KEGG/MetaCyc databases.

## Inputs

- KO abundance table (tab-delimited KEGG Orthology abundances; e.g. the example KO abundance file included with the PICRUSt2 repository)
- Reference pathway databases: KEGG and MetaCyc (with BioCyc pathway/genome collections)

## Outputs

- path_abun.tsv — tab-delimited pathway abundance table with pathway identifiers and abundance columns

## How to apply

Install PICRUSt2 following the official wiki installation page (https://github.com/picrust/picrust2/wiki/Installation), which bundles the MinPath source code distributed under the GNU General Public License. Load the KO abundance table (e.g. the example KO file shipped with the repository) as input to the pathway-inference step. Run MinPath to perform minimum-parsimony reconstruction over the KEGG/MetaCyc pathway databases, mapping KO abundances to pathway abundances. Write the reconstructed pathway abundance table to path_abun.tsv. Consult the PICRUSt2 wiki for how to install and run PICRUSt2 and how the pipeline works before troubleshooting; post problems to the PICRUSt users Google group rather than filing GitHub issues unless a genuine run defect is observed.

## Related tools

- **PICRUSt2** (Pipeline within which the MinPath pathway-inference step is executed; installation instructions are in the PICRUSt2 wiki) — https://github.com/picrust/picrust2
- **MinPath** (Minimum-parsimony pathway-inference component whose source code is bundled in the PICRUSt2 repository and distributed under the GNU General Public License) — https://github.com/picrust/picrust2

## Evaluation signals

- path_abun.tsv exists and is a tab-delimited table with pathway identifiers and abundance columns.
- The reconstructed pathway abundance table is consistent with the reference pathway abundance output produced by the PICRUSt2 tutorial workflow.
- Pathway identifiers correspond to the MetaCyc/KEGG pathway databases used in the reconstruction.
- The PICRUSt2 installation completed according to the official wiki installation instructions before the pathway-inference step was run.

## Limitations

- The README directs users to the PICRUSt wiki/FAQ and the PICRUSt users Google group for questions; GitHub issues are only for genuine run problems, and no changelog was found in the source document.
- MinPath is bundled under the GNU General Public License, which imposes license obligations when the code is reused or redistributed.
- No abstract body text was available in the provided document; only bibliographic metadata and the README/article index describe the component.

## Evidence
