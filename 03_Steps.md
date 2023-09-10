# Data Preparation
- Download the dataset https://www.kaggle.com/datasets/tunguz/online-retail
- Store the csv file in `include/dataset/online_retail.csv`
- In requirements.txt, add **apache-airflow-providers-google==10.3.0** restart Airflow

# Ingest Raw Dataset
- Create a GCS bucket with a unique name `april_online_retail`
- Create a service account with a name `airflow-online-retail`
    - Go to IAM & Admin -> service account -> Add GCS(cloud storage) admin role & BigQuery admin role
    - Click on the service account → Keys → Add Key → Copy the JSON content
    - Create a new file `service_account.json` in `include/gcp/` ->  `include/gcp/service_account.json`
```
{
    "type": "service_account",
    "project_id": "airtube-390719",
    "private_key_id": "58cfee8a937e7bfc66ae6465a848db53cf4fb919",
    "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQCs1DzaHpGl7zcw\nD/Ab2VDs1YxKK7rnT8Oi+pJwxomLGEg15q2mgbS8tebrs65lC8Ihzs3SGfaBycWK\nlMqOLP95gTnsYgMpveoCv/L9OU8UiYTbPdJEk7YxJi0u7jYl0BD2WJ5NnpUdBFYH\nEzAs9X5/3sHRqQItLtxMVEQ5kG4Q0kPeGFGMhABksuBkZ8EtwdVxsNQFUk9GAdJb\nWLa6Co3DraFd3A3QXCfO8HHmdR4YxqX6MKOpAipDied6PYUhf+uZoW9FFXg4VLSz\n0INYkmIhxmv+3jNcHS0wCUFS/GunRzvk9gh61HpF6KvM4OBuRSKLWt1Jezsww4hl\nQbJYOnbTAgMBAAECggEAJ6IgNlDuS6Q4/q+Y+3nxge5S1quCmAsFrTlTHcOZxSkT\nXjEBP37dKK16QDEbXBa/NSuMrZLAofDYeTg33zTYfU+yLdAoM4lWwbytB3798JK8\nwd5CevF4xXqgv/NmvXMigKu/2cL1JQtagxLWaGj/0mkN/3uHgT8Oy/5DCwRhCUAe\nIFtpzCLp4fFrkpRcXXcKKL7zsxt9x4ya0qiYA/q1p7y4zl3734ZxGw48MPVeLGSE\ne46cVMBBlDYgdsnwLJA7IexbIrg5viw4m7HG9QMUaoU1vb7xgQSEX0VM+4z3mznH\nYEJ6CAYlEDmcqAegTNayyMRFWej6ZdbgN7h+ju0uoQKBgQDf9Q8eIaM6TDlOJBrX\nBhv8eIraaGbP6EBFbc8m7jp8JwtuHrdcPzxH5euxwRWnpF8HzH7ea7+EXKvRhuTf\n5y5+biMquK5USGEp1pN/ehaOpXjgMvNZ9qRTVPWjgKM4hjER12xSuVEjqFP3yTG6\nqZyGqu/ylVC8glJp1Zdm1atvowKBgQDFjn4REXBVdDajED/ZydYPNllBi6CegNJs\nshDDlgIElyjKn7pqgrEK3F+sJSRWHJni9Z4mRjokgW/H4us9Bh6rzOEqJRZOs8WS\ndnNBp24W8iprBj8K7l8xCbzTRqwz0wgM0oS/irILGUSyn6ye9I65WqaZ28xCTMhm\nNkKSYVSPEQKBgQDc25j7CAUmmsDwlJ57aqTyyBV26fpqEgo/7diZ9dlrUj3tbRE6\nQYo7BTz4YQfv+SNWV47N3chSyekPik3vmNa7C/ZWTSZuK6rWTavLzSStq/WWc+iU\n0aygGWrcwSE1vvBpPd6vfd3MolWcSKdoA5g/HhffTOz/2i1X/bF/UjvsrQKBgQCr\nr9EZjjk82pldDxMed40jfU0GbIzzEutMcVemUmiAisl1hmjghaHM2YX/ueuhNov6\nNRDzHFcNQLvfT/K1/uqKzavlD4Qac5tBVNWHejVvlZeNmUkSe+SYXmkOh73B8CVv\n10hsmeFvSc9tGN1Q6yJaLVDaJ62U9Nu4EHG8ev+csQKBgCI8ivxrXdn0NwTThb2j\nHfw0CNAHNl1c2ml7lhmSE+pF5uIxlVHwETP7gz2+hSd85AqxG6+mkHZhu4kzwcGh\niI0iIMYARP9Al9SC1dgMjQz28VEctBRs3aiDUPKBkFESig7ZHhy6cxGUkZZdUpKZ\n6jaiEqIIqBLNAATTx+AO/b+R\n-----END PRIVATE KEY-----\n",
    "client_email": "airflow-online-retail@airtube-390719.iam.gserviceaccount.com",
    "client_id": "117189738213106346790",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/airflow-online-retail%40airtube-390719.iam.gserviceaccount.com",
    "universe_domain": "googleapis.com"
}
```

- Run `astro dev restart`
- Create connection in Airflow that use the `gcp/service_account.json` file
  - Go to localhost:8080 UI
  - Go to Admin -> create new connection
    - Connection Id: gcp
    - Connectin Type: Google Cloud
    - Keyfile JSON: usr/local/airflow/include/gcp/service_account.json 
  - Click test to verify connection
    - To enable the test button
    - Add AIRFLOW__CORE__TEST_CONNECTION=enable in .env
    - Restart the instance

# Create DAG
## retail.py
How to find `LocalFilesystemToGCSOperator`: 
- Go to astro registry
- Search local gcs
  
```python
from airflow.decorators import dag, task
from datetime import datetime

from airflow.providers.google.cloud.transfers.local_to_gcs import LocalFilesystemToGCSOperator

@dag(
    start_date=datetime(2023, 1, 1),
    schedule=None,
    catchup=False,
    tags=['retail'],
)
def retail():

    upload_csv_to_gcs = LocalFilesystemToGCSOperator(
        task_id='upload_csv_to_gcs',
        src='include/dataset/online_retail.csv', # local file path
        dst='raw/online_retail.csv', # gcs destinatin path
        bucket='april_online_retail', # gcs path
        gcp_conn_id='gcp', # gcs connection id created on Airflow UI
        mime_type='text/csv',
    )

retail()
```

Test the `upload_csv_to_gcs` task
```
astro dev bash
airflow tasks test retail upload_csv_to_gcs 2023-01-01
```

## Upload csv to Bigquery
### Create an empty Dataset (schema equivalent)
```python
from airflow.providers.google.cloud.operators.bigquery import BigQueryCreateEmptyDatasetOperator

create_retail_dataset = BigQueryCreateEmptyDatasetOperator(
        task_id='create_retail_dataset',
        dataset_id='retail',
        gcp_conn_id='gcp',
    )
```
### Create the task to load the file into a BigQuery raw_invoices table
```python
from astro import sql as aql
from astro.files import File
from astro.sql.table import Table, Metadata
from astro.constants import FileType

gcs_to_raw = aql.load_file(
        task_id='gcs_to_raw',
        input_file=File(
            'gs://marclamberti_online_retail/raw/online_retail.csv', # file path
            conn_id='gcp',
            filetype=FileType.CSV,
        ),
        output_table=Table( 
            name='raw_invoices', # name of table
            conn_id='gcp',
            metadata=Metadata(schema='retail') # under retail dataset
        ),
        use_native_support=False,
    )
```

