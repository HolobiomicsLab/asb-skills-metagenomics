---
name: protein-contact-precision-evaluation-at-top-l-thresholds
description: Use when benchmarking a contact predictor on CASP target domains and
  you need precision at multiple top-k cutoffs (L/10 to L) and separation ranges to
  compare against PSICOV, CCMpred, DeepCov, or ResPRE.
when_to_use_negative:
- Evaluating 3D model quality rather than contact precision — use TMscore on CONFOLD-constructed
  structures instead
- Comparing against RaptorX-Contact on the complete test set — server errors prevent
  prediction retrieval for 3 of 105 CASP11 and 1 of 55 CASP12 domains
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0082
tools:
- name: DeepECA
  role: Provides the predicted contact probabilities being evaluated, via pipeline.py
    storing results in ./contact_pred
  repo: github.com/tomiilab/DeepECA
- name: PSICOV
  role: Baseline ECA method whose contact predictions provide comparison precision
    values
- name: CCMpred
  role: Baseline ECA method whose contact predictions provide comparison precision
    values
- name: DeepCov
  role: DNN comparator that also uses covariance matrices as DNN input
- name: ResPRE
  role: Comparator using a precision matrix instead of a covariance matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:2ecce6dd02bced0039213b7ec4c60fe6784a6dc5d293ec149e5bf7481599be18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCMpred
  - DeepCov
  - DeepECA
  - PSICOV
  - ResPRE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# protein-contact-precision-evaluation-at-top-l-thresholds

## Summary

Evaluates protein residue-contact predictions by computing precision TP/(TP+FP) at the top L/10, L/5, L/2, and L predicted pairs for short-, medium-, and long-range sequence separations, enabling direct comparison of the Baseline Model against ECA methods on CASP11 (105 domains) and CASP12 (55 domains).

## When to use

Use when benchmarking a contact predictor on CASP target domains and you need precision at multiple top-k cutoffs (L/10 to L) and separation ranges to compare against PSICOV, CCMpred, DeepCov, or ResPRE.

## When NOT to use

- Evaluating 3D model quality rather than contact precision — use TMscore on CONFOLD-constructed structures instead
- Comparing against RaptorX-Contact on the complete test set — server errors prevent prediction retrieval for 3 of 105 CASP11 and 1 of 55 CASP12 domains

## Inputs

- Predicted contact probability lists per domain (from ./contact_pred, produced by pipeline.py)
- Ground-truth contact maps from CASP11 (105 domains) and CASP12 (55 domains) domain structures
- Target domain sequences/lengths used to define the top-L cutoffs

## Outputs

- Precision TP/(TP+FP) values at top L/10, L/5, L/2, and L predicted pairs
- Per-separation-range precision for short, medium, and long sequence separations
- Comparison table (Tables 11 and 12 format) against PSICOV, CCMpred, DeepCov, and ResPRE

## How to apply

Generate or obtain predicted contact probability lists for each target domain (contacts defined as Cβ atom pairs within 8 Å; Cα for glycine). Rank predicted pairs by probability, take the top L/10, L/5, L/2, and L pairs (L = sequence length), and compute precision as TP/(TP+FP) for short-, medium-, and long-separation ranges separately. Run this over the full CASP11 (105 domains) and CASP12 (55 domains) test sets using the DeepECA pipeline (pipeline.py, with results in ./contact_pred). Correct application is judged by whether the Baseline Model's precision values reproduce Tables 11 and 12 and outperform PSICOV, CCMpred, DeepCov, and ResPRE at every distance and prediction count.

## Related tools

- **DeepECA** (Provides the predicted contact probabilities being evaluated, via pipeline.py storing results in ./contact_pred) — github.com/tomiilab/DeepECA
- **PSICOV** (Baseline ECA method whose contact predictions provide comparison precision values)
- **CCMpred** (Baseline ECA method whose contact predictions provide comparison precision values)
- **DeepCov** (DNN comparator that also uses covariance matrices as DNN input)
- **ResPRE** (Comparator using a precision matrix instead of a covariance matrix)

## Examples

```
python pipeline.py
```

## Evaluation signals

- Computed Baseline precision values match the Baseline rows of Tables 11 and 12 on CASP11 and CASP12
- Baseline Model precision exceeds PSICOV, CCMpred, DeepCov, and ResPRE at every distance and prediction count (top L/10, L/5, L/2, L)
- Ground-truth contacts use the Cβ < 8 Å definition (Cα for glycine) consistently across evaluation
- Precision is reported separately for short, medium, and long sequence-separation ranges

## Limitations

- RaptorX-Contact comparison results are missing for 3 of 105 CASP11 domains and 1 of 55 CASP12 domains due to server errors
- RaptorX-Contact predictions rely on a web server (no local execution file) and its MSAs differ from those used for other methods
- Residue pairs with missing Cβ coordinates are excluded (marked NaN), affecting counts for some targets

## Evidence

- [discussion] Contact definition: Cβ atoms within 8 Å (Cα for glycine): "we defined a contact if the distance of Cβ atoms of the residue pair was less than 8 Å"
- [methods] Precision at top L thresholds on CASP11 and CASP12: [evidence span withheld — anchor status: not_found]
- [results] Baseline outperforms existing methods: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline output directory: "The results will be stored in ./contact_pred and ./ss_pred directory."
