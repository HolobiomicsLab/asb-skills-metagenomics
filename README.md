<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Metagenomics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for computational metagenomics: shotgun community sequencing, assembly, binning, and taxonomic and functional profiling.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-2%2C754-00284b" alt="2,754 skills">
  <img src="https://img.shields.io/badge/tools-717-00284b" alt="717 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-117-00284b" alt="117 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the metagenomics skill collection of AgenticScienceBuilder (ASB): 2,754 skills over 717 software tools and 2 composite workflows, distilled from 117 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 2,235 of the 2,754 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/metagenomics/v1` | ASB Metagenomics Skill Collection v1 | 2754 | 717 | 2026-11-14 | [10.5281/zenodo.24224628](https://doi.org/10.5281/zenodo.24224628) |

## Status

Collection `v1`, built from corpus cut `95f48a24cf3b`, is released on `main` at tag `metagenomics-v1.1.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24224628](https://doi.org/10.5281/zenodo.24224628).

| Item | State |
|---|---|
| Release gate | `receipts/95f48a24cf3b/gate_report.json` reports `pass`: 14 of 16 checks pass and 2 are not applicable (`cut_manifest`, and `peer_review_claim` because the corpus holds no preprint or deposit DOI). [`receipts/95f48a24cf3b/VERIFY.md`](receipts/95f48a24cf3b/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/metagenomics/v1` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/metagenomics/v1` |
| IRI resolution (2026-11-14) | Registered at w3id.org. It redirects (HTTP 302) to the collection landing page in the asb-skill-collections registry (HTTP 200), which names this repository and the Zenodo record. |
| DOI | [10.5281/zenodo.24224628](https://doi.org/10.5281/zenodo.24224628) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093086](https://doi.org/10.5281/zenodo.24093086) (all versions) |

## What is inside

| Item | Count | Path under `collections/metagenomics/v1/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 2,754 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 717 | `tools/` |
| Source papers | 117 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/95f48a24cf3b/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `assembly-mag-quality-taxonomy` | paired-end short-read metagenomes must be trimmed, assembled per sample, binned into MAGs, split by BUSCO quality, and assigned taxonomy with GTDB-Tk or the CAT/BAT fallback route | 5 |
| `read-taxonomic-functional-profiling` | paired-end shotgun metagenomic reads require adapter trimming, host removal, a Kraken2 taxonomic profile, and a gene-length-normalized KEGG ortholog abundance profile from UniRef90 read mapping | 5 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 2,150 |
| `noncommercial` | 439 |
| `restricted` | 0 |
| `unknown` | 165 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-metagenomics
claude plugin install metagenomics@asb-skills-metagenomics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/metagenomics/v1/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-metagenomics/collections/metagenomics/v1/leaves/metagenome-assembly-and-binning <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-metagenomics
python3 collections/metagenomics/v1/bin/search_skills.py --query "metagenome binning" -k 3
python3 collections/metagenomics/v1/bin/search_skills.py --target workflows --query "metagenome assembly and binning" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. metagenome-assembly-and-binning
2. metagenome-binning-quality-assessment
3. viral-metagenome-genome-binning

1. assembly-mag-quality-taxonomy
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/95f48a24cf3b/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `afa4911`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

All 117 included papers are open access according to `corpus.yaml` (`access.is_oa`).

Each leaf names its source paper in `provenance.source_papers` (2,754 of 2,754 leaves). 2,235 of the 2,754 quote it, or its repository README, in their `## Evidence` section. In the other 519, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 518 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 1 the section holds a quote in a form the gate parser does not read.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- `indicium-adapters-metagenomics`: the Indicium domain adapter for the same domain. Repository follows.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24224628](https://doi.org/10.5281/zenodo.24224628); its concept DOI, which covers all versions, is [10.5281/zenodo.24093086](https://doi.org/10.5281/zenodo.24093086).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
