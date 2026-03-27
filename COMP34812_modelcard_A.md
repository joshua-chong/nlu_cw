---
{}
---
language: en
license: cc-by-4.0
tags:
- text-classification
- authorship-verification
- traditional-ml
- tf-idf
- stylometry
repo: https://github.com/joshua-chong/nlu_cw

---

# Model Card for w51312jc-m84507sc-AV

<!-- Provide a quick summary of what the model is/does. -->

A supervised binary classification mode for authorship verification.
It predicts whether two input texts were written by the same author using traditional machine learning features and an ensemble classifier.


## Model Details

### Model Description

<!-- Provide a longer summary of what this model is. -->

This solution is a traditional machine learning approach for authorship verification.
It combines character-level TF-IDF n-grams, word-level TF-IDF n-grams, handcrafted stylometric features and
Normalised Compression Distance (NCD)- based similarity features. Two classifiers, Logistic Regression and
a calibrated SGD classifier are trained on the features and combined in an ensemble for final binary prediction.

- **Developed by:** Joshua Chong and Stanley Ching
- **Language(s):** English
- **Model type:** Supervised
- **Model architecture:** TF-IDF + Stylometric Features + Normalised Compression Distance + LR/SGD Ensemble
- **Finetuned from model [optional]:** N/A

### Model Resources

<!-- Provide links where applicable. -->

- **Repository:** N/A
- **Paper or documentation:** N/A

## Training Details

### Training Data

<!-- This is a short stub of information on the training data that was used, and documentation related to data pre-processing or additional filtering (if applicable). -->

The model was trained on the provided AV training set which contains 27,643 pairs of texts. Each of them includes text_1, text_2 
and a binary label indicating if both of the texts is written by the same author. The set had 13,950 negative samples and 13,693 positive samples.
Missing texts found in the set were replaced with empty string. Text fields were stripped and cast
to strings before feature extraction.

### Training Procedure

<!-- This relates heavily to the Technical Specifications. Content here should link to that section when it is relevant to the training procedure. -->

#### Training Hyperparameters

<!-- This is a summary of the values of hyperparameters used in training the model. -->


    - seed: 42
    - character TF-IDF analyzer: char_wb
    - character TF-IDF ngram_range: (3, 5)
    - character TF-IDF max_features: 10000
    - word TF-IDF ngram_range: (1, 2)
    - word TF-IDF max_features: 10000
    - Logistic Regression C: 5.0
    - Logistic Regression max_iter: 2000
    - Logistic Regression solver: lbfgs
    - SGD loss: modified_huber
    - SGD alpha: 0.00005
    - SGD max_iter: 2000

#### Speeds, Sizes, Times

<!-- This section provides information about how roughly how long it takes to train the model and the size of the resulting model. -->


    - overall training time: 8-10 minutes total with T4 GPU
    - lr_model.pkl: 158 KB
    - sgd_model.pkl: 473 KB
    - char_tfidf.pkl: 333 KB
    - word_tfidf.pkl: 366 KB
    - scaler.pkl: 2 KB
    - total artifact size for the final ensemble pipeline: approximately 1.3 MB
    

## Evaluation

<!-- This section describes the evaluation protocols and provides the results. -->

### Testing Data & Metrics

#### Testing Data

<!-- This should describe any evaluation data used (e.g., the development/validation set provided). -->

For now, the model is evaluated on the provided development set containing 5993 text pairs.
    The label distribution was 2,937 negative samples and 3,056 positive samples.

#### Metrics

<!-- These are the evaluation metrics being used. -->


    - Precision
    - Recall
    - F1-score
    - Accuracy

### Results

The model obtained an F1-score of 69.28% and an accuracy of 69.28%.
    - Logistic Regression macro F1: 0.6905
    - SGD macro F1: 0.6774
    - Ensemble macro F1: 0.6928
    - Optimised threshold: 0.50
    - Final accuracy: 0.6928

## Technical Specifications

### Hardware


    - CPU-only execution
    - Standard RAM sufficient for sparse feature matrices
    - No GPU required
      

### Software


    - Python
    - scikit-learn
    - pandas
    - NumPy
    - SciPy
    - matplotlib
    - seaborn
    - pickle
      

## Bias, Risks, and Limitations

<!-- This section is meant to convey both technical and sociotechnical limitations. -->

The model is designed for English-language authorship verification
and may not generalise well to other languages or domains. Because it relies heavily on
surface-level features such as character n-grams, word n-grams, punctuation patterns, and
compression-based similarity, it may capture topic or genre-related regularities in addition
to authorial style. The stylometric feature set is handcrafted and limited to 16 features per
text, so it may miss deeper discourse-level or semantic stylistic patterns. Performance may
also vary when texts are very short, noisy, or come from domains that differ substantially
from the training data.

## Additional Information

<!-- Any other information that would be useful for other people to know. -->

This approach combines several complementary feature types:
character TF-IDF for subword style, word TF-IDF for lexical patterns, stylometric descriptors
for writing habits, and Normalized Compression Distance for information-theoretic similarity.
The final system uses an ensemble of Logistic Regression and a calibrated SGD classifier by
averaging predicted probabilities. Threshold optimisation on the development set did not improve
over the default threshold of 0.50.
