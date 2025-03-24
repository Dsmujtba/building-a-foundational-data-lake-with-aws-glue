# simple-data-lake-with-awsglue

# Simple Data Lake with AWS Glue

This project demonstrates a basic data lake implementation using Amazon S3 for storage, AWS Glue for ETL and data cataloging, and Amazon Athena for querying.  It simulates a scenario where you process raw Amazon review data and product metadata.

## Project Overview

This project simulates a data engineering task for a retailer specializing in scale models.  You'll be working with Amazon review data (toys and games category) and product metadata. The goal is to clean, transform, and prepare this data for analysis, making it accessible via SQL queries using Amazon Athena.

The project involves the following steps:

1.  **Raw Data Exploration:** Exploring sample datasets of customer reviews and product metadata (JSON format).
2.  **Data Transformation (Python):** Implementing Python functions to clean and transform the raw data, preparing it for efficient storage and querying.
3.  **AWS Glue ETL Jobs:** Creating and running AWS Glue ETL jobs (using PySpark) to process the full datasets. These jobs utilize the transformation functions you developed. The processed data is stored in Parquet format in S3.
4.  **AWS Glue Data Catalog:** Creating and running an AWS Glue Crawler to automatically discover the schema of the processed data and populate the AWS Glue Data Catalog.
5.  **Data Querying with Amazon Athena:** Using Amazon Athena to execute SQL queries against the processed data stored in S3, leveraging the metadata in the Glue Data Catalog.
6. **Optional Experiments**: Partitioning and Compression.

## Project Structure

The project directory is organized as follows:
simple-data-lake/
├── README.md               <- This file
├── scripts/
│   └── setup.sh            <- Setup script (likely for environment setup)
├── terraform/
│   ├── assets/
│   │   ├── de-c3w2-reviews-transform-job.py   <- Glue ETL script for reviews
│   │   └── de-c3w2-metadata-transform-job.py  <- Glue ETL script for metadata
│   └── glue.tf             <- Terraform configuration for Glue resources
├── images/
│   └── data_lake.png     <- Architecture diagram
├── .aws/                   <- (Potentially) AWS configuration files (should be ignored by Git)
├── errors.txt            <- For potential terrafrom error debugging
└── C3_W2_Lab_1_Data_Lake_Solution.ipynb <- Jupyter Notebook with the main project.


## Prerequisites

*   An AWS account with access to the following services:
    *   S3 (Simple Storage Service)
    *   AWS Glue
    *   Amazon Athena
    *   IAM (Identity and Access Management)
*   Basic familiarity with Python and Pandas.
*   Basic familiarity with SQL.
*   Terraform installed and configured.
*  AWS CLI installed and configured.
* Jupyter Notebook.

## Setup and Execution

1.  **AWS Account and Credentials:**
    *   Ensure you have an active AWS account. The AWS Free Tier is sufficient for this lab.
    *   Configure your AWS credentials (access key ID and secret access key) using the AWS CLI or environment variables.  The provided `setup.sh` script likely handles some of this setup.

2.  **Clone the Repository (Hypothetical - since this is a lab):**
    ```bash
    git clone <repository_url>
    cd simple-data-lake
    ```

3.  **Explore the Raw Data (Exercises 1 & 2):**
    *   Open the Jupyter Notebook `C3_W2_Lab_1_Data_Lake_Solution.ipynb`.
    *   Run the provided code cells to load and explore the sample datasets (`reviews_Toys_and_Games_sample.json.gz` and `meta_Toys_and_Games_sample.json.gz`).
    *   Complete Exercises 1 and 2 to familiarize yourself with the data structure.

4.  **Implement Data Transformation Functions (Exercises 3 & 4):**
    *   Complete the `process_review()` function in the notebook (Exercise 3).
    *   Complete the `process_metadata()` function in the notebook (Exercise 4).
    *   These functions define the data cleaning and transformation logic.

5.  **Prepare Glue ETL Scripts (Exercises 5 & 6):**
    *   Open the provided Glue ETL scripts:
        *   `terraform/assets/de-c3w2-reviews-transform-job.py`
        *   `terraform/assets/de-c3w2-metadata-transform-job.py`
    *   Complete the `transform()` function in *each* script by copying the relevant code from Exercises 3 and 4.
    *   *Save the changes* to these files.
    *   Upload script to S3 bucket:
        ```bash
          aws s3 cp ./terraform/assets/de-c3w2-reviews-transform-job.py s3://{SCRIPTS_BUCKET_NAME}/de-c3w2-reviews-transform-job.py
          aws s3 cp ./terraform/assets/de-c3w2-metadata-transform-job.py s3://{SCRIPTS_BUCKET_NAME}/de-c3w2-metadata-transform-job.py
         ```

6.  **Deploy AWS Glue Jobs with Terraform:**
    *   Navigate to the `terraform` directory: `cd terraform`
    *   Initialize Terraform: `terraform init`
    *   Review the Terraform plan: `terraform plan`
    *   Apply the Terraform configuration: `terraform apply` (Type `yes` and press Enter to confirm). This creates the Glue jobs and necessary IAM roles. Note down the `glue_role`, `reviews_glue_job` and `metadata_glue_job` from the output.

7.  **Run AWS Glue ETL Jobs:**
    *   Execute the Glue jobs using the AWS CLI (replace `<GLUE-JOB-NAME>` with the actual job names from the Terraform output):
        ```bash
        aws glue start-job-run --job-name <GLUE-JOB-NAME> | jq -r '.JobRunId'
        ```
    *   Check the status of the jobs (replace `<GLUE-JOB-NAME>` and `<JOB-RUN-ID>`):
        ```bash
        aws glue get-job-run --job-name <GLUE-JOB-NAME> --run-id <JOB-RUN-ID> --output text --query "JobRun.JobRunState"
        ```
        Wait for the jobs to complete with a `SUCCEEDED` status.

8.  **Create and Run AWS Glue Crawler (Exercise 7):**
    *   Complete Exercise 7 in the Jupyter Notebook to create and configure the Glue crawler.  This involves using the `boto3` library to interact with the Glue API.  *Make sure to use the correct IAM role and S3 paths for your transformed data.*
     ```python
        glue_client = boto3.client('glue',region_name="us-east-1")
        configuration= {"Version": 1.0,"Grouping": {"TableGroupingPolicy": "CombineCompatibleSchemas" }}

        response = glue_client.create_crawler(
            Name='de-c3w2lab1-crawler',
            Role= '<GLUE_ROLE>', # Paste the value here
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
         ```
           response = glue_client.start_crawler(Name='de-c3w2lab1-crawler')
         ```
    *  Verify table creation.

9.  **Query Data with Amazon Athena:**
    *   Use the provided SQL queries in the notebook (Section 6) to query the processed data using Amazon Athena.  These queries demonstrate how to join data from the two tables and perform aggregations.

10. **Optional Experiments (Section 7):**
    *   Explore the effects of different compression algorithms (Snappy, Gzip, uncompressed) and partitioning strategies on storage size and query performance.

11. **Clean Up (Section 8):**
     *   Run `terraform destroy` in terraform folder to delete all created resources.

**Data Sources:**

*   **Amazon Customer Reviews Dataset:** [https://s3.amazonaws.com/amazon-reviews-pds/readme.html](https://s3.amazonaws.com/amazon-reviews-pds/readme.html)
*   **SNAP (Stanford Large Network Dataset Collection):** [https://snap.stanford.edu/data/](https://snap.stanford.edu/data/)

**AWS Services Used:**

*   **Amazon S3:** Data lake storage.
*   **AWS Glue:** ETL service and Data Catalog.
*   **AWS Glue Crawler:** Automated schema discovery.
*   **Amazon Athena:** Serverless SQL query engine.
*   **IAM:** Identity and Access Management (for permissions).
*   **Terraform:** Infrastructure as Code (IaC) for resource provisioning.

**Key Concepts:**

*   Data Lake
*   ETL (Extract, Transform, Load)
*   Schema-on-Read
*   Data Catalog
*   Serverless Computing
*   Parquet File Format
*   Data Partitioning
*   Data Compression
*   Infrastructure as Code (IaC)

**Troubleshooting**
*   **Terraform Errors:** If you encounter errors during `terraform apply`, carefully review the error messages. Common issues include incorrect AWS credentials, syntax errors in the Terraform configuration files, or resource naming conflicts. Use `-no-color  2> errors.txt` to catch the errors.
*   **Glue Job Failures:** If a Glue job fails, check the job logs in the AWS Glue console (or CloudWatch Logs) for detailed error messages.  Common issues include incorrect S3 paths, missing IAM permissions, or errors in your PySpark code.
*  **Crawler Issues**: Crawler might not create tables due to perimission, incorrect path, or data format.
* **Athena Query Errors:** If an Athena query fails, check the query syntax and ensure that the table names and column names are correct. Also, verify that the Glue Data Catalog contains the correct metadata for your data.

This comprehensive README.md file provides a clear and well-organized guide to the project, making it 
