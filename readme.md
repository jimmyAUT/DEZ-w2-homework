# # DEZ_w2_homework

## Data Engineering Zoomcamp Week 2 homework

## NYC Taxi Data ETL Pipeline using Kestra

This project utilizes Kestra to build an ETL pipeline for processing NYC taxi data.

Overview
The pipeline automates the extraction, transformation, and loading (ETL) of raw NYC taxi trip data from the NYC Taxi Dataset Repository. The process includes downloading, decompressing, and uploading the data to a Google Cloud Storage (GCS) bucket before loading it into BigQuery for analysis.

    Workflow
        1. ForEach Loop Execution:
            - The ForEach task iterates over all Year-Month and Taxi Type combinations.
            - The lists defined in the Kestra flow

            ```yaml
            ["2019-01", "2019-02", "2019-03", ..., "2021-05", "2021-06", "2021-07"]

            ["yellow", "green"]
            ```
            - Each combination triggers a subflow that includes tasks for labeling, extraction, and uploading to GCS.
        2. Data Processing Tasks:
            - Set Labels: Assigns metadata labels for tracking.
            - Extract Data: Downloads and decompresses raw CSV files.
            - Upload to GCS: Stores extracted data in a cloud storage bucket.
        3. Conditional Processing:
            The if statement differentiates between "green_taxi" and "yellow_taxi" to apply specific transformations.
        4. BigQuery Data Load:
            - External Table on GCS: A BigQuery external table references the GCS files, allowing queries without occupying storage.
            - Staging Table: Temporary storage for raw data before final insertion.
            - Main Table Load: Data is loaded from staging to the main table, and the staging table is truncated after each task completion.

    * Note: 
    In Kestra, the parents array stores all parent task runs of the current task, ordered from the most recent parent at the front to the outermost parent. In your flow, changes in the index of parents will affect the behavior of conditional evaluations (e.g., if conditions) and logging (log.Log). Adjusting the index reference is crucial to ensure the correct task hierarchy and expected behavior.

    ```yaml
    tasks:
    - id: outer-loop
        type: io.kestra.plugin.core.flow.ForEach
        values: ["yellow", "green", "bule"]
        tasks:
        - id: inner-loop
            type: io.kestra.plugin.core.flow.ForEach
            values: ["2020-01", "2020-02", "2020-03"] 
            tasks:
            - id: if_yellow_taxi
                type: io.kestra.plugin.core.flow.If
                condition: "{{parents[0].taskrun.value == 'yellow'}}" 
                # if_yellow_taxi is a subtask of the inner loop, which directly takes over the taskrun.value of the outer loop(parents[0])
                then:
                - id: print_yellow_combination
                    type: io.kestra.plugin.core.log.Log
                    message: "{{parents[1].taskrun.value}} - {{ parents[0].taskrun.value }}"
                    # Now the parents[0] is becoming the inner-loop(year-month), parents[1] is outer-loop (taxi-type)
                    # yellow - 2020-01

            - id: if_green_taxi
                type: io.kestra.plugin.core.flow.If
                condition: "{{parents[0].taskrun.value == 'green'}}"
                then:
                - id: print_green_combination
                    type: io.kestra.plugin.core.log.Log
                    message: "{{parents[1].taskrun.value}} - {{ parents[0].taskrun.value }}"
    ```

Q1: Within the execution for Yellow Taxi data for the year 2020 and month 12: what is the uncompressed file size (i.e. the output file yellow_tripdata_2020-12.csv of the extract task)?
**Ans: 128.3MB**

Q2: What is the rendered value of the variable file when the inputs taxi is set to green, year is set to 2020, and month is set to 04 during execution?
**Ans: green_tripdata_2020-04.csv**

Q3: How many rows are there for the Yellow Taxi data for all CSV files in the year 2020?
**Ans: 24648499**

```sql
SELECT COUNT(unique_row_id) FROM `dez-jimmyh.w2_kestra_dataset.yellow_tripdata` 
WHERE filename LIKE "%2020-%" 
```

Q4: How many rows are there for the Green Taxi data for all CSV files in the year 2020?
**Ans: 1734051**

```sql
SELECT COUNT(unique_row_id) FROM `dez-jimmyh.w2_kestra_dataset.green_tripdata` 
WHERE filename LIKE "%2020-%" 
```

Q5: How many rows are there for the Yellow Taxi data for the March 2021 CSV file?
**Ans: 3007292**

```sql
SELECT COUNT(unique_row_id) FROM `dez-jimmyh.w2_kestra_dataset.yellow_tripdata` 
WHERE filename = "yellow_tripdata_2020-03" 
```

Q6: How would you configure the timezone to New York in a Schedule trigger?
**Ans: Add a timezone property set to America/New_York in the Schedule trigger configuration**

```yaml
triggers:
  - id: daily-trigger
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"  # daily UTC 09:00 trigger
    timezone: "America/New_York"  # set up the timezone
```