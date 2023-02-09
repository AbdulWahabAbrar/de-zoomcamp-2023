# Week 2 Homework

# Question 1: Using the etl_web_to_gcs.py flow that loads taxi data into GCS as a guide, create a flow that loads the green taxi CSV dataset for January 2020 into GCS and run it. 
Look at the logs to find out how many rows the dataset has.

```python
from pathlib import Path
import pandas as pd
from prefect import flow, task
from prefect_gcp.cloud_storage import GcsBucket
from random import randint


@task(retries=3)
def fetch(dataset_url: str) -> pd.DataFrame:
    """Read taxi data from web into pandas DataFrame"""
    # if randint(0, 1) > 0:
    #     raise Exception

    df = pd.read_csv(dataset_url)
    return df


@task(log_prints=True)
def clean(df: pd.DataFrame) -> pd.DataFrame:
    """Fix dtype issues"""
    df["lpep_pickup_datetime"] = pd.to_datetime(df["lpep_pickup_datetime"])
    df["lpep_dropoff_datetime"] = pd.to_datetime(df["lpep_dropoff_datetime"])
    print(df.head(2))
    print(f"columns: {df.dtypes}")
    print(f"rows: {len(df)}")
    return df


@task()
def write_local(df: pd.DataFrame, color: str, dataset_file: str) -> Path:
    """Write DataFrame out locally as CSV file"""
    path = Path(f"/home/abdulwahab/data-engineering-zoomcamp/week_2_workflow_orchestration/02_gcp/{dataset_file}.csv")
    df.to_csv(path)
    return path


@task()
def write_gcs(path: Path) -> None:
    """Upload local CSV file to GCS"""
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.upload_from_path(from_path=path, to_path=path)
    return


@flow()
def etl_web_to_gcs() -> None:
    """The main ETL function"""
    color = "green"
    year = 2020
    month = 1
    dataset_file = f"{color}_tripdata_{year}-{month:02}"
    dataset_url = f"https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{color}/{dataset_file}.csv.gz"

    df = fetch(dataset_url)
    df_clean = clean(df)
    path = write_local(df_clean, color, dataset_file)
    write_gcs(path)


if __name__ == "__main__":
    etl_web_to_gcs()
 
 ```
 
 # Question 2: Using the flow in etl_web_to_gcs.py, create a deployment to run on the first of every month at 5am UTC. What’s the cron schedule for that?
 
 Below image shows the deployment tab of Prefect Orion UI, where I have 4 deployments, out of which the first deployment is scheduled for first of every month at 5 AM UTC,
 it's cron schedule is 0 5 1 * * 
 
 ![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/Cron%20Job.png "Cron Job Screenshot")
 
 # Question 3: Using etl_gcs_to_bq.py as a starting point, modify the script for extracting data from GCS and loading it into BigQuery for the months Feb and March of 2019.
This new script should not fill or remove rows with missing values. (The script is really just doing the E and L parts of ETL).

```python
from pathlib import Path
import pandas as pd
from prefect import flow, task
from prefect_gcp.cloud_storage import GcsBucket
from prefect_gcp import GcpCredentials


@task(retries=3)
def extract_from_gcs(color: str, year: int, month: int) -> Path:
    """Download trip data from GCS"""
    gcs_path = f"/home/abdulwahab/data-engineering-zoomcamp/week_2_workflow_orchestration/03_deployments/{color}_tripdata_{year}-{month:02}.parquet"
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.get_directory(from_path=gcs_path, local_path="/home/abdulwahab/data-engineering-zoomcamp/week_2_workflow_orchestration/03_deployments/")
    return Path(gcs_path)



@task()
def transform(path: Path) -> pd.DataFrame:
    """Data cleaning example"""
    df = pd.read_parquet(path)
    df.drop(df.columns[0], axis=1, inplace=True)
    return df


@task()
def write_bq(df: pd.DataFrame) -> None:
    """Write DataFrame to BiqQuery"""

    gcp_credentials_block = GcpCredentials.load("zoom-gcp-creds")

    df.to_gbq(
        destination_table="trips_data_all.yellow_rides",
        project_id="igneous-ethos-375619",
        credentials=gcp_credentials_block.get_credentials_from_service_account(),
        chunksize=500_000,
        if_exists="append",
    )


@flow(log_prints=True)
def etl_gcs_to_bq(color: str, year: int, months: list[int]):
    """Main ETL flow to load data into Big Query"""

    paths = []
    for month in months:
        path = extract_from_gcs(color, year, month)
        paths.append(path)

    dfs = [transform(path) for path in paths]
    df = pd.concat(dfs, axis=0)
    write_bq(df)

if __name__ == "__main__":
    etl_gcs_to_bq(color="yellow", year=2019, months=[2, 3])
```

### Below is the screenshot from BigQuery which shows the count of rows processed by the flow

 ![alt text](https://github.com/AbdulWahabAbrar/de-zoomcamp-2023/blob/main/bq_yellowrides.png "Count of Rows Processed")

# Question 4: GitHub storage block

Following are the steps I used to solve Question 4
1. Moved my code repository to GitHub account and connected my local github repository to commit changes using GitHub Personal Account Token agter making necessary changes to the code
2. After that I created a GitHub block using Prefect UI and put the required details like Repository name and Block name
3. Then, I created a python file called github_deploy.py where I gave entrypoint of my github flow, my github block name  and applied the deployment using .apply()
```python
#github_deploy.py file
from prefect.deployments import Deployment
from etl_web_to_gcs import etl_web_to_gcs
from prefect.filesystems import GitHub 

storage = GitHub.load("zoom-github")

deployment = Deployment.build_from_flow(
     flow=etl_web_to_gcs,
     name="notif_github",
     storage=storage,
     entrypoint="week_2_workflow_orchestration/02_gcp/etl_web_to_gcs.py:etl_web_to_gcs")

if __name__ == "__main__":
    deployment.apply()
```   
4. After that, I run the above file using "python github_deploy.py" and then I started the prefect agent using ```bash prefect agent start -q default ```
5. Then, I ran the deployment using ```bash prefect deployment run etl-web-to-gcs/deploy-github ``` and the result can be seen in the logs of the flow
6. Below is the code for etl_web_to_gcs that was used for the Question 4 of Homework

```python
from pathlib import Path
import pandas as pd
from prefect import flow, task
from prefect_gcp.cloud_storage import GcsBucket
from random import randint


@task(retries=3)
def fetch(dataset_url: str) -> pd.DataFrame:
    """Read taxi data from web into pandas DataFrame"""
    # if randint(0, 1) > 0:
    #     raise Exception

    df = pd.read_csv(dataset_url)
    return df


@task(log_prints=True)
def clean(df: pd.DataFrame) -> pd.DataFrame:
    """Fix dtype issues"""
    df["lpep_pickup_datetime"] = pd.to_datetime(df["lpep_pickup_datetime"])
    df["lpep_dropoff_datetime"] = pd.to_datetime(df["lpep_dropoff_datetime"])
    print(df.head(2))
    print(f"columns: {df.dtypes}")
    print(f"rows: {len(df)}")
    return df


@task()
def write_local(df: pd.DataFrame, color: str, dataset_file: str) -> Path:
    """Write DataFrame out locally as CSV file"""
    path = Path(f"/home/abdulwahab/data-engineering-zoomcamp/week_2_workflow_orchestration/02_gcp/{dataset_file}.csv")
    df.to_csv(path)
    return path


@task()
def write_gcs(path: Path) -> None:
    """Upload local CSV file to GCS"""
    gcs_block = GcsBucket.load("zoom-gcs")
    gcs_block.upload_from_path(from_path=path, to_path=path)
    return


@flow()
def etl_web_to_gcs() -> None:
    """The main ETL function"""
    color = "green"
    year = 2020
    month = 11
    dataset_file = f"{color}_tripdata_{year}-{month:02}"
    dataset_url = f"https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{color}/{dataset_file}.csv.gz"

    df = fetch(dataset_url)
    df_clean = clean(df)
    path = write_local(df_clean, color, dataset_file)
    write_gcs(path)


if __name__ == "__main__":
    etl_web_to_gcs()
```

# Question 5: Email or Slack notifications

I used Slack notifications for this Question 5.
