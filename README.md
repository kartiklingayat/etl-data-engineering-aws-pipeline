📦 ETL Data Engineering on AWS — etl-data-engineering-aws-pipeline
🚀 End-to-End ETL Pipeline using AWS Glue, PySpark, Amazon S3 & Amazon Redshift

This repository showcases a complete, production-style ETL (Extract → Transform → Load) Data Engineering Pipeline leveraging modern AWS cloud services.

🔧 Technologies Used

Apache Spark (PySpark)

AWS Glue

Amazon S3

AWS Glue Data Catalog

Amazon Redshift

🔄 Pipeline Workflow

The project demonstrates the full data lifecycle:

➡️ Extract → 🔁 Transform → 📤 Load → 🔍 Query → 📊 Analyze

⚠️ Important Note
Running AWS Glue, Redshift Clusters, and Crawlers may generate cost.
Always delete Glue Jobs, Crawlers, and Redshift Clusters after testing.

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

![extra](https://user-images.githubusercontent.com/63555029/228253564-6e65992a-1c0c-4f53-be04-aabec063a6f1.png)

---

## **2.1 Create a Bucket in Amazon S3**

AWS S3 → Create Bucket
Bucket Name: **etl-project-for-medium**

![s3\_bucket](https://user-images.githubusercontent.com/63555029/228254010-97443b14-b3d3-460c-b71f-e989b9c0d8d0.png)

---

## **2.2 Create Database Folder**

AWS S3 → `etl-project-for-medium` → Create Folder
Folder Name: **etl-project-for-medium-database**

![folder](https://user-images.githubusercontent.com/63555029/228255263-cfd3e59b-70dc-402e-9e00-19900116e586.png)

---

## **2.3 Create raw_data and transformed_data folders**

Inside `etl-project-for-medium-database/` create:

* raw_data
* transformed_data

![folder2](https://user-images.githubusercontent.com/63555029/228255709-5f5314ac-807b-4273-8158-67033dbcbe46.png)

---

## **2.4 Upload Data to raw_data**

S3 → raw_data → Upload `marketing_campaign.csv`

![upload](https://user-images.githubusercontent.com/63555029/228255897-2cffbb16-4c32-4cf8-a0c8-65a907f563b9.png)

---

## **3.1 Create a Glue Database**

AWS Glue → Data Catalog → Databases → Add Database
Database Name: **etl-project-for-medium-database**

![db](https://user-images.githubusercontent.com/63555029/228256298-65829739-c071-4207-814d-dfd569e0a74e.png)

---

## **3.2 Create Glue Crawler**

Crawler Name: **etl-project-for-medium-crawler**
Source: raw_data folder
IAM Role: **IAM-Role-etl-project**
Target DB: **etl-project-for-medium-database**
Schedule: On-demand

![crawler](https://user-images.githubusercontent.com/63555029/228259725-eaa8a949-6345-4f20-bdc5-058e4676de8f.png)

---

## **3.3 Run the Crawler**

![c1](https://user-images.githubusercontent.com/63555029/228256629-e504361a-a655-4072-a918-8442a7d3d11f.png)

---

## **3.4 Table & Schema Created Successfully**

![c2](https://user-images.githubusercontent.com/63555029/228256846-6b620a1a-33ac-4edf-8276-d0a6b6faf950.png)

---

# 🟦 **PART 2 — TRANSFORM**

![Slide1\_transform](https://github.com/askintamanli/Data-Engineer-ETL-Project-Using-Spark-with-AWS-Glue/assets/63555029/8132c39b-994d-4b05-be82-0af06b6b23ae)

---

## **4. Create AWS Glue ETL Job**

AWS Glue → Data Integration → Interactive Sessions → Notebooks
Job Name: **etl-project-for-medium-job**
IAM Role: **IAM-Role-etl-project**
Kernel: Spark

![job](https://user-images.githubusercontent.com/63555029/228258375-5680b1be-1b76-4eb6-b00e-bce0ed3b711f.png)

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

![transformed](https://user-images.githubusercontent.com/63555029/228259450-80d283e1-a6b2-406a-b150-b15dbec04de2.png)

---

# 🟥 **PART 3 — LOAD**

![Slide1\_load](https://user-images.githubusercontent.com/63555029/228977183-c3091fb1-6e57-4608-bf88-d24807af46bd.jpg)

---

## **6.1 Create IAM Role for Redshift**

Service: Redshift
Policy: AdministratorAccess
Role Name: **IAM-Role-etl-project-redshift**

![r1](https://user-images.githubusercontent.com/63555029/228977738-f61c5f3b-bc19-4c4d-9a50-869e305646f3.png)

---

## **6.2 Create Redshift Cluster**

Cluster Name: **etl-project-cluster**
Node Type: dc2.large
Nodes: 1
IAM Role: **IAM-Role-etl-project-redshift**

![r2](https://user-images.githubusercontent.com/63555029/228977775-6261a957-da08-4041-9317-e84476210d5d.png)

---

## **6.3 Open Redshift Query Editor**

![r3](https://user-images.githubusercontent.com/63555029/228977819-75df9364-b1da-47ad-a744-ead14f27b940.png)

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

![sql\_create](https://user-images.githubusercontent.com/63555029/228978528-c2c266b4-1183-453d-a213-1a2fa31dddd5.png)

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

![sql\_output](https://user-images.githubusercontent.com/63555029/228979014-87d30860-754e-4e6e-937a-029d326324e2.png)

---

## **7.4 Analytics Query**

```sql
SELECT education, COUNT(id), AVG(income)
FROM etl_project_transformed_data_table
GROUP BY education;
```

![analytics](https://user-images.githubusercontent.com/63555029/228979117-7e85568a-9ff8-443c-8e2f-930d5de922fd.png)

---

# 🎉 **Project Completed**

Thank you for following this ETL project.
This covers the complete workflow from **Extract → Transform → Load → Query → Analyze** on AWS.

---

# ⭐ **If this helped you, please star the repo!**

---

## भाऊ, हे READY आहे — direct GitHub मध्ये paste कर.

काही हवं असेल तर मला सांग, मी लगेच तयार करून देतो.
