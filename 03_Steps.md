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

## Install Soda Core
- In requirements.txt: `soda-core-bigquery==3.0.45`
- Create a folder /soda
- Create a `configuration.yml`:
```
-- include/soda/configuration.yml
data_source retail:
  type: bigquery
  connection:
    account_info_json_path: /usr/local/airflow/include/gcp/service_account.json
    auth_scopes:
    - https://www.googleapis.com/auth/bigquery
    - https://www.googleapis.com/auth/cloud-platform
    - https://www.googleapis.com/auth/drive
    project_id: 'airtube-390719' -- replace with your project id
    dataset: retail
```
- Restart instance to install soda defined in requirements: `astro dev restart`
- Create a Soda Cloud account
- Create an API → Profile → API Keys → Create API → Copy API in configuration.yml
```
data_source retail:
  type: bigquery
  connection:
    account_info_json_path: /usr/local/airflow/include/gcp/service_account.json
    auth_scopes:
    - https://www.googleapis.com/auth/bigquery
    - https://www.googleapis.com/auth/cloud-platform
    - https://www.googleapis.com/auth/drive
    project_id: 'airtube-390719'
    dataset: retail
soda_cloud:
  host: cloud.soda.io
  api_key_id: ec1bd081-1cfc-4037-93cc-d2ba94de930c
  api_key_secret: CcCCtCBhMIdju4VLKovy6Tha8by7sm7AD4woIW5tlwJvI57Dt0H1sQ
```
- Test the connection: soda core connect with bigquery
```
astro dev bash
soda test connection -d retail -c include/soda/configuratino.yml
```


## Soda Checks
Create the first test for include/soda/checks/sources/raw_invoices.yml：
```
checks for raw_invoices:
  - schema:
      fail:
        when required column missing: [InvoiceNo, StockCode, Quantity, InvoiceDate, UnitPrice, CustomerID, Country]
        when wrong column type:
          InvoiceNo: string
          StockCode: string
          Quantity: integer
          InvoiceDate: string
          UnitPrice: float64
          CustomerID: float64
          Country: string
```
Run checks: `soda scan -d retail -c include/soda/configuration.yml include/soda/checks/sources/raw_invoices.yml`

## Soda Check Code
ExternalPython uses an existing python virtual environment with dependencies pre-installed. That makes it faster to run than the VirtualPython where dependencies are installed at each run.For example if you have soda 1 or 2 Installed in your computer but your data quality check needs soda 3, then you can create a python vitural environment us the python operator so you can run quality check without impacting your computer

include/soda/check_function.py:
```python
def check(scan_name, checks_subpath=None, data_source='retail', project_root='include'): # scan_name = 'check_load', check_subpatch='sources'
    from soda.scan import Scan

    print('Running Soda Scan ...')
    config_file = f'{project_root}/soda/configuration.yml'
    checks_path = f'{project_root}/soda/checks'

    if checks_subpath:
        checks_path += f'/{checks_subpath}' # /soda/checks/sources

    scan = Scan()
    scan.set_verbose()
    scan.add_configuration_yaml_file(config_file)
    scan.set_data_source_name(data_source)
    scan.add_sodacl_yaml_files(checks_path)
    scan.set_scan_definition_name(scan_name)

    result = scan.execute()
    print(scan.get_logs_text())

    if result != 0:
        raise ValueError('Soda Scan failed')

    return result
```

Dockerfile: create the python virtual env:
```
# install soda into a virtual environment
RUN python -m venv soda_venv && source soda_venv/bin/activate && \ # define soda env and install soda-core related libraries
    pip install --no-cache-dir soda-core-bigquery==3.0.45 &&\
    pip install --no-cache-dir soda-core-scientific==3.0.45 && deactivate
```

In the DAG, create a new task: call check() in include/soda/check_function.py 
```python
@task.external_python(python='/usr/local/airflow/soda_venv/bin/python') # where your python env is 
    def check_load(scan_name='check_load', checks_subpath='sources'): # where the checks are sources # soda/checks/sources/raw_invoices
        from include.soda.check_function import check
        return check(scan_name, checks_subpath)
check_load() # because we use decorator to create that task, you need to expicitly call these tasks in your tag otherwise the task doesn't exist
```

Test the task:
```
astro dev bash
airflow tasks test retail check_load 2023-01-01
```



