# Machine Learning-Based Optimization of Drug-Delivery Microneedles

## Project overview

This undergraduate team project examined a computationally defined microneedle design score and a gelatin-based penetration experiment. My contribution focused on the data analysis and computational machine-learning workflow: preprocessing, J-score calculation, Random Forest surrogate modeling, SHAP analysis, and candidate ranking over the observed design catalog.

The Random Forest approximates the calculated J-score. It does not directly predict drug permeation, biological efficacy, or clinical performance.

## Data and design space

The analysis used 26 unique drug records and 144 unique microneedle configurations, producing 3,744 drug-design combinations.

| Variable | Values in source design matrix |
|---|---|
| Needle length | 350, 450, 550, 650 um |
| Base size | 100, 300 um |
| Tip diameter | 1.44, 1.93, 2.42 um |
| Inner diameter | 50, 150 um |
| Cone radius | 2.88, 3.86, 4.84 um |
| Needle density | 150, 500, 900 cm^-2 |
| Insertion speed | 5 mm/s (fixed) |
| Drug concentration | 0.5% (fixed) |

The molecular-weight range across the deduplicated drug table is 131.13-6,046.0 g/mol. The five example drugs used in candidate ranking cover a narrower range.

## J-score

The analysis defines:

$$J = \left(\frac{C}{\sqrt{MW}}\right) \times \left(N \times d_{\mathrm{inner}}^2\right) \times L$$

where C is drug concentration, MW is molecular weight, N is needle density, d_inner is inner diameter, and L is needle length. Min-max normalization is calculated over the full 3,744-row drug-by-design dataset.

J-score is a computationally defined proxy/objective. It is not a direct measurement of drug transport or efficacy, and it does not represent the full mechanics of skin insertion and transport.

## Random Forest surrogate and interpretation

A Random Forest Regressor with 100 trees and random_state=42 approximates normalized J-score from four drug descriptors and eight design variables. Feature scaling is fitted within each model pipeline.

All records sharing a design configuration stay together in the grouped holdout and five-fold GroupKFold splits. This evaluates approximation of the calculated score on held-out listed designs. It is not experimental penetration prediction or generalization to entirely unseen drugs; drug records recur across design groups. Candidate scores are out-of-fold predictions from different fold-specific models, not outputs from one final model fitted on all rows.

| Evaluation | R2 | MAE | RMSE |
|---|---:|---:|---:|
| Grouped holdout | 0.999985 | 0.000273 | 0.000864 |
| 5-fold GroupKFold out-of-fold | 0.999996 | 0.000111 | 0.000381 |

These metrics describe numerical approximation of the defined J-score only. SHAP summarizes feature contributions to the fitted RF surrogate predictions; it does not establish causal effects or experimentally measured physical importance.

| SHAP rank | Feature | Mean absolute SHAP value |
|---:|---|---:|
| 1 | Inner diameter | 0.125036 |
| 2 | Needle density | 0.083022 |
| 3 | Needle length | 0.034153 |
| 4 | Molecular weight | 0.027349 |

## Candidate design ranking

The source matrix is a finite catalog of 144 observed configurations. The notebooks exhaustively rank that catalog for five example drugs using grouped out-of-fold predictions. 

| Drug | Design ID | Length (um) | Base (um) | Tip (um) | Inner diameter (um) | Cone radius (um) | Density (cm^-2) | Concentration (%) | OOF predicted normalized J |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Ascorbic acid | 132 | 650 | 300 | 1.93 | 150 | 3.86 | 900 | 0.5 | 0.862671 |
| Estradiol | 132 | 650 | 300 | 1.93 | 150 | 3.86 | 900 | 0.5 | 0.693198 |
| Ibuprofen | 120 | 650 | 100 | 1.44 | 150 | 2.88 | 900 | 0.5 | 0.796964 |
| Lidocaine | 120 | 650 | 100 | 1.44 | 150 | 2.88 | 900 | 0.5 | 0.747739 |
| Paroxetine | 114 | 650 | 100 | 1.93 | 150 | 3.86 | 900 | 0.5 | 0.630447 |

Concentration remains fixed at the observed 0.5%. These candidates are computational rankings of observed configurations, not experimentally validated optima.

## Gelatin-based experiment

Model-derived designs were fabricated using 3D printing and evaluated using a gelatin-based penetration experiment. A five-prototype summary, preserving the values previously entered in the project notebook, is saved separately in results/tables/experimental_validation_summary.csv.

The experiment is limited to the conducted gelatin-based model and is not clinical validation. No claim about human skin performance or clinical drug delivery is made.

## Limitations

- J-score is a simplified computational proxy, not a direct measure of biological or clinical drug-delivery efficacy.
- The Random Forest approximates the calculated J-score; its metrics do not measure experimental or clinical prediction performance.
- Candidate ranking is constrained to configurations supported by the available design matrix. Insertion speed and drug concentration are fixed in that data. The evaluation does not establish generalization to unseen drug compounds.
- SHAP describes this fitted model and does not support causal interpretation.
- The gelatin-based experiment is a limited physical model. 
