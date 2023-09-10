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



