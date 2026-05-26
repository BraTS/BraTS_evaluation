# BraTS Evaluation

The Brain TumorS aka Brain Tumor Segmentation (BraTS) challenge is a globally recognized community benchmark for the evaluation of automated segmentation algorithms in neuro-oncology. Over the years, BraTS has expanded to encompass a variety of specialized tasks, including:

*   **Glioma Segmentation**: The flagship task, focusing on the delineation of distinct sub-regions (e.g., enhancing tumor, tumor core, and whole tumor) in adult gliomas.
*   **Pediatric Tumor Segmentation**: Targeting brain tumors in pediatric patients, addressing the distinct anatomical and pathological characteristics seen in this population.
*   **Brain Metastasis Segmentation**: Focusing on the detection and segmentation of metastatic brain lesions, which are often small, numerous, and anatomically diverse.
*   **Meningioma Segmentation**: Evaluating the accurate boundary delineation of meningiomas, the most common primary central nervous system tumor.

Robust, and rigorous evaluation of segmentation algorithms across these diverse tasks is essential to accurately gauge clinical applicability and algorithmic performance.

---

## Panoptica: Instance-Wise Evaluation

Panoptica is a comprehensive Python library designed to bridge the gap between global semantic evaluation and clinical necessity by enabling rigorous instance-wise and lesion-wise quantification.
While traditional metrics like the whole-volume Dice score often mask critical individual detection errors, Panoptica isolates and evaluates discrete structures such as tumor subregions through a robust pipeline of instance approximation, matching, and evaluation. 

It computes a comprehensive suite of vital detection and segmentation metrics like:

* **Detection metics** True Positive, False Positive, and False Negative detection rates, 
* **Instance-specific overlap metrics** including Intersection over Union (IoU), instance-level Dice scores, and Average Precision (AP). 
* **Instance-specific distance metrics** such as Normalized surface distance (NSD), and Hausdorff distance (HD95).

* This makes Panoptica a reliable tool for benchmarking deep learning models in medical image segmentation tasks,
standardizing clinical research pipelines, and ensuring that medical image segmentation models are evaluated on their true clinical utility rather than just gross volumetric overlap.

---

## Installation

To set up the evaluation environment, you need to install the required Python packages, primarily `panoptica` and `pandas`.

```bash
# It is recommended to use a virtual environment
conda create -n brats_eval python=3.10
conda activate brats_eval
git clone https://github.com/BraTS/BraTS_evaluation.git
cd BraTS_evaluation
poetry install
```
(Note: If Poetry is not installed, you can do as follow: `curl -sSL https://install.python-poetry.org | python3 -`)

---

## Usage

The evaluation pipeline consists of two main steps: running the evaluation to generate a JSON summary, and parsing the JSON to create a structured CSV report.

### 1. Running the Evaluation (`evaluation.py`)

This script evaluates prediction NIfTI files against reference (ground truth) NIfTI files using the Panoptica framework.

**Command:**
```bash
python evaluation.py \
    --ref_path /path/to/reference/niftis/ \
    --pred_path /path/to/prediction/niftis/ \
    --config_path /path/to/panoptica_config.yaml \
    --summary_json ./panoptica_evaluation_summary.json \
    --num_subjects 
```

**Arguments:**
*   `--ref_path`: Path to the directory containing reference (ground truth) NIfTI files.
*   `--pred_path`: Path to the directory containing prediction NIfTI files.
*   `--config_path`: Path to the Panoptica configuration YAML file (e.g., `./brats-configs/config_mets.yaml`).
*   `--summary_json`: (Optional) Output path for the JSON file summarizing all evaluation metrics. Default: `./panoptica_evaluation_summary.json`.
*   `--num_subjects`: (Optional) Number of subjects to process. Useful for quick testing.

### 2. Parsing the Results (`metrics_parser.py`)

Once the evaluation is complete, a `JSON` file will be created which includes all the quantified metrics. 
In order to extract only the metrics which are used for the BraTS Leaderboard and ranking, 
use the parser script to extract these metrics into a clean CSV format. 

The parser supports two commands: `seg` (for all segmentation tasks except for the Metastasis) and `mets` 
(for only the Metastasis task which needs both segmentation and detection metrics).

**Command (Basic Segmentation Metrics):**
```bash
python metrics_parser.py seg \
    --json_path ./panoptica_evaluation_summary.json \
    --output_csv_path ./parsed_panoptica_seg_stats.csv
```

**Command (Metastasis/Detailed Instance Metrics):**
```bash
python metrics_parser.py mets \
    --json_path ./panoptica_evaluation_summary.json \
    --vol_threshold 20.0 \
    --overlap_threshold 0.1 \
    --output_csv_path ./parsed_panoptica_mets_stats.csv
```

**Arguments for `mets` command:**
*   `--vol_threshold`: Volume threshold to differentiate between large and small lesions (e.g., 20.0 voxels/mm3 depending on your config).
*   `--overlap_threshold`: Dice score threshold to classify small lesions as True Positive (TP) or False Negative (FN).

### Example
For a complete, step-by-step walkthrough of the evaluation and parsing process, please refer to the detailed Jupyter Notebook example available at: **[`./example/brats_mets.ipynb`](./example/brats_mets.ipynb)**.

---

## References

1.  **BraTS Challenge**: [Brain TumorS (BraTS) Challenge](https://www.synapse.org/Synapse:syn74274097/wiki/639571)
2.  **Panoptica Library**: [Panoptica evaluation framework](https://github.com/BrainLesion/panoptica)