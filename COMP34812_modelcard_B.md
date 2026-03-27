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

A supervised binary classification mode for authorship verification.
It predicts whether two input texts were written by the same author using non-transformer neural network architecture.


## Model Details

### Model Description

<!-- Provide a longer summary of what this model is. -->

This solution is a Siamese neural network for authorship verification.
It combines three feature streams: 
1) Word-level representations using pretrained GloVe embeddings and a shared BiLSTM encoder with attention pooling,
2) character-level representation using CharCNN to capture spelling and punctuation patterns, and 3) 29 handcrafted
stylometric feature differences between the two inputs. The final classifier uses the combined representation to produce a binary prediction

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

The model was trained on the provided AV training set which contains 27,643 pairs of texts. Each of them includes text_1, text_2 
and a binary label indicating if both of the texts is written by the same author. The set had 13,950 negative samples and 13,693 positive samples.
Missing texts found in the set were replaced with empty string. Stylometric feature vectors are extracted from each pair and standardised using a StandardScaler. 

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
    - best_siamese_bilstm_v2.pt: 74.2 MB
    - vocab.pkl: 640 KB
    - style_scaler.pkl: 1 KB
    - total artifact size for the final pipeline: approximately 74.8 MB

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

The model obtained an F1-score of 75.97% and an accuracy of 75.97%.

## Technical Specifications

### Hardware


    - RAM: at least 16 GB
    - Storage: at least 2GB,
    - GPU: CUDA-capable GPU recommended for training,
    - The notebook was run on device: cuda,
    

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

The model is designed to handle English-language authorship verification and may not
generate well to other languages or domains. Because it uses stylometric features and language-specific signals.
Inputs are also truncated to 200 word tokens and 800 characters, so performance may degrade with longer inputs as 
important information may be removed. The model also depends on handcrafted stylometric features, 
which may not capture deeper discourse-level writing style

## Additional Information

<!-- Any other information that would be useful for other people to know. -->

The model includes swap augmentation during training so that text order doesnt affect performance.
Stylometric features are extracted as absolute difference between the inputs and standardised before passing into the model.
Combining word-level, character-level and handcrafted stylistic features to capture different aspect of writing styles.
