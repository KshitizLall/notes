# Apache Airflow - Complete Interview Guide

## Table of Contents
1. [Introduction to Apache Airflow](#introduction-to-apache-airflow)
2. [Core Concepts](#core-concepts)
3. [DAG (Directed Acyclic Graph)](#dag-directed-acyclic-graph)
4. [Operators](#operators)
5. [Tasks](#tasks)
6. [Task Dependencies](#task-dependencies)
7. [Sensors](#sensors)
8. [Hooks](#hooks)
9. [Executors](#executors)
10. [Scheduling](#scheduling)
11. [XComs (Cross-Communications)](#xcoms-cross-communications)
12. [Variables & Connections](#variables--connections)
13. [Branching & Conditional Logic](#branching--conditional-logic)
14. [Error Handling & Retries](#error-handling--retries)
15. [Airflow Architecture](#airflow-architecture)
16. [Common Interview Questions](#common-interview-questions)

---

## Introduction to Apache Airflow

### What is Apache Airflow?
Apache Airflow is an **open-source workflow orchestration platform** that allows you to:
- Define, schedule, and monitor workflows programmatically
- Handle complex dependencies between tasks
- Scale across multiple machines
- Provide rich visibility and monitoring capabilities

### Key Use Cases
- **Data Pipelines**: ETL/ELT processes
- **Machine Learning**: Model training and deployment workflows
- **Data Engineering**: Data ingestion, transformation, and loading
- **Batch Processing**: Processing large datasets on a schedule
- **Microservice Orchestration**: Coordinating multiple services
- **Reporting**: Scheduled data processing and report generation

### Why Airflow?

| Feature | Airflow | Cron | Oozie | Other Schedulers |
|---------|---------|------|-------|------------------|
| Programmatic | Yes | No | YAML | Varies |
| Complexity | High workflows | Limited | Complex | Varies |
| Dependency Management | Excellent | Limited | Good | Good |
| Monitoring | Excellent | Poor | Decent | Varies |
| Scalability | High | Low | High | Varies |
| Community | Large | N/A | Declining | Varies |
| Learning Curve | Moderate-High | Low | High | Varies |

### Airflow Philosophy
1. **Dynamic**: Pipelines are defined as code (Python)
2. **Extensible**: Easy to extend with custom operators
3. **Elegant**: Clean, parameterized code organization
4. **Scalable**: Can run hundreds of thousands of tasks

---

## Core Concepts

### DAG (Directed Acyclic Graph)
A DAG is the basic unit of work in Airflow. It represents a workflow with:
- **Directed**: Tasks flow in one direction
- **Acyclic**: No circular dependencies allowed
- **Graph**: Visual representation of task relationships

### Task
A task is a single unit of work in a DAG. Each task is an instance of an operator.

### Operator
An operator is a template for a task that defines what the task should do.

### Task Instance
A specific execution of a task at a particular time.

### Execution Date (run_date/logical_date)
The date/time that a DAG run is supposed to represent. Important for data processing dates.

---

## DAG (Directed Acyclic Graph)

### Creating a Basic DAG

#### Method 1: Using Context Manager (Modern Approach)
```python
from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonOperator

def hello_world():
    print("Hello World!")

with DAG(
    dag_id='my_first_dag',
    default_args={
        'owner': 'airflow',
        'start_date': datetime(2024, 1, 1),
        'retries': 1,
    },
    description='A simple DAG',
    schedule_interval='@daily',  # Run daily
    catchup=False,  # Don't backfill past runs
) as dag:

    task1 = PythonOperator(
        task_id='say_hello',
        python_callable=hello_world,
    )
```

#### Method 2: Using DAG Constructor (Traditional Approach)
```python
from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonOperator

dag = DAG(
    dag_id='my_first_dag',
    default_args={
        'owner': 'airflow',
        'start_date': datetime(2024, 1, 1),
        'retries': 1,
    },
    description='A simple DAG',
    schedule_interval='@daily',
    catchup=False,
)

def hello_world():
    print("Hello World!")

task1 = PythonOperator(
    task_id='say_hello',
    python_callable=hello_world,
    dag=dag,
)
```

### DAG Parameters Explained

```python
with DAG(
    dag_id='example_dag',  # Unique identifier for the DAG

    # Default arguments applied to all tasks
    default_args={
        'owner': 'data_team',  # Who owns this DAG
        'start_date': datetime(2024, 1, 1),  # When to start scheduling
        'end_date': datetime(2024, 12, 31),  # When to stop (optional)
        'retries': 2,  # Number of retries on failure
        'retry_delay': timedelta(minutes=5),  # Wait before retry
        'depends_on_past': False,  # Does task depend on previous execution?
        'wait_for_downstream': False,  # Wait for downstream tasks?
        'pool': 'default_pool',  # Resource pool for task execution
        'priority_weight': 1,  # Priority for scheduling
        'queue': 'default',  # Celery queue
        'email': ['admin@example.com'],  # Email on failure
        'email_on_failure': True,
        'email_on_retry': False,
    },

    description='Example DAG with full parameters',

    # Schedule interval
    schedule_interval='0 2 * * *',  # Cron format (2 AM daily)
    # OR use preset strings: '@hourly', '@daily', '@weekly', '@monthly'
    # OR use timedelta: timedelta(hours=1)
    # OR None for manual triggering

    catchup=False,  # Don't run past scheduled intervals on creation

    tags=['etl', 'production'],  # For organizing DAGs

    # Additional parameters
    doc_md='# DAG Documentation\nThis is a sample DAG',
    max_active_runs=1,  # Max parallel DAG runs
    max_active_tasks=10,  # Max parallel tasks in a run
    default_view='graph',  # Default graph view
    orientation='LR',  # Left to Right graph orientation
) as dag:
    pass
```

### default_args Inheritance
Tasks inherit values from default_args, but task-level parameters override them:

```python
default_args = {
    'owner': 'airflow',
    'retries': 2,
}

with DAG(
    dag_id='example',
    default_args=default_args,
    schedule_interval='@daily',
) as dag:

    # Task will have 2 retries (from default_args)
    task1 = PythonOperator(
        task_id='task1',
        python_callable=lambda: print('Task 1'),
    )

    # Task will have 0 retries (overrides default_args)
    task2 = PythonOperator(
        task_id='task2',
        python_callable=lambda: print('Task 2'),
        retries=0,
    )
```

### DAG Lifecycle

```
DAG Creation → Scheduling → Task Instantiation → Execution → Monitoring
```

1. **DAG Definition**: Python file is parsed
2. **Registration**: DAG is registered in Airflow's metadata database
3. **Scheduling**: Scheduler checks if DAG should run based on schedule_interval
4. **Task Instantiation**: Task instances are created for the DAG run
5. **Execution**: Executor runs tasks according to dependencies
6. **Completion**: DAG run is marked as succeeded/failed

---

## Operators

### What is an Operator?
An operator is a class that encapsulates the logic to perform a specific action. It's a template for a task.

### Common Operators

#### 1. PythonOperator
Execute Python functions:

```python
from airflow.operators.python import PythonOperator

def my_function(ti):  # ti = task instance
    print("Executing Python function")
    return "result"

task = PythonOperator(
    task_id='python_task',
    python_callable=my_function,
    op_args=[1, 2],  # Positional arguments
    op_kwargs={'key': 'value'},  # Keyword arguments
)
```

#### 2. BashOperator
Execute Bash commands:

```python
from airflow.operators.bash import BashOperator

task = BashOperator(
    task_id='bash_task',
    bash_command='echo "Hello from Bash"',
)
```

#### 3. EmailOperator
Send emails:

```python
from airflow.operators.email import EmailOperator

task = EmailOperator(
    task_id='send_email',
    to='admin@example.com',
    subject='DAG Completed',
    html_content='<h3>DAG Execution Successful</h3>',
)
```

#### 4. DummyOperator / EmptyOperator
Placeholder task for control flow:

```python
from airflow.operators.empty import EmptyOperator

task = EmptyOperator(task_id='dummy_task')
```

#### 5. SQLOperator
Execute SQL queries:

```python
from airflow.operators.sql import SQLExecuteQueryOperator

task = SQLExecuteQueryOperator(
    task_id='query_task',
    sql='SELECT COUNT(*) FROM table;',
    conn_id='postgres_conn',
)
```

#### 6. S3Operator
Interact with AWS S3:

```python
from airflow.providers.amazon.aws.operators.s3 import S3CopyObjectOperator

task = S3CopyObjectOperator(
    task_id='s3_copy',
    source_bucket_name='source-bucket',
    source_bucket_key='source/key',
    dest_bucket_name='dest-bucket',
    dest_bucket_key='dest/key',
    aws_conn_id='aws_default',
)
```

#### 7. HttpOperator
Make HTTP requests:

```python
from airflow.providers.http.operators.http import SimpleHttpOperator

task = SimpleHttpOperator(
    task_id='http_task',
    http_conn_id='http_default',
    endpoint='/api/data',
    method='POST',
    data=json.dumps({'key': 'value'}),
)
```

### Operator vs Hook
- **Operator**: Defines a task (what to do)
- **Hook**: Provides a reusable connection (how to connect)

```python
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook

def extract_data():
    hook = PostgresHook(postgres_conn_id='postgres_default')
    result = hook.get_records("SELECT * FROM table")
    return result

task = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
)
```

### Operator Categories

| Category | Purpose | Examples |
|----------|---------|----------|
| Action | Perform actual work | PythonOperator, BashOperator, EmailOperator |
| Transfer | Move data | S3CopyObjectOperator, BigQueryLoadJobOperator |
| Sensor | Wait for condition | FileSensor, TimeDeltaSensor, S3KeySensor |
| Check | Validate data | SQLCheckOperator, ThresholdCheckOperator |
| Sub-DAG | Embed DAG | SubDagOperator |

---

## Tasks

### Task Definition
A task is an instance of an operator. It has:
- **task_id**: Unique identifier within a DAG
- **Operator**: Type of work to perform
- **Parameters**: Configuration for the operator
- **Dependencies**: Which tasks it depends on

```python
task = PythonOperator(
    task_id='unique_task_id',  # Must be unique in DAG
    python_callable=my_function,
    depends_on_past=True,  # Task depends on previous run success
    pool='limited_pool',  # Use specific resource pool
    priority_weight=10,  # Higher = higher priority
    queue='priority_queue',  # Celery queue
    trigger_rule='all_success',  # When to trigger this task
)
```

### Task Instance
A task instance is a specific execution of a task:

```python
# In code, you reference the task
task1 = PythonOperator(task_id='task1', ...)

# When the DAG runs, Airflow creates a TaskInstance
# TaskInstance = DAG Run + Task
# So: task1 execution on 2024-01-01 is a TaskInstance
```

### Task States

```
                 no_status
                    |
          ┌─────────┴─────────┐
          |                   |
       scheduled          queued
          |                   |
          └─────────┬─────────┘
                    |
                 running
                 /  |  \
              /     |     \
          success  failed  upstream_failed
                    |
                 retrying
```

Task states:
- **scheduled**: Task waiting to be picked up by executor
- **queued**: Task is in executor queue
- **running**: Task is currently executing
- **success**: Task completed successfully
- **failed**: Task failed (retries exhausted)
- **upstream_failed**: Upstream task failed
- **skipped**: Task was skipped
- **removed**: Task removed from DAG
- **no_status**: Default initial state

### Trigger Rules
Control when a task should execute:

```python
task = PythonOperator(
    task_id='task',
    python_callable=my_function,
    trigger_rule='all_success',  # DEFAULT - all upstream successful
)

# Other trigger rules:
# 'all_done' - all upstream complete (success or failed)
# 'all_failed' - all upstream failed
# 'one_success' - at least one upstream successful
# 'one_failed' - at least one upstream failed
# 'none_failed' - all upstream successful or skipped
# 'none_skipped' - no upstream task skipped
# 'dummy' - always trigger
```

Example:
```python
with DAG('example', ...) as dag:
    task1 = PythonOperator(task_id='task1', ...)
    task2 = PythonOperator(task_id='task2', ...)
    task3 = PythonOperator(
        task_id='task3',
        trigger_rule='one_success',  # Run if either task1 or task2 succeeds
    )

    [task1, task2] >> task3
```

---

## Task Dependencies

### Setting Dependencies

#### Method 1: Bitwise Operators
```python
# >> sets downstream
task1 >> task2  # task2 depends on task1

# << sets upstream
task2 << task1  # Same as above

# Chain multiple tasks
task1 >> task2 >> task3 >> task4

# Fan-out
task1 >> [task2, task3, task4]

# Fan-in
[task1, task2, task3] >> task4

# Complex
[task1, task2] >> task3 >> [task4, task5]
```

#### Method 2: set_upstream/set_downstream
```python
task1.set_downstream(task2)  # task1 >> task2
task2.set_upstream(task1)     # task1 >> task2
task1.set_downstream([task2, task3])  # Fan-out
```

### Complete Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def task1_func():
    return "Task 1 result"

def task2_func():
    return "Task 2 result"

def task3_func():
    return "Task 3 result"

with DAG(
    dag_id='dependency_example',
    default_args={'owner': 'airflow', 'start_date': datetime(2024, 1, 1)},
    schedule_interval='@daily',
) as dag:

    task1 = PythonOperator(task_id='task1', python_callable=task1_func)
    task2 = PythonOperator(task_id='task2', python_callable=task2_func)
    task3 = PythonOperator(task_id='task3', python_callable=task3_func)
    task4 = PythonOperator(task_id='task4', python_callable=lambda: print('Task 4'))

    # task1 must complete first
    # then task2 and task3 run in parallel
    # finally task4 runs
    task1 >> [task2, task3] >> task4
```

### Dependency Validation
Airflow validates DAGs to ensure:
- No circular dependencies
- All task_ids are unique
- All task dependencies exist
- No cycles in the graph

Invalid DAG example:
```python
# This will raise an error - circular dependency!
task1 >> task2 >> task3 >> task1  # INVALID!
```

---

## Sensors

### What is a Sensor?
A sensor is an operator that waits for an event to occur. It's useful for waiting on external conditions.

### Sensor Behavior
```python
# Sensors have a mode parameter:
# 'poke' mode (default): Sleep and poll frequently (blocks worker slot)
# 'reschedule' mode: Free up worker slot and retry later (better for long waits)

from airflow.sensors.filesystem import FileSensor

task = FileSensor(
    task_id='wait_for_file',
    filepath='/data/input.csv',
    poke_interval=30,  # Check every 30 seconds
    timeout=3600,  # Timeout after 1 hour
    mode='poke',  # or 'reschedule'
)
```

### Common Sensors

#### 1. FileSensor
Wait for a file to arrive:

```python
from airflow.sensors.filesystem import FileSensor

task = FileSensor(
    task_id='wait_for_file',
    filepath='/data/input.csv',
    poke_interval=60,
    timeout=86400,  # 24 hours
)
```

#### 2. TimeDeltaSensor
Wait for a time duration:

```python
from airflow.sensors.time_delta import TimeDeltaSensor
from datetime import timedelta

task = TimeDeltaSensor(
    task_id='wait_10_minutes',
    delta=timedelta(minutes=10),
)
```

#### 3. TimeSensor
Wait until a specific time:

```python
from airflow.sensors.time_delta import TimeSensor
from datetime import time

task = TimeSensor(
    task_id='wait_until_2pm',
    target_time=time(14, 0),  # 2:00 PM
)
```

#### 4. ExternalTaskSensor
Wait for another DAG/task to complete:

```python
from airflow.sensors.external_task import ExternalTaskSensor

task = ExternalTaskSensor(
    task_id='wait_for_upstream_dag',
    external_dag_id='upstream_dag',
    external_task_id='upstream_task',
    poke_interval=60,
    timeout=3600,
)
```

#### 5. S3KeySensor
Wait for a file in S3:

```python
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor

task = S3KeySensor(
    task_id='wait_for_s3_file',
    bucket_name='my-bucket',
    bucket_key='data/input.csv',
    aws_conn_id='aws_default',
    poke_interval=60,
    timeout=3600,
)
```

#### 6. SqlSensor
Wait for a SQL condition:

```python
from airflow.providers.common.sql.sensors.sql import SqlSensor

task = SqlSensor(
    task_id='wait_for_data',
    conn_id='postgres_conn',
    sql='SELECT COUNT(*) FROM table WHERE status = "ready";',
    poke_interval=60,
)
```

### Poke vs Reschedule Mode

```python
# Poke Mode (blocks slot)
sensor = FileSensor(
    task_id='sensor',
    filepath='/data/file.csv',
    mode='poke',  # Keeps slot occupied
    poke_interval=30,
)

# Reschedule Mode (frees slot)
sensor = FileSensor(
    task_id='sensor',
    filepath='/data/file.csv',
    mode='reschedule',  # Releases slot, retries later
    poke_interval=30,
)
```

---

## Hooks

### What is a Hook?
A hook is a bridge to external systems. It provides connection logic and reusable methods for interacting with external services.

### Hook vs Operator
```python
# Hook - low level connection and methods
from airflow.providers.postgres.hooks.postgres import PostgresHook

hook = PostgresHook(postgres_conn_id='postgres_default')
result = hook.get_records("SELECT * FROM table LIMIT 10")
# This is reusable and flexible

# Operator - high level task definition
from airflow.operators.sql import SQLExecuteQueryOperator

op = SQLExecuteQueryOperator(
    task_id='execute_sql',
    sql='SELECT * FROM table LIMIT 10',
    conn_id='postgres_conn',
)
# This is a specific task with predefined behavior
```

### Common Hooks

#### 1. PostgresHook
```python
from airflow.providers.postgres.hooks.postgres import PostgresHook

def extract_data(ti):
    hook = PostgresHook(postgres_conn_id='postgres_conn')
    records = hook.get_records("SELECT * FROM users;")
    ti.xcom_push(key='user_records', value=records)

task = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
)
```

#### 2. MySqlHook
```python
from airflow.providers.mysql.hooks.mysql import MySqlHook

hook = MySqlHook(mysql_conn_id='mysql_default')
records = hook.get_records("SELECT * FROM table;")
```

#### 3. S3Hook
```python
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

def upload_to_s3(ti):
    hook = S3Hook(aws_conn_id='aws_default')
    hook.load_file(
        filename='/local/file.csv',
        key='s3/path/file.csv',
        bucket_name='my-bucket',
        replace=True,
    )

task = PythonOperator(task_id='upload', python_callable=upload_to_s3)
```

#### 4. HttpHook
```python
from airflow.providers.http.hooks.http import HttpHook

def call_api(ti):
    hook = HttpHook(http_conn_id='http_conn', method='GET')
    response = hook.run(endpoint='/api/data')
    return response.json()

task = PythonOperator(task_id='api_call', python_callable=call_api)
```

### Creating Custom Hooks
```python
from airflow.hooks.base import BaseHook
import requests

class CustomApiHook(BaseHook):
    def __init__(self, conn_id):
        self.conn_id = conn_id
        self.base_url = self.get_connection(conn_id).host

    def get_data(self, endpoint):
        url = f"{self.base_url}{endpoint}"
        response = requests.get(url)
        return response.json()

# Usage
def fetch_data(ti):
    hook = CustomApiHook(conn_id='custom_api')
    data = hook.get_data('/endpoint')
    return data
```

---

## Executors

### What is an Executor?
An executor determines HOW tasks are executed - it defines the parallelization strategy and how tasks are distributed.

### Executor Types

#### 1. SequentialExecutor
- Executes tasks sequentially (one at a time)
- Uses SQLite backend
- **Best for**: Development, testing
- **Downside**: No parallelization

```python
# Single-threaded, useful for testing
# Configuration in airflow.cfg:
# executor = SequentialExecutor
```

#### 2. LocalExecutor
- Executes tasks in parallel on a single machine
- Uses multiple processes
- **Best for**: Development, small deployments
- **Requirement**: Metadata database (PostgreSQL, MySQL)

```python
# Parallel execution on single machine
# Configuration:
# executor = LocalExecutor
# sql_alchemy_conn = postgresql://user:pass@localhost/airflow
```

#### 3. CeleryExecutor
- Distributed task execution across multiple machines
- Uses message broker (RabbitMQ, Redis)
- **Best for**: Production, large-scale deployments
- **Components**: Scheduler, Workers, Message Broker, Result Backend

```
┌─────────────┐
│  Scheduler  │──┐
└─────────────┘  │
                 │ Tasks Queue
            ┌────▼───────┐
            │   Broker   │
            │ (RabbitMQ) │
            └────┬───────┘
                 │
        ┌────────┼────────┐
        │        │        │
    ┌───▼──┐ ┌──▼───┐ ┌──▼───┐
    │Worker│ │Worker│ │Worker│
    └──────┘ └──────┘ └──────┘
```

#### 4. KubernetesExecutor
- Executes each task in a Kubernetes pod
- **Best for**: Cloud-native, scalable deployments
- **Requirement**: Kubernetes cluster

#### 5. DaskExecutor
- Uses Dask for distributed computing
- **Best for**: Data science workflows

### Executor Comparison

| Executor | Parallelization | Scalability | Setup | Use Case |
|----------|-----------------|-------------|-------|----------|
| Sequential | No | None | Simple | Development |
| Local | Single Machine | Limited | Simple | Dev/Small |
| Celery | Multiple Machines | High | Complex | Production |
| Kubernetes | Dynamic Pods | Very High | Complex | Cloud-native |
| Dask | Distributed | High | Moderate | Data Science |

### Executor Selection Logic
```python
# Development/Testing → SequentialExecutor or LocalExecutor
# Small Production → LocalExecutor or CeleryExecutor (1-2 workers)
# Large Production → CeleryExecutor (many workers) or KubernetesExecutor
# Cloud-native → KubernetesExecutor
```

---

## Scheduling

### Schedule Intervals

#### Cron Format
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday to Saturday)
│ │ │ │ │
│ │ │ │ │
* * * * *
```

Common examples:
```python
'0 0 * * *'      # Every day at midnight
'0 2 * * *'      # Every day at 2 AM
'0 */6 * * *'    # Every 6 hours
'0 9 * * 1-5'    # Monday to Friday at 9 AM
'0 0 1 * *'      # First day of month at midnight
'*/30 * * * *'   # Every 30 minutes
'0 12 * * 1'     # Every Monday at noon
```

#### Preset Intervals
```python
'@yearly'        # Once a year at midnight of Jan 1
'@annually'      # Same as @yearly
'@monthly'       # Once a month at midnight of first day
'@weekly'        # Once a week at midnight of Sunday
'@daily'         # Once a day at midnight
'@midnight'      # Same as @daily
'@hourly'        # Once an hour at the beginning of the hour
'@once'          # Execute only once (no recurrence)
```

#### Timedelta
```python
from datetime import timedelta

with DAG(
    dag_id='example',
    schedule_interval=timedelta(hours=1),  # Every hour
    ...
) as dag:
    pass

# Other examples:
timedelta(days=1)        # Daily
timedelta(hours=6)       # Every 6 hours
timedelta(minutes=30)    # Every 30 minutes
```

#### Manual Triggering
```python
with DAG(
    dag_id='manual_dag',
    schedule_interval=None,  # No automatic scheduling
    ...
) as dag:
    pass
```

### Execution Date vs Start Date

```python
# start_date: When to BEGIN scheduling
# execution_date: The logical date for which the task is running
# (the end of the period it should process)

default_args = {
    'start_date': datetime(2024, 1, 1),  # Start scheduling from Jan 1
}

with DAG(dag_id='example', default_args=default_args, schedule_interval='@daily'):
    pass

# Timeline:
# 2024-01-01 00:00 - First DAG run created with execution_date=2024-01-01
# 2024-01-02 00:00 - Second DAG run created with execution_date=2024-01-02
# ...
```

### Backfilling
Running DAG for past dates:

```python
# In command line:
airflow dags backfill my_dag --start-date 2024-01-01 --end-date 2024-01-31

# In code:
with DAG(
    dag_id='example',
    default_args={'start_date': datetime(2024, 1, 1)},
    schedule_interval='@daily',
    catchup=True,  # Run all missed intervals on enable
    ...
) as dag:
    pass
```

### max_active_runs & max_active_tasks
```python
with DAG(
    dag_id='example',
    max_active_runs=1,    # Only 1 DAG run at a time
    max_active_tasks=10,  # Max 10 tasks per DAG run
    ...
) as dag:
    pass
```

---

## XComs (Cross-Communications)

### What are XComs?
XComs allow tasks to exchange data. They are stored in the metadata database.

### Pushing XCom
```python
from airflow.operators.python import PythonOperator

def task_1(ti):
    result = "Hello from Task 1"
    ti.xcom_push(key='my_key', value=result)
    # Or return value (automatically pushed with key='return_value')
    return result

task1 = PythonOperator(
    task_id='task_1',
    python_callable=task_1,
)
```

### Pulling XCom
```python
def task_2(ti):
    # Get XCom from upstream task
    value = ti.xcom_pull(task_ids='task_1', key='my_key')
    print(f"Received: {value}")

    # Or pull return_value
    return_value = ti.xcom_pull(task_ids='task_1', key='return_value')
    print(f"Return value: {return_value}")

task2 = PythonOperator(
    task_id='task_2',
    python_callable=task_2,
)

task1 >> task2
```

### Complete XCom Example
```python
with DAG('xcom_example', default_args={'owner': 'airflow'}) as dag:

    def extract(ti):
        data = {'users': 100, 'orders': 500}
        ti.xcom_push(key='extracted_data', value=data)
        return data

    def transform(ti):
        data = ti.xcom_pull(task_ids='extract_task', key='extracted_data')
        transformed = {k: v * 2 for k, v in data.items()}
        ti.xcom_push(key='transformed_data', value=transformed)

    def load(ti):
        data = ti.xcom_pull(task_ids='transform_task', key='transformed_data')
        print(f"Loading data: {data}")

    extract_task = PythonOperator(task_id='extract_task', python_callable=extract)
    transform_task = PythonOperator(task_id='transform_task', python_callable=transform)
    load_task = PythonOperator(task_id='load_task', python_callable=load)

    extract_task >> transform_task >> load_task
```

### XCom Limitations
- Stored in metadata database (limited by database size)
- Default limit is 48KB per XCom message
- Not suitable for large data transfers
- Consider using external storage (S3, databases) for large data

---

## Variables & Connections

### Variables
Variables store key-value pairs for configuration:

```python
from airflow.models import Variable

# Retrieve a variable
api_key = Variable.get("api_key")
db_host = Variable.get("db_host", default_var="localhost")

# Use in DAG
with DAG('example', ...) as dag:
    def use_variable():
        api_key = Variable.get("api_key")
        print(f"Using API key: {api_key}")

    task = PythonOperator(task_id='task', python_callable=use_variable)
```

Setting variables:
```bash
# CLI
airflow variables set api_key "my_secret_key"
airflow variables get api_key

# Python
from airflow.models import Variable
Variable.set("api_key", "my_secret_key")
```

### Connections
Connections store credentials and connection strings:

```python
from airflow.models import Connection

# Retrieve a connection
conn = Connection.get_connection_from_secrets('postgres_conn')
print(conn.host, conn.login, conn.password)

# Use in operators/hooks
from airflow.providers.postgres.hooks.postgres import PostgresHook

hook = PostgresHook(postgres_conn_id='postgres_conn')
```

Connection anatomy:
```
postgres_conn = postgresql://username:password@host:5432/database

Format: scheme://[login[:password]@]host[:port][/schema]
```

Connection parameters:
```python
Connection(
    conn_id='my_postgres',
    conn_type='postgres',
    host='localhost',
    login='user',
    password='pass',
    schema='database',
    port=5432,
)
```

### Secrets Backend
Store sensitive data securely:

```python
# In airflow.cfg:
# secrets_backend = airflow.providers.google.cloud.secrets_manager.CloudSecretManagerBackend

from airflow.models import Variable

# Retrieves from secrets backend (e.g., AWS Secrets Manager, GCP Secret Manager)
secret = Variable.get("my_secret")
```

---

## Branching & Conditional Logic

### Task Branching
Execute different tasks based on conditions:

```python
from airflow.operators.python import PythonOperator, BranchPythonOperator

def check_value():
    value = 10
    if value > 5:
        return 'task_a'  # Return task_id to execute
    else:
        return 'task_b'

with DAG('branch_example', ...) as dag:
    branch_task = BranchPythonOperator(
        task_id='branch',
        python_callable=check_value,
    )

    task_a = PythonOperator(task_id='task_a', python_callable=lambda: print('A'))
    task_b = PythonOperator(task_id='task_b', python_callable=lambda: print('B'))
    task_c = PythonOperator(task_id='task_c', python_callable=lambda: print('C'))

    branch_task >> [task_a, task_b] >> task_c
```

### Multiple Branches
```python
def complex_branch():
    import random
    choice = random.choice(['option_a', 'option_b', 'option_c'])
    return choice

branch_task = BranchPythonOperator(
    task_id='branch',
    python_callable=complex_branch,
)

task_a = PythonOperator(task_id='option_a', python_callable=lambda: print('A'))
task_b = PythonOperator(task_id='option_b', python_callable=lambda: print('B'))
task_c = PythonOperator(task_id='option_c', python_callable=lambda: print('C'))
task_d = PythonOperator(task_id='final_task', python_callable=lambda: print('D'))

branch_task >> [task_a, task_b, task_c] >> task_d
```

### Dynamic Task Mapping (Airflow 2.3+)
```python
from airflow.operators.python import PythonOperator

def process_item(item):
    print(f"Processing {item}")

with DAG('dynamic_example', ...) as dag:
    items = ['a', 'b', 'c', 'd']

    process_task = PythonOperator.partial(
        task_id='process',
        python_callable=process_item,
    ).expand(op_kwargs=[{'item': i} for i in items])
```

---

## Error Handling & Retries

### Retry Configuration
```python
from datetime import timedelta

default_args = {
    'retries': 3,  # Number of retries
    'retry_delay': timedelta(minutes=5),  # Wait before retry
    'retry_exponential_backoff': True,  # Exponential backoff
    'max_retry_delay': timedelta(minutes=60),  # Max wait time
}

with DAG('example', default_args=default_args, ...) as dag:
    task = PythonOperator(task_id='task', python_callable=my_func)
```

### Task-Level Retry
```python
task = PythonOperator(
    task_id='task',
    python_callable=my_func,
    retries=5,
    retry_delay=timedelta(minutes=10),
)
```

### On Failure Callback
```python
def handle_failure(context):
    task_instance = context['task_instance']
    print(f"Task {task_instance.task_id} failed!")
    print(f"Exception: {context['exception']}")
    # Send alert, log, etc.

task = PythonOperator(
    task_id='task',
    python_callable=my_func,
    on_failure_callback=handle_failure,
)
```

### On Success Callback
```python
def handle_success(context):
    print("Task succeeded!")

task = PythonOperator(
    task_id='task',
    python_callable=my_func,
    on_success_callback=handle_success,
)
```

### On Retry Callback
```python
def handle_retry(context):
    print("Task is being retried!")

task = PythonOperator(
    task_id='task',
    python_callable=my_func,
    on_retry_callback=handle_retry,
)
```

### Timeout
```python
task = PythonOperator(
    task_id='task',
    python_callable=my_func,
    execution_timeout=timedelta(hours=1),  # Fail if not done in 1 hour
)
```

### Error Handling in Python
```python
from airflow.exceptions import AirflowException

def my_func():
    try:
        # Do something
        pass
    except Exception as e:
        raise AirflowException(f"Task failed: {str(e)}")

task = PythonOperator(
    task_id='task',
    python_callable=my_func,
)
```

---

## Airflow Architecture

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Airflow Architecture                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Web Server (UI)                         │  │
│  │  - DAG Visualization                                 │  │
│  │  - Task Monitoring                                   │  │
│  │  - Log Access                                        │  │
│  │  - Manual Triggering                                 │  │
│  └──────────────────────────────────────────────────────┘  │
│                            │                                 │
│                    ┌───────┴────────┐                       │
│                    │                │                       │
│  ┌─────────────────┴──────┐  ┌──────┴──────────────────┐   │
│  │     Scheduler          │  │   Task Executor        │   │
│  │                        │  │                        │   │
│  │ - DAG Parsing          │  │ - Runs Tasks           │   │
│  │ - Scheduling DAG runs  │  │ - Manages Parallelism  │   │
│  │ - Creating task        │  │                        │   │
│  │   instances            │  │                        │   │
│  └─────────────────┬──────┘  └──────┬──────────────────┘   │
│                    │                │                       │
│                    └───────┬────────┘                       │
│                            │                                 │
│  ┌─────────────────────────┴────────────────────────────┐  │
│  │         Metadata Database                            │  │
│  │                                                      │  │
│  │ - DAG definitions                                   │  │
│  │ - Task instances                                    │  │
│  │ - Variables                                         │  │
│  │ - Connections                                       │  │
│  │ - XComs                                             │  │
│  │ - Task logs                                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Components

#### 1. Scheduler
- Parses DAGs from Python files
- Creates task instances based on schedule_interval
- Monitors task execution and updates states
- Manages retries and SLAs
- Typically runs in a single instance (though can be HA)

#### 2. Executor
- Physically executes the tasks
- Manages parallelization
- Can be SequentialExecutor, LocalExecutor, CeleryExecutor, etc.

#### 3. Web Server
- Provides REST API and web UI
- Displays DAGs, tasks, logs
- Allows manual triggering
- Monitors health

#### 4. Metadata Database
- Stores all state information
- DAG definitions, task instances, logs
- Connections and variables
- Must be PostgreSQL or MySQL (not SQLite in production)

#### 5. Executor Workers (for CeleryExecutor)
- Execute tasks
- Report back to scheduler/executor
- Can be scaled horizontally

### High-Level Flow

```
1. User writes DAG in Python
   ↓
2. Scheduler parses DAG file every 1-2 minutes
   ↓
3. For each DAG, scheduler checks if it should run
   ↓
4. If yes, creates TaskInstances and queues them
   ↓
5. Executor picks up tasks and runs them
   ↓
6. Task updates its state in metadata database
   ↓
7. Web UI shows updated status
   ↓
8. Task completes (success/failed)
```

---

## Common Interview Questions

### 1. What is a DAG in Airflow?
A DAG (Directed Acyclic Graph) is a collection of tasks organized with dependencies. It ensures:
- **Directed**: Flow goes in one direction
- **Acyclic**: No circular dependencies
- Each task is represented as a node, dependencies as edges

### 2. Difference between Operator and Task?
- **Operator**: A template/class that defines WHAT to do (e.g., PythonOperator)
- **Task**: An instance of an operator with parameters (e.g., task1 = PythonOperator(...))

### 3. What is XCom and when would you use it?
XCom allows inter-task communication. Use it for:
- Passing small amounts of data between tasks
- Sharing results from one task to another
- Not suitable for large data (use S3, databases instead)

### 4. What's the difference between start_date and execution_date?
- **start_date**: When Airflow should begin scheduling
- **execution_date**: The logical date for which the task is running (end of period)

### 5. Explain the catchup parameter.
When `catchup=True`, Airflow automatically runs all missed DAG intervals when the DAG is enabled. When `False`, only runs from the current date forward.

### 6. What are trigger rules?
Trigger rules determine when a task should execute:
- `all_success`: All upstream succeeded (default)
- `one_success`: At least one upstream succeeded
- `all_done`: All upstream completed
- `none_failed`: All upstream succeeded or skipped

### 7. Explain the difference between poke and reschedule mode in sensors.
- **poke mode**: Polls continuously (blocks worker slot) - good for short waits
- **reschedule mode**: Releases slot and retries later - good for long waits

### 8. What's the best executor for production?
CeleryExecutor or KubernetesExecutor:
- CeleryExecutor: Requires message broker, good for most use cases
- KubernetesExecutor: Cloud-native, dynamic scaling

### 9. How do you handle failures in Airflow?
Use:
- `retries`: Number of automatic retries
- `retry_delay`: Wait between retries
- `on_failure_callback`: Custom logic on failure
- `execution_timeout`: Max execution time
- Error boundaries/exception handling in tasks

### 10. What's the maximum recommended size for XCom?
Default is 48KB. For larger data, use external storage (S3, databases, etc.).

### 11. How does the scheduler work?
The scheduler:
1. Parses all DAG files every 1-2 minutes
2. Determines which DAGs should run based on schedule_interval
3. Creates TaskInstances for DAG runs
4. Pushes tasks to executor queue
5. Monitors task execution and updates states

### 12. What's the difference between max_active_runs and max_active_tasks?
- **max_active_runs**: Maximum number of concurrent DAG runs
- **max_active_tasks**: Maximum number of concurrent tasks within a single DAG run

### 13. How would you perform data validation in an Airflow pipeline?
Use check operators:
- `SQLCheckOperator`: Validate SQL query results
- `SQLThresholdCheckOperator`: Check if value exceeds threshold
- `BranchPythonOperator`: Custom validation logic

### 14. Explain how you would handle late-arriving data in Airflow.
Options:
- Use sensors to wait for data
- Set `depends_on_past=True` to handle out-of-order executions
- Implement SLAs with alerts
- Create manual re-run capability

### 15. What's the best practice for DAG design?
- Keep DAGs simple and focused (single responsibility)
- Use descriptive task_ids
- Implement proper error handling
- Add documentation (doc_md)
- Use default_args for common settings
- Avoid hardcoding values (use Variables/Connections)
- Make tasks idempotent
- Monitor and set SLAs

---

## Best Practices Summary

### DAG Design
✓ Keep DAGs focused and maintainable
✓ Use meaningful task_ids
✓ Document your DAGs (doc_md)
✓ Use default_args for DRY principle
✓ Avoid circular dependencies
✓ Make tasks idempotent (rerunnable)

### Task Design
✓ One responsibility per task
✓ Handle errors and retries
✓ Use appropriate operators
✓ Set reasonable timeouts
✓ Implement logging

### Data Movement
✓ Use XCom for small data only
✓ External storage (S3, DB) for large data
✓ Never hardcode credentials (use Connections)
✓ Use Variables for configuration

### Monitoring
✓ Set up alerting on task failures
✓ Monitor executor health
✓ Check scheduler logs
✓ Set SLAs for critical tasks

### Deployment
✓ Use version control for DAGs
✓ Test DAGs before production
✓ Use separate environments
✓ Document dependencies
✓ Have runbooks for common issues

---

## Quick Reference

### Most Important Concepts
```
DAG - Workflow definition
Task - Unit of work
Operator - Task template
Executor - Task runner
Scheduler - DAG parser and trigger
XCom - Inter-task communication
```

### Common Patterns
```
ETL: Extract → Transform → Load
Fan-out: One task → Multiple tasks
Fan-in: Multiple tasks → One task
Branching: Conditional task execution
```

### Key Parameters
```
start_date - When to begin scheduling
schedule_interval - How often to run
retries - Number of automatic retries
timeout - Max execution time
trigger_rule - When to execute task
```

Good luck with your Apache Airflow interviews!
