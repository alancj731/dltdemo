## Make sure you are using Python 3.9-3.13 and have pip installed
```bash
python --version
pip --version
```

## Install uv and reate virtual enviroment
```bash
# first navigate to your project folder 
pip install uv
uv venv
source .venv/bin/activate
# now you are in your virtual enviroment, enter deactivate to exit
```

## Install dlt
```bash
uv pip install -U dlt "dlt[postgres]"
# use dlt to create template for loading data from one sql database(source) to a postgres(destination) database
dlt init sql_database postgres --eject
# install required packages for the template
uv pip install -r requirements.txt
# install psycopg2
uv pip install psycopg2
```
# Setup the secrets.toml file:
edit and set the required value in .dlt/secrets.toml file, below shows an example
```toml
[sources.sql_database.credentials]
drivername = "postgresql+psycopg2" #
database = "sourcedatabasename" # fill this in!
password = "postgres" # fill this in!
username = "postgres" # fill this in!
host = "xxx.us-west-2.elb.amazonaws.com" # fill this in!
port = 5432 # fill this in!

[destination.postgres.credentials]
database = "destdatabasename" # fill this in!
password = "postgres" # fill this in!
username = "postgres" # fill this in!
host = "127.0.0.1" # fill this in!
port = 5432 # fill this in!
connect_timeout = 15
```

## change the sql_database_pipeline.py to customize the pipeline
The example pipeline copy a "tags" table from source db to destination db
```py
#...
def load_select_tables_from_database() -> None:
    """Use the sql_database source to reflect an entire database schema and load select tables from it.

    This example sources data from the public Rfam MySQL database.
    """
    # Create a pipeline, data loaded will go into a new schema called "rfam_data"
    pipeline = dlt.pipeline(pipeline_name="rfam", destination='postgres', dataset_name="rfam_data")

    # Configure the source to load a few select tables incrementally
    credentials = dlt.secrets["sources"]["sql_database"]["credentials"] # load settings from .dlt/secrets.toml 
    source_1 = sql_database(credentials).with_resources("tags")

    # Run the pipeline. The merge write disposition merges existing rows in the destination by primary key
    info = pipeline.run(source_1, write_disposition="merge")
    print(info)
#...

if __name__ == "__main__":
    # Load selected tables with different settings
    load_select_tables_from_database()
```

## Run the pipeline
```bash
 python sql_database_pipeline.py
 ```

