---
{}
---
language: en
license: cc-by-4.0
tags:
- text-classification
- authorship-verification
- bilstm
- stylometry
repo: https://github.com/joshua-chong/nlu_cw

---

# Model Card for w51312jc-m84507sc-AV

<!-- Provide a quick summary of what the model is/does. -->

A supervised binary classification model for authorship verification.
It predicts whether two input texts were written by the same author using a non-transformer neural network architecture.


## Model Details

### Model Description

<!-- Provide a longer summary of what this model is. -->

This solution is a Siamese neural network for authorship verification.
It combines three feature streams: 
1) Word-level representations using pretrained GloVe embeddings and a shared BiLSTM encoder with attention pooling,
2) character-level representation using CharCNN to capture spelling and punctuation patterns, and
3) 29 handcrafted stylometric feature differences between the two inputs.
The final classifier uses the combined representation to produce a binary prediction.

- **Developed by:** Joshua Chong and Stanley Ching
- **Language(s):** English
- **Model type:** Supervised
- **Model architecture:** Siamese BiLSTM + Character CNN + Stylometric Features
- **Finetuned from model [optional]:** GloVe pretrained word embeddings

### Model Resources

<!-- Provide links where applicable. -->

- **Repository:** https://nlp.stanford.edu/projects/glove/
- **Paper or documentation:** https://aclanthology.org/D14-1162.pdf

## Training Details

### Training Data

<!-- This is a short stub of information on the training data that was used, and documentation related to data pre-processing or additional filtering (if applicable). -->

The model was trained on the provided AV training set which contains 27,643 pairs of texts. Each pair includes text_1, text_2
and a binary label indicating whether both texts were written by the same author. The set had 13,950 negative samples and 13,693 positive samples.
Missing texts found in the set were replaced with empty strings. Stylometric feature vectors are extracted from each pair and standardised using a StandardScaler.

### Training Procedure

<!-- This relates heavily to the Technical Specifications. Content here should link to that section when it is relevant to the training procedure. -->

#### Training Hyperparameters

<!-- This is a summary of the values of hyperparameters used in training the model. -->


    - max_len: 200 word tokens
    - batch_size: 64
    - accum_steps: 1
    - effective_batch_size: 64
    - embedding_dim: 300
    - max_vocab: 50000
    - hidden_dim: 128
    - num_layers: 2
    - dropout: 0.25
    - embedding_dropout: 0.15
    - spatial_dropout: 0.1
    - char_vocab_size: 128
    - char_embed_dim: 30
    - char_cnn_filters: 64
    - char_max_len: 800
    - learning_rate: 2e-3
    - weight_decay: 1e-4
    - label_smoothing: 0.02
    - num_epochs: 20
    - patience: 5
    - optimizer: AdamW
    - scheduler: CosineAnnealingLR
    - loss: BCEWithLogitsLoss
    - seed: 42

#### Speeds, Sizes, Times

<!-- This section provides information about how roughly how long it takes to train the model and the size of the resulting model. -->


    - overall training time: 2 hours
    - duration per training epoch: 6 minutes
    - best epoch: 19
    - approach_b_model.pt: 74.2 MB
    - vocab.pkl: 640 KB
    - style_scaler.pkl: 1 KB
    - total artifact size for the final pipeline: approximately 74.8 MB

## Evaluation

<!-- This section describes the evaluation protocols and provides the results. -->

### Testing Data & Metrics

#### Testing Data

<!-- This should describe any evaluation data used (e.g., the development/validation set provided). -->

The model is evaluated on the provided development set containing 5,993 text pairs.
The label distribution was 2,937 negative samples and 3,056 positive samples.

#### Metrics

<!-- These are the evaluation metrics being used. -->


    - Precision
    - Recall
    - F1-score
    - Accuracy

### Results

The model achieved a macro F1-score of 76.33% and an accuracy of 76.36% at the default threshold of 0.5.
Threshold optimisation on the dev set identified an optimal threshold of 0.48, yielding a marginal improvement to F1 76.36%.

Per-class results at threshold 0.5:

    - Different Author (0): precision 0.76, recall 0.75, F1 0.76
    - Same Author (1):      precision 0.76, recall 0.78, F1 0.77

## Technical Specifications

### Hardware


    - RAM: at least 16 GB
    - Storage: at least 2 GB
    - GPU: CUDA-capable GPU recommended for training
    - The notebook was run on device: cuda


### Software


    - Python
    - PyTorch
    - NumPy
    - pandas
    - scikit-learn
    - NLTK
    - matplotlib
    - seaborn
    - tqdm


## Bias, Risks, and Limitations

<!-- This section is meant to convey both technical and sociotechnical limitations. -->

The model is designed for English-language authorship verification and may not generalise well to other languages or domains,
as it relies on stylometric features and language-specific signals. Inputs are truncated to 200 word tokens and 800 characters,
so performance may degrade on longer texts where important stylistic information is removed. The model also depends on
handcrafted stylometric features which may not capture deeper discourse-level writing style. The model shows signs of
overconfidence on training data (train loss 0.25 vs dev loss 0.57 at best checkpoint), though dev F1 remains stable,
suggesting the classifier generalises despite miscalibrated probabilities.

## Additional Information

<!-- Any other information that would be useful for other people to know. -->

The model uses swap augmentation during training so that text order does not affect performance.
Stylometric features are extracted as absolute differences between the two inputs and standardised before being passed into the model.
Word-level, character-level, and handcrafted stylistic features are combined to capture different aspects of writing style.
