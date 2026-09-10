# Metadata-Driven RAG Call-Centre Decision-Support Framework

**Author:** Lineo Nkoebe  
**Degree:** MSc Computing, University of South Africa (UNISA)  
**Purpose:** Computational implementation accompanying the final dissertation.

## 1. Purpose of this repository
This repository contains the research implementation of the metadata-driven call-centre decision-support framework described in the dissertation. The implementation demonstrates the computational workflow from structured call metadata through preprocessing, exploratory analysis, metadata-document construction, similarity retrieval, retrieval-augmented generation (RAG), constrained `TAG | ACTION` output generation, and reproducibility checks.

The repository is written so that a supervisor or reviewer can understand the implementation without the researcher being present.

## 2. Important data-availability notice
The original operational call-centre dataset is **not distributed in this repository** because it is restricted research data. It must not be uploaded to GitHub. An authorised user who has been supplied with the dataset through an approved channel can place a file named `CallCenterData.csv` in the repository root to execute the data-dependent workflow.


## 3. Repository contents
- `Lineo_Nkoebe_LLM_Call_Center.ipynb` - main research Jupyter Notebook.
- `requirements.txt` - Python package requirements.
- `TECHNICAL_DEFENCE.md` - detailed explanation and technical justification of the implementation.
- `.gitignore` - prevents CSV research data and common local files from being committed accidentally.

## 4. How to run the code
Recommended Python version: **Python 3.10-3.12**.

1. Download or clone this repository.
2. Create/activate a Python environment if desired.
3. Install the dependencies with: `python -m pip install -r requirements.txt`
4. If authorised access to the research dataset has been provided, place `CallCenterData.csv` in the same folder as the notebook.
5. Start Jupyter with: `jupyter notebook`
6. Open `Lineo_Nkoebe_LLM_Call_Center.ipynb`.
7. Restart the kernel and select **Run All** so that the notebook executes from the first cell to the last cell in sequence.
8. A successful complete run reaches the marker `PIPELINE EXECUTION COMPLETE`.

On the first connected run, the primary model path may download the MiniLM and FLAN-T5 model files if they are not already cached locally.

## 5. Computational pipeline
The primary implementation uses `sentence-transformers/all-MiniLM-L6-v2` to represent privacy-preserving metadata documents as semantic embeddings. Similar historical records are retrieved using FAISS `IndexFlatIP`; because MiniLM embeddings are L2-normalised, inner-product search is equivalent to cosine-similarity ranking. Retrieved metadata is supplied as context to `google/flan-t5-small`, which generates a deliberately constrained `TAG | ACTION` response. A strict parser normalises the generated tag to a predefined set and provides a human-escalation fallback when a valid response cannot be established.

The primary retrieval implementation uses FAISS, matching the system architecture described in the dissertation. If the primary model dependencies are unavailable, the notebook switches to a clearly labelled scikit-learn retrieval fallback only for software-flow validation.

## 6. Offline validation fallback
If the primary `sentence-transformers`, `transformers`, `torch`, `faiss-cpu`, or model weights are unavailable, the notebook contains a clearly labelled `OFFLINE_VALIDATION_FALLBACK`. It uses TF-IDF + scikit-learn retrieval and deterministic rule-based generation only to test the notebook's data flow, retrieval orchestration, parser, and top-to-bottom execution. **Fallback outputs are not MiniLM/FAISS/FLAN-T5 research results and must not be interpreted as such.**

## 7. Privacy and confidentiality
Raw telephone/customer identifiers are not included in model prompts. The preprocessing code contains one-way hashing for identifier fields, while metadata documents are constructed only from operational attributes needed by the proof of concept. The repository itself contains no operational CSV data and no raw record outputs saved in the notebook.

## 8. Technical defence and reproducibility


## Original exploratory figures

Two exploratory figures produced from the authorised research dataset are included in the repository root to preserve the visual outputs used during development and review:

- `call_type_distribution.png` — Call Type Distribution.
- `hold_time_histogram_99pct.png` — Hold time (sec) histogram clipped at the 99th percentile.

The corresponding plotting code is retained in Cells 7 and 8 of the Jupyter notebook. The hold-time figure is intentionally retained even though the distribution is concentrated at zero; this reflects the observed source data rather than a plotting error. The restricted raw call-centre dataset is not distributed in this repository.

## Data Availability

The original operational call-centre dataset used in this study is not included in this repository because it contains restricted research data and is subject to confidentiality and data-governance requirements. The repository therefore provides the computational implementation and supporting documentation without distributing the underlying dataset.

## Reproducibility

The notebook is structured for sequential execution using repository-relative paths and documented software dependencies. A fixed random seed is used where applicable to support repeatability. Exact reproduction of data-dependent results requires authorised access to the original research dataset. The `requirements.txt` file specifies the Python packages required to execute the computational workflow.
