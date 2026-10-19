# Comparative Evaluation of Traditional Machine Learning and Deep Learning Models for Phishing URL Detection

This project compares traditional machine learning algorithms with a TensorFlow/Keras neural network for classifying URLs as **phishing** or **legitimate**. It uses the PhiUSIIL Phishing URL (Website) Dataset from the UCI Machine Learning Repository and evaluates performance, error patterns, and computational cost.

> **Research caution:** The recorded scores are extremely high on the notebook's hostname-disjoint test split, but the dataset audit shows strong formatting differences between the legitimate and phishing URL classes. The scores should not be interpreted as proof of real-world effectiveness. An independent time-based or external-source evaluation is still needed.

## Project links

- **Dataset:** [PhiUSIIL Phishing URL (Website) — UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+website+dataset)
- **Project artifacts and saved models:** [Google Drive project folder](https://drive.google.com/drive/folders/1iBHgXvqrKnh63gd48lj5lkOU8B_C-JGH?usp=drive_link)
- **Research report:** `reports/Phishing_URL_ML_vs_Deep_Learning_Research_Report.pdf`
- **Notebook:** `notebooks/Phishing_URL_ML_vs_Deep_Learning_FINAL_Annotated.ipynb`
- **Demonstration video:** [Video Link](https://youtu.be/CHd1K5BDUcc)

The Google Drive folder is the location for the zipped raw dataset and saved model artifacts.

## Research question

Which approach—traditional machine learning or deep learning—provides the best balance of phishing detection performance and computational efficiency when evaluated using URL-derived features and a hostname-disjoint test set?

## Dataset

The notebook downloads the dataset from the official UCI repository during execution. The raw file loaded in the recorded run contained **235,795 rows and 56 columns**. The dataset includes URL and website-related fields and a binary label.

The notebook audits the schema, class distribution, missing values, duplicate URLs, and URL-format patterns. For the primary experiments, it constructs **42 features from the URL string** rather than using the dataset's precomputed website-derived fields. After removing 425 exact duplicate URL rows, 235,370 records remained.

For consistency within this project, the target is encoded as:

- `0` — legitimate URL
- `1` — phishing URL (positive class)

The original UCI label convention is handled during preprocessing. See the notebook for the actual mapping and validation checks.

### Important dataset limitation

In the recorded data audit, **100% of legitimate URLs** began with `https://www.`, compared with **2.43% of phishing URLs**. The audit also found that no legitimate records had a non-root path or query string, while some phishing records did. This raises a risk that models may learn dataset-collection or formatting patterns instead of robust signals that generalize to new attacks.

The notebook also demonstrates that `https://www.example.com/` receives very high phishing probabilities from both selected models. That output is a sanity-check warning—not verified ground truth for that exact input—and is discussed as a reason not to treat the prototype as a production security tool.

## Methodology

The notebook follows a reproducible workflow:

1. Download and validate the official dataset, confirm the schema and label values, and inspect missing values, duplicates, and class balance.
2. Audit URL-format patterns and the potential for dataset bias.
3. Engineer 42 URL-only features and remove exact duplicate URL rows.
4. Split the data by hostname into training, validation, and held-out test partitions to reduce host overlap across splits.
5. Train and compare six traditional machine learning configurations and three TensorFlow/Keras neural-network configurations.
6. Select one model from each paradigm using validation PR-AUC, with ROC-AUC as a tie-break, and choose the decision threshold using validation data.
7. Evaluate the selected models once on the held-out hostname-disjoint test set and save metrics, plots, model artifacts, and experiment logs.

The recorded split contains 150,426 training rows, 37,609 validation rows, and 47,335 test rows, with zero hostname overlap reported between splits.

## Experiments

### Traditional machine learning (Scikit-Learn)

- `ML01_Logistic_Baseline` — Logistic Regression baseline
- `ML02_Logistic_FeatureSelection` — Logistic Regression with feature selection
- `ML03_RandomForest_Flexible` — flexible Random Forest configuration
- `ML04_RandomForest_Regularized` — regularized Random Forest configuration
- `ML05_ExtraTrees` — Extra Trees classifier
- `ML06_HistGradientBoosting` — Histogram-based Gradient Boosting classifier

### Deep learning (TensorFlow/Keras)

- `DL07_MLP_Baseline` — baseline multilayer perceptron
- `DL08_MLP_Regularized` — regularized multilayer perceptron
- `DL09_MLP_LowerLearningRate` — multilayer perceptron with a lower learning rate

The neural-network experiments use Keras with the `tf.data` input pipeline. The notebook records the model settings, training and prediction times, metrics, and TensorBoard summaries.

## Recorded results

The following figures come from the notebook's recorded held-out test evaluation. HistGradientBoosting was selected as the traditional model and the regularized MLP was selected as the deep learning model using validation results.

| Metric | HistGradientBoosting | Regularized MLP |
|---|---:|---:|
| Accuracy | 99.81% | 99.80% |
| Precision | 99.97% | 99.93% |
| Recall | 99.59% | 99.61% |
| F1-score | 99.78% | 99.77% |
| ROC-AUC | 0.9986 | 0.9988 |
| Average precision (reported as PR-AUC) | 0.9990 | 0.9991 |
| False positives | 6 | 14 |
| False negatives | 83 | 79 |
| Recorded fit time | 9.79 s | 92.05 s |
| Prediction time per 1,000 URLs | 22.15 ms | 7.65 ms |

The two approaches performed similarly on this test split. HistGradientBoosting trained much faster and produced fewer false positives at the selected threshold. The MLP had slightly higher ROC-AUC, average precision, and recall, but missed only four fewer phishing URLs while flagging eight more legitimate URLs. These are dataset-specific results; they do not establish real-world generalization.

## Evaluation and generated outputs

The notebook generates or saves:

- A nine-experiment results table in CSV and JSON formats
- A final test-set comparison table
- Confusion matrices for the selected models
- ROC and precision–recall curves
- Deep-learning learning curves and a training-size comparison
- A CSV of misclassified test records
- Selected traditional and deep-learning model artifacts
- Preprocessing objects, feature-schema information, and model metadata
- TensorBoard event logs

The output directory used in the recorded Colab run is `/content/phishing_url_outputs`. Exported deliverables and saved model artifacts are linked in the Google Drive project folder above.

**Known diagnostic limitation:** The notebook's feature-importance attempt reported that the selected estimator did not expose a direct importance attribute. A permutation-importance analysis has not been claimed as completed in these results. The initial error preview also emphasizes the traditional model because those rows appear first; use the full saved error table when reviewing both models.

## Run the notebook in Google Colab

Google Colab is the recommended environment for this project.

1. Clone or download this repository, or open the notebook through Colab's GitHub upload option once the repository is public/shared.
2. Open `notebooks/Phishing_URL_ML_vs_Deep_Learning_FINAL_Annotated.ipynb` in Google Colab.
3. Run the notebook from the first cell to the last cell. The notebook attempts to download the official UCI dataset itself; no raw dataset should be committed to GitHub.
4. Review the experiment table, evaluation plots, error analysis, and generated artifacts in the notebook output directory.
5. If reproducing the run in a different environment, record the Python and package versions and note differences in hardware and execution time.

The recorded run used Python 3.13.16, TensorFlow 2.21.0, pandas 2.2.3, and random seed `42`. No GPU device was detected in that run, so the recorded timings reflect the available CPU runtime. Results and timings may vary slightly with software versions and hardware.

### Dependencies

The repository's `requirements.txt` lists the main packages used by the notebook. For a strict environment reproduction, pin package versions after confirming them in the execution environment. The notebook is designed around Google Colab paths and runtime behavior; local execution may require path and environment adjustments.

## Repository structure

```text
phishing-url-detection-ml-vs-deep-learning/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── Phishing_URL_ML_vs_Deep_Learning_FINAL_Annotated.ipynb
├── reports/
│   └── Phishing_URL_ML_vs_Deep_Learning_Research_Report.pdf
├── results/
│   ├── experiment_results.csv
│   ├── experiment_results.json
│   ├── final_test_comparison.csv
```

## Reproducibility and responsible interpretation

The notebook sets a random seed, records model configurations and evaluation metrics, separates training/validation/test data by hostname, selects decision thresholds using validation data, and logs experiment summaries. These steps support reproducibility, but do not eliminate all sources of bias or uncertainty.

Before treating the model as reliable for deployment, future work should evaluate an independent dataset or time-based holdout, test robustness to URL formatting changes, improve and validate feature-importance diagnostics, review false negatives and false positives by error category, and check probability calibration. A predicted phishing probability is a model score, not a guarantee that a URL is malicious or safe.

## References and attribution

1. A. Prasad and S. Chandra, “PhiUSIIL Phishing URL (Website),” *UCI Machine Learning Repository*, 2024. [https://doi.org/10.1016/j.cose.2023.103545](https://doi.org/10.1016/j.cose.2023.103545).
2. UCI Machine Learning Repository, “PhiUSIIL Phishing URL (Website) Dataset,” [dataset page](https://archive.ics.uci.edu/dataset/967/phiusiil+phishing+url+website+dataset).

Please read the research report for the broader literature review and the full reference list.
