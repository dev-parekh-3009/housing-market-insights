Notebook structure for pipelines

TO DO

Done
- Tables are now created in a a separate notebook
- Remove create table statements from pipelines
- If table doesn't exist, exit pipeline

In progress
- Find a way to make re-runs possible when setting up files for re-run

To Do
- Write source filename to the table
- Write rejected records to table
  - Write reason for rejection - how to derive this?
- Add further table optimisations:
  - Partitioning
  - Z-ordering
  - Delta constraints
- Create a new table for tracking files processed (filename, hash, timestamp, status, row count)
- Update incremental logic to use files instead of max ingestion timestamp
- Think about gold layer ideas (see chat)

0 - Source data
- Downloaded from https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads
- Create a new schema called 'landing'
- Create a volume in the landing schema called property_prices
- Create directories in the volume for new, processed, and failed
- Upload files to landing/property_prices/new

1 - Ingestion (Bronze)

1.1 - Set variables for 
- Catalog name
- Database name 
- Bronze table name  

1.2 - Check the existing data
- Check if the table exists
- Get total row count in bronze table

1.3 - Read data from source
- Define source path (landing zone)
- List CSV files in landing zone
- Define column names of expected data
- Instantiate variable to count number of processed records
- For each CSV file, 
    - Validate column names match expected list (reject file if not)
    - Read into dataframe
    - Add column for ingestion time
    - Write to bronze table in delta append mode
    - Update count of processed records

1.4 - Validate data processed
- Get total row count in bronze table
- Subtract from previous total row count
- Compare with count of processed records from files
- Write warning message if missing rows

1.5 - Move files
- Rejected files move to landing/property_prices/failed
- Successful files move to landing/property_prices/processed


2 - Transformation (Silver)

2.1 - Set variables for 
- Catalog name
- Database name 
- Bronze table name  
- Silver table name

2.2 - Check the existing data
- Check if the bronze table exists (fail if not)
- Check if the silver table exists (create if not)
  - Open question: How to enforce updated schema validation? 
  - Silver table schema should not exactly match bronze table
- Get total row count in bronze table
- Get total row count in silver table

2.3 - Calculate delta records
- Read bronze table into dataframe
- Read silver table into dataframe
- Calculate max ingestion time from silver table
- Filter bronze table on ingestion time
- Count new bronze records to process
- Exit if no new bronze records

2.4 - Clean and transform data
- Filter new records to remove invalid data:
    - transaction_id is not null
    - price is not null
    - price > 0
    - postcode is not null
    - transfer_date is not null
    - transfer_date > year 2020
- De-duplicate based on transaction_id, keeping the latest ingestion_time
- Apply trim and uppercase to string columns
- Capture rows filtered out due to invalid data in a separate df
- Count new silver records to write

2.5 - Merge (upsert) data into silver table
- Match records on transaction_id
- Update when matched
- Insert when not matched

2.6 - Write failed records to new table

3 - Create analytics tables

3.1 - Average price by region

3.2 - Monthly price trends

3.3 - Property type analysis


4 - Dashboard with insights

4.1 - London prices vs UK average

4.2 - Fastest growing regions

