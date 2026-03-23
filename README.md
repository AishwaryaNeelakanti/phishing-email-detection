[README.md](https://github.com/user-attachments/files/26189721/README.md)
# Phishing Email Detection Using Fine-Tuned GPT-4.1 Mini

A thesis project investigating the effectiveness of fine-tuned large language models for phishing email detection, compared against traditional deep learning baselines (BERT, RoBERTa, BiLSTM).

## Project Overview

This research fine-tunes OpenAI's GPT-4.1 Mini model on a curated phishing email dataset and evaluates its performance against established NLP baselines. Synthetic phishing email augmentation is performed using Claude Sonnet 4 to expand training data across multiple attack domains.

## Repository Structure

```
Thesis_ICT/
├── Fine_tune_GPT4.1.ipynb              # Fine-tuning GPT-4.1 Mini via OpenAI API
├── Final_Evaluation.ipynb               # Evaluation of fine-tuned model (21,489 emails)
├── Baseline_Models.ipynb                # BERT, RoBERTa, BiLSTM baseline training & evaluation
├── PromptingMailCreation.ipynb          # Synthetic phishing email generation using Claude Sonnet 4
├── model_train.jsonl                    # Fine-tuning training data (4,092 samples)
├── model_test.jsonl                     # Fine-tuning test/validation data (1,024 samples)
├── Enron.csv                            # Enron email corpus (legitimate emails)
├── Nazario_5.csv                        # Nazario phishing email corpus
├── Nazario_2021_2022_augmented_final.csv# Augmented Nazario phishing samples
├── phishing_dataset_20251116_101546.csv # Generated phishing dataset
├── test_generation_20251116_095734.csv  # Test generation output
└── README.md
```

## Methodology

### 1. Data Collection and Preparation

| Source | Type | Samples |
|--------|------|---------|
| Enron Email Corpus | Legitimate | ~4,000 (sampled) |
| Nazario Phishing Corpus | Phishing | ~13,600 |
| Synthetic (Claude Sonnet 4) | Phishing | ~3,900 |
| **Total Evaluation Set** | **Mixed** | **21,489** |

The fine-tuning dataset uses a separate split of 4,092 training and 1,024 validation samples formatted as OpenAI chat completions in JSONL.

### 2. Synthetic Phishing Email Generation

`PromptingMailCreation.ipynb` uses Claude Sonnet 4 to generate phishing email variations across five attack domains:

- **Financial BEC** (Business Email Compromise)
- **Healthcare Credential Harvesting**
- **Financial Credential Theft**
- **Healthcare BEC**
- **Education BEC**

Each domain includes 10 variation strategies spanning low to high sophistication levels, covering different psychological manipulation tactics (urgency, authority, fear).

### 3. Fine-Tuning GPT-4.1 Mini

`Fine_tune_GPT4.1.ipynb` fine-tunes the `gpt-4.1-mini-2025-04-14` model using the OpenAI fine-tuning API:

- **Base model**: GPT-4.1 Mini
- **Method**: Supervised fine-tuning
- **Epochs**: 1
- **Training samples**: 4,092
- **Validation samples**: 1,024
- **Training steps**: 2,046
- **Final validation loss**: 0.01

The model is fine-tuned to perform binary classification (phishing vs legitimate) with structured analysis output including classification, risk level, and recommendation.

### 4. Baseline Models

`Baseline_Models.ipynb` implements three baseline models for comparison:

| Model | Architecture | Tokenization |
|-------|-------------|--------------|
| BERT | `bert-base-uncased` + classification head | WordPiece |
| RoBERTa | `roberta-base` + classification head | BPE |
| BiLSTM | 2-layer bidirectional LSTM | Whitespace |

All baselines are trained for 1 epoch with learning rate 2e-5, batch size 32, and max sequence length 512.

### 5. Evaluation

`Final_Evaluation.ipynb` evaluates the fine-tuned GPT-4.1 Mini on the full 21,489-sample evaluation dataset using parallel async API calls (100 concurrent requests).

The LLM response parser extracts classifications using multiple regex patterns with fallback keyword counting.

## Results

### Baseline Model Performance (20% held-out test set, n=4,298)

| Model | Accuracy | Precision | Recall | F1 Score | FN |
|-------|----------|-----------|--------|----------|-----|
| BERT | 99.26% | 99.71% | 99.37% | 0.9954 | 22 |
| **RoBERTa** | **99.65%** | **99.94%** | **99.63%** | **0.9979** | **13** |
| BiLSTM | 85.71% | 85.34% | 99.54% | 0.9190 | 16 |

### Fine-Tuned GPT-4.1 Mini Performance (full evaluation set, n=21,489)

| Metric | Value |
|--------|-------|
| Overall Accuracy | 89.59% |
| Phishing Precision | 88.67% |
| Phishing Recall | 99.98% |
| Phishing F1 | 0.9399 |
| Legitimate Precision | 99.83% |
| Legitimate Recall | 44.15% |
| Legitimate F1 | 0.6122 |

### Performance by Data Source

| Source | Samples | Accuracy |
|--------|---------|----------|
| Enron (Legitimate) | 4,000 | 44.15% |
| Enron Phishing | 13,617 | 99.99% |
| Synthetic Phishing | 3,872 | 99.97% |

## Requirements

### Python Dependencies

```
torch
transformers
openai
anthropic
pandas
numpy
scikit-learn
tqdm
nest_asyncio
```

### Hardware

- Baseline models were trained on NVIDIA A100-SXM4-80GB
- Fine-tuning and evaluation use the OpenAI API (cloud)
- Synthetic generation uses the Anthropic API (cloud)

### API Keys

The notebooks require:
- **OpenAI API key** for fine-tuning and evaluation
- **Anthropic API key** for synthetic phishing email generation

Set these as environment variables:
```bash
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
```

## Usage

### 1. Generate Synthetic Phishing Emails

Run `PromptingMailCreation.ipynb` to generate augmented phishing training data across multiple attack domains.

### 2. Fine-Tune GPT-4.1 Mini

Run `Fine_tune_GPT4.1.ipynb` to upload the JSONL training data and launch a fine-tuning job via the OpenAI API. The notebook monitors training progress and saves the fine-tuned model name.

### 3. Train Baseline Models

Run `Baseline_Models.ipynb` to train and evaluate BERT, RoBERTa, and BiLSTM on the evaluation dataset. Requires GPU for reasonable training times.

### 4. Evaluate Fine-Tuned Model

Run `Final_Evaluation.ipynb` to evaluate the fine-tuned GPT-4.1 Mini against the full evaluation dataset. Supports parallel async processing for faster evaluation.

## Known Limitations

- **Class imbalance**: The evaluation dataset has a 4.37:1 phishing-to-legitimate ratio, which impacts metric interpretation
- **Legitimate email recall**: The fine-tuned GPT-4.1 Mini achieves only 44.15% recall on legitimate emails, misclassifying 2,234 out of 4,000 as phishing
- **Evaluation split difference**: Baselines use a 20% held-out split while the fine-tuned model is evaluated on the full dataset, limiting direct comparison
- **Single epoch training**: All models are trained for only 1 epoch
- **BiLSTM tokenization**: Uses simple whitespace tokenization rather than subword tokenization, putting it at a disadvantage relative to transformer baselines

## License

This project is developed as part of an ICT thesis for academic research purposes.

## Acknowledgements

- [Enron Email Dataset](https://www.cs.cmu.edu/~enron/) for legitimate email samples
- [Nazario Phishing Corpus](https://monkey.org/~jose/phishing/) for phishing email samples
- OpenAI for GPT-4.1 Mini fine-tuning API
- Anthropic for Claude Sonnet 4 synthetic data generation
