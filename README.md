# Airflow ETL Weather Data Project
This project demonstrates a straightforward Extract, Transform, Load (ETL) pipeline using Apache Airflow to process weather data. The pipeline fetches weather information from an API, processes it, and stores it in a PostgreSQL database.

## Project Structure

```
airflow_etl_project/
├── dags/
│   ├── src/
│   │   ├── __init__.py
│   │   ├── get_weather.py
│   │   ├── load_table.py
│   │   └── transform_data.py
│   └── weatherDag.py
├── .gitignore
└── README.md
```

## Setup Instructions

### 1. Environment Setup
Set up your Airflow environment:

```
export AIRFLOW_HOME=$PWD
python -m venv venv
source venv/bin/activate
```
Install Dependency (for local run)
```
sudo apt install libpq-dev python-dev

pip install -r dags/requirements.txt
```

### 2. Database Configuration
Install and configure PostgreSQL:
```
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Create a database and user:
```
sudo -u postgres psql
CREATE DATABASE weather_db;
CREATE USER airflow_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE weather_db TO airflow_user;
```
Create the weather table:
```
CREATE TABLE IF NOT EXISTS weather_data (
    city TEXT,
    country TEXT,
    latitude REAL,
    longitude REAL,
    date DATE,
    humidity REAL,
    pressure REAL,
    min_temp REAL,
    max_temp REAL,
    temp REAL,
    conditions TEXT
);
```

### 3. Airflow Connections
Set up the following connections in Airflow:
#### 1. PostgreSQL connection
- In the Airflow UI, select menu Admin > Connections
- Select tab Create
- Fill database credentials, for example:
```
Conn Id = weatherdb_postgres_conn
Conn Type = PostgreSQL
Host = <your-db-host>
Schema = <your-db-name>
Login = <your-db-user>
Password = <your-db-password>
Port = 5432
```

#### 2. OpenWeatherMap API connection
- Register a free account on [openweathermap](https://openweathermap.org/api) and get the API key.
- On Airflow UI, go to Admin → Variables menu, and create new Variable
- Fill with these values:
```
key = OPEN_WEATHER_API_KEY
value = <your-api-key>
```

#### 3. Google Cloud Storage connection
In project are used Google Cloud Platform to save tasks' output file.
- Create a new Service Account in GCP IAM. Make sure it is assigned to a Role that has permission to read and write GCS bucket.
- Create a Key and download it as JSON file. Keep this file as you cannot download it after this time.
- In the Airflow UI, go to an Admin → Connections menu, and create a new connection.
- Fill these value on create connection form:
```
Conn Id = gcp_airflow_lab
Conn Type = Google Cloud Platform
Project Id = <your-gcp-project-id>
Keyfile path: left empty
Keyfile JSON: copy-paste the content of JSON keyfile that you downloaded before
Scopes = https://www.googleapis.com/auth/devstorage.read_write  
```

### 4. Airflow Variables
Set the following Airflow variables:
- `WEATHER_API_KEY`: Your OpenWeatherMap API key
- `GCS_BUCKET`: Your Google Cloud Storage bucket name.
Example:
```
key = GCS_BUCKET
value = <your-gcs-bucket-name>
```

### Running the DAG

To test individual tasks. Create a DAG with three tasks:

- get_weather
- transform_data
- load_table

```
airflow tasks test weather_etl_dag get_weather 2023-01-01
airflow tasks test weather_etl_dag transform_data 2023-01-01
airflow tasks test weather_etl_dag load_table 2023-01-01
```

To run a backfill:
```
airflow dags backfill weather_etl_dag -s 2023-01-01 -e 2023-01-07
```

### Deployment
To deploy to a production Airflow environment (e.g., Cloud Composer):

1. Create a Composer environment

2. Upload DAG files to the corresponding DAG folder in GCS. You can upload it manually or use gsutil with this command (assuming you are in the project's root folder).
```
gsutil rsync -r -x ".*\.pyc$" dags/  gs://<your-composer-environment-bucket>/dags
```
Make sure this directory structure is uploaded to GCS bucket:
```
dags/
   weatherDag.py
   src/
      __init__.py
      get_weather.py
      transform_data.py
      load_table.py
```
3. Check the Airflow UI to see your new deployed DAG.

### Development Notes
To clear task instances:
```
airflow clear <your-dag-id>
```
In Cloud Composer
```
gcloud composer environments run <your-composer-environment-name> --location=<your-composer-environment-location> clear -- <your-dag-id> -c
```
In general, to run Airflow CLI in Cloud Composer:
```
gcloud composer environments run <your-composer-environment-name> --location=<your-composer-environment-location> <airflow-subcommand> -- <parameters-and-options>
```