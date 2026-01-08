# Docker ML Demo

A machine learning sentiment analysis application that classifies tweet emotions into three categories: happiness, sadness, and neutral. The project includes a complete ML pipeline orchestrated with DVC and a Flask web application containerized with Docker.

## Overview

This project implements an end-to-end machine learning pipeline for sentiment analysis using tweet emotion data. It includes data ingestion, preprocessing, feature engineering, model training, and a web interface for making predictions.

## Features

- Complete ML pipeline with DVC orchestration
- Text preprocessing with NLTK (stopword removal, lemmatization, URL removal, etc.)
- Bag of Words feature extraction with CountVectorizer
- Logistic Regression classifier
- Flask web interface for predictions
- Docker containerization with multi-stage builds
- Prediction audit logging
- Support for MLflow model tracking (commented out, requires configuration)

## Project Structure

```
docker-ml-demo/
├── src/
│   ├── data/
│   │   ├── data_ingestion.py      # Downloads and splits dataset
│   │   └── data_preprocessing.py   # Text normalization
│   ├── features/
│   │   └── feature_engineering.py  # Bag of Words transformation
│   └── model/
│       ├── model_building.py       # Model training
│       ├── model_evaluation.py     # Model evaluation (commented)
│       └── register_model.py       # MLflow registration (commented)
├── flask_app/
│   ├── app.py                      # Flask application
│   ├── preprocessing_utility.py    # Text preprocessing utilities
│   ├── requirements.txt            # Flask app dependencies
│   └── templates/
│       └── index.html              # Web interface
├── data/
│   ├── raw/                        # Raw train/test splits
│   ├── interim/                    # Preprocessed data
│   └── processed/                  # Feature-engineered data
├── models/
│   ├── model.pkl                   # Trained model
│   └── vectorizer.pkl              # Fitted vectorizer
├── audit/
│   └── predictions.txt             # Prediction audit log
├── dvc.yaml                        # DVC pipeline definition
├── params.yaml                     # Configuration parameters
├── Dockerfile                      # Multi-stage Docker build
├── compose.yaml                    # Docker Compose configuration
└── pyproject.toml                  # Python project configuration
```

## Requirements

- Python >= 3.11 (project) or Python 3.10 (Docker)
- Docker and Docker Compose
- DVC (for pipeline orchestration)

## Installation

### Local Development

1. Clone the repository:
```bash
git clone <repository-url>
cd docker-ml-demo
```

2. Install dependencies using uv or pip:
```bash
uv sync
# or
pip install -e .
```

3. Install development dependencies (DVC):
```bash
uv sync --group dev
# or
pip install dvc>=3.61.0
```

### Docker Setup

The application can be run entirely in Docker without local Python installation.

## Usage

### Running the ML Pipeline

Execute the complete pipeline using DVC:

```bash
dvc repro
```

This will run all stages defined in `dvc.yaml`:
1. **data_ingestion**: Downloads tweet emotion dataset from GitHub and creates train/test splits
2. **data_preprocessing**: Applies text normalization (lowercase, stopword removal, lemmatization, etc.)
3. **feature_engineering**: Creates Bag of Words features using CountVectorizer
4. **model_building**: Trains a Logistic Regression classifier

Individual stages can be run separately:
```bash
dvc repro data_ingestion
dvc repro data_preprocessing
dvc repro feature_engineering
dvc repro model_building
```

### Running the Flask Application

#### Using Docker Compose

```bash
docker compose up --build
```

The application will be available at `http://localhost:8000`.

#### Using Docker directly

```bash
docker build -t sentiment-analysis .
docker run -p 8000:8000 -v ./audit:/app/audit sentiment-analysis
```

#### Local Flask development

```bash
cd flask_app
export PORT=8000
python app.py
```

### Making Predictions

1. Navigate to `http://localhost:8000` (or the configured port)
2. Enter text in the text area
3. Click "Predict" to get sentiment classification
4. Results are classified as: Happy (1), Sad (0), or Neutral (2)
5. All predictions are logged to `audit/predictions.txt`

## Configuration

Configuration parameters are defined in `params.yaml`:

```yaml
data_ingestion:
  test_size: 0.25  # 25% of data for testing

feature_engineering:
  max_features: 5000  # Maximum features for Bag of Words
```

## Model Details

- **Algorithm**: Logistic Regression
- **Solver**: lbfgs
- **Penalty**: L2
- **C**: 1.0
- **Feature Extraction**: CountVectorizer (Bag of Words)
- **Max Features**: 5000
- **Preprocessing**: Lowercase conversion, stopword removal, number removal, punctuation removal, URL removal, lemmatization

## Data Source

The project uses tweet emotion data from:
```
https://raw.githubusercontent.com/campusx-official/jupyter-masterclass/main/tweet_emotions.csv
```

The dataset is automatically downloaded during the data ingestion stage.

## Text Preprocessing Pipeline

The preprocessing pipeline applies the following transformations in order:

1. Convert to lowercase
2. Remove stop words (English)
3. Remove numbers
4. Remove punctuation
5. Remove URLs
6. Lemmatization using WordNet

## Docker Details

The project uses a multi-stage Docker build:
- **Builder stage**: Installs dependencies and downloads NLTK data
- **Final stage**: Copies only necessary files for runtime

Base image: `python:3.10-slim-bookworm`

Port configuration via environment variable `PORT` (default: 5000 in Dockerfile, 8000 in compose.yaml)

## Dependencies

### Main Dependencies
- python-dotenv >= 1.1.1
- dagshub >= 0.5.10
- flask >= 3.1.1
- mlflow >= 2.8,<3.0
- nltk >= 3.9.1
- numpy >= 2.3.1
- pandas >= 2.3.1
- scikit-learn >= 1.7.1

### Flask App Dependencies
- Flask == 3.0.3
- nltk == 3.8.1
- numpy == 1.26.4
- scikit-learn == 1.5.0

### Development Dependencies
- dvc >= 3.61.0

## Logging

The pipeline includes comprehensive logging:
- Console logging at DEBUG level
- File logging for errors to:
  - `errors.log` (data ingestion)
  - `transformation_errors.log` (data preprocessing)
  - `feature_engineering_errors.log` (feature engineering)
  - `model_building_errors.log` (model training)

## MLflow Integration

The project includes MLflow integration for model tracking and registration (currently commented out). To enable:

1. Set up DagsHub or MLflow tracking server
2. Configure `DAGSHUB_PAT` environment variable
3. Uncomment the model_evaluation and model_registration stages in `dvc.yaml`
4. Uncomment code in `src/model/model_evaluation.py` and `src/model/register_model.py`

## Prediction Auditing

All predictions made through the web interface are logged to `audit/predictions.txt` in the format:
```
<input_text>, <sentiment_label>
```

Where sentiment_label is one of: Happpy, Sad, or Neutral.

## License

[Add license information if applicable]

## Contributing

[Add contributing guidelines if applicable]

