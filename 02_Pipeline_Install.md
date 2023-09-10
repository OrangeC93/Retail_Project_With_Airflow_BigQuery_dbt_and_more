Pipeline: 
![image](pics/pipeline.png)


Install:
- Docker
- [Astro CLI](https://docs.astronomer.io/astro/cli/overview): get started with apache airflow
  - In terminal: `brew install astro`
  - In vs code terminal: `astro dev init` to initialize your airflow development environment
    - /dag: data pipeline .py
    - /include: input thins that are not data pipelines in the data pipelines
    - /plugins: customize airflow instance 
    - .env: export environment variables
    - airflow_settings.yaml: persist connections variables
    - dockerfile: astro is a wrapper on docker
    - packages.txt: install operating system dependencies and requirements
  - In vs code terminal: `astro dev start` to execute airflow
  - localhost:8080 admin admin
- Soda: https://www.soda.io/
- GC account
- Cosmos: integrate dbt with airflow
