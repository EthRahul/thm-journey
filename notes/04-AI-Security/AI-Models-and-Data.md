# AI Models & Data
Tags: #THM #AI-Security #MLSecurity

---

## Training Data

- AI models are only as good as the data they're trained on
- **Training data** = the dataset used to teach the model patterns and relationships
- Data must be: large, diverse, representative, and clean
- **Structured data** → tables, spreadsheets (easy for ML)
- **Unstructured data** → text, images, audio (harder, needs preprocessing)

### Data Pipeline
Raw Data → Collection → Cleaning → Labelling → Feature Extraction → Training

### Security Risks in Training Data
- **Data Poisoning** → attacker injects malicious samples to corrupt model behaviour
- **Biased data** → model inherits and amplifies biases present in training set
- **Sensitive data leakage** → model memorises and can regurgitate PII from training data

---

## Building the Model

- **Model** = mathematical function that maps inputs to outputs after learning from training data
- Training = adjusting model weights to minimise prediction error (loss function)
- **Overfitting** → model memorises training data, fails on new data
- **Underfitting** → model too simple, can't learn the pattern

### Common Model Types
| Type | Use Case |
|---|---|
| Linear/Logistic Regression | Simple classification/prediction |
| Decision Trees / Random Forests | Structured data classification |
| Neural Networks | Complex patterns, images, text |
| Transformers | LLMs, NLP tasks |

### Security Relevance
- Model weights = intellectual property → target for theft (model extraction attacks)
- Serialised model files (`.pkl`, `.h5`) can contain malicious code → never load untrusted models

---

## The Inheritance Problem

- Models **inherit** the properties — including flaws — of their training data
- If training data is biased, discriminatory, or poisoned → model inherits those traits
- **Transfer Learning** → taking a pre-trained model and fine-tuning it on new data
  - Risk: the base model's inherited biases/backdoors carry over to the fine-tuned model
- **Foundation Models** (GPT, BERT, etc.) are used as bases for thousands of downstream models
  - A vulnerability in a foundation model propagates to ALL models built on top of it
- Supply chain risk: downloading pre-trained models from public repos without verification

---

## The Black Box Problem

- Most powerful ML models (deep neural networks) are **not interpretable** — even their creators can't fully explain their decisions
- Input goes in → output comes out → the reasoning in between is opaque
- **Black Box** = you can query the model but cannot inspect its internals

### Why This Is a Security Problem
- Hard to audit for bias, backdoors, or malicious behaviour
- Attackers can exploit unpredictable edge-case behaviour
- Regulators increasingly require **explainability** (especially in finance/healthcare)

### Approaches to Address It
- **XAI (Explainable AI)** → techniques like LIME, SHAP that approximate why a model made a decision
- **Model Cards** → documentation of what a model does, its limitations, and training data
- **Adversarial testing** → probing the model with edge cases to find unexpected behaviour

---

## Key Takeaways

- Data quality directly determines model security and reliability
- Training pipelines are an attack surface — poisoned data = poisoned model
- Inherited flaws from base models propagate silently through transfer learning
- Black box models are hard to audit → trust but verify through adversarial testing
- Serialised model files are dangerous — treat like executable code
