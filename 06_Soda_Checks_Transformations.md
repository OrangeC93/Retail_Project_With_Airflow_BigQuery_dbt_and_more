# Check the transformations after dbt
In soda/transform/dim_customer.yml:
```
checks for dim_customer:
  - schema:
      fail:
        when required column missing: 
          [customer_id, country]
        when wrong column type:
          customer_id: string
          country: string
  - duplicate_count(customer_id) = 0:
      name: All customers are unique
  - missing_count(customer_id) = 0:
      name: All customers have a key
```

In soda/transform/dim_datetime.yml:
```
checks for dim_datetime:
  # Check fails when product_key or english_product_name is missing, OR
  # when the data type of those columns is other than specified
  - schema:
      fail:
        when required column missing: [datetime_id, datetime]
        when wrong column type:
          datetime_id: string
          datetime: datetime
  # Check failes when weekday is not in range 0-6
  - invalid_count(weekday) = 0:
      name: All weekdays are in range 0-6
      valid min: 0
      valid max: 6
  # Check fails when customer_id is not unique
  - duplicate_count(datetime_id) = 0:
      name: All datetimes are unique
  # Check fails when any NULL values exist in the column
  - missing_count(datetime_id) = 0:
      name: All datetimes have a key
```
In soda/transform/dim_product.yml:
```
checks for dim_product:
  # Check fails when product_key or english_product_name is missing, OR
  # when the data type of those columns is other than specified
  - schema:
      fail:
        when required column missing: [product_id, description, price]
        when wrong column type:
          product_id: string
          description: string
          price: float64
  # Check fails when customer_id is not unique
  - duplicate_count(product_id) = 0:
      name: All products are unique
  # Check fails when any NULL values exist in the column
  - missing_count(product_id) = 0:
      name: All products have a key
  # Check fails when any prices are negative
  - min(price):
      fail: when < 0
```
In soda/transform/fct_invoices.yml:
```
checks for fct_invoices:
  # Check fails when invoice_id, quantity, total is missing or
  # when the data type of those columns is other than specified
  - schema:
      fail:
        when required column missing: 
          [invoice_id, product_id, customer_id, datetime_id, quantity, total]
        when wrong column type:
          invoice_id: string
          product_id: string
          customer_id: string
          datetime_id: string
          quantity: int
          total: float64
  # Check fails when NULL values in the column
  - missing_count(invoice_id) = 0:
      name: All invoices have a key
  # Check fails when the total of any invoices is negative
  - failed rows:
      name: All invoices have a positive total amount
      fail query: |
        SELECT invoice_id, total
        FROM fct_invoices
        WHERE total < 0
```
# Check for task
```sql
@task.external_python(python='/usr/local/airflow/soda_venv/bin/python')
    def check_transform(scan_name='check_transform', checks_subpath='transform'):
        from include.soda.check_function import check

        return check(scan_name, checks_subpath)
check_transform()
```

Test in terminal: `airflow tasks test retail check_transform 2023-01-01`

