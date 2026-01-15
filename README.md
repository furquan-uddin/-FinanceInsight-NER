
# Financial NER Extraction Pipeline

Extract structured financial insights from 10-K/10-Q reports using custom-trained FinBERT NER model.

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/downloads/release/python-31019/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 🎯 Features

- ✅ **Custom FinBERT NER** trained on scraped finance articles
- ✅ **PDF Table Extraction** with `pdfplumber` (preserves multi-column structure)
- ✅ **Section Detection** (MD&A, Financial Statements, Risk Factors, etc.)
- ✅ **Entity Extraction** (ORG, MONEY, DATE, PERCENT, TICKER, etc.)
- ✅ **Multi-Year Table Parsing** (2024, 2023, 2022 columns)
- ✅ **Clean JSON Output** (`insights.json`)
- ✅ **Interactive CLI** for quick testing

---

## 📦 Installation

### Prerequisites
- Python 3.10.19
- CUDA (optional, for GPU training)

### Setup

```bash
# Clone repository
git clone https://github.com/DevilsBreath/-FinanceInsight-NER.git
cd -FinanceInsight-NER

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

---

## 🚀 Usage

### 1. Train the Model

Train custom FinBERT NER on your financial dataset:

```bash
# Run complete training pipeline (preprocess → train → evaluate)
python train.py --step all

# Or run individual steps
python train.py --step preprocess  # Prepare data
python train.py --step train       # Train model
python train.py --step evaluate    # Evaluate results
```

**Expected Output:**
```
================================================================================
STEP 1: PREPROCESSING
================================================================================
✓ Processed 5000 training samples
✓ Created train/val/test splits (80/10/10)

================================================================================
STEP 2: TRAINING
================================================================================
Epoch 1/50: Loss=0.245, F1=0.58
Epoch 10/50: Loss=0.089, F1=0.71
✓ Model saved to: models/finbert_ner

================================================================================
STEP 3: EVALUATION
================================================================================
Overall F1: 0.93
  ORG:        Precision=0.74, Recall=0.72, F1=0.73
  MONEY:      Precision=0.76, Recall=0.75, F1=0.75
  DATE:       Precision=0.71, Recall=0.69, F1=0.70
```

---

### 2. Extract Insights from PDF (Production)

Process complete 10-K/10-Q reports end-to-end:

```bash
# Basic usage
python pipeline.py --pdf data/pdfs/apple_10k_2023.pdf

# Custom output filename
python pipeline.py --pdf data/pdfs/tesla_10q_q4.pdf --output tesla_q4_insights.json

# Use specific model checkpoint
python pipeline.py --pdf data/pdfs/microsoft_annual.pdf --model models/finbert_ner_v2
```

**Output:** Saves to `output/<filename>_insights.json`

**Sample Output Structure:**
```json
{
  "document_info": {
    "company": "Apple Inc.",
    "fiscal_year": "2023",
    "currency": "USD"
  },
  "sections": {
    "management_discussion": "...",
    "financial_statements": "...",
    "risk_factors": "..."
  },
  "tables": [
    {
      "page": 12,
      "header": ["Item", "2024", "2023", "2022"],
      "rows": [
        {"item": "Revenue", "values": ["394.3B", "383.9B", "365.8B"]}
      ]
    }
  ],
  "metadata": {
    "total_sections": 5,
    "entities_extracted": 247,
    "tables_parsed": 8
  }
}
```

---

### 3. Quick Testing with CLI

Test NER extraction on sentences or PDFs without full pipeline:

#### Test a Sentence
```bash
python cli.py --text "Apple Inc. reported Q4 revenue of $89.5 billion, up 8% YoY."
```

**Output:**
```
================================================================================
EXTRACTED ENTITIES (5 found)
================================================================================

1. "Apple Inc."
   Type: ORG
   Position: [0:10]
   Context: ...Apple Inc. reported Q4 revenue of $89.5 billion...

2. "$89.5 billion"
   Type: MONEY
   Position: [35:48]
   Context: ...reported Q4 revenue of $89.5 billion, up 8% YoY...

3. "8%"
   Type: PERCENT
   Position: [53:55]
   Context: ...$89.5 billion, up 8% YoY...
```

#### Test a PDF Document
```bash
# Basic extraction
python cli.py --pdf data/pdfs/tesla_10k.pdf

# With statistics
python cli.py --pdf data/pdfs/amazon_annual.pdf --stats

# JSON output
python cli.py --pdf data/pdfs/microsoft_10q.pdf --format json --output msft_entities.json

# Hide context snippets for cleaner output
python cli.py --pdf data/pdfs/google_earnings.pdf --no-context --stats
```

**With `--stats` flag:**
```
================================================================================
ENTITY STATISTICS
================================================================================

Unique Entity Types: 9

Breakdown by Type:
  ORG            :   89 ( 26.0%) ██████████████
  MONEY          :   67 ( 19.6%) ██████████
  DATE           :   54 ( 15.8%) ████████
  PERCENT        :   48 ( 14.0%) ███████
  METRIC         :   31 (  9.1%) █████
  PERSON         :   23 (  6.7%) ███
  TICKER         :   15 (  4.4%) ██
  GPE            :   10 (  2.9%) █
  PRODUCT        :    5 (  1.5%) █
```

---

## 📂 Project Structure

```
financial-ner-extraction/
├── data/
│   ├── pdfs/              # Input PDF files
│   ├── raw/               # Raw training data (CoNLL format)
│   └── processed/         # Preprocessed training data
├── models/
│   └── finbert_ner/       # Trained model checkpoint
├── output/                # Extracted insights (JSON)
├── src/
│   ├── extractors/
│   │   ├── pdf_extractor.py
│   │   ├── section_detector.py
│   │   ├── ner_extractor.py
│   │   ├── table_parser.py
│   │   └── insight_generator.py
├── training/
│   ├── config.py
│   ├── predict.py
│   ├── preprocess.py
│   ├── train_ner.py
│   └── evaluate.py
├── pipeline.py            # Main extraction pipeline
├── train.py               # Training entry point
├── cli.py                 # Interactive testing CLI
└── requirements.txt
```

---

## 🏷️ Supported Entity Types

| Entity | Description | Example |
|--------|-------------|---------|
| `ORG` | Organizations | Apple Inc., SEC, Federal Reserve |
| `MONEY` | Monetary values | $394.3 billion, €50M, ¥100K |
| `DATE` | Dates and periods | Q4 2023, December 31, FY2024 |
| `PERCENT` | Percentages | 8%, 12.5%, 3.2 percentage points |
| `PERSON` | People | Tim Cook, Jerome Powell |
| `TICKER` | Stock symbols | AAPL, TSLA, MSFT |
| `METRIC` | Financial metrics | EPS, P/E ratio, revenue growth |
| `GPE` | Locations | United States, California, EMEA |
| `PRODUCT` | Products | iPhone 15, Azure, Model 3 |
| `INSTRUMENT` | Financial instruments | bonds, derivatives, options |
| `EVENT` | Events | merger, acquisition, IPO |

---

## 🛠️ Command Reference

### Training Commands
```bash
python train.py --step all           # Full pipeline
python train.py --step preprocess    # Data preprocessing only
python train.py --step train         # Training only
python train.py --step evaluate      # Evaluation only
```

### Pipeline Commands
```bash
python pipeline.py --pdf <path>                  # Extract insights
python pipeline.py --pdf <path> --output <name>  # Custom output name
python pipeline.py --pdf <path> --model <path>   # Use specific model
```

### CLI Commands
```bash
python cli.py --text "<sentence>"                # Test sentence
python cli.py --pdf <path>                       # Test PDF
python cli.py --pdf <path> --stats               # Show statistics
python cli.py --pdf <path> --format json         # JSON output
python cli.py --pdf <path> --output <name>       # Save results
python cli.py --pdf <path> --no-context          # Hide context
```

---

## 📧 Contact

For questions or issues, open an issue on [GitHub](https://github.com/DevilsBreath/-FinanceInsight-NER/issues).
