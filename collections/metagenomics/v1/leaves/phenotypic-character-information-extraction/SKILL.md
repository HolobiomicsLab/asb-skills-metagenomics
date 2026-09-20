---
name: phenotypic-character-information-extraction
description: Use when the goal is to semi-automatically extract phenotypic characters
  (e.g., cell shape, Gram stain, NaCl/pH/temperature ranges, antibiotic sensitivity)
  from taxonomic description text into a taxon-by-character matrix for phylogenetic
  or phenomic analysis.
when_to_use_negative:
- Descriptions are morphological/semi-structured botanical texts better handled by
  CharaParser — MicroPIE targets prokaryotic descriptions rich in physiological and
  ecological traits
- The task requires domain-general open extraction without curated term lists — OIE/DeepDive-style
  systems are a different (inadequate per the paper) approach
- You need characters outside the 42 supported phenotypic character types of MicroPIE
  0.1.0
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_0091
tools:
- name: MicroPIE
  role: 'Main NLP pipeline: Preprocessor, SVM Character Predictor, and rule-based
    Character Extractors producing the taxon-by-character matrix'
  repo: github.com/biosemantics/micropie2
- name: MicroPIE 0.1.0
  role: Specific release (Apr 25, 2016) covering 42 phenotypic character types, compiled
    with Maven into micropie-0.1.0.jar
  repo: github.com/biosemantics/micropie2
- name: SVM (Support Vector Machine)
  role: Supervised classifier in the Character Predictor that routes sentences to
    the correct Character Extractor; removing it drops accuracy from 79.0% to 54.6%
- name: Maven
  role: Build system used to transfer the downloaded project into a Maven project
    and compile it into a .jar file
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1396-8
    title: 'Microbial phenomics information extractor (MicroPIE): a natural language
      processing tool for the automated acquisition of prokaryotic phenotypic characters
      from text sources'
  source_content_hash: sha256:3c9820ad111fde9bada98cb92133c1e355ce3700469de8fd1f5b94281c878101
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - MicroPIE
  - MicroPIE 0.1.0
  - SVM (Support Vector Machine)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# phenotypic-character-information-extraction

## Summary

Extract microbial phenotypic characters and character states from natural-language prokaryotic taxonomic descriptions using MicroPIE, which combines an SVM-based Character Predictor with rule-based, term-list-driven extraction to produce a structured taxon-by-character matrix.

## When to use

Use when the goal is to semi-automatically extract phenotypic characters (e.g., cell shape, Gram stain, NaCl/pH/temperature ranges, antibiotic sensitivity) from taxonomic description text into a taxon-by-character matrix for phylogenetic or phenomic analysis.

## When NOT to use

- Descriptions are morphological/semi-structured botanical texts better handled by CharaParser — MicroPIE targets prokaryotic descriptions rich in physiological and ecological traits
- The task requires domain-general open extraction without curated term lists — OIE/DeepDive-style systems are a different (inadequate per the paper) approach
- You need characters outside the 42 supported phenotypic character types of MicroPIE 0.1.0

## Inputs

- Folder of natural-language prokaryotic taxonomic descriptions (e.g., Part_One_111_final, 111 descriptions)
- MicroPIE models folder with SVM-trained character models (models/character_model0.1.0)
- Character Term Lists and extractor configuration files (models/CharacterValueExtractors0.1.0)
- Hand-generated gold standard taxon-by-character matrix for evaluation

## Outputs

- Taxon-by-character matrix written to the -o output folder
- Per-character Relaxed_P / Relaxed_R / Relaxed_F1 scores (e.g., relaxed_scores.csv)
- Performance tier counts: number of characters with Relaxed_F1 > 0.80 and with 0.50–0.80

## How to apply

Transfer the downloaded MicroPIE project into a Maven project (Java sources under 'src', 'models' folder placed in the workspace) and compile it into a .jar file. Run MicroPIE from the command line passing -i the folder of taxonomic descriptions (e.g., Part_One_111_final with 111 descriptions), -o the output folder for the generated taxon-by-character matrix, and -m the models folder containing SVM-trained character models (models/character_model0.1.0) and character Term Lists (models/CharacterValueExtractors0.1.0). Internally, sentences flow from the Preprocessor to the SVM Character Predictor, which identifies which of the 42 designated characters each sentence describes, then rule-based Character Extractors segment noun phrases and match them against per-character Term Lists; when one conjunct of a coordinated noun phrase matches a term, all conjuncts are extracted as values. Keep the SVM component enabled: the ablation showed disabling it drops overall accuracy to 54.6% (3260/5969) versus 79.0% with SVM. Validate the output matrix against a hand-generated gold standard by computing Relaxed precision, recall, and F1 per character and binning scores into tiers (F1 > 0.80, 0.50–0.80).

## Related tools

- **MicroPIE** (Main NLP pipeline: Preprocessor, SVM Character Predictor, and rule-based Character Extractors producing the taxon-by-character matrix) — github.com/biosemantics/micropie2
- **MicroPIE 0.1.0** (Specific release (Apr 25, 2016) covering 42 phenotypic character types, compiled with Maven into micropie-0.1.0.jar) — github.com/biosemantics/micropie2
- **SVM (Support Vector Machine)** (Supervised classifier in the Character Predictor that routes sentences to the correct Character Extractor; removing it drops accuracy from 79.0% to 54.6%)
- **Maven** (Build system used to transfer the downloaded project into a Maven project and compile it into a .jar file)

## Examples

```
Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models
```

## Evaluation signals

- Against the gold standard matrix, expect 21 characters (50%) with Relaxed_F1 > 0.80 and 16 characters (38%) between 0.50 and 0.80, with ~79.0% of character values extracted
- With the SVM Character Predictor enabled, expect higher Relaxed_Precision than the no-SVM configuration on 34 of the 42 characters
- Overall accuracy with SVM should be ~79.0% versus ~54.6% (3260/5969) without it; ~2000 incorrect values indicate the Character Predictor was bypassed
- Output folder should contain a taxon-by-character matrix whose rows/columns correspond to taxa and the designated phenotypic characters
- On the 12 student-compared characters (e.g., antibiotic production, biofilm formation, filterability, host), MicroPIE's relaxed hit score should exceed the student matrix (445 vs 200), significant at p < 0.01 by Student's t-Test

## Limitations

- Nearly half of designated characters had Relaxed_F1 not above 0.80, so extraction quality is character-dependent
- Term-list coverage is insufficient for some characters; coordinated noun-phrase expansion partially compensates but further development, including incorporation of ontologies, is necessary
- The authors acknowledge the system is not yet robust and expect future improvements will make it more robust and effective
- No changelog found in the repository, so tracking fixes across versions (0.1.0 vs 0.2.0) requires reading release notes elsewhere

## Evidence

- [abstract] uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions: "uses a robust supervised classification algorithm (Support Vector Machine) to identify characters from sentences in prokaryotic taxonomic descriptions"
- [abstract] MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores between 0.50 and 0.80: "MicroPIE extracted 21 characters (50%) with a Relaxed F1 score > 0.80 and 16 characters (38%) with Relaxed F1 scores ranging between 0.50 and 0.80."
- [methods] After downloading the project, transfer it into a Maven project. You can compile the project into a .jar file.: "After downloading the project, transfer it into a <a href="https://maven.apache.org/">Maven(https://maven.apache.org/)</a> project. You can compile the project into a .jar file."
- [readme] Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models: "Java –cp micropieproject_lib –jar micropie-0.1.0.jar -i F:/MicroPIE/datasets/Part_One_111_final -o F:/MicroPIE/ext/craft -m F:/MicroPIE/MicroPIEWEB/models"
- [readme] The trained SVM models are in the folder, models\character_model0.1.0. The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0.: "The trained SVM models are in the folder, models\character_model0.1.0.

The Term Lists of characters are in the folder, models\CharacterValueExtractors0.1.0."
- [discussion] MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%: "MicroPIE performed well (Relaxed_F1 score > 0.80) on over half of designated characters and achieved an overall accuracy of 79.0%"
