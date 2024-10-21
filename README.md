# ETL-Data-enginering-project-part-2

## Part 2 - Data Quality, Orchestration and Visualization

For the second part of the capstone project, you will further develop the data pipeline. You will integrate data quality checks and orchestration to improve the robustness of the architecture and include data visualization and analytical views to showcase the insights you can generate with the data pipeline. 

# Table of Contents

- [ 1 - Introduction](#1)
- [ 2 - Deployment of the Previous Architecture](#2)
- [ 3 - Data Quality with AWS Glue](#3)
  - [ 3.1 - Configuring the Rule Sets](#3.1)
  - [ 3.2 - Creating Materialized Views with *dbt*](#3.2)
- [ 4 - Orchestration with Apache Airflow](#4)
  - [ 4.1 - Accessing Apache Airflow](#4.1)
  - [ 4.2 - DAG for Songs Data in RDS Source](#4.2)
  - [ 4.3 - DAG for Users and Sessions Data from API Source](#4.3)
- [ 5 - Data Visualization with Apache Superset](#5)


## 1 - Introduction

DeFtunes is a new company in the music industry, offering a subscription-based app for streaming songs. Recently, they have expanded their services to include digital song purchases. With this new retail feature, DeFtunes requires a data pipeline to extract purchase data from their new API and operational database, enrich and model this data, and ultimately deliver the comprehensive data model for the Data Analysis team to review and gain insights. You and your team have developed an initial version of the pipeline, now there are some new requirements to improve it.

The new requirements for this project are:

![alt text](<Screenshot 2024-10-19 at 01.02.18.png>)

1. The pipeline should allow for incremental ingestion of new data from the data sources.
2. The pipeline should run daily using data orchestration (you will use Airflow).
3. Data quality checks should be implemented to verify the quality of newly ingested and cleansed data.
4. Analytical views should be added on top of the star schema data model.
5. A dashboard should be added to the architecture, to allow the visualization of the analytical views and insights (you will use Apache Superset).


## 2 - Deployment of the Previous Architecture

You will recreate the data architecture from the first part of the capstone, this is a refresher of the elements:

<img width="990" alt="Screenshot 2024-10-18 at 13 27 54" src="https://github.com/user-attachments/assets/f93814f4-0932-47d5-9477-0b16ca6bdcc3">

1. **Data Sources**:
   1. *DeFtunes API*: Contains the `users` and `sessions` endpoints, used to gather information about sales parametrized by a start and end date.
   2. *DeFtunes Operational RDS*: Contains the `songs` table, with all the information related to the available songs in the platform
2. **Extract Jobs**: Three AWS Glue jobs in charge of extracting information from the data sources into the `landing zone`. A new argument has been added to the jobs known as `ingest_date`, this will help with the incremental load of new data.
3. **Transform Jobs**: Two AWS Glue jobs in charge of transforming the raw information coming from the `landing zone` and saving the cleansed data into Apache Iceberg tables in the `transformation zone`.
4. **Redshift Spectrum + Glue Catalog**: Enable Redshift to query Apache Iceberg tables in the `transformation zone`
5. **Redshift**: Data warehouse solution used as the `serving layer`, the data modelling is done using *dbt*.

To deploy this infrastructure again, you have been provided with some Terraform files. 

2.1. Go to the AWS console and search for **CloudFormation**. Click on the alphanumeric ID stack and then open the **Outputs** tab. You will see the key `APIEndpoint`, copy the corresponding **Value**. 

Open the glue file `terraform/modules/extract_job/glue.tf`, replace the `<API_ENDPOINT>` placeholders with the API Endpoint value (in two places). 

Take a look at the ingestion date which is in the format `YYYY-MM-DD`. Jobs will be running each month to extract data from the previous month, starting by ingesting the data of January 2020. So the ingestion date will need to be the first day of the next month, `"2020-02-01"`. Replace the placeholders `<INGEST_DATE_YYYY-MM-DD>` with the ingestion date `"2020-02-01"` in the format `YYYY-MM-DD` (in three places).

Save changes.

Next Steps in the notebook_details_2.ipynb file