# Cleaning a Deliberately Messy Dataset — Manual + Automated

A 500+ row student dataset with layered, real-world-style data quality issues, cleaned two ways: by hand in LibreOffice Calc, then reproduced through **AutoDS**, a cleaning/EDA/ML tool I built myself.

## The problems in the raw data

This wasn't a simple "drop the nulls" exercise — the file had several independent failure modes stacked on top of each other, each requiring a different diagnostic approach:

- **Inconsistent delimiters within a single file** — the first ~100 rows used `|`, later rows switched to `,`, so a naive import silently collapsed entire records into one column
- **Corrupted character encoding** (cp1250, not UTF-8) — garbled currency symbols that looked like stray `?` characters, which turned out to not even be the literal "?" key when parsed programmatically, breaking naive string-matching formulas
- **Concatenated duplicate records** — some rows contained two full student entries mashed into a single line, which broke pandas' CSV parser outright (`ParserError: Expected 15 fields, saw 22`) until I wrote detection logic to isolate and split them
- **Colliding primary keys** — multiple different people sharing the same Student_ID, requiring manual judgment calls (not blanket renumbering) to resolve correctly
- **Mixed currencies with a hidden unit error** — payments mixed $ and £, and a subset of values turned out to be off by exactly 100x (a cents-vs-whole-unit entry bug) — caught only by comparing outliers against the known-good distribution, not by any single formula
- **Three different date formats in one column**, including an ambiguous DD-MM-YY numeric format that `DATEVALUE()` silently misparsed on some rows (flipping day/month) — required building explicit per-format parsers instead of trusting auto-detection

## Two approaches, compared

**Manual (LibreOffice Calc):** every fix required first diagnosing *why* a row broke, then building a formula specific to that failure mode — `EXACT()` to catch lost surnames, `REGEX()` to strip unknown symbols regardless of what they actually were, custom `DATE()` builders per format.

**Automated (AutoDS):** ran the same raw file through a cleaning/EDA/ML app I built — automated type-fixing and a chat-style assistant (`"replace Web Developmen with Web Development in course"`) applying the same category of fixes programmatically, then auto-generating EDA visualizations and training a model (Stacking Ensemble, 76% accuracy) to predict course enrollment from the cleaned data.

## Repo structure

```
student-data-cleaning/
├── README.md
├── manual-cleaning/
│   └── cleaned_dataset.csv          # output of the Calc cleaning pass
├── autods-pipeline/
│   ├── Unclean_Dataset_1_autods_notebook.ipynb
│   ├── original_data.csv
│   ├── cleaned_data.csv
│   └── autods_cleaning_tools/       # clean.py, clean_chat.py, nlp.py, assistant.py
└── screenshots/                     # EDA charts, model leaderboard
```

## Takeaway

Manual cleaning forces you to understand *why* data breaks. Automating it forces you to generalize that understanding into reusable rules. Doing both on the same dataset was more valuable than either alone.
