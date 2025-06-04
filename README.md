# AWS Data Pipeline for Product Review Analysis (Glue, PySpark, S3, Athena & Terraform)

This project demonstrates my ability to design, implement, and manage an end-to-end, scalable data processing pipeline on AWS. I engineered a solution to ingest, transform, catalog, and analyze Amazon product review and metadata JSON datasets (specifically for the "Toys & Games" category), making insights readily available via SQL queries. This project was developed to gain practical, hands-on experience with core AWS data services and infrastructure automation.

![Architecture Diagram](./images/your-architecture-diagram.png)
*(Ensure your architecture diagram, if you have one for this project, is in an `images` folder (or similar) and correctly linked. If not, you can omit this line or create a simple diagram.)*

## Key Achievements & Skills Demonstrated:

* **End-to-End Pipeline Development:** Successfully built a data pipeline that takes raw JSON data from an S3 landing zone, processes it through AWS Glue, stores it in an optimized format in a processed S3 zone, catalogs it, and enables querying via Amazon Athena.
* **Serverless ETL with PySpark & AWS Glue:**
    * Developed **AWS Glue ETL jobs using PySpark** to perform key transformations: schema definition, data type casting, cleaning (e.g., handling missing values or inconsistencies), and efficient conversion from JSON to Parquet format.
    * Engineered logic within these scripts to process both customer reviews and product metadata.
* **Data Lake Storage & Optimization on Amazon S3:**
    * Utilized Amazon S3 as the central data lake for storing raw, intermediate, and processed data.
    * Implemented **data partitioning strategies** (e.g., by year/month for reviews; by sales category for metadata) and **Snappy compression** for the Parquet files, significantly improving query performance in Athena and reducing storage costs.
* **Automated Data Cataloging & Schema Management:**
    * Deployed and configured an **AWS Glue Crawler** to automatically discover schemas from the processed Parquet data in S3 and populate the AWS Glue Data Catalog, enabling seamless data discovery and querying.
    * Programmatically created and managed the Glue Crawler using **Python (Boto3)**, demonstrating API interaction with AWS services.
* **Infrastructure as Code (IaC) with Terraform:**
    * Authored **Terraform configurations** from scratch to reliably provision and manage all AWS resources. This includes S3 buckets, Glue jobs, Glue crawlers, IAM roles with least-privilege permissions, and associated configurations.
    * This approach ensures reproducibility, version control, and efficient management of the cloud infrastructure.
* **Interactive Data Analysis & Insights with Amazon Athena:**
    * Enabled ad-hoc data analysis by making the processed data queryable via **Amazon Athena**.
    * Crafted SQL queries to join review and metadata tables and derive meaningful insights, such as identifying top-reviewed products, average ratings by brand, and top products by average rating with a significant number of reviews.
* **AWS Cloud Services Proficiency:** Demonstrated practical application and integration of key AWS services:
    * Amazon S3 (Data Lake Storage)
    * AWS Glue (ETL, Data Catalog, Crawlers)
    * Amazon Athena (Interactive Querying)
    * AWS IAM (Identity & Access Management)
* **Programming & Scripting:** Applied skills in:
    * Python (PySpark for ETL, Boto3 for AWS SDK interaction, AWSWrangler for Athena/Catalog operations)
    * SQL (for Athena queries and defining transformations)
    * HCL (HashiCorp Configuration Language for Terraform)
    * Shell scripting (for deployment automation tasks)

## Project Details:

### Data Sources:

This project utilizes datasets from the Amazon Customer Reviews Dataset, simulating real-world data scenarios:

* **Reviews Data:** `reviews_Toys_and_Games_5.json.gz` - Contains customer reviews including reviewer ID, product ASIN, review text, rating, etc.
* **Product Metadata:** `meta_Toys_and_Games.json.gz` - Contains product information such as ASIN, title, description, price, brand, and sales rank.

### Core Processing Steps:

1.  **Infrastructure Deployment:** All AWS resources are defined and deployed using Terraform.
2.  **Data Ingestion (Simulated):** Raw JSON datasets are uploaded to a designated S3 "landing" bucket.
3.  **ETL Transformation (AWS Glue & PySpark):**
    * Glue jobs, written in PySpark, are triggered.
    * These jobs read the raw JSON data, apply necessary transformations (cleaning, structuring, schema enforcement), partition the data, and write it to a "processed" S3 bucket in optimized Parquet format with Snappy compression.
4.  **Data Cataloging (AWS Glue Crawler):**
    * A Glue Crawler, created via Boto3 script, runs over the processed data in S3.
    * It infers the schema and creates/updates tables in the AWS Glue Data Catalog, making the data discoverable.
5.  **Data Analysis (Amazon Athena):**
    * Users can connect to Athena and query the newly cataloged tables using standard SQL to extract insights.

### Example Insights Queried via Athena:

This project enabled the extraction of various insights from the processed Amazon review and metadata. Below are examples of SQL queries run in Amazon Athena against the cataloged tables:

1.  **Top 5 Most Reviewed Products:** Identifies products with the highest number of distinct reviewers.

    ```sql
    SELECT
        met.title,
        COUNT(DISTINCT toy.reviewerid) AS review_count
    FROM toys_metadata met -- Assumes 'toys_metadata' is the table name in Athena for product metadata
    LEFT JOIN toys_reviews toy -- Assumes 'toys_reviews' is the table name in Athena for reviews
        ON met.asin = toy.asin
    WHERE met.title <> '' -- Excludes products without titles
    GROUP BY met.title
    ORDER BY review_count DESC
    LIMIT 5;
    ```

2.  **Top 10 Products by Average Rating (Minimum 1000 Reviews):** Lists top-rated products that have a substantial review base, ensuring statistical significance.

    ```sql
    SELECT
        met.title,
        -- Ensure 'met.sales_category' exists in your 'toys_metadata' table or remove/adjust this line
        -- met.sales_category, 
        AVG(toy.overall) AS average_rating
    FROM toys_metadata met
    LEFT JOIN toys_reviews toy
        ON met.asin = toy.asin
    GROUP BY met.title -- Add 'met.sales_category' here if you include it in the SELECT
    HAVING COUNT(DISTINCT toy.reviewerid) > 1000 -- Filters for products with more than 1000 unique reviews
    ORDER BY average_rating DESC
    LIMIT 10;
    ```

3.  **Average Rating and Number of Products by Brand:** Provides an overview of brand performance and product count.

    ```sql
    SELECT
        met.brand,
        COUNT(DISTINCT met.asin) AS product_count,
        AVG(toy.overall) AS average_rating
    FROM toys_metadata met
    LEFT JOIN toys_reviews toy
        ON met.asin = toy.asin
    WHERE met.brand <> '' -- Excludes entries without a brand name
    GROUP BY met.brand
    ORDER BY product_count DESC -- Orders by the number of products per brand
    LIMIT 10;
    ```


## Technologies Used:

* **Cloud Platform:** AWS
    * **Storage:** Amazon S3
    * **ETL:** AWS Glue (PySpark Jobs, Crawlers, Data Catalog)
    * **Querying:** Amazon Athena
    * **Identity & Access Management:** AWS IAM
* **Infrastructure as Code:** Terraform
* **Programming Languages:** Python (PySpark, Boto3, AWSWrangler), SQL, HCL (Terraform), Shell
* **Data Formats:** JSON (source), Parquet (processed)
* **Compression:** Snappy
* **Version Control:** Git & GitHub

---
