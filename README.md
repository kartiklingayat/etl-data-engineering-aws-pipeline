# 📦 **etl-data-engineering-aws-pipeline**
## 🚀 **End-to-End ETL Pipeline using AWS Glue, PySpark, Amazon S3 & Amazon Redshift**

This repository provides a fully implemented, production-style **ETL (Extract → Transform → Load) Data Engineering Pipeline** built using modern AWS cloud services.

It demonstrates how data moves through the pipeline using:

- **Apache Spark (PySpark)**
- **AWS Glue**
- **Amazon S3**
- **AWS Glue Data Catalog**
- **Amazon Redshift**

---

## 🔄 **Pipeline Lifecycle**
### **Extract → Transform → Load → Query → Analyze**

This workflow shows how raw data is ingested, processed, cleaned, transformed, stored, and finally queried for analytics.

---

![export](https://github.com/askintamanli/Data-Engineer-ETL-Project-Using-Spark-with-AWS-Glue/assets/63555029/b866df61-9b20-47db-a648-83fbb24e1974)

---

# 📌 **What we will do (Step-by-Step)**

1. Create IAM Role for the project
2. Create an S3 Bucket and upload data
3. Create AWS Glue Database & Table
4. Create Glue Studio Notebook
5. Transform data using PySpark
6. Create Amazon Redshift Cluster
7. Load transformed data into Redshift

---

# 🟧 **PART 1 — EXTRACT**

![Slide1\_extract](https://github.com/askintamanli/Data-Engineer-ETL-Project-Using-Spark-with-AWS-Glue/assets/63555029/d8aea115-9c32-42e7-bf9a-ffebf47e0230)

---

## **1.1 Create IAM Role for AWS Glue**

AWS Console → IAM → Roles → Create Role
Service: **Glue**
Policy: **AdministratorAccess**
Role Name: **IAM-Role-etl-project**

<img width="1296" height="129" alt="228253564-6e65992a-1c0c-4f53-be04-aabec063a6f1" src="https://github.com/user-attachments/assets/92bf7db7-722f-466f-b272-d0ffe8cda5a7" />


---

## **2.1 Create a Bucket in Amazon S3**

AWS S3 → Create Bucket
Bucket Name: **etl-project-for-medium**


---

## **2.2 Create Database Folder**

AWS S3 → `etl-project-for-medium` → Create Folder
Folder Name: **etl-project-for-medium-database**



---

## **2.3 Create raw_data and transformed_data folders**

Inside `etl-project-for-medium-database/` create:

* raw_data
* transformed_data

<img width="1543" height="161" alt="228255709-5f5314ac-807b-4273-8158-67033dbcbe46" src="https://github.com/user-attachments/assets/592d39b1-cbba-4f10-a882-750cb77bc632" />


---

## **2.4 Upload Data to raw_data**

S3 → raw_data → Upload `marketing_campaign.csv`

<img width="1491" height="155" alt="228255897-2cffbb16-4c32-4cf8-a0c8-65a907f563b9" src="https://github.com/user-attachments/assets/78288ec9-2f72-43ae-899f-c2c92030124e" />


---

## **3.1 Create a Glue Database**

AWS Glue → Data Catalog → Databases → Add Database
Database Name: **etl-project-for-medium-database**


---

## **3.2 Create Glue Crawler**

Crawler Name: **etl-project-for-medium-crawler**
Source: raw_data folder
IAM Role: **IAM-Role-etl-project**
Target DB: **etl-project-for-medium-database**
Schedule: On-demand


---

## **3.3 Run the Crawler**



---

## **3.4 Table & Schema Created Successfully**

<img width="1250" height="777" alt="228256846-6b620a1a-33ac-4edf-8276-d0a6b6faf950" src="https://github.com/user-attachments/assets/0813b1a8-de0e-47fb-b6cb-a3be8397197c" />


---

# 🟦 **PART 2 — TRANSFORM**

![Slide1\_transform](https://github.com/askintamanli/Data-Engineer-ETL-Project-Using-Spark-with-AWS-Glue/assets/63555029/8132c39b-994d-4b05-be82-0af06b6b23ae)

---

## **4. Create AWS Glue ETL Job**

AWS Glue → Data Integration → Interactive Sessions → Notebooks
Job Name: **etl-project-for-medium-job**
IAM Role: **IAM-Role-etl-project**
Kernel: Spark



---

# 🧪 **5. PySpark Code (Original Code — NO CHANGES)**

---

## **5.1 Initialize Session**

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
```

---

## **5.2 Create DynamicFrame**

```python
dyf = glueContext.create_dynamic_frame.from_catalog(
    database='etl-project-for-medium-database',
    table_name='raw_data'
)
dyf.printSchema()
```

---

## **5.3 Convert to DataFrame**

```python
df = dyf.toDF()
df.show()
```

---

## **5.4 Drop Unnecessary Columns**

```python
df = df["id","year_birth","education","marital_status","income","dt_customer"]
df.show()
```

---

## **5.5 Check Null Values**

```python
from pyspark.sql.functions import *
df.select([count(when(col(c).isNull(),c)).alias(c) for c in df.columns]).show()
```

---

## **5.6 Fill Missing Values**

```python
mean_value = df.select(mean(col('income'))).collect()[0][0]
df = df.fillna(mean_value, subset=['income'])
df.select([count(when(col(c).isNull(),c)).alias(c) for c in df.columns]).show()
```

---

## **5.7 Save CSV to S3**

```python
df.write \
  .format("csv") \
  .mode("append") \
  .option("header", "true") \
  .save("s3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/")
```

---

## **5.8 Save JSON to S3**

```python
df.write \
  .format("json") \
  .mode("append") \
  .save("s3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/")
```

---

## **5.9 Check Transformed Bucket**



---

# 🟥 **PART 3 — LOAD**

![Slide1\_load](https://user-images.githubusercontent.com/63555029/228977183-c3091fb1-6e57-4608-bf88-d24807af46bd.jpg)

---

## **6.1 Create IAM Role for Redshift**

Service: Redshift
Policy: AdministratorAccess
Role Name: **IAM-Role-etl-project-redshift**



---

## **6.2 Create Redshift Cluster**

Cluster Name: **etl-project-cluster**
Node Type: dc2.large
Nodes: 1
IAM Role: **IAM-Role-etl-project-redshift**



---

## **6.3 Open Redshift Query Editor**



---

# 📝 **7. Load Transformed Data into Redshift**

---

## **7.1 Create Table**

```sql
CREATE TABLE etl_project_transformed_data_table(
"id" INTEGER NULL,
"year_birth" INTEGER NULL,
"education" VARCHAR NULL,
"marital_status" VARCHAR NULL,
"income" INTEGER NULL,
"dt_customer" DATE NULL
) ENCODE AUTO;
```

<img width="1578" height="724" alt="228978528-c2c266b4-1183-453d-a213-1a2fa31dddd5" src="https://github.com/user-attachments/assets/7a315c92-4b42-4014-b790-0cabab1690a0" />


---

## **7.2 COPY Data from S3**

```sql
COPY etl_project_transformed_data_table
FROM 's3://etl-project-for-medium/etl-project-for-medium-database/transformed_data/part-00000-6429f588-c5f4-4f6e-88df-b8bd3506113e-c000.csv'
IAM_ROLE 'arn:aws:iam::835769464848:role/IAM-Role-etl-project-redshift'
IGNOREHEADER 1
DELIMITER ',';
```

---

## **7.3 Validate Data**

```sql
SELECT * FROM etl_project_transformed_data_table;
```

<img width="1630" height="750" alt="228979014-87d30860-754e-4e6e-937a-029d326324e2" src="https://github.com/user-attachments/assets/376ca2d7-41e2-4550-864b-54f0866b260a" />


---

## **7.4 Analytics Query**

```sql
SELECT education, COUNT(id), AVG(income)
FROM etl_project_transformed_data_table
GROUP BY education;
```

![analytics](https://user-images.githubusercontent.com/63555029/228979117-7e85568a-9ff8-443c-8e2f-930d5de922fd.png)

---

