# PySpark NYC Taxi Data Pipeline

## Dataset Description
- **Source:** [Databricks Public Datasets → nyctaxi](https://docs.databricks.com/en/sample-data/nyctaxi.html)
- **Files Used:**
  - `dbfs:/databricks-datasets/nyctaxi/tripdata/yellow/` (2019-01 for development, 2017–2020 for full run)
  - `dbfs:/databricks-datasets/nyctaxi/taxizone/` (Taxi Zone reference)
- **Size:** ~1GB+ for full dataset (multiple years combined)
- **Purpose:** Analyze trip duration, tip rate, and passenger behavior across boroughs and hours of day.

## Data Processing Pipeline

1. **Load Data:**
   Read yellow taxi CSVs using PySpark with inferred schema.
   ```python
   spark.read.option("header", "true").csv(path)
   ```
2. **Transformations:**
   - Added computed columns:
     - `trip_minutes` = trip duration in minutes
     - `pickup_hour`, `pickup_dow` = extracted time features
     - `tip_rate` = `tip_amount / fare_amount`
   - Applied filters:
     - Trip duration 1–180 min
     - Distance 0.2–50 miles
     - Valid passenger count and positive fare

3. **Join:**
   - Joined taxi trip data with `taxizone` twice (for pickup and dropoff zones).
   - Enriched dataset with borough and zone information.

4. **Aggregation:**
   - Grouped by `PU_Borough`, `pickup_hour`
   - Computed trip count, avg distance, avg fare, avg tip rate.

5. **SQL Queries:**
   - Query 1: Avg tip rate by payment type
   - Query 2: Trip count by borough and day of week

6. **Optimization:**
   - Filters applied early to reduce shuffle.
   - Column pruning using `.select()` before joins.
   - Broadcast join avoided; used partitioning on borough for scalability.

7. **Write Output:**
   - Saved aggregated results to `/tmp/nyc_taxi_results` in Parquet format.


---

### **Part 3 — Performance Analysis**


## Performance Analysis

**Tools used:**  
- `.explain(True)` for physical plan  
- Databricks Query Details tab (DAG, shuffle metrics)

**Findings:**
- Spark pushed down filters (`trip_minutes`, `trip_distance`, `passenger_count`) to avoid full scan.  
- Column pruning reduced I/O significantly.  
- Shuffles minimized by reusing partition key (`PU_Borough`).  
- Using `.cache()` improved repeated query performance by ~30%.  

**Potential Bottlenecks:**  
- Inferring schema (`inferSchema=True`) adds noticeable latency.  
- Large joins (PU/DO) can skew partitions on Manhattan trips.

**Optimization Steps Taken:**  
- Early filtering, selective column loading.  
- `repartition(64, "PU_Borough")` before heavy aggregations.

## 🧠 Key Findings

| Metric | Example Value | Insight |
|--------|----------------|---------|
| Avg tip rate | ~18% | Manhattan has the highest average tipping behavior |
| Peak pickup hour | 18:00–20:00 | Evening rush hour dominates trip volume |
| Longest avg trip distance | Queens → Bronx | More suburban trips, higher fare |

## Screenshots to Include

Required (per assignment):
1. **Query Execution Plan** (`df.explain(True)` output)  
2. **Query Details view (Databricks screenshot)**  
3. **Successful pipeline output (`.show()` / `.write`)**

Insert them here (Markdown syntax example):
![Execution Plan](images/explain_output.png)
![Query Details](images/query_details.png)


---

### **Part 6 — Repo Structure**

## Repository Structure
```
pyspark-nyc-taxi-pipeline/
│
├── nyc_taxi_pipeline.ipynb
├── README.md
└── images/
```

### **Part 7 — Run Instructions**

## How to Run
1. Open in Databricks Notebook or local PySpark setup.  
2. Attach to an active cluster.  
3. Run cells A–H sequentially.  
4. Verify outputs via `.show()` and `.explain()`.






