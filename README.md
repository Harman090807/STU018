# STU019
# CTF_STU019-BookDetective  
_A forensic book & review analysis project for a dataset of books and reviews_

## Description  
This project implements a workflow to detect a manipulated book in a dataset of books + reviews, locate a fake review that contains a hidden hash, and then apply a machine-learning based model (with interpretability) to distinguish suspicious reviews from genuine ones. The goal is to:  
- Compute required hashes for identification.  
- Scan the dataset to find the manipulated book and the fake review.  
- Analyze reviews using an ML classifier to distinguish suspicious vs genuine reviews.  
- Use interpretability (e.g. SHAP) to understand which words/features contribute to genuine reviews.

## Table of Contents  
- Description  
- Prerequisites  
- Installation  
- Usage / Quick Start  
- Project Structure  
- Approach Overview  
- Flags Output (or Analysis Output)  
- How to Reproduce / Run  
- Notes / Caveats  

## Prerequisites  
- Python 3.8 or higher (recommend 3.9+)  
- `pandas >= 1.5.0`  
- `numpy >= 1.21.0`  
- `scikit-learn >= 1.0.0`  
- `nltk >= 3.8.0` (with punkt tokenizer)  
- `shap` library for interpretability  
- (Optional but recommended) Use a virtual environment (venv or conda)  

## Installation  

```bash
# Example using a virtual environment
python -m venv venv
source venv/bin/activate         # Linux / macOS
# venv\Scripts\activate          # Windows

pip install pandas numpy scikit-learn nltk shap
python -m nltk.downloader punkt
```  

If you prefer, you can maintain a `requirements.txt` and install using `pip install -r requirements.txt`.

## Usage / Quick Start  

1. Place your dataset (books + reviews) into a folder, e.g. `data/`.  
2. Run the main script to find the manipulated book and fake review:  
   ```bash
   python solver.py --step find_book --data_path data/your_books_file.json
   ```  
3. After identifying the target book and fake review, run the review-analysis & model training:  
   ```bash
   python solver.py --step analyze_reviews --data_path data/your_reviews_file.json
   ```  
4. (Optional) You may also include a Jupyter notebook or separate analysis script to train model, compute interpretability metrics, and output results.  
5. All results / flags (or findings) will be stored in `flags.txt` (or an output file you choose).  

## Project Structure  

```
CTF_STU160-BookDetective/
│
├── README.md              # this file  
├── solver.py              # main script: detection + analysis  
├── data/                  # raw dataset (books + reviews)  
├── analysis/              # (optional) ML training, SHAP analysis, EDA, notebooks  
├── flags.txt              # output: identified flags or results  
└── reflection.md          # explanation of approach, reasoning, observations  
```  

## Approach Overview  

- **Hash detection & filtering**  
  - Use standard SHA-256 hashing (e.g. via Python’s `hashlib`) to compute the hash for your identifier string.  
  - Filter books by rating criteria (e.g. `rating_number = 1234`, `average_rating = 5.0`) — to narrow down candidate books.  
  - Iterate over reviews of candidate books and search for your computed hash string in the review text (case-insensitive).  
  - If found, mark that book as the manipulated target.  

- **Title-based hash for Identification**  
  - Take the first 8 non-space characters from the selected book’s title (concatenated, without spaces).  
  - Compute SHA-256 on that substring (encoded consistently, e.g. UTF-8) to derive the “title-hash” for output or record.  

- **Fake-Review Identification**  
  - The review containing your computed hash is likely the fake / manipulated review.  

- **Review-Spam Detection + Interpretability**  
  - Preprocess review texts (tokenize, clean, optionally remove stopwords or punctuation).  
  - Vectorize reviews (e.g. TF-IDF, n-grams, embeddings) or use other features (review length, metadata if available).  
  - Train a classifier (e.g. Random Forest, Logistic Regression) to distinguish suspicious vs genuine reviews.  
    - If dataset has no explicit labels, treat the detected fake review as “suspicious” and others as candidates for “genuine,” or consider semi-supervised approach.  
  - Use an interpretability tool (e.g. SHAP) on the “genuine” reviews (low-suspicion) to extract the top words/features that most reduce suspicion (i.e. contribute to being classified as genuine).  

## Output / Flags (or Analysis Output)  

After running the full workflow, you should have a file (e.g. `flags.txt`) or output containing:  
```
FLAG1 = <computed_hash_from_title>  
FLAG2 = FLAG2{<your_computed_hash>}  
FLAG3 = FLAG3{<computed_hash_from_top_words_and_ID>}  
```  

*(If your project isn’t a CTF but rather analysis-oriented, you may instead output a summary of detected fake review, suspiciousness scores, SHAP-based feature importance, etc.)*  

## How to Reproduce / Run  

- Clone the repo:  
  ```bash
  git clone <your_repo_url>
  cd CTF_STU160-BookDetective
  ```  
- Ensure dependencies installed (see Installation).  
- Put data files in `data/`.  
- Run detection & analysis as per Usage instructions.  
- Review output (flags or analysis) and optionally examine logs / shap outputs / model performance.  

## Notes & Caveats  

- Ensure consistent string encoding (e.g. UTF-8) when computing hashes — even small differences (spaces, case, hidden characters) can change the SHA-256 drastically.  
- If using a machine-learning model for review classification, results aren’t guaranteed to be perfect — interpret them carefully, maybe manually inspect borderline cases.  
- When training a model, fix random seeds (for reproducibility), and document any hyperparameters or preprocessing steps.  

