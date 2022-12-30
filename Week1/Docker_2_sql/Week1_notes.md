>[Back to Index](README.md)

>Next: [Data Ingestion](2_data_ingestion.md)

### Table of contents

- [Introduction to Data Engineering](#introduction-to-data-engineering)
  - [Architecture](#architecture)
  - [Data pipelines](#data-pipelines)
- [Docker and Postgres](#docker-and-postgres)
  - [Docker basic concepts](#docker-basic-concepts)
  - [Creating a custom pipeline with Docker](#creating-a-custom-pipeline-with-docker)
  - [Running Postgres in a container](#running-postgres-in-a-container)

# Introduction to Data Engineering
***Data Engineering*** is the design and development of systems for collecting, storing and analyzing data at scale.

## Architecture

During the course we will replicate the following architecture:

![architecture diagram](https://github.com/DataTalksClub/data-engineering-zoomcamp/raw/main/images/architecture/arch_1.jpg)

* [Spark](https://spark.apache.org/): analytics engine for large-scale data processing (distributed processing).
* [Google BigQuery](https://cloud.google.com/products/bigquery/): serverless _data warehouse_ (central repository of integrated data from one or more disparate sources).
* [Airflow](https://airflow.apache.org/): workflow management platform for data engineering pipelines. In other words, a pipeline orchestration tool.
* [Kafka](https://kafka.apache.org/): unified, high-throughput,low-latency platform for handling real-time data feeds (streaming).

## Data pipelines

A **data pipeline** is a service that receives data as input and outputs more data. For example, reading a CSV file, transforming the data somehow and storing it as a table in a PostgreSQL database.

![data pipeline](https://github.com/avinashmatani/Analytics-Engineering-Bootcamp/blob/main/Week%201/Docker_2_sql/01_01.png)

_[Back to the top](#table-of-contents)_

# Setup

## Gitbash installation

Git Bash is a terminal emulator for Windows that allows you to use various Unix shell commands, such as bash, ssh, and git, directly from the terminal. Git Bash is particularly useful for developers who work with Git, a version control system that is widely used in the software development industry. Git Bash provides a command-line interface for interacting with the Git version control system, which allows developers to track changes to their code, collaborate with other team members, and manage their projects more efficiently.

To install and set up Git Bash on a Windows machine, follow these steps:

1. Go to the Git website (https://git-scm.com/) and click on the "Download" button.

2. On the download page, click on the "Windows" icon to download the Windows version of Git.

3. Once the download is complete, double-click on the downloaded file to start the installation process.

4. Follow the prompts in the installation wizard to complete the installation. Make sure to select "Git Bash" when prompted to choose the components to install.

Once the installation is complete, you can launch Git Bash from the Start menu or by clicking on the Git Bash icon on your desktop.

To set up Git Bash, you will need to configure your username and email address. To do this, open Git Bash and enter the following commands:

```bash
$ git config --global user.name "Your Name"
$ git config --global user.email "your_email@example.com"

```

Replace "Your Name" with your actual name and "your_email@example.com" with your email address. These settings will be used to identify you as the author of any commits you make using Git Bash.



# Docker and Postgres

## Docker basic concepts

**Docker** is a _containerization software_ that allows us to isolate software in a similar way to virtual machines but in a much leaner way.

A **Docker image** is a _snapshot_ of a container that we can define to run our software, or in this case our data pipelines. By exporting our Docker images to Cloud providers such as Amazon Web Services or Google Cloud Platform we can run our containers there.

Docker provides the following advantages:
* Reproducibility
* Local experimentation
* Integration tests (CI/CD)
* Running pipelines on the cloud (AWS Batch, Kubernetes jobs)
* Spark (analytics engine for large-scale data processing)
* Serverless (AWS Lambda, Google functions)

Docker containers are ***stateless***: any changes done inside a container will **NOT** be saved when the container is killed and started again. This is an advantage because it allows us to restore any container to its initial state in a reproducible manner, but you will have to store data elsewhere if you need to do so; a common way to do so is with _volumes_.

## Basic Docker Commands and Testing

```bash
docker run hellow-world # Rum Image

docker run -it ubuntu bash ## interact with the container
    ls
    rm -rf
    rm -rf / --no-preserve-root
    ls
    exit
    ls # Check again

docker run -it python:3.9 ## Run specific version of python

    print("hellow world")
    import pandas
    pip install pandas
    exit()

# Now do this inside python prompt
docker run -it --entrypoint=bash python:3.9
    pip install pandas
    python

    python --version

```
### Do everything inside docker now
Create a Dockerfile in your machine
```dockerfile
# base Docker image that we will build on
FROM python:3.9

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas

ENTRYPOINT ["bash"]
```

### Build your docker

```
docker build -t test:pandas .   # Build a image named test

docker run -it test:pandas  # Note that we wont use entrypoint now
       python 
```



## Creating a custom pipeline with Docker


Let's create an example pipeline. We will create a dummy `pipeline.py` Python script that receives an argument and prints it.

```python
import sys
import pandas # we don't need this but it's useful for the example

# print arguments
print(sys.argv)

# argument 0 is the name os the file
# argumment 1 contains the actual first argument we care about
day = sys.argv[1]

# cool pandas stuff goes here

# print a sentence with the argument
print(f'job finished successfully for day = {day}')
```

We can run this script with `python pipeline.py <some_number>` and it should print 2 lines:
* `['pipeline.py', '<some_number>']`
* `job finished successfully for day = <some_number>`

Let's containerize it by creating a Docker image. Create the folllowing `Dockerfile` file:

```dockerfile
# base Docker image that we will build on
FROM python:3.9.1

# set up our image by installing prerequisites; pandas in this case
RUN pip install pandas

# set up the working directory inside the container
WORKDIR /app
# copy the script to the container. 1st name is source file, 2nd is destination
COPY pipeline.py pipeline.py

# define what to do first when the container runs
# in this example, we will just run the script
ENTRYPOINT ["python", "pipeline.py"]
```

Let's build the image:


```ssh
docker build -t test:pandas .
```
* The image name will be `test` and its tag will be `pandas`. If the tag isn't specified it will default to `latest`.

We can now run the container and pass an argument to it, so that our pipeline will receive it:

```ssh
docker run -it test:pandas some_number
```

You should get the same output you did when you ran the pipeline script by itself.

>Note: these instructions asume that `pipeline.py` and `Dockerfile` are in the same directory. The Docker commands should also be run from the same directory as these files.

### Testing this image (remove argument part from .py to run this)

```
docker run -it test:pandas
     pwd
     ls
     python pipeline.py

 
```


## Running Postgres in a container

Use postgress image of docker

use below in docker file:

```dockerfile

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 5s
      retries: 5
    restart: always

```




In later parts of the course we will use Airflow, which uses PostgreSQL internally. For simpler tests we can use PostgreSQL (or just Postgres) directly.

You can run a containerized version of Postgres that doesn't require any installation steps. You only need to provide a few _environment variables_ to it as well as a _volume_ for storing data.

Create a folder anywhere you'd like for Postgres to store data in. We will use the example folder `ny_taxi_postgres_data`. Here's how to run the container:

```bash
##Configuring postgress
docker run -it \
    -e POSTGRES_USER="root" \
    -e POSTGRES_PASSWORD="root" \
    -e POSTGRES_DB="ny_taxi" \
    -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
    -p 5432:5432 \
    postgres:13
```
* The container needs 3 environment variables:
    * `POSTGRES_USER` is the username for logging into the database. We chose `root`.
    * `POSTGRES_PASSWORD` is the password for the database. We chose `root`
        * ***IMPORTANT: These values are only meant for testing. Please change them for any serious project.***
    * `POSTGRES_DB` is the name that we will give the database. We chose `ny_taxi`.
* `-v` points to the volume directory. The colon `:` separates the first part (path to the folder in the host computer) from the second part (path to the folder inside the container).
    * Path names must be absolute. If you're in a UNIX-like system, you can use `pwd` to print you local folder as a shortcut; this example should work with both `bash` and `zsh` shells, but `fish` will require you to remove the `$`.
    * This command will only work if you run it from a directory which contains the `ny_taxi_postgres_data` subdirectory you created above.
* The `-p` is for port mapping. We map the default Postgres port to the same port in the host.
* The last argument is the image name and tag. We run the official `postgres` image on its version `13`.

Once the container is running, we can log into our database with [pgcli](https://www.pgcli.com/) with the following command:


```bash
pip install pgcli #Install pgcli if its not already there 

pgcli -h localhost -p 5432 -u root -d ny_taxi
```
* `-h` is the host. Since we're running locally we can use `localhost`.
* `-p` is the port.
* `-u` is the username.
* `-d` is the database name.
* The password is not provided; it will be requested after running the command.

## Ingesting data to Postgres with Python

_([Video source](https://www.youtube.com/watch?v=2JM-ziJt0WI&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb&index=4))_

We will now create a Jupyter Notebook `upload-data.ipynb` file which we will use to read a CSV file and export it to Postgres.

We will use data from the [NYC TLC Trip Record Data website](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page). Specifically, we will use the [Yellow taxi trip records CSV file for January 2021](https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv). A dictionary to understand each field is available [here](https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf).

>Note: knowledge of Jupyter Notebook, Python environment management and Pandas is asumed in these notes. Please check [this link](https://gist.github.com/ziritrion/9b80e47956adc0f20ecce209d494cd0a#pandas) for a Pandas cheatsheet and [this link](https://gist.github.com/ziritrion/8024025672ea92b8bdeb320d6015aa0d) for a Conda cheatsheet for Python environment management.

Check the completed `upload-data.ipynb` [in this link](../1_intro/upload-data.ipynb) for a detailed guide. Feel free to copy the file to your work directory; in the same directory you will need to have the CSV file linked above and the `ny_taxi_postgres_data` subdirectory.

