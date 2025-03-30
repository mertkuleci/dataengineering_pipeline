# Project Title
Buluttan Data Engineer Intern Technical Assessment

# Overview
This project demonstrates a data engineering pipeline that:

1. Extracts weather data (hourly temperature observations) for 2023 and 2024 from Environment Canada’s bulk_data_e endpoint (for two stations: 26953 and 31688).
2. Checks for data quality issues (nulls, outliers, missing days).
3. Stores the raw data in CSV format under the raw_data/ folder.
4. Transforms the data by loading it into SQLite:
  • Cleans invalid rows (e.g., null or out-of-range temps).
  • Aggregates monthly stats (average, min, max).
  • Computes year-over-year (yoy) average temperature deltas.
  • Joins with geonames.csv to fetch feature_id and map.

5. Outputs the final monthly table to a CSV (final_output.csv) and logs quality issues to an Excel file (transform_data_quality.xlsx).

# Data Flow / Architecture

## 1. ingest.py

  • Loops over station IDs [26953, 31688], years [2023, 2024], months [1..12].
  
  • Builds a dynamic URL (e.g. https://climate.weather.gc.ca/...).
  
  • Downloads each CSV and saves unmodified to raw_data/.

## 2. transform.py

  • Creates (or resets) a local SQLite DB (weather_db.sqlite).
  
  • Loads geonames.csv into geonames_dim.
  
  • Loads all weather_station_*.csv from raw_data/ into weather_raw.
  
  • Checks for nulls, outliers, missing coverage (via SQL). Records issues in data_quality_issues.
  
  • Cleans the data (removes rows with invalid year or temperature).
  
  • Aggregates monthly stats in monthly_agg, calculates YOY difference in monthly_agg_yoy.
  
  • Joins geonames_dim to produce final_data.
  
  • Exports final_data to final_output.csv.
  
  • Writes all data quality issues to transform_data_quality.xlsx.
  

# Setup & Requirements

Python 3.8+

pip install -r requirements.txt (if you have one; otherwise manually install):

  requests (for ingestion)
  openpyxl (for Excel)
  boto3 (if cloud upload to S3)
  pytest (if you want to run tests)

Ensure the script is run from the project root so it can find raw_data/ and geonames.csv.

# Usage
1. Clone or download this repository.

2. Run ingestion:
  python ingest.py

    This creates raw_data/ folder and saves monthly CSVs for station IDs 26953 and 31688, years 2023–2024.

3. Run transformation:
  python transform.py
    This creates/updates weather_db.sqlite, logs issues to transform_data_quality.xlsx, and writes final_output.csv.

4. Check the final results:
  final_output.csv → aggregated monthly table with yoy differences.
  transform_data_quality.xlsx → any data issues found.
  raw_data/ → raw downloads (unmodified).


# Data Quality Checks
  • Null Checks: Finds rows where temp_c is null or year/month is missing.
  • Outlier Checks: Flags if temp_c < -50 or temp_c > 50.
  • Missing Days: If actual row count < 80% of the expected (days_in_month * 24), it flags coverage issues.


# Testing
We have a tests/ folder, you can run:
  pytest

# Known Limitations
  • If the source CSV from Environment Canada is empty (no temperature columns), you’ll see many “NO_VALID_TEMP” or “NULL_VALUES” warnings.
  • The final join with geonames.csv may fail to match lat/lon exactly, leading to null feature_id or map.
  • YOY difference is only meaningful if data for the same month in the previous year exists.

# Future Enhancements
• Incremental ingestion (only fetch new days).
• Partitioned raw data (e.g. raw_data/2023/01/station_31688.csv).
• Logging to a file rather than Excel-based checks.
• Orchestration with Airflow or Luigi.
• Testing more components with unit tests.
