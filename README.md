# Phishing Email Classification

A machine learning pipeline to classify emails as **phishing** or **legitimate** using TF-IDF vectorization and a Random Forest classifier.

```mermaid
graph TD
    A[Raw Email Dataset] --> B[Preprocessing<br/>TF-IDF Vectorization]
    B --> C[Train/Test Split<br/>80/20]
    C --> D[Random Forest<br/>Classifier]
    D --> E[Trained Model]
    E --> F[Prediction API]
    F --> G{Is Phishing?}
    G -->|Yes| H[🚨 Phishing]
    G -->|No| I[✅ Legitimate]

    style A fill:#4a6fa5,color:#fff
    style B fill:#7b68ee,color:#fff
    style C fill:#9370db,color:#fff
    style D fill:#6a5acd,color:#fff
    style E fill:#483d8b,color:#fff
    style F fill:#5b4c9a,color:#fff
    style G fill:#ff8c00,color:#fff
    style H fill:#dc143c,color:#fff
    style I fill:#2e8b57,color:#fff
```

## Project Pipeline

The project follows a three-stage pipeline:

```mermaid
flowchart LR
    subgraph Stage1 [Stage 1: Preprocess]
        A1[CSV Dataset<br/>text + label] --> A2[TF-IDF<br/>Vectorizer]
        A2 --> A3[Feature Matrix X<br/>Label Vector y]
        A3 --> A4[Save: preprocessed_data.pkl]
    end

    subgraph Stage2 [Stage 2: Train]
        B1[Load .pkl] --> B2[Train/Test Split]
        B2 --> B3[Random Forest]
        B3 --> B4[Evaluation]
        B4 --> B5[Save: phishing_detector.pkl]
    end

    subgraph Stage3 [Stage 3: Predict]
        C1[New Email] --> C2[Vectorize]
        C2 --> C3[Load Model]
        C3 --> C4[Predict]
        C4 --> C5[Phishing / Not Phishing]
    end

    Stage1 --> Stage2 --> Stage3
```

## Workflow

```mermaid
sequenceDiagram
    participant User
    participant Preprocess
    participant Trainer
    participant Predictor
    participant Model

    User->>Preprocess: python preprocess.py
    Preprocess->>Preprocess: Load phishing_email.csv
    Preprocess->>Preprocess: TF-IDF vectorization
    Preprocess-->>User: preprocessed_data.pkl

    User->>Trainer: python train.py
    Trainer->>Trainer: Load preprocessed_data.pkl
    Trainer->>Trainer: Train/test split (80/20)
    Trainer->>Trainer: Train RandomForest
    Trainer->>Trainer: Evaluate accuracy
    Trainer-->>User: phishing_detector.pkl

    User->>Predictor: python predict.py
    Predictor->>Predictor: Load model + vectorizer
    Predictor->>Predictor: Vectorize input email
    Predictor->>Model: predict()
    Model-->>Predictor: Prediction result
    Predictor-->>User: "Phishing" or "Not Phishing"
```

## Data Flow

```mermaid
flowchart TD
    CSV[("phishing_email.csv<br/>13,614 emails")] --> LOAD[pandas.read_csv]
    DS2[("dataset-2.csv<br/>10,519 emails")] --> LOAD
    LOAD --> COL{Columns: text & label?}
    COL -->|Yes| TFIDF[TfidfVectorizer<br/>fit_transform]
    COL -->|No| ERR[Raise ValueError]
    TFIDF --> X[("X: Feature Matrix<br/>(n_samples, n_features)")]
    TFIDF --> VECT[("Vectorizer Object")]
    LOAD --> Y[("y: Label Vector<br/>(0 = legitimate, 1 = phishing)")]
    X & Y & VECT --> PKL[("preprocessed_data.pkl")]
    PKL --> SPLIT[Train/Test Split<br/>test_size=0.2]
    SPLIT --> RF[RandomForestClassifier<br/>random_state=42]
    RF --> ACC[Accuracy Score]
    RF --> MODEL[("phishing_detector.pkl")]

    style CSV fill:#4a6fa5,color:#fff
    style DS2 fill:#4a6fa5,color:#fff
    style PKL fill:#6a5acd,color:#fff
    style MODEL fill:#483d8b,color:#fff
    style RF fill:#ff8c00,color:#fff
```

## Model Architecture

```mermaid
classDiagram
    class Preprocess {
        +load_csv(path)
        +tfidf_vectorize(text)
        +save_preprocessed(data, file)
    }
    class Train {
        +load_preprocessed(file)
        +split_data(X, y)
        +train_random_forest()
        +evaluate(y_test, y_pred)
        +save_model(model, file)
    }
    class Predict {
        +load_model(file)
        +load_vectorizer(file)
        +vectorize_email(text)
        +predict(email_vector)
    }
    class RandomForest {
        +n_estimators: 100
        +random_state: 42
        +criterion: gini
        +fit(X_train, y_train)
        +predict(X_test)
    }
    Preprocess --> Train : preprocessed_data.pkl
    Train --> Predict : model.pkl
    Train --> RandomForest : trains
    RandomForest --> Predict : uses
```

## Files

| File | Purpose |
|------|---------|
| `preprocess.py` | Loads CSV, vectorizes text with TF-IDF, saves preprocessed data |
| `train.py` | Loads preprocessed data, trains RandomForest, evaluates, saves model |
| `predict.py` | Loads saved model, vectorizes input, predicts phishing/legitimate |
| `phishing_email.csv` | Primary dataset (13,614 emails) |
| `dataset-2.csv` | Secondary dataset (10,519 emails) |
| `Pipfile` | Python 3.12 dependency specification |

## Installation

```bash
pip install pandas scikit-learn
```

Or using Pipenv:

```bash
pipenv install
pipenv shell
```

## Usage

### 1. Preprocess the data

```bash
python preprocess.py
```

Output: `preprocessed_data.pkl` containing the TF-IDF feature matrix, label vector, and vectorizer.

### 2. Train the model

```bash
python train.py
```

Output: Trained `phishing_detector.pkl` with accuracy printed to console.

### 3. Make predictions

```bash
python predict.py
```

The script includes a sample prediction. Modify `email_text` in `predict.py` to classify custom emails.

## How It Works

```mermaid
flowchart RL
    subgraph Text [Text Vectorization]
        T1[Raw Email Text] --> T2[TF-IDF<br/>Term Frequency /<br/>Inverse Document<br/>Frequency]
        T2 --> T3[Sparse Numerical<br/>Feature Matrix]
    end

    subgraph Classifier [Classification]
        C1[Feature Matrix] --> C2[Random Forest<br/>Ensemble of<br/>Decision Trees]
        C2 --> C3[Majority Vote]
        C3 --> C4[Prediction]
    end

    subgraph Output [Output]
        O1[0 = Legitimate]
        O2[1 = Phishing]
    end

    Text --> Classifier --> Output
```

## Dataset Format

The CSV files must contain two columns:

```
text,label
"Your account has been compromised...",1
"Meeting scheduled for tomorrow",0
```

- **text**: Email body content
- **label**: 0 for legitimate, 1 for phishing

## License

MIT License — see [LICENSE](LICENSE).
