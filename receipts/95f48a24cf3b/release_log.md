# Release log: metagenomics v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:23:02Z | 1c19d4ed5497840adf07bdf57b6e8e073c4b31ef | `{"candidate_tree": "sha256:0d5ce1a9c4c1800ddc993d7c9d6e7abe86c7724176b7efcba38fa20b0a62484f", "gate_report": "e0413c9ea94bc3870204a60c7e9cdf58a5da7f2ee37a72f263c7dd63a1e5f7e9"}` |
| deposit-plan | 2026-11-09T14:23:03Z | 1c19d4ed5497840adf07bdf57b6e8e073c4b31ef | `{"archive_sha256": "987361ea3f42b18d5c751bbd9cc782ab68cf31e7dfddce9a254b9f0d2f7d317f"}` |
| deposit-draft | 2026-11-09T21:05:46Z | 1c19d4ed5497840adf07bdf57b6e8e073c4b31ef | `{"archive_sha256": "987361ea3f42b18d5c751bbd9cc782ab68cf31e7dfddce9a254b9f0d2f7d317f"}` |
| record-doi | 2026-11-10T10:44:52Z | 1c19d4ed5497840adf07bdf57b6e8e073c4b31ef | `{"CITATION.cff": "c5595d09c63ba88489af68de10e8062a1ddeb7fcc930fa170ea106c2841f06e2", "README.md": "67ad9d9c70093479defc77c64a62b70d525e73a3a6ce09029dbb9b6c03d7e56c", "RELEASE.yaml": "882e98b056446fe978fde97ad56a8410d96372c3c6fe3d2ff31de2f8b362e73e", "marketplace.json": "21fc2244bdc5e256a8bd12bce8c9666c0e5c2bfdc9f0341532ddc55cc78fd880", "plugin.json": "c6465b1f40f33996e58b97ac34d6c439927bea15bfae74bc6fe568b6ca8a584c"}` |
| receipt | 2026-11-10T10:46:35Z | 1c19d4ed5497840adf07bdf57b6e8e073c4b31ef | `{"receipt": "f5b729929e561e72e4071e07feda6af3183b6477158cf94b1cc27e81afaf8dbe"}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093087; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
