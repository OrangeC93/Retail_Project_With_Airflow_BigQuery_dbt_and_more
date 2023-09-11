In retail.py:
```python
from airflow.models.baseoperator import chain

chain(
  upload_csv_to_gcs,
  create_retail_dataset,
  gcs_to_raw,
  check_load(),
  transform,
  check_transform(),
  report,
  check_report()
)

# please remove the check_load(), check_transform(), check_report() in above code

```
