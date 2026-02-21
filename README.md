# 🚀 ETL Data Engineering Pipeline on AWS

This project demonstrates a complete **end-to-end ETL Data Engineering Pipeline**
built using **AWS Glue, PySpark, Amazon S3, Glue Data Catalog, and Amazon Redshift**.

The goal of this project is to show how raw data is:
**Extracted → Transformed → Loaded → Queried → Analyzed**  
using a fully serverless and scalable cloud architecture.

---

# 🏗️ Architecture Overview
(Replace this with your actual screenshot)

![architecture](screenshots/architecture.png)

---

# 📁 Project Structure


etl-data-engineering-aws-pipeline/
│── data/
│ └── marketing_campaign.csv
│
│── notebooks/
│ └── etl-project-transform-data.ipynb
│
│── src/
│ └── pyspark_etl_script.py
│
│── screenshots/
│ ├── architecture.png
│ ├── extract.png
│ ├── transform.png
│ ├── load.png
│
│── README.md
│── .gitignore
│── .gitattributes


---

# 🔄 ETL Pipeline Stages  
**Extract → Transform → Load**

---

# 🟧 PART 1 — EXTRACT  
_Load Raw Data into AWS S3_

![extract](screenshots/extract.png)

---

## ✔ 1. Create IAM Role for AWS Glue

AWS Console → IAM → Roles → Create Role  
Service: **Glue**  
Permissions: **AdministratorAccess**

**Role Name:**

IAM-Role-etl-project


---

## ✔ 2. Create S3 Bucket & Folders

Bucket Name:

etl-project-for-medium


Inside folder structure:

etl-project-for-medium-database/
├── raw_data/
└── transformed_data/


Upload dataset:

marketing_campaign.csv


---

## ✔ 3. Create Glue Database & Table (using Crawler)

### 3.1 Create Database
AWS Glue → Data Catalog → Databases → Add Database  

etl-project-for-medium-database


### 3.2 Create Glue Crawler
Crawler Name:

etl-project-for-medium-crawler

Source: raw_data folder  
IAM Role: `IAM-Role-etl-project`  
Target DB: `etl-project-for-medium-database`

Run crawler → table created.

---

# 🟦 PART 2 — TRANSFORM  
_Transform data using PySpark on AWS Glue_

![transform](screenshots/transform.png)

---

## ✔ 4. Create AWS Glue Interactive Notebook

Job Name:

etl-project-for-medium-job


IAM Role: `IAM-Role-etl-project`  
Kernel: Spark  
Workers: 5  
Worker Type: G.1X

---

# 🧪 5. PySpark Code (No Changes Done)

## ▶ 5.1 Initialize Session
```python
%idle_timeout 2880
%glue_version 3.0
%worker_type G.1X
%number_of_workers 5

import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job

sc = SparkContext.getOrCreate()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
▶ 5.2 Load Data From Glue Catalog
dyf = glueContext.create_dynamic_frame.from_catalog(
    database='etl-project-for-medium-database',
    table_name='raw_data'
)
dyf.printSchema()
▶ 5.3 Convert to DataFrame
df = dyf.toDF()
df.show()
▶ 5.4 Select Required Columns
df = df["id","year_birth","education","marital_status","income","dt_customer"]
df.show()
▶ 5.5 Check NULL Values
from pyspark.sql.functions import *
df.select([count(when(col(c).isNull(),c)).alias(c) for c in df.columns]).show()
▶ 5.6 Fill NULL Income with Mean
mean_value = df.select(mean(col('income'))).collect()[0][0]
df = df.fillna(mean_value, subset=['income'])
df.select([count(when(col(c).isNull(),c)).alias(c) for c in df.columns]).show()
▶ 5.7 Save Transformed Data (CSV)
df.write \
  .format("csv") \
  .mode("append") \
  .option("header", "true") \
  .save("s3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/")
▶ 5.8 Save Transformed Data (JSON)
df.write \
 .format("json") \
 .mode("append") \
 .save("s3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/")
🟩 PART 3 — LOAD

Load transformed data into Amazon Redshift

✔ 6. Create IAM Role for Redshift

Service: Redshift
Permission: AdministratorAccess

Role Name:

IAM-Role-etl-project-redshift
✔ 7. Create Amazon Redshift Cluster

Cluster ID:

etl-project-cluster

Node type: dc2.large
Nodes: 1
Attach IAM Role: IAM-Role-etl-project-redshift

🟥 7 — Load Data into Redshift
▶ 7.1 Create Table
CREATE TABLE etl_project_transformed_data_table(
"id" INTEGER NULL,
"year_birth" INTEGER NULL,
"education" VARCHAR NULL,
"marital_status" VARCHAR NULL,
"income" INTEGER NULL,
"dt_customer" DATE NULL
) ENCODE AUTO;
▶ 7.2 COPY Data from S3 into Redshift
COPY etl_project_transformed_data_table
FROM 's3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/part-00000-6429f588-c5f4-4f6e-88df-b8bd3506113e-c000.csv'
IAM_ROLE 'arn:aws:iam::835769464848:role/IAM-Role-etl-project-redshift'
IGNOREHEADER 1
DELIMITER ',';
▶ 7.3 Verify Table
SELECT * FROM etl_project_transformed_data_table;
▶ 7.4 Analytics Query
SELECT education, COUNT(id), AVG(income)
FROM etl_project_transformed_data_table
GROUP BY education;
