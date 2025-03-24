# Data Lake Processing with AWS Glue and Athena

This project demonstrates a simple data lake implementation on AWS, focusing on processing Amazon review and product metadata. It covers data ingestion, transformation, cataloging, and querying using AWS Glue, Amazon S3, and Amazon Athena.

## Project Overview

This project simulates a data engineering task for a retailer analyzing Amazon product reviews and metadata (specifically for the "Toys & Games" category). Raw data (JSON format) is ingested into an S3 data lake, transformed using AWS Glue ETL jobs (written in PySpark), and stored in Parquet format. An AWS Glue Crawler populates the Glue Data Catalog, enabling SQL queries via Amazon Athena. The project also explores data partitioning and compression.

## Architecture

![Data Lake Architecture](images/data_lake.png)

This project utilizes the following AWS services:

*   **Amazon S3:** Used as the data lake for storing both raw and processed data.
*   **AWS Glue:** Used for ETL processing and data cataloging.
    *   **Glue ETL Jobs:** PySpark scripts that transform the raw JSON data into Parquet format.
    *   **Glue Crawler:** Automatically discovers the schema of the processed data and populates the Glue Data Catalog.
*   **Amazon Athena:** Used for interactive SQL queries against the processed data in S3.
*   **AWS IAM:** Used for managing permissions for Glue jobs and the crawler.
* **Terraform**: Used to define infrastructure.

## Data Sources

The project uses two datasets from the Amazon Customer Reviews Dataset:

*   **Reviews Data:**  `reviews_Toys_and_Games_5.json.gz` (Full dataset) and `reviews_Toys_and_Games_sample.json.gz` (Sample). Contains customer reviews, including reviewer ID, product ID (ASIN), review text, rating, and timestamps.
*   **Product Metadata:** `meta_Toys_and_Games.json.gz` (Full dataset) and `meta_Toys_and_Games_sample.json.gz` (Sample). Contains product information, including ASIN, title, description, price, brand, and sales rank.

The data is stored in the following S3 bucket: `s3://<YOUR_BUCKET_NAME>/` where `<YOUR_BUCKET_NAME>` is a unique bucket name created during setup.

**Data Schema (Reviews):**

| Column Name     | Data Type | Description                                      |
| --------------- | --------- | ------------------------------------------------ |
| `reviewerID`    | STRING    | Unique ID of the reviewer.                       |
| `asin`          | STRING    | Unique ID of the product (ASIN).                 |
| `reviewerName`  | STRING    | Name of the reviewer.                             |
| `helpful`       | ARRAY     | Helpfulness rating (e.g., \[2, 3] = 2 out of 3). |
| `reviewText`    | STRING    | Text of the review.                               |
| `overall`       | DOUBLE    | Rating of the product (1-5).                       |
| `summary`       | STRING    | Summary of the review.                            |
| `unixReviewTime`| INT       | Time of the review (Unix timestamp).             |
| `reviewTime`    | STRING    | Time of the review (raw string format).           |

**Data Schema (Metadata):**

| Column Name      | Data Type | Description                                           |
| ---------------- | --------- | ----------------------------------------------------- |
| `asin`           | STRING    | Unique ID of the product (ASIN).                      |
| `description`    | STRING    | Description of the product.                            |
| `title`          | STRING    | Name of the product.                                  |
| `price`          | DOUBLE    | Price of the product.                                 |
| `imUrl`          | STRING    | URL of the product image.                             |
| `related`        | MAP       | Related products (also_bought, also_viewed, etc.).     |
| `salesRank`      | MAP       | Sales rank information.                               |
| `brand`          | STRING    | Brand name.                                           |
| `categories`     | ARRAY     | List of categories the product belongs to.            |

## Prerequisites

*   An AWS account with access to S3, Glue, Athena, and IAM.
*   Python 3.7+
*   AWS CLI installed and configured.
*   Terraform installed and configured.
*   `boto3`, `awswrangler`, and `pandas` Python libraries.
*  Jupyter Notebook.

## Setup and Execution

1.  **Configure AWS Credentials:** Ensure your AWS CLI is configured with appropriate credentials.

2.  **Deploy Infrastructure (Terraform):**
    *   Navigate to the `terraform` directory: `cd terraform`
    *   Initialize Terraform: `terraform init`
    *   Review the Terraform plan: `terraform plan`
    *   Apply the Terraform configuration: `terraform apply` (Type `yes` and press Enter). *Note the output values: `glue_role`, `reviews_glue_job`, `metadata_glue_job`.*

3.  **Upload Glue Scripts to S3:**
    *   Copy the provided Glue scripts to your S3 scripts bucket. Replace `<SCRIPTS_BUCKET_NAME>` with the actual bucket name:
    ```bash
    aws s3 cp ./terraform/assets/de-c3w2-reviews-transform-job.py s3://<SCRIPTS_BUCKET_NAME>/de-c3w2-reviews-transform-job.py
    aws s3 cp ./terraform/assets/de-c3w2-metadata-transform-job.py s3://<SCRIPTS_BUCKET_NAME>/de-c3w2-metadata-transform-job.py
    ```

4.  **Run Glue ETL Jobs:**
    *   Run the Glue jobs using the AWS CLI (replace `<GLUE-JOB-NAME>` with the actual job names from the Terraform output):
        ```bash
        aws glue start-job-run --job-name <GLUE-JOB-NAME> | jq -r '.JobRunId'
        ```
        Example:
        ```bash
        aws glue start-job-run --job-name de-c3w2lab1-reviews-etl-job | jq -r '.JobRunId'
        aws glue start-job-run --job-name de-c3w2lab1-metadata-etl-job | jq -r '.JobRunId'
        ```

    *   Check job status (replace `<GLUE-JOB-NAME>` and `<JOB-RUN-ID>`):
        ```bash
        aws glue get-job-run --job-name <GLUE-JOB-NAME> --run-id <JOB-RUN-ID> --output text --query "JobRun.JobRunState"
        ```
        Wait for both jobs to reach the `SUCCEEDED` state.

5.  **Create and Run Glue Crawler:**
      * Create Glue crawler using boto3

        ```python
        DATABASE_NAME = "de-c3w2lab1-aws-reviews" # Or any database name
        glue_client = boto3.client('glue',region_name="us-east-1")
        configuration= {"Version": 1.0,"Grouping": {"TableGroupingPolicy": "CombineCompatibleSchemas" }}

        response = glue_client.create_crawler(
            Name='de-c3w2lab1-crawler',
            Role= '<GLUE_ROLE>', # Paste Glue Role Here
            DatabaseName=DATABASE_NAME,
            Description= 'Amazon Reviews for Toys',
            Targets={
                'S3Targets': [
                    {
                        'Path': f's3://{BUCKET_NAME}/processed_data/snappy/partition_by_year_month/toys_reviews/',
                    },
                    {
                        'Path': f's3://{BUCKET_NAME}/processed_data/snappy/partition_by_sales_category/toys_metadata/',
                    }
                ]}

            )
        ```
     *   Start the crawler:
        ```bash
        aws glue start-crawler --name de-c3w2lab1-crawler
        ```
        *   Check that the crawler created the tables:
        ```python
         import awswrangler as wr
         wr.catalog.tables(database=DATABASE_NAME)
        ```

6.  **Query Data with Athena:**

    *   Open the Amazon Athena console.
    *   Select the database created by the crawler (e.g., `de-c3w2lab1-aws-reviews`).
    *   Run SQL queries against the created tables (e.g., `toys_reviews`, `toys_metadata`).

    **Example Queries:**

    *   **Top 5 reviewed products:**

        ```sql
        SELECT met.title, count(distinct toy.reviewerid) as review_count
        FROM toys_metadata met
        LEFT JOIN toys_reviews toy
        ON met.asin = toy.asin
        WHERE met.title <> ''
        GROUP BY met.title
        ORDER BY count(distinct toy.reviewerid) DESC
        LIMIT 5;
        ```

    *   **Top 10 products by average rating (at least 1000 reviews):**

        ```sql
        SELECT met.title, met.sales_category, avg(toy.overall) as review_avg
        FROM toys_metadata met
        LEFT JOIN toys_reviews toy
        ON met.asin = toy.asin
        GROUP BY met.title, met.sales_category
        HAVING count(distinct toy.reviewerid) > 1000
        ORDER BY avg(toy.overall) DESC
        LIMIT 10;
        ```
    * **Average rating and number of product by brand**:
       ```sql
        SELECT met.brand, count(distinct met.asin) as product_count, avg(toy.overall) as review_avg
        FROM toys_metadata met
        LEFT JOIN toys_reviews toy
        ON met.asin = toy.asin
        WHERE met.brand <> ''
        GROUP BY met.brand
        ORDER BY count(distinct toy.asin) DESC
        LIMIT 10;
       ```

## Troubleshooting

*   **Glue Job Failures:** Check the Glue job logs in the AWS Management Console or CloudWatch Logs for error messages. Common issues include incorrect S3 paths, IAM permissions, or errors in the PySpark code.
*   **Athena Query Errors:** Verify table and column names in your SQL queries. Ensure the Glue crawler has run successfully and populated the Data Catalog.
* **Terraform issues:** use `-no-color  2> errors.txt`

## Cleanup

To remove all resources created by this project, run the following command from the `terraform` directory:

```bash
terraform destroy
