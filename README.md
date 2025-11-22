# Retail-Data-Processing-Hackathon
## Overview

Small end-to-end example that demonstrates a typical retail data pipeline:

1. **Phase 1 (Data Ingestion & Quality Validation / ETL)** — load raw CSVs, clean data (duplicates, dates, nulls, orphaned records), fix obvious data issues, and build a single `clean_master_data.csv` file at the line-item (most granular) level.
2. **Phase 3 (Analytics & Modeling)** — load the cleaned master file and run a set of analytics and simple models (inventory vs sales correlation, a Random Forest-based customer future-spend prediction, promotion/discount logic, stockout opportunity analysis, and an impact/revenue forecast).

> NOTE: The repository code uses two top-level functions: `clean_and_prep_data()` (Phase 1) and `run_analytics_logic()` (Phase 3). Phase 2 (if any) is not present in this snippet.

---

## File / Input expectations

The ETL function expects the following CSV files in the working directory (exact filenames):

* `stores.csv` — must contain at least `store_id`, `opening_date` (parsable as date).
* `products.csv` — must contain at least `product_id`, `unit_price`, `product_category`, `current_stock_level`.
* `promotion_details.csv` — must contain at least `promotion_id`, `discount_percent`.
* `customer_details.csv` — must contain at least `customer_id`, `loyalty_status`, `historic_annual_spend`.
* `store_sales_header.csv` — transaction header table; must contain `transaction_id`, `transaction_date`, `store_id`, `customer_id`.
* `store_sales_line_items.csv` — line-item table; must contain `transaction_id`, `product_id`, `quantity`, `promotion_id` (optional).

Outputs produced by the pipeline:

* `clean_master_data.csv` — merged, cleaned line-item-level dataset used by the analytics stage.
* Console logs — information about duplicates, future-dated transactions, bad promotions, negative quantities, model metrics, and quick recommendations.

---

## Quick setup & dependencies

Create a virtual environment and install the required Python packages (tested with Python 3.8+):

```bash
python -m venv venv
source venv/bin/activate    # macOS / Linux
venv\Scripts\activate      # Windows
pip install pandas numpy scikit-learn
```

---

## How to run

1. Place the required CSV files (see "File / Input expectations") in the same folder as the script.
2. Run the ETL function to produce `clean_master_data.csv`:

```bash
python your_script_name.py   # ensure the file containing clean_and_prep_data() is executed
```

3. Run the analytics & modeling stage (it reads `clean_master_data.csv` and `products.csv`):

```bash
python your_script_name.py   # ensure the file containing run_analytics_logic() is executed
```

**Important:** The provided snippet uses `if _name_ == "_main_":` which is a bug. Replace those occurrences with the correct check:

```python
if __name__ == "__main__":
    clean_and_prep_data()

# and later
if __name__ == "__main__":
    run_analytics_logic()
```

If both functions are in the same file, guard their execution appropriately (or call them from a small CLI). Example minimal runner at bottom of file:

```python
if __name__ == "__main__":
    clean_and_prep_data()
    run_analytics_logic()
```

---

## Key behaviors & assumptions

* Future-dated transactions are dropped; future store opening dates are set to today.
* Duplicate transactions (by `transaction_id`) are dropped (first kept).
* Line items with non-positive `quantity` are removed.
* Negative `unit_price` values in `products.csv` are converted to absolute values.
* Promotions that are present but invalid are nullified (keeps the sale record but removes the invalid promotion).
* The predictive target (`target_future_spend`) is a synthetic target derived from historic spend and observed totals; replace with a business target when available.

---

## Quick troubleshooting

* If the script prints `Hey, I can't find <file>`, ensure the CSV filename exactly matches (case sensitive on some OS) and is in the working directory.
* If modeling prints "Not enough data points...", you need at least 3 distinct customers in the cleaned data for the included train/test split.
* If you see parsing errors for dates, check date formats in `store_sales_header.csv` and `stores.csv`.

---

## Extending this project (ideas)

* Add unit tests and sample fixtures (small CSVs) for automated CI validation.
* Export model artifacts (pickle) and provide a scoring function / REST endpoint.
* Enhance feature engineering (recency, frequency, monetary, product affinity, seasonality).
* Add a Phase 2 that performs schema validation and column type enforcement with a library like `pandera`.

---

## License & Author

* Author: (your name)
* License: MIT (or your preferred license)

---

If you want, I can also produce a `requirements.txt`, a CLI wrapper, or convert this README into a `README.md` file in the repo for you.
