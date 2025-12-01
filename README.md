# Economic-PPI-Health-Indicator

## Overview
Hi, I'm Nathan Reid - a current senior at UF - I'm attempting to build a AI-driven data analysis tool that is designed to interpret PPI data to evaluate the importance of commodities within the US economy. Millions of individuals - from farmers and manufacturers to investors and brokerages - rely on the production, import, and export of commodities like corn, cotton, palladium, and other natural resources. By analyzing multi-year datasets published by the US Census Bureau and the Bureau of Labor Statistics, my tool seeks to identify trends, correlations, and shifts in economic influence across different sectors of the economy. My goal is to leverage machine learning to see how we can transform data into actionable government insights that can lead to decision making at every level. Whether it be research, finance, or policy, I want to make a tool that better helps the user understand market dynamics, forecast changes in production (weather, labor shortages, etc) and make informed decisions. This project also serves as a deep-dive into how AI can enhance traditional economic analysis by processing large volumes of data and uncovering patterns that might otherwise go unnoticed!

### Authors
Nathan Reid 

BS, Construction Management

University of Florida - Rinker School

### Example Input and Output
| Input | Output |
|----------------------------|---------------------------|
|![Input](ppi-comrlp.xlsx)|![Output](aggregated_ppi_analysis.xlsx)|


## How To Use

### PPI Relative Importance Analysis (2023–2024)

This repository contains a small, explainable AI-style pipeline that analyzes
the **Producer Price Index (PPI) "relative importance" table** from the
U.S. Bureau of Labor Statistics (BLS) and generates:

- Percent change in relative importance from **December 2023 to December 2024**
- A categorical label describing the magnitude of change
- A short natural-language interpretation for each commodity

The code is intentionally lightweight and transparent so that the mapping
from data to explanation is easy to follow.

### Files

- `ppi_relative_importance_analysis.py`  
  Main script. Loads a local Excel file (`ppi-comrlp.xlsx`), computes
  percent changes, assigns change categories, and writes a new Excel file
  with the results.

- `requirements.txt`  
  Python dependencies (mainly `pandas` and `openpyxl`).

- `data/ppi-comrlp.xlsx`  
  The file `comrlp25` with columns:
  - `Commodity code`
  - `Index` (commodity description)
  - `Relative importance December 2023`
  - `Relative importance December 2024`

### Installation

Create and activate a virtual environment (optional but recommended), then
install dependencies:

```bash
pip install -r requirements.txt
```

### Usage

Run this script in an environment:
```
python ppi_relative_importance_analysis.py \
  --input data/ppi-comrlp.xlsx \
  --output data/ppi_comrlp_2023_2024_change_analysis.xlsx

```

After running, you will get an output file that will contain:

- Commodity – text description (from the Index column in the source file)
- RI_2023 – relative importance weight for December 2023
- RI_2024 – relative importance weight for December 2024
- Pct_Change – percent change in relative importance (2023 → 2024)
- Change_Category – one of Stable, Small change, Moderate change, Large change, Unknown
- NLG_Interpretation – a short natural-language explanation based on the change

## Notes and Limitations

The script operates on relative-importance weights, which are BLS
index weights, not prices or quantities. Large percent changes can
occur even for small absolute movements, especially when the 2023 weight
is very small.

Extremely large changes (e.g. >100%) should be interpreted as shifts
in PPI weighting, not literally as a doubling of production or
demand.
