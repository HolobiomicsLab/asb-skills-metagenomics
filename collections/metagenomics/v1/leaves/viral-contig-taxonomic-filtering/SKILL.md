---
name: viral-contig-taxonomic-filtering
description: Use when improving or finalizing a viral metagenomic assembly and you
  need to drop non-viral contigs based on reference-alignment taxonomy before LCA-based
  classification.
when_to_use_negative:
- Input is raw reads rather than assembled contigs — the filter operates on contigs
  with accumulated per-taxon bits-per-base alignment scores
- You need contigs classified against a database lacking viral superkingdom taxonomy
  labels, since the filter's top-taxon criterion requires superkingdom resolution
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0622
tools:
- name: VirMAP
  role: Hosts Virmap.pl, whose pass-one/two taxonomic scoring implements the viral-superkingdom
    contig filter
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide alignment of contigs to the VirMAP/Genbank reference databases
    feeding the filter
- name: DIAMOND
  role: Translated/protein alignment of contigs used alongside BBMap results in the
    filter's taxonomic scoring
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:96d3529f112854908ced2710c77f2178a961f625e986ecb015f2380be7df3b02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BLAST+
  - DIAMOND
  - MEGAHIT
  - Perl
  - Tadpole
  - VirMAP
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Viral contig taxonomic filtering

## Summary

VirMAP's filter step that removes assembled contigs whose alignment profile does not resolve to the viral superkingdom, ensuring the final dual assembly consists solely of viral sequences before taxonomic classification.

## When to use

Use when improving or finalizing a viral metagenomic assembly and you need to drop non-viral contigs based on reference-alignment taxonomy before LCA-based classification.

## When NOT to use

- Input is raw reads rather than assembled contigs — the filter operates on contigs with accumulated per-taxon bits-per-base alignment scores
- You need contigs classified against a database lacking viral superkingdom taxonomy labels, since the filter's top-taxon criterion requires superkingdom resolution

## Inputs

- Improved contigs/super-scaffolds from the VirMAP improvement (merge-and-rebuild) stage
- VirMAP reference databases: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd (built from Genbank gbvrl/gbphg divisions)

## Outputs

- Filtered set of viral-only contigs retained for pass-two LCA classification
- Per-contig taxonomic assignments, scores, and LCA ranks in taxonomic_assignments.tsv

## How to apply

Align the improved contigs to VirMAP's reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) using BBMap for nucleotide alignments and DIAMOND for translated/protein alignments. Run pass one of Virmap.pl, accumulating the maximum bit-score-per-base for every taxon aligned to each contig. Apply the two-part filter: remove a contig if its top scoring pass-one taxon is not in the superkingdom Virus, or if more than 50% of its aligning bases do not resolve to the superkingdom Virus. Only retained contigs proceed to pass-two weighting and the rank-wise LCA algorithm with dynamically computed majority thresholds. The rationale is to guarantee the dual assembly is virus-only before downstream classification.

## Related tools

- **VirMAP** (Hosts Virmap.pl, whose pass-one/two taxonomic scoring implements the viral-superkingdom contig filter) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide alignment of contigs to the VirMAP/Genbank reference databases feeding the filter)
- **DIAMOND** (Translated/protein alignment of contigs used alongside BBMap results in the filter's taxonomic scoring) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Every unfiltered (retained) contig has a top pass-one taxon in the superkingdom Virus
- For each retained contig, more than 50% of its aligning bases resolve to the superkingdom Virus
- Each retained contig has a valid per-contig row in taxonomic_assignments.tsv with a top taxon, score, and LCA rank
- The VirmapRun output directory is populated with taxonomic classifications and abundance results, with run diagnostics captured in VirmapRun.err without fatal errors

## Limitations

- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the paper text; sensitivity/specificity tuning parameters are only in Supplementary Data 14, which may not be present in this document
- The filter depends on the comprehensiveness and version of the Genbank database; no database snapshot or download date is specified
- Divergent or low-coverage viruses with no aligning reference may fail to produce a viral top taxon and be incorrectly filtered

## Evidence

- [discussion] A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus.: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [discussion] using BBMap19 and DIAMOND20, respectively: "using BBMap19 and DIAMOND20, respectively"
- [discussion] Contigs that map outside the viral superkingdom are filtered.: "Contigs that map outside the viral superkingdom are filtered."
- [methods] Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
