# COMP34812 NLU Coursework — Group 31

## Track: C — Authorship Verification

**Task:** Given two text sequences, predict whether they were written by the same author (1) or different authors (0).

**Group Members:**
- Stanley Ching Tek Guan — 11398249
- Joshua Chong Yong Ping — 11317171

---

## Repository Structure

```
├── APPROACH_A.ipynb          # Approach A: training, evaluation, and demo code
├── APPROACH_B.ipynb          # Approach B: training, evaluation, and demo code
├── Group_31_A.csv            # Approach A predictions on test data
├── Group_31_B.csv            # Approach B predictions on test data
├── model_card_A.md           # Model card for Approach A
├── model_card_B.md           # Model card for Approach B
├── README.md                 # This file
```

## Drive link to models
https://drive.google.com/drive/folders/1Uf6xdqmeKhhWq5YLStrCbcrDLBDFL5l4

---

## Approach A — Traditional ML (Category A)

**TF-IDF + Stylometric Features + Compression Distance + LR/SGD Ensemble**

### Description

A traditional machine learning pipeline combining multiple feature engineering strategies with an ensemble of classifiers. No deep learning or GPU required.

### Features Used
- **Character TF-IDF (3-5 grams, 10K features):** Captures subword stylistic patterns such as spelling habits, word endings, and punctuation usage
- **Word TF-IDF (1-2 grams, 10K features):** Captures lexical patterns and word choice
- **16 Handcrafted Stylometric Features per text:** Vocabulary richness (type-token ratio, Yule's K, hapax legomena ratio), function word ratio, punctuation rates (commas, semicolons, exclamations, questions), average word/sentence length, contraction usage, uppercase/digit ratios, short/long word ratios
- **Normalized Compression Distance (NCD):** Information-theoretic similarity measure inspired by Jiang et al. (ACL 2023). Measures how well two texts compress together — same-author texts share stylistic patterns that compress more efficiently
- **Pairwise Features:** Absolute differences and element-wise products of stylometric vectors, plus TF-IDF cosine similarity at both character and word level

### Classifiers
- Logistic Regression (C=5.0, LBFGS solver)
- SGD Classifier (modified Huber loss, calibrated with 3-fold CV)
- Ensemble: averaged probabilities from both classifiers with threshold optimisation

### How to Run

The notebook `APPROACH_A.ipynb` contains all code in sequential sections:

1. **Sections 1-5:** Imports, data loading, feature extraction (stylometric + NCD + TF-IDF)
2. **Section 6:** Classifier training (Logistic Regression + SGD)
3. **Sections 7-8:** Ensemble and threshold optimisation
4. **Section 9:** Feature importance analysis
5. **Section 10:** Error analysis
6. **Section 11:** Save models to disk
7. **Section 12:** Inference function — generates predictions for new test data

**To run inference on new test data:**
1. Run all cells from Sections 1-5 to set up the environment and feature extraction functions
2. Run the Save Models cell (Section 11) if models are not yet saved
3. Update the test path in Section 12 and run:
```python
test_preds = run_inference(
    test_path="/content/test.csv",
    output_path="/content/Group_31_A.csv",
    threshold=best_thresh,
)
```

**Runtime:** ~5-10 minutes on CPU (no GPU required)

### Dev Set Performance
- Logistic Regression: ~0.685 macro F1
- SGD Classifier: ~0.680 macro F1
- Ensemble: ~0.683 macro F1
- Baseline (SVM): 0.561 macro F1

---

## Approach B — Deep Learning without Transformers (Category B)

**Siamese BiLSTM + CharCNN with Multi-Head Attention and Stylometric Features**

### Description

A Siamese neural network architecture where both texts pass through a shared-weight encoder combining word-level BiLSTM, character-level CNN, and handcrafted stylometric features.

### Architecture Components
- **GloVe 6B 300d Embeddings:** Pre-trained word vectors (frozen during training)
- **Siamese BiLSTM (2-layer, 128 hidden, bidirectional):** Shared-weight encoder for both texts
- **CharCNN:** Character-level CNN with multiple filter sizes (3, 4, 5, 7) capturing punctuation and spelling patterns
- **Multi-Head Attention Pooling (4 heads):** Learns to attend to stylistically relevant parts of the sequence
- **SpatialDropout1D:** Drops entire embedding channels for better regularisation
- **29 Handcrafted Stylometric Features:** Lexical, punctuation, structural, and character-level style features
- **Comparison Layer:** `[|w1-w2|; w1*w2; w1; w2; |c1-c2|; c1*c2; style_features]`
- **Classifier:** 3-layer MLP with LayerNorm and residual connections

### Training Details
- Optimizer: AdamW with cosine annealing LR schedule
- Gradient accumulation (effective batch size 128)
- Swap augmentation (randomly swap text_1 and text_2 during training)
- Early stopping with patience on dev F1

### How to Run

The notebook `APPROACH_B.ipynb` contains all code in sequential sections:

1. **Sections 1-6:** Setup, data loading, stylometric features, GloVe embeddings, dataset
2. **Section 7:** Model architecture definition
3. **Section 8:** Training loop
4. **Section 9:** Evaluation on dev set
5. **Section 10:** Demo code / inference function
6. **Section 11:** Error analysis

**To run inference on new test data:**
1. Ensure the trained model file `best_siamese_bilstm_v2.pt` is available (download from OneDrive link below if needed)
2. Run the inference function in Section 10:
```python
predict_from_file(
    input_path="test.csv",
    output_path="Group_31_B.csv",
    model_path="best_siamese_bilstm_v2.pt",
    threshold=best_threshold,
)
```

**Runtime:** ~15-20 minutes training on GPU (T4 or above); inference takes ~1 minute

### Dev Set Performance
- Macro F1: ~0.770
- Baseline (LSTM): 0.623 macro F1

---

## Saved Models

### Approach A (stored in Google Drive: `models_approach_a/`)
- `lr_model.pkl` — Trained Logistic Regression model
- `sgd_model.pkl` — Trained SGD Classifier (calibrated)
- `char_tfidf.pkl` — Fitted character-level TF-IDF vectorizer
- `word_tfidf.pkl` — Fitted word-level TF-IDF vectorizer
- `scaler.pkl` — Fitted StandardScaler for dense features

All files are under 10MB and included in the submission.

### Approach B (stored on OneDrive)
- `best_siamese_bilstm_v2.pt` — Trained Siamese BiLSTM model checkpoint (includes model weights, vocabulary, and style scaler)
- `vocab.pkl` — Vocabulary mapping (backup)
- `style_scaler.pkl` — Stylometric feature scaler (backup)

**OneDrive Link:** [INSERT YOUR ONEDRIVE LINK HERE]

---

## Data Sources & Attribution

- **Training/Dev Data:** Provided by the COMP34812 course team (closed-mode shared task — no external datasets used)
- **GloVe Embeddings (Approach B):** Pennington et al. (2014) "GloVe: Global Vectors for Word Representation" — downloaded from https://nlp.stanford.edu/projects/glove/
- **NCD Method (Approach A):** Inspired by Jiang et al. (2023) "Low-Resource Text Classification: A Parameter-Free Classification Method with Compressors", ACL 2023
- **scikit-learn:** Used for Logistic Regression, SGD Classifier, TF-IDF vectorization, and evaluation metrics
- **PyTorch:** Used for neural network implementation in Approach B
- **NLTK:** Used for sentence tokenization and stopword lists

---

## Use of Generative AI Tools

Large language models (LLMs) were used during development for the following purposes:
- Researching improvement techniques and feature engineering strategies for authorship verification
- Sentence structuring and documentation writing
- Debugging runtime errors

All generated code was reviewed, understood, modified, and tested by the team members. The final architectural decisions, experimental design, and analysis were performed by the team.

---

## Evaluation

Both approaches were evaluated on the dev set using macro F1-score and benchmarked against the course-provided baselines using the local scorer.

| Approach | Category | Dev F1 | Baseline F1 | Improvement |
|----------|----------|--------|-------------|-------------|
| A — TF-IDF + NCD + LR/SGD | A (Traditional ML) | 0.683 | 0.561 | +0.122 |
| B — Siamese BiLSTM + CharCNN | B (Deep Learning, no transformers) | 0.770 | 0.623 | +0.147 |

Additional evaluation includes error analysis (false positive/negative inspection), feature importance analysis (Approach A), and threshold optimisation for both approaches.
