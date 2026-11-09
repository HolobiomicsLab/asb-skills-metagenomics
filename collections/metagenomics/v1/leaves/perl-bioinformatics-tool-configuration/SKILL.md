---
name: perl-bioinformatics-tool-configuration
description: 'Use when deploying CopyRighter from source in a Perl environment: setting
  up Makefile.PL-driven installation, resolving CPAN dependencies (Bio::Community,
  Getopt::Euclid >= 0.4.3, Method::Signatures, Scalar::Util), or handling non-administrator
  installation via local::lib.'
when_to_use_negative:
- CopyRighter is already installed and COPYRIGHTER_DB is set — proceed directly to
  running the GCN correction rather than reinstalling
- The task is the downstream GCN correction itself (converting an OTU table, running
  correction, computing Hellinger/euclidean distances) — this skill only covers installation
  and configuration
- Your pipeline does not involve CopyRighter or other Makefile.PL-based Perl bioinformatics
  tools — a generic dependency manager is more appropriate
tools:
- name: CopyRighter
  role: Target Perl tool being installed and configured from source; correct installation
    is the prerequisite for lineage-specific GCN correction of 16S rRNA amplicon profiles
  repo: github.com/fangly/AmpliCopyrighter
- name: Perl >= 5.10.0
  role: Runtime language required by CopyRighter; version floor verified before installation
- name: GNU make
  role: Build tool used with 'perl Makefile.PL', 'make', and 'make install' to compile
    and install CopyRighter
- name: Bio::Community
  role: CPAN dependency pulled in by Makefile.PL; provides community-file handling
    utilities (e.g., bc_use_repr_id for replacing OTU IDs by Greengenes IDs)
  repo: github.com/fangly/AmpliCopyrighter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2049-2618-2-11
    title: 'CopyRighter: a rapid tool for improving the accuracy of microbial community
      profiles through lineage-specific gene copy number correction'
  source_content_hash: sha256:93d41116ec1ac6aaaabb9eecb0a726e0ab47dd7d4bb19f6309237765a80f5dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bio::Community
  - CopyRighter
  - GNU make
  - Perl
  - Perl >= 5.10.0
  - make
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# perl-bioinformatics-tool-configuration

## Summary

Installs and configures the Perl-based CopyRighter (fangly/AmpliCopyrighter, version 0.46) from source using perl Makefile.PL, make, and make install, pulling in CPAN dependencies so that 16S rRNA gene copy number (GCN) correction can be run on microbial community profiles.

## When to use

Use when deploying CopyRighter from source in a Perl environment: setting up Makefile.PL-driven installation, resolving CPAN dependencies (Bio::Community, Getopt::Euclid >= 0.4.3, Method::Signatures, Scalar::Util), or handling non-administrator installation via local::lib.

## When NOT to use

- CopyRighter is already installed and COPYRIGHTER_DB is set — proceed directly to running the GCN correction rather than reinstalling
- The task is the downstream GCN correction itself (converting an OTU table, running correction, computing Hellinger/euclidean distances) — this skill only covers installation and configuration
- Your pipeline does not involve CopyRighter or other Makefile.PL-based Perl bioinformatics tools — a generic dependency manager is more appropriate

## Inputs

- CopyRighter source distribution from fangly/AmpliCopyrighter (includes Makefile.PL and distributed trait-value data files)
- Perl >= 5.10.0 runtime and a make implementation (GNU make recommended)
- Administrator privileges, or a local::lib setup when admin rights are unavailable
- CPAN access for dependency modules: Bio::Community, Getopt::Euclid (>= 0.4.3), Method::Signatures, Scalar::Util

## Outputs

- Working 'copyrighter' command-line executable (version 0.46) accepting -i/-o community files in biom, QIIME, GAAS, Unifrac, or generic tabular format
- Configured COPYRIGHTER_DB environment variable pointing to the 16S rRNA gene copy number and genome length estimate files
- Installed CPAN dependencies (Bio::Community, Getopt::Euclid, Method::Signatures, Scalar::Util)

## How to apply

Verify Perl >= 5.10.0 and make (e.g., GNU make) are present, then run 'perl Makefile.PL' followed by 'make'; install globally with administrator privileges via 'make install', or follow the local::lib instructions first if you lack admin rights. Makefile.PL declares the additional CPAN dependencies (Bio::Community, Getopt::Euclid >= 0.4.3, Method::Signatures, Scalar::Util) that must be installed for the preprocessing utilities and the copyrighter script itself. After installation, confirm the tool responds with its synopsis (e.g., 'copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic') and that the COPYRIGHTER_DB environment variable points to the distributed 16S rRNA gene copy number / genome length data files (IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy). Rationale: CopyRighter is a Perl open-source tool, so correct dependency resolution is a prerequisite for any downstream GCN or genome-length bias correction.

## Related tools

- **CopyRighter** (Target Perl tool being installed and configured from source; correct installation is the prerequisite for lineage-specific GCN correction of 16S rRNA amplicon profiles) — github.com/fangly/AmpliCopyrighter
- **Perl >= 5.10.0** (Runtime language required by CopyRighter; version floor verified before installation)
- **GNU make** (Build tool used with 'perl Makefile.PL', 'make', and 'make install' to compile and install CopyRighter)
- **Bio::Community** (CPAN dependency pulled in by Makefile.PL; provides community-file handling utilities (e.g., bc_use_repr_id for replacing OTU IDs by Greengenes IDs)) — github.com/fangly/AmpliCopyrighter

## Examples

```
perl Makefile.PL && make && make install
```

## Evaluation signals

- 'perl Makefile.PL' completes and lists/installs all CPAN dependencies (Bio::Community, Getopt::Euclid >= 0.4.3, Method::Signatures, Scalar::Util) without error
- 'make' and 'make install' finish without failures; when admin rights are absent, local::lib was installed first per the methods
- Running 'copyrighter' without arguments or with -h prints the documented synopsis ('copyrighter -i otu_table.qiime -o otu_table_copyrighted.generic') and the README VERSION line reports version 0.46
- COPYRIGHTER_DB resolves to a tab-delimited trait file with two columns (ID or Greengenes taxstring, 16S rRNA count) as described in the README's -d argument
- Test run with -v (verbose) displays trait value assignments, confirming taxa are processed with the intended Greengenes 2012/10 taxonomic strings

## Limitations

- Global 'make install' requires administrator privileges; non-administrators must install local::lib first
- Additional CPAN dependencies for preprocessing utilities must be installed manually per the Makefile.PL file
- Documented version is 0.46; the README notes bugs should be reported at the GitHub issue tracker, and a Galaxy webserver (Galaxy QLD) was offered as an alternative graphical environment
- Distributed trait data files cover 16S rRNA gene copy number and genome length based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy; other taxonomies (e.g., Silva) are not yet covered

## Evidence

- [methods] To install CopyRighter globally on your system, run the following commands: "To install CopyRighter globally on your system, run the following commands"
- [methods] First, follow the instructions to install local::lib: "First, follow the instructions to install local::lib"
- [methods] you will need to install additional CPAN dependencies. See the Makefile.PL file for details: "you will need to install additional CPAN dependencies. See the Makefile.PL file for details"
- [readme] This document refers to copyrighter version 0.46: [evidence span withheld — anchor status: external_doc]
- [readme] Data files of 16S rRNA gene copy number and genome length (based on IMG 4.0 genomes mapped onto the Oct 2012 Greengenes taxonomy) are distributed with CopyRighter.: [evidence span withheld — anchor status: external_doc]
