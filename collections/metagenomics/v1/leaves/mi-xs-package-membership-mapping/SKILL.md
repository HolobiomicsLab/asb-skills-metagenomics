---
name: mi-xs-package-membership-mapping
description: Use when a MIxS extension proposal designates terms for specific environmental
  packages and you must confirm the term-to-package mapping against the authoritative
  schema file mixs.yaml.
when_to_use_negative:
- The analysis goal is contributing or editing new terms to MIxS rather than verifying
  existing schema placement (use the repo's contributing/ticket workflow instead).
- The schema of interest is not represented in LinkML YAML — i.e., pre-v6.0 MIxS versions
  without a mixs.yaml file.
- You need ontology semantics of environment terms (ENVO classes) rather than slot-to-package
  membership in the MIxS schema.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: LinkML
  role: Framework used to load and enumerate class/slot definitions from the YAML-encoded
    MIxS schema (mixs.yaml)
  repo: https://github.com/GenomicsStandardsConsortium/mixs
- name: GenomicsStandardsConsortium/mixs
  role: Source repository providing the authoritative mixs.yaml schema file and contributing
    documentation
  repo: https://github.com/GenomicsStandardsConsortium/mixs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43705-022-00092-w
    title: 'MIxS-SA: a MIxS extension defining the minimum information standard for
      sequence data from symbiont-associated micro-organisms'
  source_content_hash: sha256:542a9eb9a761add455526fb8f765ca7e505743ce2dede688da2095737e61813d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GenomicsStandardsConsortium/mixs
  tools:
  - GenomicsStandardsConsortium/mixs
  - LinkML
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mi(xs)-package-membership-mapping

## Summary

Verifies and maps the placement of newly introduced MIxS-SA terms (observed host symbionts, biotic relationship, relationship to other samples) into the designated MIxS environmental and core packages by parsing the published LinkML mixs.yaml schema.

## When to use

Use when a MIxS extension proposal designates terms for specific environmental packages and you must confirm the term-to-package mapping against the authoritative schema file mixs.yaml.

## When NOT to use

- The analysis goal is contributing or editing new terms to MIxS rather than verifying existing schema placement (use the repo's contributing/ticket workflow instead).
- The schema of interest is not represented in LinkML YAML — i.e., pre-v6.0 MIxS versions without a mixs.yaml file.
- You need ontology semantics of environment terms (ENVO classes) rather than slot-to-package membership in the MIxS schema.

## Inputs

- src/mixs/schema/mixs.yaml — LinkML YAML schema file from GenomicsStandardsConsortium/mixs
- MIxS-SA proposal term definitions: MIXS:0001309 (observed host symbionts), MIXS:0000028 (biotic relationship), 'relationship to other samples'

## Outputs

- mixs_term_package_map. — dictionary mapping each term identifier to the packages where it was found, including verification status flags for the conditional (required: false) and TBD checks

## How to apply

Obtain the published schema file src/mixs/schema/mixs.yaml from the GenomicsStandardsConsortium/mixs repository, which since MIxS v6.1 is maintained in LinkML (YAML). Load it with a YAML/LinkML schema loader and enumerate the class (slot) definitions and package (classes) groupings. Programmatically check that slot MIXS:0001309 ('observed host symbionts') appears in exactly the host-associated, human-associated, plant-associated, human-vaginal, human-skin, human-oral and human-gut package class definitions; that MIXS:0000028 ('biotic relationship') carries a conditional (required: false / if-required) status in the core package; and that the slot 'relationship to other samples' has its description/range marked TBD in the core package, as proposed in the MIxS-SA paper. Build a dictionary mapping each term identifier to its list of packages plus verification status flags for the conditional and TBD checks, then serialize the dictionary to mixs_term_package_map.. Validation consists of confirming one entry per term, exact match of MIXS:0001309 to the seven expected packages, and both the conditional and TBD flags recorded as verified.

## Related tools

- **LinkML** (Framework used to load and enumerate class/slot definitions from the YAML-encoded MIxS schema (mixs.yaml)) — https://github.com/GenomicsStandardsConsortium/mixs
- **GenomicsStandardsConsortium/mixs** (Source repository providing the authoritative mixs.yaml schema file and contributing documentation) — https://github.com/GenomicsStandardsConsortium/mixs

## Examples

```
python -c "import yaml, ; s=yaml.safe_load(open('src/mixs/schema/mixs.yaml')); cls=s['classes']; pkgs=['host-associated','human-associated','plant-associated','human-vaginal','human-skin','human-oral','human-gut']; hits=[p for p in pkgs if any('0001309' in str(a) for a in cls[p].get('attributes',{}).values())]; .dump({'MIXS:0001309':hits}, open('mixs_term_package_map.','w'))"
```

## Evaluation signals

- The JSON record contains an entry for each of the three terms (MIXS:0001309, MIXS:0000028, 'relationship to other samples').
- The package list for MIXS:0001309 matches exactly the seven expected packages: host-associated, human-associated, plant-associated, human-vaginal, human-skin, human-oral and human-gut.
- The conditional status of MIXS:0000028 (required: false / if-required in the core package) is recorded as verified.
- The TBD status of 'relationship to other samples' (description/range in the core package) is recorded as verified.

## Limitations

- The schema is maintained in LinkML and regenerated artifacts (project/) are autogenerated; the mapping is valid only against the specific mixs.yaml version checked out.
- The term 'relationship to other samples' is proposed to be added to all environmental packages across the domains of life, so its placement may extend beyond the core package and the TBD status is provisional.
- Terms can be proposed/modified by the community via GitHub tickets, so package membership may change between schema releases.

## Evidence

- [readme] observed host symbionts designated for seven packages: [evidence span withheld — anchor status: not_found]
- [readme] LinkML transition: "Following the release of MIxS v6.0, subsequent releases (e.g. MIxS 6.1) are represented in and maintained using the [LinkML](https://linkml.io/linkml/) framework. LinkML uses YAML to define schemas."
- [intro] proposal to add relationship term: "We also propose adding the term ‘relationship to other packages’ to all environmental packages across the domains of life, to allow for integrated analysis of symbiont and host microbiota"
- [intro] community modification workflow: "Changes to the package (addition of terms, modification etc.) can be proposed by the community by creating a ticket on the MIxS GitHub page"
