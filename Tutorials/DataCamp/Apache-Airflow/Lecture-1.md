

## Lecture 1: Introduction to Apache Airflow


You've just started looking at using Airflow within your company and would like to try to run a task within the Airflow platform. You remember that you can use the airflow run command to execute a specific task within a workflow. Note that an error while using airflow run will return airflow.exceptions.AirflowException: on the last line of output. An Airflow DAG is set up for you with a dag_id of etl_pipeline. The task_id is download_file and the start_date is 2020-01-08. All other components needed are defined for you.

For example, this `airflow run etl_pipeline download_file 2020-01-08` can accomplish the above purpose.

### Basic Airflow Commands
* `list_dags`: shows all available DAGs defined within Airflow.
* `test`: method of testing workflows in Airflow.
* `scheduler`: subcommand starts a scheduling process for Airflow.
* `airflow -h`: help

### Directed Acyclic Graphs
 * These are directed and does not contain any cycles (individual components can not be rerun)
 * Written in Python but can support others such as bash scripts etc.
 * These are made up of component (for example, tasks) to be completed.
 * 
 ```# Import the DAG object
# Import the DAG object
from airflow.models import DAG
```


```# Import the DAG object
from airflow.models import DAG

# Define the default_args dictionary
default_args = {
  'owner': "dsmith",
  "start_date": datetime(2020,1,14),
  "retries": 2
}
````

Instantiate the DAG object to a variable called etl_dag with a DAG named example_etl.
And, Add the default_args dictionary to the appropriate argument.

```# Import the DAG object
from airflow.models import DAG

# Define the default_args dictionary
default_args = {
  'owner': 'dsmith',
  'start_date': datetime(2020, 1, 14),
  'retries': 2
}

# Instantiate the DAG object
etl_dag = DAG("example_etl", default_args=default_args)
```


### Airflow Web Interface
You've successfully created some DAGs within Airflow using the command-line tools, but notice that it can be a bit tricky to handle scheduling / troubleshooting / etc. After reading the documentation further, you realize that you'd like to access the Airflow web interface. For security reasons, you'd like to start the webserver on port 9090. Which airflow command would you use to start the webserver on port 9090?

Airflow is installed and accessible from the command line. Remember to use the airflow -h command if needed. airflow <subcommand> -h will provide further detail.

```
airflow webserver -p 9090
```
  
