# Check reports SQL before publish
In include/soda/checks/report/report_customer_invoices.yml:
```
checks for report_customer_invoices:
  # Check fails if the country column has any missing values
  - missing_count(country) = 0:
      name: All customers have a country
  # Check fails if the total invoices is lower or equal to 0
  - min(total_invoices):
      fail: when <= 0
```

In include/soda/checks/report/report_product_invoices.yml:
```
checks for report_product_invoices:
  # Check fails if the stock_code column has any missing values
  - missing_count(stock_code) = 0:
      name: All products have a stock code
  # Check fails if the total quanity sold is lower or equal to 0
  - min(total_quantity_sold):
      fail: when <= 0
```
In include/soda/checks/report/report_year_invoices.yml:
```
checks for report_year_invoices:
  # Check fails if the number of invoices for a year is negative
  - min(num_invoices):
      fail: when < 0
```

Test
```
cd /usr/local/airflow
source /usr/local/airflow/soda_venv/bin/activate
soda scan -d retail -c include/soda/configuration.yml include/soda/checks/report/*
```

# Add final task
```
@task.external_python(python='/usr/local/airflow/soda_venv/bin/python')
    def check_report(scan_name='check_report', checks_subpath='report'):
        from include.soda.check_function import check

        return check(scan_name, checks_subpath)
check_report()
```


