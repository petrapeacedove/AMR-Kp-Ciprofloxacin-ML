# Machine Learning-Based Prediction of Antimicrobial Resistance Phenotypes from Whole-Genome Sequencing Data

## Overview

Antimicrobial resistance (AMR) is a major challenge in infectious disease treatment. Whole-genome sequencing (WGS) provides a large amount of genomic information that can potentially be used to predict whether a bacterial isolate is resistant or susceptible to an antibiotic.

This project investigates **machine-learning-based prediction of ciprofloxacin resistance in *Klebsiella pneumoniae*** using genomic information obtained from the **BV-BRC (Bacterial and Viral Bioinformatics Resource Center)**.

The project combines:

* AMR phenotype data
* Whole-genome sequencing data
* Genome quality control
* Known antimicrobial-resistance-associated genomic features
* Logistic Regression
* Random Forest
* MLST-based population-structure analysis
* Initial exploration of whole-genome k-mer representation

A major focus of the project is not only predictive performance, but also understanding **whether a model is learning genuine resistance-associated genomic signals or simply capturing bacterial population structure**.

---

## Research Question

> **Can genomic information from whole-genome sequencing be used to predict ciprofloxacin resistance in *Klebsiella pneumoniae*, and how much does bacterial population structure influence the prediction?**

---

## Dataset

The data were obtained from **BV-BRC**, using *Klebsiella pneumoniae* and ciprofloxacin-associated AMR records.

### Initial phenotype data

The BV-BRC ciprofloxacin AMR data contained thousands of records from *K. pneumoniae*.

For the primary analysis, laboratory-method phenotype records were filtered to retain:

* Resistant
* Susceptible

Intermediate and records without a usable Resistant/Susceptible phenotype were excluded from the binary classification dataset.

After removing duplicate genome IDs:

| Category    | Count |
| ----------- | ----: |
| Resistant   | 2,547 |
| Susceptible | 1,180 |
| Total       | 3,727 |

The final dataset therefore contained **3,727 unique genomes with binary ciprofloxacin phenotypes**.

---

## Genome Quality Control

Genome metadata were retrieved from BV-BRC and evaluated before machine-learning analysis.

Quality-control criteria included:

* Genome status
* Genome quality
* CheckM completeness
* Contamination
* Genome fragmentation

The primary filtering criteria removed genomes that were:

* Deprecated
* Marked as poor quality
* Less than 95% complete
* Greater than 5% contamination

After quality control:

> **3,662 genomes remained.**

All retained genomes were marked as **Good** quality.

The retained dataset contained:

* 3,574 WGS genomes
* 88 complete genomes

Highly fragmented genomes were also examined as potential outliers. A small number contained more than 500 contigs, but they were retained because their other quality metrics were acceptable.

---

## Whole-Genome Sequences

FASTA sequences were obtained for the 3,662 quality-controlled genomes.

Six genomes that could not be retrieved directly through the BV-BRC sequence endpoint were recovered using their assembly accessions from NCBI.

Final sequence verification showed:

```text
QC-passed genomes: 3,662
FASTA genomes:     3,662
Missing genomes:   0
Extra genomes:     0
```

Therefore, every genome used in the final ML dataset had a corresponding genome sequence.

---

## Project Dataset

The final master dataset contained:

```text
3,662 genomes
125 metadata/features columns
2,497 Resistant
1,165 Susceptible
```

The phenotype distribution after QC was therefore approximately:

* Resistant: 68.2%
* Susceptible: 31.8%

The final dataset was divided using a stratified 80/20 train-test split.

### Training set

```text
2,929 genomes
1,997 Resistant
  932 Susceptible
```

### Test set

```text
733 genomes
500 Resistant
233 Susceptible
```

The test genomes were kept separate from model training.

---

# Known AMR-Associated Genomic Features

Known ciprofloxacin-associated genomic features were extracted from BV-BRC's specialty-gene annotations.

The final feature set included:

* `gyrA_variant`
* `gyrB_variant`
* `parC_variant`
* `QnrB10`
* `QnrB_family`
* `oqxA`
* `oqxB`

These features represent database-derived AMR-associated annotations.

They should **not automatically be interpreted as independently confirmed isolate-specific mutations**, because the project did not perform independent sequence-level validation of every variant call.

The resulting feature matrix contained seven primary genomic features for the initial ML experiments.

---

# Machine Learning

Two supervised machine-learning approaches were evaluated:

## 1. Logistic Regression

Logistic Regression was used as a simple linear baseline.

The model was trained using the seven known AMR-associated genomic features.

Five-fold cross-validation was performed using the training set only.

The final untouched test set was then used for evaluation.

### Cross-validation

Approximate results:

| Metric            | Result |
| ----------------- | -----: |
| Accuracy          |  0.682 |
| ROC-AUC           |  0.535 |
| Average Precision |  0.708 |
| Recall            |  1.000 |
| F1                |  0.811 |

### Test set

| Metric      | Result |
| ----------- | -----: |
| Accuracy    |  0.682 |
| ROC-AUC     |  0.540 |
| PR-AUC      |  0.711 |
| Sensitivity |  1.000 |
| F1          |  0.811 |

However, inspection of the predictions showed that the model essentially predicted the majority class for the test data.

Therefore, the approximately 68% accuracy should **not** be interpreted as strong predictive performance.

The ROC-AUC close to 0.5 also indicates that the seven-feature model provided little useful discrimination between resistant and susceptible isolates.

---

## 2. Random Forest

A Random Forest classifier was also trained using the same seven genomic features.

### Cross-validation

| Metric            | Result |
| ----------------- | -----: |
| Accuracy          |  0.680 |
| ROC-AUC           |  0.534 |
| Average Precision |  0.706 |

### Test set

| Metric      | Result |
| ----------- | -----: |
| Accuracy    |  0.682 |
| ROC-AUC     |  0.541 |
| PR-AUC      |  0.711 |
| Sensitivity | ~0.998 |
| F1          |  0.811 |

The Random Forest did not provide a meaningful improvement over Logistic Regression.

This suggests that the seven selected known AMR-associated features alone were insufficient to build a useful ciprofloxacin-resistance classifier in this dataset.

---

# Population Structure and MLST

Because bacterial genomes are not independent observations from a single homogeneous population, population structure was investigated using **MLST (Multilocus Sequence Typing)**.

The dataset contained:

```text
557 unique MLST sequence types
79 genomes without MLST
```

Some sequence types were highly represented.

Examples include:

| Sequence Type | Genomes |
| ------------- | ------: |
| ST307         |     623 |
| ST258         |     522 |
| ST15          |     162 |
| ST147         |     113 |
| ST16          |     108 |
| ST11          |      82 |
| ST512         |      80 |
| ST14          |      77 |
| ST405         |      73 |
| ST37          |      70 |

This showed substantial population structure within the dataset.

---

# MLST-Only Baseline

An MLST-only baseline was constructed to investigate whether sequence type itself could explain a substantial amount of the observed phenotype structure.

For the random train/test split, the resistance frequency of each MLST in the training set was used to generate predictions for the test set.

This produced:

```text
Accuracy: 0.823
ROC-AUC:  0.925
```

This result is substantially stronger than the seven known-AMR-feature models.

However, this should **not** be interpreted as proof that MLST is a direct biological cause of ciprofloxacin resistance.

Instead, it demonstrates that **population structure contains substantial information associated with the phenotype in this dataset**.

---

# Grouped MLST Evaluation

To test whether the MLST-based signal would generalize to previously unseen lineages, a grouped train/test split was performed so that the same MLST did not occur in both training and test sets.

The grouped dataset contained:

```text
3,583 genomes with MLST information
```

The split produced:

```text
Training: 3,025
Testing:    558
```

with **no MLST overlap between the two sets**.

When an unseen MLST was encountered, the model used the overall training-set resistance prevalence.

The resulting baseline performed poorly:

```text
Accuracy: 0.523
ROC-AUC:  0.500
```

This is an important finding.

It indicates that the strong random-split MLST performance does not necessarily represent generalization to previously unseen sequence types.

---

# Interpretation

The experiments highlight an important issue in genomic AMR prediction:

> **A model can achieve apparently strong predictive performance because of bacterial population structure rather than because it has learned transferable resistance-associated genomic mechanisms.**

The MLST experiments demonstrated this clearly.

A random split allows closely related or identical sequence types to appear in both training and test sets. Under those conditions, sequence-type-associated phenotype patterns can be exploited by a model.

A grouped split is much more challenging because the model must generalize across previously unseen lineages.

Therefore, evaluating AMR prediction using only a random split can potentially overestimate how well a model generalizes to new bacterial populations.

---

# Whole-Genome k-mer Exploration

The project also explored representing bacterial genomes using **k-mers**, with 21-mers selected as the initial representation.

The goal was to move beyond a small set of predefined AMR-associated features and represent genomic variation across the whole genome.

The KMC software package was used for efficient k-mer counting.

The workflow explored:

```text
Genome FASTA
     ↓
21-mer extraction
     ↓
Per-genome k-mer database
     ↓
Binary presence/absence representation
     ↓
Genome prevalence
     ↓
Potential ML feature matrix
```

Individual KMC databases were successfully generated for the 2,929 training genomes.

A binary representation was also tested, where each k-mer count was converted to presence/absence before combining genomes.

The k-mer databases were then partially combined using prevalence-based union operations.

---

## Status of the k-mer Analysis

The complete k-mer machine-learning pipeline was **not completed**.

Specifically, the following final steps were not completed:

```text
Complete k-mer prevalence matrix
        ↓
k-mer feature selection
        ↓
k-mer ML classifier
        ↓
Final independent test evaluation
```

Therefore, **no k-mer classification performance is reported in this project**.

This is intentional: incomplete exploratory work is not presented as a validated prediction result.

---

# Key Findings

### 1. Known AMR features alone were insufficient

The seven selected ciprofloxacin-associated features did not produce useful predictive discrimination.

Both Logistic Regression and Random Forest produced ROC-AUC values close to 0.5.

---

### 2. Accuracy alone can be misleading

The models achieved approximately 68% accuracy, but this was largely explained by majority-class behavior.

Therefore, accuracy was interpreted alongside:

* ROC-AUC
* PR-AUC
* sensitivity
* F1 score
* prediction distribution

---

### 3. Population structure was highly informative

MLST-based prediction produced substantially stronger performance under a random split.

This indicates that bacterial lineage/population structure is strongly associated with the observed resistance phenotype in the dataset.

---

### 4. Generalization to unseen lineages is difficult

When MLST groups were separated between training and testing, the MLST-only baseline lost its predictive ability.

This demonstrates why evaluation design is particularly important in bacterial genomic ML.

---

### 5. Whole-genome representations are a logical next step

The limited performance of the predefined feature set motivates the use of broader genomic representations such as:

* k-mers
* sequence variants
* gene presence/absence
* resistance-associated mutations
* accessory genome features

However, these approaches require careful feature selection and leakage-aware evaluation.

---

# Limitations

This project has several limitations.

### Phenotype limitations

The analysis was restricted to available BV-BRC ciprofloxacin Resistant/Susceptible records.

Phenotype measurements can vary according to:

* laboratory method
* testing conditions
* reporting practices
* dataset composition

### Population structure

The dataset contains substantial MLST structure.

Random train/test splitting can therefore produce optimistic estimates when closely related isolates occur in both partitions.

### Known AMR feature representation

The initial model used a relatively small set of database-derived AMR-associated features.

This does not capture the full genomic basis of ciprofloxacin resistance.

### Variant interpretation

The specialty-gene annotations are database-derived features and were not independently validated as isolate-specific sequence mutations.

### k-mer analysis

The whole-genome k-mer classifier was not completed and therefore no k-mer performance claims are made.

---

# Reproducibility

The project was developed using:

* Python
* Google Colab
* Google Drive
* BV-BRC data
* NCBI assembly data for recovery of a small number of sequences
* KMC 3.2.4 for k-mer processing
* scikit-learn for machine learning
* pandas
* NumPy

The main project structure is:

```text
AMR_Kp_Ciprofloxacin/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── genomes/
│
├── scripts/
│
├── results/
│
├── figures/
│
└── README.md
```

Large datasets, genome FASTA files, and KMC databases are **not included in this repository**.

---

# Main Workflow

```text
BV-BRC
  │
  ├── AMR phenotype records
  │
  └── Genome metadata
          │
          ▼
  Resistant / Susceptible filtering
          │
          ▼
  Duplicate removal
          │
          ▼
  Genome quality control
          │
          ▼
  3,662 QC-passed genomes
          │
          ├───────────────┐
          │               │
          ▼               ▼
  Known AMR features    MLST
          │               │
          ▼               ▼
  Logistic Regression   Population
  Random Forest         structure analysis
          │               │
          └───────┬───────┘
                  ▼
          Model evaluation
                  │
                  ▼
       Generalization analysis

                  +
                  
          Whole-genome k-mer
             exploration
```

---

# Conclusion

This project demonstrates a complete exploratory framework for studying antimicrobial-resistance prediction from bacterial whole-genome data.

The main result was not simply a high classification accuracy. Instead, the analysis showed that **evaluation strategy and bacterial population structure are critical considerations in genomic AMR prediction**.

A small predefined set of ciprofloxacin-associated genomic features did not provide strong predictive discrimination. In contrast, MLST contained substantial information about the observed phenotype under a random split, but this signal did not generalize to unseen sequence types.

These results motivate future work using broader whole-genome representations while using **lineage-aware or grouped evaluation strategies** to obtain more realistic estimates of generalization.

---

# Future Work

Potential extensions include:

1. Complete the whole-genome k-mer feature matrix.
2. Perform training-only k-mer feature selection.
3. Train Logistic Regression, Random Forest, and other classifiers using k-mer features.
4. Compare random-split and MLST-grouped performance.
5. Investigate gene presence/absence features.
6. Incorporate sequence-level resistance mutations.
7. Evaluate models across multiple antibiotics.
8. Test external or temporally separated datasets.
9. Investigate interpretable genomic features driving model predictions.

---

# Author

**Petra Peace Dove X**

GitHub: `petrapeacedove`

---

# Disclaimer

This repository represents an academic machine-learning and bioinformatics project. Model performance reported here is specific to the analyzed dataset and evaluation design and should not be interpreted as a clinical diagnostic or treatment recommendation.
