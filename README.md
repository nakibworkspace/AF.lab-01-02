# Introduction to Apache Airflow

Tags: Airflow, Docker, MLOps
Class: AF-1, AF-2
Created: October 4, 2024 2:55 PM

# Objectives:

- This guide will introduce you to Apache Airflow, it’s fundamentals and how it works.
- The creations of DAGs using various operators in Airflow will get familiar.
- Will help you to build connections between databases (Postgres) and Airflow.

# Table of Contents:

- Introduction and Basic Concepts of Apache Airflow.
- Run Airflow 2.0 in docker container.
- Creating first DAG with BashOperator.
- Creating DAG with PythonOperator.
- Creating DAG with TaskFlow API.
- DAG with catchup and backfill.
- Scheduling with CRON expression.
- Airflow connection to connect Postgres.
- Use case of BranchPythonOperators.
- SubDAGs & TaskGroups.
- XComs.

## Introduction and Basic Concepts of Apache Airflow

- **What is Apache Airflow?**
    
    Apache Airflow is an open-source workflow management platform for data engineering pipelines.
    

- DAG(Directed Acyclic Graph), Task, Operator

![Screenshot 2024-10-04 at 3.00.12 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/a7ea4380-eb64-4930-97de-655235d2481c.png)

- Execution Date, Task Instance, Dag Run

![Screenshot 2024-10-02 at 12.10.07 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/745838b0-0a39-40ff-94d9-a4ed4ede970a.png)

- Airflow Task Lifecycle

![Screenshot 2024-10-02 at 12.13.36 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/41adc6b9-12ce-46f9-adb0-f5b5716ebe59.png)

![Screenshot 2024-10-02 at 12.19.39 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/c0e1d843-69b9-4ade-85f4-e23551f22ffa.png)

No status: Scheduler created empty task instance

Scheduled: scheduler determined task instance needs to run

Skipped: task is skipped

Queued: scheduler sent task to executor to run on the queue

Running: worker picked up a task and is now running it

Shutdown: task run has been shutdown

Up for retry: rerun the task

Up for reschedule: reschedule task every certain time interval

- Basic Architecture

![Screenshot 2024-10-02 at 11.46.39 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/84db0da2-13fe-477f-98b0-a90c6249fe43.png)

## Run Airflow 2.0 in docker container

1. Installing docker community edition.
2. Creating a new folder named “Airflow docker” and open it in VSCode.
3. Checking if the docker community edition & the docker compose version working fine.
    
    
    ![Screenshot 2024-10-06 at 12.34.52 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-06_at_12.34.52_AM.png)
    
4. Fetching the docker-compose.yaml file 

```markup
curl -LfO '[https://airflow.apache.org/docs/apache-airflow/2.10.4/docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/2.10.4/docker-compose.yaml)'
```

1. The docker-compose.yaml file will show up on directory.

![Screenshot 2024-10-06 at 12.37.29 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/cf2a6635-98cb-414c-b929-06c9f6f8eb1d.png)

1. Editing the yaml file AIRFLOW_CORE_EXECUTOR from CeleryExecutor to LocalExecutor. Deleting not needed lines such:

```python
AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
AIRFLOW__CELERY__BROKER_URL: redis://:@redis:6379/0
```

redis, airflow-worker, flower.

**ISSUE-01:** Was having problem regarding the edition of the yaml file. 

Solution: 

```python
docker-compose down
```

1. Copying the first line and run it on terminal. Second line is for linux user, as I am currently using mac, I don’t need to run it.
    
    ![Screenshot 2024-10-06 at 2.20.12 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-06_at_2.20.12_AM.png)
    

1. Initializing the database

```python
docker-compose up airflow-init
```

1. If you get this output, that means the airflow initialization is complete.
    
    ![Screenshot 2024-10-12 at 1.48.06 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_1.48.06_AM.png)
    

1. Running the Airflow

```python
docker-compose up -d
```

The output might look like this:

![Screenshot 2024-10-12 at 1.49.43 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_1.49.43_AM.png)

1. To check what containers are running in the background,

```python
docker ps
```

The output should look like:

![Screenshot 2024-10-12 at 1.50.35 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_1.50.35_AM.png)

Meaning the airflow-webserver, airflow-scheduler and postgres database is running.

**Launching it using the Load Balancer**

![Screenshot 2024-11-20 at 8.47.51 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-11-20_at_8.47.51_PM.png)

1. Try getting the ip of your local machine using the command.
    
    ![Screenshot 2024-11-20 at 8.48.41 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-11-20_at_8.48.41_PM.png)
    
2. Create Load Balancer and launch it.
    
    ![Screenshot 2024-11-20 at 8.49.13 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-11-20_at_8.49.13_PM.png)
    

## Creating first DAG with BashOperator

1. After logging in to the Apache Airflow using [http://localhost:8080/home](http://localhost:8080/home), the dashboard is filled with example dags.
    
    ![Screenshot 2024-10-12 at 2.18.43 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/f1876ec3-6686-4faa-857a-3bb31972b389.png)
    

1. To remove them following steps are needed to be taken:

```python
docker-compose down -v
```

- will show this output
    
    ![Screenshot 2024-10-12 at 2.19.47 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_2.19.47_AM.png)
    
- Change the AIRFLOW_CORE_LOAD_EXAMPLE from “true” to “false” in the yaml file.
    
    ![Screenshot 2024-10-12 at 2.20.11 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_2.20.11_AM.png)
    

- Then reinitialize the database as did before. Will get output like this.
    
    ![Screenshot 2024-10-12 at 2.23.29 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/d78faf2f-9fa0-4c71-adab-7cabf12d0dcc.png)
    

1. Open a new file on “dag” folder.

![Screenshot 2024-10-12 at 3.02.25 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_3.02.25_AM.png)

1. The code to create the first dag

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator

default_args= {
    "owner": "nakib",
    "retries" : 5,
    "retry_delay": timedelta(minutes=2)
}

with DAG(
    dag_id= "Our_first_dag",
    default_args= default_args, 
    description="This is our first dag that we write",
    start_date= datetime(2024, 10, 12, 2),
    schedule="@daily"
) as dag:
    task1 = BashOperator(
        task_id= "first_task",
        bash_command="hello world, this is the first task"
    )
    task1
```

1. After a successful creation of DAG, it might look like this in the airflow dag dashboard.

![Screenshot 2024-10-12 at 2.58.47 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_2.58.47_AM.png)

The grid view

![Screenshot 2024-10-12 at 3.30.32 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/52a0d8f3-8fc2-4600-84d2-4704f141e9c2.png)

1. Creating the second dag will be another easy task, using the same code with some changes.

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.bash import BashOperator

default_args= {
    "owner": "nakib",
    "retries" : 5,
    "retry_delay": timedelta(minutes=2)
}

with DAG(
    dag_id= "Our_first_dag_v2",
    default_args= default_args, 
    description="This is our first dag that we write",
    start_date= datetime(2024, 10, 12, 2),
    schedule="@daily"
) as dag:
    task1 = BashOperator(
        task_id= "first_task",
        bash_command="hello world, this is the first task"
    )
    task2 = BashOperator(
        task_id= "second_task",
        bash_command="I'm the second task and I'll be running after the first task"
    )
    task1.set_downstream(task2)
```

1. After running it, the airflow output will be seen like this
    
    ![Screenshot 2024-10-12 at 3.30.49 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/3d56d8f8-d82f-4ee4-a35e-5e1850e2106f.png)
    

1. In the grid view
    
    ![Screenshot 2024-10-12 at 3.29.57 AM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/4a5b5c8d-c55d-4360-b011-27df6b1a8a95.png)
    

## Creating DAG using python operator

1. Code

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator

default_args= {
    'owner': 'nakib',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}
def greet(name, age):
    print(f"Hello world! My name is {name}, "
          f"I am {age} years old. ")

with DAG(
    default_args=default_args,
    dag_id= 'First_dag_with_python_operator_v02',
    description= 'first dag creation with python operator',
    start_date= datetime(2024, 10, 12, 4),
    schedule= '@daily'
) as dag:
    task1 = PythonOperator(
        task_id= 'greet',
        python_callable= greet,
        op_kwargs= {'name ': 'Tom', 'age':20}
    )
    task1
```

1. After running the code and refreshing the DAG dashboard, our first dag with python operator is created!
    
    ![Screenshot 2024-10-12 at 2.14.25 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_2.14.25_PM.png)
    

> When we want to share informations between tasks, we got to use XCom. Basically we can push information in one task and push that information in other task. Here is how to do it.
> 

1. The function “greet” pulled the information that was pushed by the function “get_name”.

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator

default_args= {
    'owner': 'nakib',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}
def greet(ti, age):  #ti -> task instance
    name= ti.xcom_pull(task_ids='get_name')
    print(f"Hello world! My name is {name}, "
          f"I am {age} years old. ")

def get_name():
    return 'Jerry'

with DAG(
    default_args=default_args,
    dag_id= 'First_dag_with_python_operator_v03',
    description= 'first dag creation with python operator',
    start_date= datetime(2024, 10, 12, 4),
    schedule= '@daily'
) as dag:
    task1 = PythonOperator(
        task_id= 'greet',
        python_callable= greet,
        op_kwargs= {'age':20}
    )
    task2= PythonOperator(
        task_id= 'get_name',
        python_callable=get_name
    )
    task2 >> task1

```

1. DAG creation done.

![Screenshot 2024-10-12 at 2.36.20 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_2.36.20_PM.png)

## Creating DAG with Taskflow API

```python
from datetime import datetime, timedelta

from airflow.decorators import dag, task

default_args= {
    'owner': 'nakib',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}

@dag(dag_id= "first_dag_with_taskflow_api",
     default_args=default_args,
     start_date=datetime(2024,10,12,6),
     schedule='@daily')

def hello_world_etl():

    @task()
    def get_name():
        return "Jerry"
    
    @task()
    def get_age():
        return 19
    
    @task()
    def greet(name, age):
        print(f"Hello world, My name is {name}, "
              f"and I am {age} years old")
        
    name= get_name()
    age= get_age()
    greet(name=name, age=age)

greet_dag = (hello_world_etl)
```

To get the multiple values out of a function, slight changes are needed.

1. Change the greet function.

```python
@task()
    def greet(first_name, last_name, age):
        print(f"Hello world, My name is {first_name} {last_name}, "
              f"and I am {age} years old")
```

1. Set multiple output parameter to true in the get_name function.

```python
@task(multiple_outputs=True)
    def get_name():
        return {
            'first_name' : 'Jerry',
            'last_name': 'fridmann'
        }
```

1. Get the name dictionary from get_name function and pass them to greet function.

```python
name_dict= get_name()
    age= get_age()
    greet(first_name= name_dict['first_name'],
          last_name= name_dict['last_name'],
          age=age)
```

1. Update the dag id version and run it.
2. It will create a new DAG.

## DAG with catchup and backfill

Both *catchup* and *backfill* are ways to run Airflow DAGs in the past. Here is an article to know the difference between both,

[https://faun.pub/airflow-catchup-vs-backfill-22f3a7d4ba6f](https://faun.pub/airflow-catchup-vs-backfill-22f3a7d4ba6f)

```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.bash import BashOperator

default_args= {
    'owner': 'nakib',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}

with DAG(
    dag_id= 'dag_with_catchup_backfilll_v01',
    default_args=default_args,
    start_date= datetime(2024, 10, 1),
    schedule='@daily',
    catchup=True

) as dag:
    task1= BashOperator(
        task_id= 'task1',
        bash_command='Echo this is simple bash command'
    )
```

![Screenshot 2024-10-12 at 3.58.18 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/b7decbc2-2807-4c8b-9bc2-c1bd9b12dfb2.png)

The DAG has started since October 1st, 2024 and restarting once in everyday.

![Screenshot 2024-10-12 at 4.05.17 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.05.17_PM.png)

```python
with DAG(
    dag_id= 'dag_with_catchup_backfilll_v02',
    default_args=default_args,
    start_date= datetime(2024, 10, 1),
    schedule='@daily',
    catchup=False
```

You can change the catchup to False to make it backfill, update the version and run it.

![Screenshot 2024-10-12 at 4.06.17 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/4332fbba-d717-4e77-8a05-ca6933b4572f.png)

You can still run the DAG using backfill. To know how follow the following steps,

1. Find the airflow scheduler container on VSCode using the command

```python
docker ps
```

![Screenshot 2024-10-12 at 4.12.35 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.12.35_PM.png)

1. Copy the scheduler id and run this command

```python
docker exec -it ff4bda0267bd /bin/bash
```

1. The output command terminal should be like this.

```python
airflow@ff4bda0267bd:/opt/airflow$
```

1. Run this command

```python
airflow dags backfill -s 2024-10-01 -e 2024-10-08
```

1. exit command to exit the container. Will see dag run from the starting date to end date.
    
    ![Screenshot 2024-10-12 at 3.58.18 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/dcfd69dd-618a-4b71-b2eb-363eccf03905.png)
    

The code in dag dashboard.

![Screenshot 2024-10-12 at 4.22.01 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.22.01_PM.png)

## Scheduling with Cron Expression

In Airflow, creating a DAG needs a schedule interval parameter. Which receives Cron Expression as String or datetime.timedelta as Object.

![Screenshot 2024-10-12 at 4.26.29 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.26.29_PM.png)

**What is a CRON Expression?**

A CRON expression is a string comprising five fields separated by white space that represents a set of times. Normally as a schedule to execute some routine.

![Screenshot 2024-10-12 at 4.30.08 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.30.08_PM.png)

Airflow already provides some presets for CRON expression. 

![Screenshot 2024-10-12 at 4.31.01 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.31.01_PM.png)

But if you want to generate customised CRON expressions, you can make it in this website.

[https://crontab.guru](https://crontab.guru/)

## Airflow connection to connect Postgres

![Screenshot 2024-10-12 at 4.39.05 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_4.39.05_PM.png)

- Go to the Admin > Connection > Add connection.
    
    ![Screenshot 2024-10-12 at 8.54.48 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_8.54.48_PM.png)
    

- Change the yaml file, add port.
    
    ![Screenshot 2024-10-12 at 5.48.08 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_5.48.08_PM.png)
    

- Download dbeaver and install it from here.

[https://dbeaver.io/download/](https://dbeaver.io/download/)

- Start the airflow postgres container using the command.

```python
docker-compose up -d —no-deps —build postgres
```

- After completation the output would look like,
    
    ![Screenshot 2024-10-12 at 5.50.00 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_5.50.00_PM.png)
    

- Open new file and run this code.

```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator

default_args= {
    "owner": "nakib",
    "retries" : 5,
    "retry_delay": timedelta(minutes=2)
}

with DAG(
    dag_id= "dag_with_postgres_operator_v02",
    default_args= default_args, 
    description="The first dag with postgres",
    start_date= datetime(2024, 10, 12),
    schedule= '0 0 * * *'
) as dag:
    task1 = PostgresOperator(
        task_id= 'create_postgres_table',
        postgres_conn_id='postgres_localhost',
        sql='''
            create table if not exist dag run (
                dt date,
                dag_id character varying,
                primary key (ds, dag_id)
                )'''
                
    )
    task1
```

- Save it and run it, it will show a new DAG in the dashboard.
    
    ![Screenshot 2024-10-12 at 9.05.52 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_9.05.52_PM.png)
    

- The grid view
    
    ![Screenshot 2024-10-12 at 9.06.10 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_9.06.10_PM.png)
    

- To insert informations to table add this bit of code and fix their task dependencies.

```python
task2 = PostgresOperator(
        task_id= 'insert_into_table',
        postgres_conn_id='postgres_localhost',
        sql="""
            insert into dag_runs (ds, dag_id) values ('{{ds}}','{{dag.dag_id}}')
        """
    )
    task1 >> task2
```

- In the same way you can delete content from table using this part of code.

```python
task3= PostgresOperator(
        task_id= 'delete_data_from_table',
        postgres_conn_id='postgres_localhost',
        sql="""
            delete from dag_runs where dt= '{{ds}}' and dag_id='{{dag.dag_id}}'
        """
    )
    task1 >> task3 >> task2
```

## Installation of python requirements via extending airflow docker image

- Open the docker project folder and open a new file “requirements.txt”.
    
    ![Screenshot 2024-10-12 at 9.41.17 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_9.41.17_PM.png)
    

- Create a docker file in project root folder and give it as input

```docker
FROM apache/airflow:2.0.1
COPY requirements.txt /requirements.txt
RUN pip install --user --upgrade pip
RUN pip install --no-cache-dir --user -r /requirements.txt
```

- In terminal run this command

```python
docker build . --tag extending_airflow:latest
```

Issue:

![Screenshot 2024-10-12 at 11.05.55 PM.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-10-12_at_11.05.55_PM.png)

Solution:

Run this following commands:

```python
docker build --platform linux/amd64 . --tag extending_airflow:latest
```

- The code to execute the DAG

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator

default_args = {
    'owner': 'nakib',
    'retries': 5,
    'retry_delay': timedelta(minutes=5)
}

def get_sklearn():
    import sklearn
    print(f'scikit-learn with version: {sklearn.__version__}')

with DAG( 
    default_args=default_args,
    dag_id= 'dag_with_python_dependencies_v01',
    start_date= datetime(2024,10,8),
    schedule='@daily'
) as dag:
    
    get_sklearn= PythonOperator(
        task_id= 'get_sklearn',
        python_callable= get_sklearn
    )

    get_sklearn
```

## **BranchPythonOperator:**

While scheduling tasks in airflow, sometimes there is a need of workflow to branch, or go through a certain path based on arbitrary condition. This is where the use of BranchPythonOperator comes for solution.

![Screenshot 2024-12-14 at 15.56.47.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/fcd95cd7-2927-42fa-8b4b-0b615f825c7e.png)

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import BranchPythonOperator
from datetime import datetime, timedelta

default_args={
    'start_date': datetime(2024,1,1)
}

def choose_best_model():
    accuracy=6
    if accuracy > 5:
        return 'accurate'
    elif accuracy <=5 :
        return 'inaccurate'

with DAG('branching', schedule_interval='@daily', default_args=default_args, catchup=False) as dag:
    
    choose_best_model= BranchPythonOperator(
        task_id= 'choose_best_model',
        python_callable=choose_best_model
    )

    accurate= BashOperator(
        task_id='accurate',
        bash_command='the model witll choose this if accuaracy is more than 5'
    )

    inaccurate= BashOperator(
        task_id='inaccurate',
        bash_command='model will choose this is accuracy is less than 5'
    )

    choose_best_model >> [accurate, inaccurate]
```

The code snippet defines a function where a condition of accuracy is given. If the condition satisfies, the accurate operator will succeed else the inaccurate operator.

Defining a function is a must thing to do here in branching. Otherwise it will send error for python_callable.

![Screenshot 2024-12-14 at 15.28.53.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-14_at_15.28.53.png)

If everything is done right it will show this as result:

![Screenshot 2024-12-14 at 15.35.14.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-14_at_15.35.14.png)

### SubDags:

When adding exactly same set of tasks regularly to every DAG, or to group several tasks into a single logical unit, the use of SubDAGs arise.

![Screenshot 2024-12-15 at 21.58.18.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_21.58.18.png)

The creation SubDAGs are divided into three phases.

1. Defining the subdag function.

```python
import pendulum
from airflow import DAG
from airflow.operators.empty import EmptyOperator

def subdag(parent_dag_name, child_dag_name, args):
    
    dag_subdag= DAG(
        dag_id=f'{parent_dag_name}.{child_dag_name}',
        default_args=args,
        start_date=pendulum.datetime(2024,1,1, tz='UTC'),
        catchup=False,
        schedule_interval='@daily',
    )

    for i in range(5):
        EmptyOperator(
            task_id= f'{child_dag_name}-task-{i+1}',
            default_args=args,
            dag=dag_subdag,
        )
        return dag_subdag
```

1. Parent DAG definition and putting the tasks in it with dependencies.

```python
from airflow import DAG
from airflow.example_dags.subdags.subdag import subdag
from airflow.operators.empty import EmptyOperator
from airflow.operators.subdag import SubDagOperator
from datetime import datetime, timedelta
from airflow.utils.dates import days_ago

DAG_NAME= 'example_subdag_operator'

with DAG(
    dag_id=DAG_NAME,
    default_args={"retries":2},
    start_date=days_ago(2),
    schedule_interval="@once",
    tags=['example'],
) as dag:

    start= EmptyOperator(
        task_id='start'
    )

    section_1= SubDagOperator(
        task_id='section-1',
        subdag=subdag(DAG_NAME, 'section-1', dag.default_args),
    )

    some_other_task= EmptyOperator(
        task_id= 'some-other-task',
    )

    section_2= SubDagOperator(
        task_id= 'section-2',
        subdag=subdag(DAG_NAME, 'section-2', dag.default_args),
    )

    end= EmptyOperator(
        task_id='end',
    )

    start >> section_1 >> some_other_task >> section_2 >> end
```

The output shows parent DAGs which consists the child DAGs in it.

![Screenshot 2024-12-15 at 22.31.04.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_22.31.04.png)

### TaskGroups:

TaskGroups work similarly as SubDAGs, but with time TaskGroups has taken over SubDAGs with it’s efficiency and reliable behaviours.

![Screenshot 2024-12-15 at 22.21.27.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_22.21.27.png)

We have divided the TaskGroup in few steps for better understanding of it.

Firstly, we tried to make the usual sequential code for Airflow scheduling.

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.empty import EmptyOperator
from datetime import datetime , timedelta
from airflow.utils.task_group import TaskGroup

default_args= {
    'owner': 'local',
    'retries': 5,
    'retry_delay': timedelta(minutes=2),
}

with DAG(
    dag_id='Taskgroups_dag',
    default_args=default_args,
    description='this is the tutorial dag for airflow taskgroups',
    start_date= datetime(2024,1,1),
    schedule='@daily',
)as dag:

    start= EmptyOperator(task_id='START')
    a= EmptyOperator(task_id='Task_A')
    a1= EmptyOperator(task_id='Task_A1')
    b= EmptyOperator(task_id='Task_B')
    c= EmptyOperator(task_id='Task_C')
    d= EmptyOperator(task_id='Task_D')
    e= EmptyOperator(task_id='Task_E')
    f= EmptyOperator(task_id='Task_F')
    g= EmptyOperator(task_id='Task_G')
    end= EmptyOperator(task_id='END')

start >> a >> a1 >> b >> c >> d >> e >> f >> g >> end

```

Which led us to get this output.

![Screenshot 2024-12-15 at 21.07.53.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_21.07.53.png)

Then, the Empty Operators (a, a1, b and c) are taken into a single group named gr_1. And we also have set the task dependencies for a and a1, where the other two tasks are independent.

```python
with TaskGroup("A-A1", tooltip="Task group for A and A1") as gr_1:
        a= EmptyOperator(task_id='Task_A')
        a1= EmptyOperator(task_id='Task_A1')
        b= EmptyOperator(task_id='Task_B')
        c= EmptyOperator(task_id='Task_C')
        a>>a1

start >> gr_1 >> d >> e >> f >> g >> end
```

The output will change to this:

![Screenshot 2024-12-15 at 21.13.21.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_21.13.21.png)

And by clicking on the A-A1 task, we get the full view of the operations that’s happening within this.

![Screenshot 2024-12-15 at 21.13.36.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-15_at_21.13.36.png)

## XComs:

XComs stands for cross communication and helps you to share messages and small amount of datas between tasks. An object where the value you want to exchange is stored inside the meta database of Airflow. So each time you are interacting with the XComs, you are interacting with the database of airflow. XComs mainly consists two arguments: Return_key which is the identity and Value, which holds the variable.

![Screenshot 2024-12-18 at 23.49.22.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-18_at_23.49.22.png)

![Screenshot 2024-12-18 at 23.46.07.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-18_at_23.46.07.png)

For the purpose of learning, we can think of these DAGs where best of these three models will be chosen. When we process the values of three models and find an answer we push the value to XComs to send it to the next task.

To get the visual of the diagram, here’s the code snippet for that, where the models will choose any number between for their accuracy in between 0.1 to 10.

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from airflow.operators.python import PythonOperator

from random import uniform
from datetime import datetime, timedelta

default_args= {
    'start_date': datetime(2024,1,1)
}

def training_model():
    accuracy= uniform(0.1, 10.0)
    print(f"model's accuracy: {accuracy}")
    return accuracy

def choose_best_model():
    print(f"choose best model")

with DAG(
    dag_id='x_coms_dag',
    default_args=default_args,
    schedule_interval='@daily',
    catchup=False
) as dag:
    downloading_data= BashOperator(
        task_id= 'downloading_data',
        bash_command='sleep 3'
    )

    training_model_task = [PythonOperator(
        task_id= f'training_model{task}',
        python_callable=training_model
    ) for task in ['A','B','C']]

    choose_model= PythonOperator(
        task_id= 'choose_model',
        python_callable=choose_best_model
    )

    downloading_data >> training_model_task >> choose_model
```

The DAGs will be created:

![Screenshot 2024-12-18 at 20.59.45.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-18_at_20.59.45.png)

We can check the XComs from the Action tab.

![Screenshot 2024-12-18 at 21.01.16.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-18_at_21.01.16.png)

You can also push the XComs with your preferred return_key. To do that some slight changes like you need to access the task instance argument, which helps you to create a XCom with parameters key and value. 

```python
def training_model(task_instance):
    accuracy= uniform(0.1, 10.0)
    print(f"model's accuracy: {accuracy}")
    task_instance.xcom_push(key='model_accuracy', value=accuracy)
```

![Screenshot 2024-12-18 at 21.08.34.png](Introduction%20to%20Apache%20Airflow%20115d4718bce880f0981dea9746a2826a/Screenshot_2024-12-18_at_21.08.34.png)

**To call a XCom value in any function,**

```markup
task_instance.xcom_pull(key="identifier as string", task_ids="task-1")
```

# Conclusion:

In this lab we completed the fundamentals of Apache Airflow including the theory and DAGs creation.