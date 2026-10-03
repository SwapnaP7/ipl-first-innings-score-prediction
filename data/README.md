# Dataset

The notebook expects the file `data/ipl_dataset.csv` (relative to the repository root). It is **not included** in this repository because the original source and licence of the file used in this project could not be verified, so redistribution rights are unknown.

## What is required

A ball-by-ball IPL first-innings dataset in CSV format with exactly these columns:

`mid`, `date`, `venue`, `batting_team`, `bowling_team`, `batsman`, `bowler`, `runs`, `wickets`, `overs`, `runs_last_5`, `wickets_last_5`, `striker`, `non-striker`, `total`

- `mid` is the match identifier, `total` is the final first-innings score (the target), and `overs` is in cricket notation (for example `5.1`; the digit after the point is 0-6).
- The reported results use a file with 76,014 rows covering 617 matches (2008-04-18 to 2017-05-21).

## How to use it

1. Obtain a dataset with these columns from a source whose terms allow your use.
2. Save it as `data/ipl_dataset.csv`.
3. Open `notebooks/ipl_first_innings_score_prediction.ipynb` and run all cells.

The notebook checks the column names and stops with an error if they differ. Do not commit the CSV to a public repository unless its terms permit redistribution.
