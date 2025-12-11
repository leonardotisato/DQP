# Data and Information Quality Project

## Project Overview

This project demonstrates a complete **data quality assessment and improvement pipeline** applied to the "Comune di Milano - Servizi alla persona: Parrucchieri e Estetisti" (City of Milan - Personal Services: Hairdressers and Aestheticians) dataset. The goal is to systematically identify, analyze, and resolve data quality issues through standardization, deduplication, and null-value handling.

## Data Source

- **Dataset:** Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti(in).csv
- **Content:** Registry of hairdressing and beauty service establishments in Milan
- **Records:** ~3,900 entries
- **Attributes:** Service type, location, address details, establishment type, working area dimensions

## Project Structure

```
.
├── project.ipynb                    # Main analysis notebook
├── Comune-di-Milano-*.csv          # Original raw dataset
├── cleaned_Comune-di-Milano-*.csv  # Cleaned and deduplicated output
├── Report*.html                     # YData profiling reports
└── README.md                        # This file
```

## Methodology & Pipeline

The project follows a structured data quality workflow with five main phases:

### 1. **Data Quality Assessment**
- **Completeness:** Calculate missing values per column and overall dataset coverage
- **Uniqueness:** Identify distinct values and uniqueness percentages
- **Duplication:** Detect exact row duplicates
- **Key Findings:**
  - 7.52% null values in "Prevalente" (primary activity)
  - 19.06% null in "Superficie altri usi" (other usage area)
  - 66.54% null in "Superficie lavorativa" (working area)
  - All records have location information

### 2. **Data Profiling**
- Generate automated data profiling reports using **YData ProfileReport**
- Create both HTML reports for visual inspection and detailed statistics
- Reports: 
  - `Report comune di Milano Servizi alla persona di parrucchieri e estetisti.html`
  - `Report cleaned Comune di Milano Servizi alla persona di parrucchieri e estetisti.html`

### 3. **Data Wrangling & Standardization**

#### Renaming & Sorting
- Rename columns for consistency:
  - `Tipo esercizio pa` → `Tipo esercizio`
  - `Prevalente` → `Attivita Primaria`
  - `ZD` → `Municipio`
- Sort by establishment type and location

#### Text Standardization
- Convert all text fields to uppercase for uniformity
- Preserve null values during transformation

#### Tipo Esercizio Transformation
The raw `Tipo esercizio` field contains semicolon-separated service types. These are parsed and converted into **5 boolean indicator columns**:

| Original Category | Mapped Boolean Column |
|---|---|
| ACCONCIATORE, PARRUCCHIERE (various types), BARBIERE | **ACCONCIATORE** |
| ESTETISTA, MANICURE, PEDICURE, TRUCCATORE | **ESTETISTA** |
| CENTRO ABBRONZATURA, CENTRO BENESSERE, CENTRO MASSAGGI | **CENTRO ABBRONZATURA** |
| TIPO C/D TRATTAMENTI ESTETICI | **TRATTAMENTO** |
| ESECUZIONE DI TATUAGGI E PIERCING | **TATUAGGI E PIERCING** |

#### Address Parsing
Regular expression extraction from `Ubicazione` field to recover structured address components:
- Street type (`Tipo via`)
- Street name (`Via`)
- House number (`Civico`)
- District code (`Municipio`)

### 4. **Null Values Handling**

| Column | Strategy | Rationale |
|---|---|---|
| `Attivita Primaria` | Fill with first True boolean column | Infer primary activity from service type flags |
| `Tipo esercizio` | Dropped after extraction | Information transferred to boolean columns |
| `Superficie lavorativa` | Median by service type group | Preserve within-group consistency |
| `Superficie altri usi` | Fill with 0 | Assume no secondary usage if unreported |
| `Codice via` | Group fill by address, then 0 | Preserve known addresses, default unknown |
| `Municipio` | Group fill by address, then 0 | Same as Codice via |
| `Civico` | Fill with 0 | Default for missing street numbers |
| Remaining nulls | Drop rows | Final cleanup of unfillable values |

### 5. **Duplicate Detection & Deduplication**

Uses **Sorted Neighbourhood + String Similarity** approach:

**Similarity Metrics:**
- Exact match on street type and municipality
- Jaro-Winkler similarity (threshold 0.75) for street name, house number, and primary activity
- Weighted scoring: Via similarity (4×) + Activity similarity (3×) + Code match (2×) + House number (1×)

**Configuration:**
- Sorted Neighbourhood window: 25 records
- Duplicate threshold score: 5.5 / 10 maximum
- Deduplication strategy: Keep first occurrence, merge boolean columns via OR operation

**Result:** Identified potential duplicates and consolidated service type information before removal

## Output & Results

### Cleaned Dataset Statistics
- **Final record count:** 3,403 (87.1% of original)
- **Dropped duplicates:** 506 records
- **Completeness:** 100% (all remaining nulls removed)
- **Columns:** 13 (original 10 + 3 address components)

### Final Schema
```
Tipo via          - Street type (CSO, VIA, PIAZZA, etc.)
Via               - Street name
Civico            - House/building number
Codice via        - City street code
Municipio         - Milan district (1-9)
Attivita Primaria - Primary service category
Superficie altri usi      - Secondary usage area (m²)
Superficie lavorativa     - Working area (m²)
ACCONCIATORE              - Boolean indicator
ESTETISTA                 - Boolean indicator
CENTRO ABBRONZATURA       - Boolean indicator
TRATTAMENTO               - Boolean indicator
TATUAGGI E PIERCING       - Boolean indicator
```

## How to Run

### Requirements
```
pandas
numpy
ydata-profiling
scikit-learn
recordlinkage
Levenshtein
pyphonetics
jaro
matplotlib
```

### Installation
```bash
pip install -r requirements.txt
```

### Execution
1. Place `Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti(in).csv` in the project directory
2. Open `project.ipynb` in Jupyter Notebook
3. Run cells sequentially or all at once
4. Outputs:
   - `cleaned_Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti(in).csv` (cleaned dataset)
   - HTML profiling reports (if enabled)

## Key Decisions & Trade-offs

1. **Attivita Primaria Imputation:** When null, filled with the first True value from boolean columns. This makes assumptions about primary activity but preserves records that would otherwise be dropped.

2. **Address Filling:** Uses group-based imputation (other records from same street) then defaults to 0 for unknown cases. Prioritizes data preservation over perfection.

3. **Duplicate Threshold:** Score of 5.5/10 chosen to minimize false positives while catching likely duplicates. Conservative approach due to dataset heterogeneity.

4. **Outlier Detection:** Skipped—dataset contains primarily categorical and flag columns; no suitable numerical patterns for statistical outlier detection.

## Results & Quality Improvements

| Metric | Before | After | Change |
|---|---|---|---|
| Missing values | 11.9% | 0% | ✓ -11.9% |
| Exact duplicates | Yes | No | ✓ Removed |
| Fuzzy duplicates | Undetected | Detected & merged | ✓ Improved |
| Standardization | Mixed case | Uppercase | ✓ Consistent |
| Address parsing | Raw strings | Structured fields | ✓ Extracted |
| Service types | Free text (103 variants) | 5 boolean flags | ✓ Normalized |

## Files

- **project.ipynb** - Complete analysis and transformation notebook
- **Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti(in).csv** - Original dataset
- **cleaned_Comune-di-Milano-Servizi-alla-persona-parrucchieri-estetisti(in).csv** - Final cleaned dataset
- **Report comune di Milano...html** - Data profile of raw dataset
- **Report cleaned Comune di Milano...html** - Data profile of cleaned dataset

## Authors & Context

**Course:** Data and Information Quality (Politecnico di Milano)  
**Dataset Source:** Comune di Milano Open Data  
**Analysis Date:** 2024-2025

## Notes

- All transformations are documented and reversible via code
- Deduplication prioritizes information preservation (OR operation on boolean columns)
- Quality improvements documented at each pipeline stage
- Final dataset ready for downstream analysis and applications