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

![DAG Example](images/image-01.png)

- Execution Date, Task Instance, Dag Run

![DAG Example](images/image-02.png)

- Airflow Task Lifecycle

![DAG Example](images/image-03.png)

![DAG Example](images/image-04.png)

No status: Scheduler created empty task instance

Scheduled: scheduler determined task instance needs to run

Skipped: task is skipped

Queued: scheduler sent task to executor to run on the queue

Running: worker picked up a task and is now running it

Shutdown: task run has been shutdown

Up for retry: rerun the task

Up for reschedule: reschedule task every certain time interval

- Basic Architecture

![DAG Example](images/image-05.png)

## Run Airflow 2.0 in docker container

1. Installing docker community edition.
2. Creating a new folder named “Airflow docker” and open it in VSCode.
3. Checking if the docker community edition & the docker compose version working fine.
    
    
    ![DAG Example](images/image-06.2.png)
    
4. Fetching the docker-compose.yaml file 

```markup
curl -LfO '[https://airflow.apache.org/docs/apache-airflow/2.10.4/docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/2.10.4/docker-compose.yaml)'
```

1. The docker-compose.yaml file will show up on directory.

![DAG Example](images/image-06.png)

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
    
    ![DAG Example](images/image-07.png)
    

1. Initializing the database

```python
docker-compose up airflow-init
```

1. If you get this output, that means the airflow initialization is complete.
    
    ![DAG Example](images/image-08.png)
    

1. Running the Airflow

```python
docker-compose up -d
```

The output might look like this:

![DAG Example](images/image-09.png)

1. To check what containers are running in the background,

```python
docker ps
```

The output should look like:

![DAG Example](images/image-10.png)

Meaning the airflow-webserver, airflow-scheduler and postgres database is running.

**Launching it using the Load Balancer**

![DAG Example](images/image-11.png)

1. Try getting the ip of your local machine using the command.
    
    ![DAG Example](images/image-12.png)
    
2. Create Load Balancer and launch it.
    
    ![DAG Example](images/image-13.png)
    

## Creating first DAG with BashOperator

1. After logging in to the Apache Airflow using [http://localhost:8080/home](http://localhost:8080/home), the dashboard is filled with example dags.
    
    ![DAG Example](images/image-14.png)
    

1. To remove them following steps are needed to be taken:

```python
docker-compose down -v
```

- will show this output
    
    ![DAG Example](images/image-15.png)
    
- Change the AIRFLOW_CORE_LOAD_EXAMPLE from “true” to “false” in the yaml file.
    
    ![DAG Example](images/image-16.png)
    

- Then reinitialize the database as did before. Will get output like this.
    
    ![DAG Example](images/image-17.1.png)
    

1. Open a new file on “dag” folder.

![DAG Example](images/image-18.png)

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

![DAG Example](images/image-19.png)

The grid view

![DAG Example](images/image-20.png)

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
    
    ![DAG Example](images/image-21.png)
    

1. In the grid view
    
    ![DAG Example](images/image-22.png)
    

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
    
    ![DAG Example](images/image-23.png)
    

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

![DAG Example](images/image-24.png)

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

![DAG Example](images/image-25.png)

The DAG has started since October 1st, 2024 and restarting once in everyday.

![DAG Example](images/image-26.png)

```python
with DAG(
    dag_id= 'dag_with_catchup_backfilll_v02',
    default_args=default_args,
    start_date= datetime(2024, 10, 1),
    schedule='@daily',
    catchup=False
```

You can change the catchup to False to make it backfill, update the version and run it.

![DAG Example](images/image-27.png)

You can still run the DAG using backfill. To know how follow the following steps,

1. Find the airflow scheduler container on VSCode using the command

```python
docker ps
```

![DAG Example](images/image-28.png)

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
    
    ![DAG Example](images/image-29.png)
    

The code in dag dashboard.

![DAG Example](images/image-30.png)

## Scheduling with Cron Expression

In Airflow, creating a DAG needs a schedule interval parameter. Which receives Cron Expression as String or datetime.timedelta as Object.

![DAG Example](images/image-31.png)

**What is a CRON Expression?**

A CRON expression is a string comprising five fields separated by white space that represents a set of times. Normally as a schedule to execute some routine.

![DAG Example](images/image-32.png)

Airflow already provides some presets for CRON expression. 

![DAG Example](images/image-33.png)

But if you want to generate customised CRON expressions, you can make it in this website.

[https://crontab.guru](https://crontab.guru/)

## Airflow connection to connect Postgres

![DAG Example](images/image-34.png)

- Go to the Admin > Connection > Add connection.
    
    ![DAG Example](images/image-35.png)
    

- Change the yaml file, add port.
    
    ![DAG Example](images/image-36.png)
    

- Download dbeaver and install it from here.

[https://dbeaver.io/download/](https://dbeaver.io/download/)

- Start the airflow postgres container using the command.

```python
docker-compose up -d —no-deps —build postgres
```

- After completation the output would look like,
    
    ![DAG Example](images/image-37.png)
    

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
    
    ![DAG Example](images/image-38.png)
    

- The grid view
    
    ![DAG Example](images/image-39.png)
    

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
    
    ![DAG Example](images/image-40.png)
    

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

![DAG Example](images/image-41.png)

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

![DAG Example](images/image-42.png)

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

![DAG Example](images/image-43.png)

If everything is done right it will show this as result:

![DAG Example](images/image-44.png)

### SubDags:

When adding exactly same set of tasks regularly to every DAG, or to group several tasks into a single logical unit, the use of SubDAGs arise.

![DAG Example](images/image-45.png)

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

![DAG Example](images/image-46.png)

### TaskGroups:

TaskGroups work similarly as SubDAGs, but with time TaskGroups has taken over SubDAGs with it’s efficiency and reliable behaviours.

![DAG Example](images/image-47.png)

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

![DAG Example](images/image-48.png)

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

![DAG Example](images/image-49.png)

And by clicking on the A-A1 task, we get the full view of the operations that’s happening within this.

![DAG Example](images/image-50.png)

## XComs:

XComs stands for cross communication and helps you to share messages and small amount of datas between tasks. An object where the value you want to exchange is stored inside the meta database of Airflow. So each time you are interacting with the XComs, you are interacting with the database of airflow. XComs mainly consists two arguments: Return_key which is the identity and Value, which holds the variable.

![DAG Example](images/image-51.png)

![DAG Example](images/image-52.png)

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

![DAG Example](images/image-53.png)

We can check the XComs from the Action tab.

![DAG Example](images/image-54.png)

You can also push the XComs with your preferred return_key. To do that some slight changes like you need to access the task instance argument, which helps you to create a XCom with parameters key and value. 

```python
def training_model(task_instance):
    accuracy= uniform(0.1, 10.0)
    print(f"model's accuracy: {accuracy}")
    task_instance.xcom_push(key='model_accuracy', value=accuracy)
```

![DAG Example](images/image-55.png)

**To call a XCom value in any function,**

```markup
task_instance.xcom_pull(key="identifier as string", task_ids="task-1")
```

# Conclusion:

In this lab we completed the fundamentals of Apache Airflow including the theory and DAGs creation.
