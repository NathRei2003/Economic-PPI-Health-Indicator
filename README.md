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

## Raw Code

```
#!/usr/bin/env python3
"""
PPI Relative Importance Analysis (2023–2024)

This script reads a BLS PPI "relative importance" Excel file (ppi-comrlp.xlsx),
extracts the December 2023 and December 2024 weights, computes percent changes,
assigns change categories, and generates a natural-language interpretation
for each commodity.

Usage:
    python ppi_relative_importance_analysis.py \
        --input data/ppi-comrlp.xlsx \
        --output data/ppi_comrlp_2023_2024_change_analysis.xlsx
"""

import argparse
from pathlib import Path

import pandas as pd


def load_relative_importance_excel(path: Path, sheet_name: str = "comrlp25") -> pd.DataFrame:
    """
    Load the BLS PPI relative-importance table from an Excel file.

    Assumes the sheet has a header row with columns:
      - 'Commodity code'
      - 'Index'
      - 'Relative importance December 2023'
      - 'Relative importance December 2024'
    """
    df = pd.read_excel(path, sheet_name=sheet_name, header=0)

    # Keep only relevant columns (raise if missing)
    required_cols = [
        "Index",
        "Relative importance December 2023",
        "Relative importance December 2024",
    ]
    missing = [c for c in required_cols if c not in df.columns]
    if missing:
        raise ValueError(f"Missing required columns in input file: {missing}")

    df = df[required_cols].copy()
    df = df.rename(
        columns={
            "Index": "Commodity",
            "Relative importance December 2023": "RI_2023",
            "Relative importance December 2024": "RI_2024",
        }
    )

    return df


def clean_relative_importance_df(df: pd.DataFrame) -> pd.DataFrame:
    """
    Clean and filter the relative-importance DataFrame:
    - Drop rows with missing commodity
    - Remove the 'All commodities' total row
    - Convert RI_2023 and RI_2024 to numeric and drop invalid rows
    """
    df = df.copy()
    df = df.dropna(subset=["Commodity"])
    df["Commodity"] = df["Commodity"].astype(str).str.strip()

    # Drop overall total row
    df = df[~df["Commodity"].str.lower().str.contains("all commodities")]

    # Convert to numeric
    df["RI_2023"] = pd.to_numeric(df["RI_2023"], errors="coerce")
    df["RI_2024"] = pd.to_numeric(df["RI_2024"], errors="coerce")

    df = df.dropna(subset=["RI_2023", "RI_2024"])

    return df


def compute_percent_change(df: pd.DataFrame) -> pd.DataFrame:
    """
    Compute percent change in relative importance from 2023 to 2024:
        Pct_Change = (RI_2024 - RI_2023) / RI_2023 * 100
    """
    df = df.copy()
    df["Pct_Change"] = (df["RI_2024"] - df["RI_2023"]) / df["RI_2023"] * 100
    return df


def interpret_change_row(row: pd.Series) -> pd.Series:
    """
    Given a row with 'Commodity' and 'Pct_Change', return a category and
    a natural-language explanation.

    Categories:
      - Stable         (|change| < 1%)
      - Small change   (1% <= |change| < 5%)
      - Moderate change(5% <= |change| < 20%)
      - Large change   (|change| >= 20%)
      - Unknown        (NaN or invalid)
    """
    p = row["Pct_Change"]
    name = row["Commodity"]

    if pd.isna(p):
        category = "Unknown"
        text = f"No valid change could be computed for {name}."
        return pd.Series([category, text])

    abs_p = abs(p)

    if abs_p < 1:
        category = "Stable"
        text = (
            f"The relative importance of {name} was essentially stable from 2023 to 2024, "
            f"changing by only {p:.2f}%."
        )
    elif abs_p < 5:
        category = "Small change"
        direction = "increased" if p > 0 else "decreased"
        text = (
            f"The relative importance of {name} {direction} slightly by {p:.2f}% between 2023 and 2024, "
            f"indicating only a minor shift in its weight in the overall PPI basket."
        )
    elif abs_p < 20:
        category = "Moderate change"
        direction = "increased" if p > 0 else "decreased"
        text = (
            f"{name} experienced a moderate {direction} of {p:.2f}% in relative importance from 2023 to 2024. "
            f"This suggests a noticeable shift, although not an extreme one."
        )
    else:
        category = "Large change"
        direction = "increase" if p > 0 else "decrease"
        text = (
            f"{name} shows a large {direction} in relative importance of {p:.2f}% from 2023 to 2024. "
            f"Because these values are relative-importance weights rather than prices, this likely reflects "
            f"changes in basket weighting or classification rather than pure price movement."
        )

    return pd.Series([category, text])


def analyze_ppi_relative_importance(input_path: Path, output_path: Path, sheet_name: str = "comrlp25") -> None:
    """
    End-to-end pipeline:
      - load Excel
      - clean
      - compute percent change
      - categorize and generate NLG interpretation
      - save result as Excel
    """
    print(f"Loading input file: {input_path}")
    df = load_relative_importance_excel(input_path, sheet_name=sheet_name)
    df = clean_relative_importance_df(df)
    df = compute_percent_change(df)

    # Apply interpretation function
    df[["Change_Category", "NLG_Interpretation"]] = df.apply(interpret_change_row, axis=1)

    # Sort by percent change (descending)
    df = df.sort_values("Pct_Change", ascending=False)

    # Ensure output directory exists
    output_path.parent.mkdir(parents=True, exist_ok=True)

    print(f"Saving analysis to: {output_path}")
    df.to_excel(output_path, index=False)

    # Print a quick preview to stdout
    print("\nTop 10 rows (largest increases):")
    print(df.head(10))


def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="PPI Relative Importance 2023–2024 Analysis"
    )
    parser.add_argument(
        "--input",
        type=str,
        required=True,
        help="Path to input ppi-comrlp.xlsx file.",
    )
    parser.add_argument(
        "--output",
        type=str,
        required=True,
        help="Path to output Excel file with analysis.",
    )
    parser.add_argument(
        "--sheet",
        type=str,
        default="comrlp25",
        help="Sheet name containing the relative-importance table (default: comrlp25).",
    )
    return parser.parse_args()


if __name__ == "__main__":
    args = parse_args()
    input_path = Path(args.input)
    output_path = Path(args.output)

    analyze_ppi_relative_importance(input_path=input_path, output_path=output_path, sheet_name=args.sheet)
```
