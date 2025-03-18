# Data Pipelines with Apache Airflow  

## Project Overview  
Sparkify, a music streaming company, is enhancing its **ETL pipeline automation and monitoring** by implementing **Apache Airflow**. This project develops **scalable, reusable, and monitored** data pipelines that allow **backfilling** and ensure **data integrity** through validation tests post-ETL.  

The raw data, stored in **Amazon S3**, contains **user activity logs and song metadata in JSON format**. The pipeline extracts, transforms, and loads this data into **Amazon Redshift** for further analysis.  

---

## ETL Workflow (DAG)  
The DAG follows these structured steps:  

1. **Begin Execution** – Initializes the pipeline and creates required tables if not already present.  
2. **Stage Events & Stage Songs** – Loads **user activity logs** and **song metadata** from **S3 to Redshift**.  
3. **Load Fact Table (Songplays)** – Transfers relevant data from staging tables into the **fact table**.  
4. **Load Dimension Tables** – Populates **users, artists, songs, and time** dimension tables using a common operator.  
5. **Run Data Quality Checks** – Verifies data integrity with SQL-based validation rules.  
6. **Stop Execution** – Marks the successful completion of the DAG.  

![ETL Pipeline DAG](images/example-dag.png)  

---

## Operators & Their Functions  

### **1. Stage Operator**  
Handles data ingestion from **S3 to Redshift** using SQL’s `COPY` command.  
**Parameters:**  
- `redshift_conn_id` – Redshift connection details.  
- `aws_credentials_id` – AWS S3 access credentials.  
- `table` – Destination table in Redshift.  
- `s3_bucket` & `s3_key` – Location of source files.  
- `region` – AWS region for S3 storage.  
- `json` – Formatting options for JSON data.  

**File:** `plugins/operators/stage_redshift.py`  

### **2. Fact Table Operator**  
Transfers processed data from staging tables into the **songplays fact table**.  
**Parameters:**  
- `redshift_conn_id` – Redshift connection details.  
- `sql_statement` – SQL query for data insertion.  

**File:** `plugins/operators/load_fact.py`  

### **3. Dimension Table Operator**  
Loads **dimension tables** using the same operator for efficiency.  
**Parameters:**  
- `redshift_conn_id` – Redshift connection details.  
- `table` – Target dimension table.  
- `sql_statement` – SQL query for data extraction.  
- `truncate-insert` – Determines if the table should be emptied before insertion.  

**File:** `plugins/operators/load_dimension.py`  

### **4. Data Quality Operator**  
Performs validation checks on tables post-ETL execution.  
**Parameters:**  
- `redshift_conn_id` – Redshift connection details.  
- `retries` – Number of retry attempts before failure.  
- `tables` – Dictionary mapping table names to validation tests.  

**File:** `plugins/operators/data_quality.py`  

---

## **Setup Requirements**  
- **Apache Airflow 1.10.2**  
- **Amazon Redshift Cluster**  

### **Airflow Configuration**  

#### **1. AWS Credentials Setup**  
Navigate to **Admin > Connections** in Airflow UI and configure AWS access:  
- **Conn Id:** `aws_credentials`  
- **Conn Type:** `Amazon Web Services`  
- **Login & Password:** Use AWS IAM access keys.  

![AWS Credentials](images/connection-aws-credentials.png)  

#### **2. Redshift Connection**  
Configure Redshift connection in Airflow:  
- **Conn Id:** `redshift`  
- **Conn Type:** `Postgres`  
- **Host:** Redshift cluster endpoint (excluding port).  
- **Schema:** `dev`  
- **Port:** `5439`  

![Redshift Connection](images/connection-redshift.png)  

---

## **Running the DAG**  
1. Upload the necessary files to the **Udacity workspace**.  
2. Start Airflow by running:  
   ```sh
   /opt/airflow/start.sh
