### Project Title
**Buluttan Data Engineer Intern Technical Assessment**

### Overview
This project demonstrates a **data engineering pipeline** that:

1. **Extracts** weather data (hourly temperature observations) for **2023 and 2024** from Environment Canada’s bulk_data_e endpoint (for stations **26953** and **31688**).
2. **Performs** data quality checks (nulls, outliers, missing days).
3. **Stores** the raw data in CSV format under the **raw_data/** folder.
4. **Transforms** the data by loading it into **SQLite**:
   - Cleans invalid rows (e.g., null or out-of-range temps).
   - Aggregates monthly stats (avg, min, max).
   - Computes year-over-year (yoy) average temperature deltas.
   - Joins with geonames.csv to fetch **feature_id** and **map**.
   - Uses **external SQL files** (in a sql/ folder) to run queries.
   - Logs progress with **logging** instead of print statements.
5. **Outputs** the final monthly table to:
   - **final_output.csv** (and also .json and .parquet)
   - Logs quality issues into **transform_data_quality.xlsx** (and also .json and .parquet)

### Data Flow / Architecture

#### 1. ingest.py
- Iterates over station IDs `[26953, 31688]`, years `[2023, 2024]`, and months `[1..12]`.
- Builds a dynamic URL (e.g. https://climate.weather.gc.ca/...).
- Downloads each CSV, saves them unmodified to **raw_data/**.
- Logs progress via **logging** messages.

#### 2. transform.py
- Creates (or resets) a local SQLite DB named **weather_db.sqlite**.
- Loads **geonames.csv** into **geonames_dim**.
- Loads all **weather_station_*.csv** from **raw_data/** into **weather_raw**.
- Reads and executes SQL scripts (`create_tables.sql`, `monthly_agg.sql`, `yoy_diff.sql`, `final_data.sql`) from a **sql/** folder.
- Checks for nulls, outliers, and missing coverage (via SQL). Records these issues in memory and finally writes them out in multiple formats (Excel, JSON, Parquet).
- Cleans the data by removing invalid rows (bad year, out-of-range temp).
- Aggregates monthly stats in `monthly_agg`, calculates YOY difference in `monthly_agg_yoy`.
- Joins geonames_dim to produce `final_data`.
- Writes `final_output.csv` (plus .json and .parquet).
- Writes all data quality issues to **transform_data_quality.xlsx**, plus JSON and Parquet equivalents.
- Uses **logging** for status and error messages.

### Setup & Requirements
- **Python 3.8+**

Install dependencies:
```bash
pip install -r requirements.txt
```
Which may include:
- `requests`
- `openpyxl`
- `pandas`, `pyarrow` (for JSON/Parquet output)
- `pytest` (for testing)
- `logging` (built-in, no separate install)

Make sure to run scripts from the project root so they can find `raw_data/`, `sql/`, and `geonames.csv`.

### Usage

1. **Clone** or download this repository.
2. **Run ingestion**:
   ```bash
   python ingest.py
   ```
   - Creates `raw_data/` folder, fetches monthly CSV for 26953 & 31688 (2023–2024).
   - Logs progress.

3. **Run transformation**:
   ```bash
   python transform.py
   ```
   - Builds `weather_db.sqlite`.
   - Executes external SQL scripts.
   - Logs data quality issues to `transform_data_quality.*` (Excel, JSON, Parquet).
   - Writes final monthly aggregates to `final_output.*` (CSV, JSON, Parquet).

4. **Check**:
   - `final_output.csv` → aggregated monthly table with YOY differences.
   - `transform_data_quality.xlsx` → data issues found.
   - Additional formats: `final_output.json`, `final_output.parquet`, `transform_data_quality.json`, `transform_data_quality.parquet`.
   - `raw_data/` → raw CSV files.

### Data Quality Checks
- **Null Checks**: Finds rows where `temp_c` is null or year/month is missing.  
- **Outlier Checks**: Flags if `temp_c` < -50 or `temp_c` > 50.  
- **Missing Days**: Coverage <80% calculated as `(row_count / (days_in_month * 24)) < 0.8`.

### Testing
There is a `test_transform.py`:
```bash
pytest
```
- Basic tests for transformation and data integrity.

### Known Limitations
- If Environment Canada’s CSV has no temperature data, expect “NO_VALID_TEMP” or “NULL_VALUES” warnings.
- The join to geonames.csv may fail if lat/long don't match exactly → `feature_id`/`map` might be null.
- YOY difference only works if prior year’s data is available.

### Future Enhancements
- **Incremental ingestion** (fetch only new data).
- **Partitioned raw data** (e.g. `raw_data/2023/01/station_31688.csv`).
- **Logging to file** instead of only console.
- **Orchestration** with Airflow or Luigi.
- **Expanded testing** for more robust coverage.

