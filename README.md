# index

+ [What is Airflow](#airflow)
+ [Benefits](#benefits)
+ [Core Components](#core-components)
+ [Core Concept](#core-concept)
+ [Other Concepts](#other-concepts)
+ [How Airflow works](#how-airflow-works)
+ [Usage of Airflow](#usage-of-airflow)
+ [Define a DAG](#define_dag)
+ [Condition on Task](#branching)
+ [Airflow Webserver Problem](#airflow-webserver-problem)
+ [Interact with Sqlite3](#interact-with-sqlite3)
+ [Deploy](#deploy)
+ [DAG Optimization](#dag-optimization)
+ [Amazing Airflow Operators](#airflow_operators)
+ [Airflow Security](#airflow_roles)
+ [version](#version)

# airflow

+ It's an Orchestrator to **execute a task** at **right time** in **right way** in the **right order**.

# **benefits**

+ **Dynamic**
  + Everything we can do here in Python so the advantages are limitless.
+ **Scalability**
  + It's possible to run as many task as we want in parallel.
+ **UI**
  + Monitor the data pipeline.
  + able to retry our tasks.
  + Data profiling:
    + run sql queries
    + show data in chart
+ **Extensible**

# core-components

+ **Web server</span>**
  + Flask server with Gunicorn the UI.

+ **Scheduler**

+ **Metastore**
  + It's related to DB where all the metadata related to airflow itself but also related to our data pipeline, plans, tasks & so on will be stored.

+ **Executor</span>**
  + It defines how our tasks are going to be executed.
  + Type
    **SequentialExecutor**: It executes tasks one after another.
    **LocalExecutor**: It can execute task parallely.

+ **Worker**
  + It defines where the task will be executed.

# **core-concept**</span>

+ **DAG**: Depends on one another but has No loop.

+ **Operator**: It's kind of wrapper around the task. Ex: we want to connect to our DB, insert data in it, we'll use an operator to do that.**One operator for one Task.**
  + **Action** : It executes fn or cmd.
  
  + **Transfer** : It allows to transfer data from src to destination.
  
  + **Sensor** : It waits for something to happen before moving to next task.
    + **poke_interval(sec)**</span> : Every n seconds the given task should wait.
    
    + **timeout(sec)**</span> : Max time limit to wait.
    
    + **softfail(bollean)**</span> : If set to **true**, will marked the task as skipped on failure.

+ **Backfilling & catchup</span>** : It basically fetches the data from previous missing dates. By default it is **True**. When catchup is set to **True** then the dag will run from last run date & when it is **false** then the dag will be triggered from current date.  
  
# **other-concepts**</span>

+ **Task Instance</span>**

+ **Workflow</span>** : It's the combination of all concepts.

+ **Hook</span>** : It embodies a connection to a remote server, service or platform. It's used to transafer data between source to destination.

+ **Pool</span>** : priority of task/worker.

+ **plugin</span>** : Airflow provides advantages to create custom plugin like operator, hook, sensor etc.

+ **.airflowignore</span>** : Dag names we want to ignore. File must be put in dags dir.

+ **zommbies/undeeds</span>** : theory.

# how-airflow-works

+ **Single node architecture**
  ```mermaid
    flowchart LR

    web_server --> metastore
    scheduler --> metastore
    executor_queue <--> metastore
    ```
  + How it works
      ```mermaid
      flowchart LR

      web_server <--parse_python_files --> folder_dags
      scheduler<--parse_python_files --> folder_dags
      scheduler--parse_the_info--> metastore
      executor--runs_the_task_and_update_metadata--> metastore
      ```
+ **Multi node architecture**
  + wip...

# usage-of-airflow

+ **airflow dir architecture**
  + **airflow.cfg</span>** : airflow configuration

    ```
    load_examples: True/False
    sql_alchemy_conn: sqlite/MySQL/postgres connection string
    ```

  + **airflow.db** : DB information
  + **logs** : log information
  + **webserver_config.py** : webserver configuration
  + **make a dir named ```dags```**
+ **airflow -h** : all available cmd
+ **DB**
  + **initialize the metastore/db(for the 1st time)**

    ```
    airflow db init (deprecated in 2.7)
    airflow db migrate
    ```

  + **Update db version. Ex: 1.10.x to 2.2.x**

    ```
    airflow db upgrade (deprecated in 2.7)
    airflow db migrate
    ```

  + **reset the DB**

    ```
    airflow db reset
    ```

  + **Check the status after changing the configuration.**

    ```
    airflow db check
    ```

+ **UI**
  + **running the UI**

    ```
    airflow webserver
    ```

+ **Connection**
  + **It returns all the connection name & detail.**

    ```
    airflow connections
    ```

+ **Create a user**

  ```
  airflow users create -u uname -f firstname -l lastname -p password  -e email -r role[Admin, Viewer, User, Op, Public]
  ```

+ **Enable scheduler**

    ```
    airflow scheduler
    ```

+ **Dags**
  + **All dag list**

    ```
    airflow dag list
    ```

  + **Exact info of that particular task**

    ```
    airflow tasks list dagName
    ```
  
  + **export DAG dependecy as img/pdf or anything**
    ```sh
    sudo apt-get install graphviz
    ```

    ```sh
    airflow dags show dag_name --save file_name.pdf
    ```

+ **Test**
  + **It shows the task is success/fail. It's a good practice to test every task before deploy.**

    ```
    airflow tasks test dag_id task_id date
    ```

+ **Tasks**:
  + **Sequential ordering** : task1 >> task2 >> task3 >> task4
  + **Parallel ordering** : task1 >> [task2, task3] >> task4
  + **Trigger a DAG from another DAG**
      ```python
      from airflow.models import DAG
      from airflow.operators.trigger_dagrun import TriggerDagRunOperator
      from airflow.operators.bash import BashOperator
      from airflow.utils.edgemodifier import Label
      from datetime import timedelta, datetime

      default_args = {
          'owner': 'admin',
          'email_on_failure': False,
          'email_on_retry': False,
          'email_on_success': False,
          'email': 'samrat.mitra@vivasoftltd.com',
          'retries': 1,
          'retry_delay': timedelta(seconds=10)
      }

      with DAG(dag_id='DAG2'
              , default_args=default_args
              , description='A simple Test Dag which runs every 2 min inerval'
              , start_date=datetime(2023, 11, 8)
              , schedule_interval=None  # only once
              , catchup=False):
          
          # Tasks
          # ==============

          task1 = BashOperator(task_id='task1'
                              , bash_command='sleep 1')
          
          task2 = BashOperator(task_id='task2'
                              , bash_command='sleep 2')
          
          task3 = BashOperator(task_id='task3'
                              , bash_command='sleep 3')
        
          # trigger DAG
          trigger_child_dag1 = TriggerDagRunOperator(task_id='trigger_child_dag1',
                                                    trigger_dag_id='DAG1',
                                                    execution_date='{{ds}}',
                                                    reset_dag_run=True,
                                                    wait_for_completion=True,
                                                    poke_interval=2
                                                    )

        # Task flow

        trigger_child_dag1 >> task1 >> task2 >> task3
      ```
  + **ScaleUp task**
    1) **airflow.cfg** :
       + **executor**: What kind of execution (sequential or parallel)
       + **sql_alchemy_conn**: DB connection
       + **parallelism**: 1....n (How many tasks will be executed in parallel for the entire airflow instance.)
       + **dag_concurrency**: 1....n (How many tasks can be run in parallel for a given DAG.)
       + **max_active_run_per_dag**:1....n (How many DAG will run in parallel at a single time.)
    2) **celery**: Distribute & execute the task asynchronously.

        ```
        pip install 'apache-airflow[celery]'
        ```

       + <span style="color: red;">**caution**</span> : **Celery can't be used with sqlite. Use MySQL/Postgres.**
    3) **redis(in memory DB)**:
        + installation:
          + [link](https://phoenixnap.com/kb/install-redis-on-ubuntu-20-04)
          + cmd:

            ```
              sudo apt update
              sudo apt intall redis-server -y
              sudo nano /etc/redis/redis.conf
              change supervised no to systemd
              run redis server: sudo systemctl restart redis.service
              check server status: sudo systemctl status redis.service
            ```

        + airflow.cfg:
          + executor: CeleryExecutor
          + broker_url: redis url (localhost or IP)
          + result_backend: sql_alchemy_conn
    4) **airflow redis package**

        ```
        pip3 install 'apache-airflow-providers-redis'
        ```

    5) The UI which allows to monitor workers by which the task is executed.

        ```
        airflow celery flower
        ```

        ```
          Problem: flower import error
          Solution: pip install --upgrade apache-airflow-providers-celery==2.0.0 (for airflow 2.1.0)
        ```

       + ip: localhost:5555
       + Add worker in celery

          ```
         airflow celery worker
          ```

    7) **TaskGroup**: To run similliar kinds of taks parallelly

        ```python
        from airflow.utils.task_group import TaskGroup
        ```

    8) **Xcom**: It's used to push/pull data
    9) **Trigger**: conditional task execution
        + [documentation](https://tinyurl.com/bddfzajn)

# define_dag

- [DAG all params](https://airflow.apache.org/docs/apache-airflow/stable/_api/airflow/models/dag/index.html)

  ```python
  from datetime import datetime, timedelta

  from airflow import DAG
  from airflow.models import Variable

  from utility.ms_teams_notification import send_fail_notification_teams_message,\
        send_success_notification_teams_message

  default_args = {
    'owner' : 'admin',
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': int(Variable.get("no_of_retry")),
    'retry_delay': timedelta(seconds=int(Variable.get("task_retry_delay_in_sec"))),
    'on_failure_callback': send_fail_notification_teams_message,
    'on_success_callback': send_success_notification_teams_message,
    'is_paused_upon_creation': True  # by default the DAG will be disabled
    }

  with DAG(dag_id='TrggerFileTransferAndIngestionDAG'
        , dag_display_name='Trigger File Transfer And Ingestion DAG'
         , default_args=default_args
         , description=f'Trigger SFTPfileTransferDefaultSource, SFTPfileTransferSaviyntIDM and KCCIngestDataToBigQuery DAG'
         , start_date=datetime(2025, 2, 21)
         , schedule_interval='0 17 * * *'  # every day at 17
         , tags=['bigquery', 'schedule', 'daily']
         , catchup=False
         , owner_links={"admin": "mailto:username@gmail.com"}
         # or, owner_links={"admin": "https://www.example.com"}
         , fail_stop=True  # the downstream tasks will skipped instead of getting failed
         , dagrun_timeout=timedelta(seconds=10)
         ):
         # define tasks
         pass
  ```

# branching

```python
from airflow.operators.python import PythonOperator, BranchPythonOperator

"""
--> define the DAG
  --> then use the code
"""

def _connect_to_api()-> str:
  """
  try to establish API connection

  --> returns task id according to status code
  --> if status_code == 200:
      returns next task id
  --> else:
      returns termination task id
  """

  falcon = CSPMRegistration(client_id=Variable.get('falcon_client_id'),
                            client_secret=Variable.get('falcon_client_secret')
                            )
  status_code = falcon.get_policy_settings(cloud_platform=cloud_provide_list).get('status_code', None)
  if status_code:
      if int(status_code) == 200:
          return 'get_api_data'  # next task
      else:
          return 'invalid_connection'  # termination task

connect_to_api = BranchPythonOperator(task_id='connect_to_api',
                                          task_display_name='🌐 connect to API',
                                          python_callable=_connect_to_api,
                                          trigger_rule="all_success")

# task #2: invalid connection
invalid_connection = PythonOperator(task_id='invalid_connection',
                                    task_display_name='🚫 Invalid connection',
                                    python_callable=lambda: print('API connectiin is failed'),
                                    trigger_rule="all_success") 

# task #3: fetch API data
get_api_data = PythonOperator(task_id='get_api_data',
                            python_callable=_get_api_data)

# taskflow
connect_to_api >> invalid_connection
connect_to_api >> get_api_data
```

# airflow-webserver-problem

+ **Problem**: server is running in PID: 4006 or whatever
  + **Solution**

      ```
      kill -9 PID
      ```

# interact-with-sqlite3

+ **sAccess DB**
  + **sqlite path/db_name.db** -> To access DB
  + **All table list**

    ```
    .tables
    ```

  + **select * from tableName</span>** -> Particular table

# deploy

+ **GCP Composer**
  + create a vpc
    - subnet creation mode: ```custom```
    - add a subnet
    - private google access: ```on```
  + create an env in GCP composer & upload the files in **DAG** folder
    1. give proper role to **default service** account(*-compute@developer.gserviceaccount.com)
        - cloud sql client
        - editor
        - Eventarc Event Receiver
    2. create a **service account**
    3. goto ```IAM```
    4. click the checkbox in middle right side
    5. find the cloud_composer_service_account like ```service-*@cloudcomposer-accounts.iam.gserviceaccount.com```, click checkbox and click Edit principal
        - Cloud Composer API Service Agent
        - Cloud Composer v2 API Service Agent Extension
    6. Click ```GRANT ACCESS```
        - Add principals
          - select the created service account(```step #2```)
        - Assign roles
          - Cloud Composer v2 API Service Agent Extension
          - Eventarc Event Receiver
          - save
    7. goto ```Service Accounts```
        - select the created service account
        - goto ```permissions```
          - select ```*-compute@developer.gserviceaccount.com```
            - role: ```Editor```
          - select ```*@mxs-cmdatalake-prd.iam.gserviceaccount.com```
            -  role: ```Cloud Composer v2 API Service Agent Extension``` and ```Service Account Token Creator```
          - select ```service-*@cloudcomposer-accounts.iam.gserviceaccount.com```
            - role: ```Cloud Composer API Service Agent```, ```Cloud Composer v2 API Service Agent Extension``` and ```Service Account Admin```
    
    8. **bind**
        ```sh
        gcloud iam service-accounts add-iam-policy-binding \
        weselect-data-dev@we-select-data-dev-422614.iam.gserviceaccount.com \
        --member serviceAccount:service-126779322718@cloudcomposer-accounts.iam.gserviceaccount.com \
        --role roles/composer.ServiceAgentV2Ext
        ```
    9. **create**
        - console

          ```sh
          gcloud composer environments create env_name \
          --location us-central1 \
          --image-version composer-2.7.1-airflow-2.7.3 \
          --service-account "weselect-data-dev@we-select-data-dev-422614.iam.gserviceaccount.com"
          ```
    10. [doc](https://cloud.google.com/composer/docs/composer-2/create-environments)
  
  + if composer in ```private``` env:
    1. goto ```cloud NAT```
    2. create ```cloud NAT gateway```
    3. NAT type ```public```
    4. Select Cloud Router
        - network: vpc
        - region: as same as ```composer```
        - cloud router: create a new router
    5. Network service tier: ```Standard```(**for dev**)

        
  + **how to access the DB from `GCP composer`**:
    + GCP composer uses the **PostgreSQL** by default which is kept in **GKE**
    + steps:
      1. get the GKE cluster name from
          ```mermaid
          flowchart LR

          composer --> env_name --> environment_configuration
          ```
      2. full sqlAlchemy conn from
          ```mermaid
          flowchart LR

          composer --> env_name --> airflow_webserver --> Admin --> Configurations
          ```
      3. save 2 IPs from composer sql proxy service
          ```mermaid
          flowchart TB

          composer --> env_name --> environment_configuration --> GKE_cluster --> details --> Networking --> service_ingress --> airflow_sqlproxy_service

          airflow_sqlproxy_service --> cluster_ip

          airflow_sqlproxy_service --> serving_pods_endpoint
          ```
      4. create virtual machine with same region and airflow_sqlproxy_service(cluster_ip)
      5. Finally, execute psql cmd to get the db details
          ```sh
          # get the dbname, user, password, port from sqlAlchemy connection

          psql -h airflow_sqlproxy_service_serving_pods_endpoint -p 3306 -U root -p password -d db_name
          ```
  
  + **User authentication**
    - composer config:
      ```mermaid
      flowchart LR

      composerName --> overwrite_airflow_congfig --> rbac_user_role:viewer
      ```
    
    - create new user
      ```bash
      gcloud composer environments run example-environment \
      --location us-central1 \
      users create -- \
      -r Op \
      -e "example-user@example.com" \
      -u "example-user@example.com" \
      -f "Name" \
      -l "Surname" \
      --use-random-password
      ```
    - update user role
      ```bash
        gcloud composer environments run ENVIRONMENT_NAME \
      --location LOCATION \
      users add-role -- -e USER_EMAIL -r Admin
      ```

# dag-optimization

- keep tasks `atomic`

- use a `static` start date 

- change the `name` of the DAG when u change the `start date` 

- Don't import `airflow variable` outside `methods/operators`, use it directly.

- Break down a big pipeline into smaller pipelines/tasks, not a single task or pipeline.

- Use `template fields`, `variable`, and `macros`.

- Executor
    + use `LocalExecutor/CeleryExecutor/DaskExecutor/
/KubernetesExecutor/CeleryKubernetesExecutor`(in Cloud we can ignore it)

- **idempotency**: operation can be applied multiple times without changing any result 

- Never pull/process large dataset using pandas/any library in airflow 

- For dataOps use `dbt`/`sqlmesh`/`whatever`.

- use `TaskGroup` to run similliar kinds of tasks simultaneously

- use loop to create dynamic task for similiar type of task 

- Make proper calculation of `parallelism, max_active_tasks_per_dag and max_active_runs_per_dag`

- `N.B:` Airflow is an Orchestrator. Don't ever process large amount of data via `airflow`. Use corresponding tool/software/library/framework (e.g., `spark`)

# airflow_operators

- **SQLExecuteQueryOperator**
  > Execute any SQL query from any SQL DB.
  > [doc](https://tinyurl.com/uy26ncne)
  ```python
  from datetime import datetime, timedelta

  from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator

  # task 1:
  execute_sql_query = SQLExecuteQueryOperator(task_id='execute_sql_query'
                                              , task_display_name='get sample data'
                                              , conn_id='postgres_local_connection'
                                              , sql='SELECT * FROM PUBLIC.ACTOR LIMIT 1;'
                                              , show_return_value_in_logs=True)
  ```

# airflow_roles

- **public**
  > Public users (anonymous) don’t have any permissions.

- **Viewer**
  > Viewer users have limited read permissions.

- **USer**
  > User users have Viewer permissions plus additional permissions.

- **Op**
  > Op users have User permissions plus additional permissions. 

- **Admin**
  > Admin users have all possible permissions, including granting or revoking permissions from other users. Admin users have Op permission plus additional permissions.

- [doc](https://airflow.apache.org/docs/apache-airflow-providers-fab/stable/auth-manager/access-control.html)

# version

+ **2.9.0**