## Lecture 2: Airflow Operators

The most common task in Airflow is an Operator. They represent a single task in workflow. The task can be anything such as sending an email. 
These tasks run independently. They usually do not exchange information (it is possible but not discussed here). 
Different operators are used to perform different tasks. 
For example, the dummy operator is used to troubleshoot or create a task that has not been implemented.

````
DummyOperator(task_id= "example_task", dag= dag)
````

And BashOperator

```
BashOperator(task_id= "bash_Script_example", bash_command= "echo "Example!"", dag=dag)
```

```
BashOperator(task_id= "bash_Script_example", bash_command= "runcleanup.sh", dag=dag)
```

### Defining a BashOperator task

The BashOperator allows you to specify any given Shell command or script and add it to an Airflow workflow. This can be a great start to implementing Airflow in your environment.
As such, you've been running some scripts manually to clean data (using a script called cleanup.sh) prior to delivery to your colleagues in the Data Analytics group. 
As you get more of these tasks assigned, you've realized it's becoming difficult to keep up with running everything manually, much less dealing with errors or retries. 
You'd like to implement a simple script as an Airflow operator. The Airflow DAG analytics_dag is already defined for you and has the appropriate configurations in place.

```# Import the BashOperator
from airflow.operators.bash_operator import BashOperator

# Define the BashOperator 
cleanup = BashOperator(
    task_id="cleanup_task",
    # Define the bash_command
    bash_command="cleanup.sh",
    # Add the task to the dag
    dag=analytics_dag
)
```
### Multiple BashOperators

Airflow DAGs can contain many operators, each performing their defined tasks.
You've successfully implemented one of your scripts as an Airflow task and have decided to continue migrating your individual scripts to a full Airflow DAG. 
You now want to add more components to the workflow. 
In addition to the cleanup.sh used in the previous exercise you have two more scripts, consolidate_data.sh and push_data.sh. 
These further process your data and copy to its final location. The DAG analytics_dag is available as before, and your cleanup task is still defined. 
The BashOperator is already imported.

Define a BashOperator called consolidate, to run consolidate_data.sh with a task_id of consolidate_task.
Add a final BashOperator called push_data, running push_data.sh and a task_id of pushdata_task.

```
# Define a second operator to run the `consolidate_data.sh` script
consolidate = BashOperator(
    task_id='consolidate_task',
    bash_command="consolidate_data.sh",
    dag=analytics_dag)

# Define a final operator to execute the `push_data.sh` script
push_data = BashOperator(
    task_id="pushdata_task",
    bash_command="push_data.sh",
    dag=analytics_dag)
```

### Airflow Tasks
* Tasks are instances of operators
* Usually assigned a variable to a python code
* Referred to by the task_id within the workflow and not the variable name
* Task dependencies: define the order of task completion. If not defined, it can be random. It can be defined upstream (beofre) or downstream (after).
* Bitshift operators: Upstream operator- task1>> task2 >> task3
* Mixed dependencies: task1 >> task2 << task3 suggests that task 1 and task 3 must run before task2

#### Define order of BashOperators

Now that you've learned about the bitshift operators, it's time to modify your workflow to include a pull step and to include the task ordering. 
You have three currently defined components, cleanup, consolidate, and push_data.
The DAG analytics_dag is available as before and the BashOperator is already imported.

Define a BashOperator called pull_sales with a bash command of wget https://salestracking/latestinfo?json.
Set the pull_sales operator to run before the cleanup task.
Configure consolidate to run next, using the downstream operator.
Set push_data to run last using either bitshift operator.
```
# Define a new pull_sales task
pull_sales = BashOperator(
    task_id='pullsales_task',
    bash_command="wget https://salestracking/latestinfo?json",
    dag=analytics_dag
)

# Set pull_sales to run prior to cleanup
pull_sales >> cleanup

# Configure consolidate to run after cleanup
cleanup >> consolidate

# Set push_data to run last
consolidate >> push_data
```

Determining the order of tasks
While looking through a colleague's workflow definition, you're trying to decipher exactly in which order the defined tasks run. The code in question shows the following:

```
pull_data << initialize_process
pull_data >> clean >> run_ml_pipeline
generate_reports << run_ml_pipeline
```

Troubleshooting DAG dependencies
You've created a DAG with intended dependencies based on your workflow but for some reason Airflow won't load / execute the DAG. Try using the terminal to:

List the DAGs.
Decipher the error message.
Use cat workspace/dags/codependent.py to view the Python code.
Determine which of the following lines should be removed from the Python code. You may want to consider the last line of the file.

```
repl:~$ airflow list_dags
[2022-08-17 22:55:07,445] {__init__.py:51} INFO - Using executor SequentialExecutor
[2022-08-17 22:55:07,770] {dagbag.py:90} INFO - Filling up the DagBag from /home/repl/workspace/dags
[2022-08-17 22:55:07,773] {dagbag.py:267} ERROR - Failed to bag_dag: /home/repl/workspace/dags/codependent.py
Traceback (most recent call last):
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dagbag.py", line 253, in process_file
    self.bag_dag(dag, parent_dag=dag, root_dag=dag)
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dagbag.py", line 324, in bag_dag
    dag.test_cycle()  # throws if a task cycle is found
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dag.py", line 1427, in test_cycle
    self._test_cycle_helper(visit_map, task_id)
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dag.py", line 1449, in _test_cycle_helper
    self._test_cycle_helper(visit_map, descendant_id)
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dag.py", line 1449, in _test_cycle_helper
    self._test_cycle_helper(visit_map, descendant_id)
  File "/usr/local/lib/python3.6/dist-packages/airflow/models/dag.py", line 1447, in _test_cycle_helper
    raise AirflowDagCycleException(msg)
airflow.exceptions.AirflowDagCycleException: Cycle detected in DAG. Faulty task: third_task to first_task
```

This means that we have to correct the code because there are some tasks that are causing a cycle to be created. It can be made clear by looking at the code below-

```
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime

default_args = {
  'owner': 'dsmith',
  'start_date': datetime(2020, 2, 12),
  'retries': 1
}

codependency_dag = DAG('codependency', default_args=default_args)

task1 = BashOperator(task_id='first_task',
                     bash_command='echo 1',
                     dag=codependency_dag)

task2 = BashOperator(task_id='second_task',
                     bash_command='echo 2',
                     dag=codependency_dag)

task3 = BashOperator(task_id='third_task',
                     bash_command='echo 3',
                     dag=codependency_dag)

# task1 must run before task2 which must run before task3
task1 >> task2
task2 >> task3
task3 >> task1
```

Therefore we can understand that the cyclic loop is caused by task3 >> task1 and hence it must be removed.

### Python Operator

Using the PythonOperator

You've implemented several Airflow tasks using the BashOperator but realize that a couple of specific tasks would be better implemented using Python. 
You'll implement a task to download and save a file to the system within Airflow.
The requests library is imported for you, and the DAG process_sales_dag is already defined.

Define a function called pull_file with two parameters, URL and savepath.
Use the print() function and Python f-strings to write a message to the logs.

```
# Define the method
def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'wb') as f:
        f.write(r.content)    
    # Use the print method for logging
    print(f"File pulled from {URL} and saved to {savepath}")
```
Import the necessary object to use the Python Operator.
```
def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'wb') as f:
        f.write(r.content)   
    # Use the print method for logging
    print(f"File pulled from {URL} and saved to {savepath}")
    
# Import the PythonOperator class
from airflow.operators.python_operator import PythonOperator
```

Create a new task assigned to the variable pull_file_task, with the id pull_file.
Add the pull_file(URL, savepath) function defined previously to the operator.
Define the arguments needed for the task.

```
def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'wb') as f:
        f.write(r.content)   
    # Use the print method for logging
    print(f"File pulled from {URL} and saved to {savepath}")

from airflow.operators.python_operator import PythonOperator

# Create the task
pull_file_task = PythonOperator(
    task_id='pull_file',
    # Add the callable
    python_callable=pull_file,
    # Define the arguments
    op_kwargs={'URL':'http://dataserver/sales.json', 'savepath':'latestsales.json'},
    dag=process_sales_dag
)
```

More PythonOperators

To continue implementing your workflow, you need to add another step to parse and save the changes of the downloaded file. 
The DAG process_sales_dag is defined and has the pull_file task already added. 
In this case, the Python function is already defined for you, parse_file(inputfile, outputfile).
Note that often when implementing Airflow tasks, you won't necessarily understand the individual steps given to you. 
As long as you understand how to wrap the steps within Airflow's structure, you'll be able to implement a desired workflow.

Define the Python task to the variable parse_file_task with the id parse_file.
Add the parse_file(inputfile, outputfile) to the Operator.
Define the arguments to pass to the callable.
Add the task to the DAG.

```
# Add another Python task
parse_file_task = PythonOperator(
    task_id="parse_file",
    # Set the function to call
    python_callable = parse_file,
    # Add the arguments
    op_kwargs={'inputfile':'latestsales.json', 'outputfile':'parsedfile.json'},
    # Add the DAG
    dag=process_sales_dag
)
```

EmailOperator and dependencies

Now that you've successfully defined the PythonOperators for your workflow, 
your manager would like to receive a copy of the parsed JSON file via email when the workflow completes.
The previous tasks are still defined and the DAG process_sales_dag is configured.

Import the class to send emails.
Define the Operator and add the appropriate arguments (to, subject, files).
Set the task order so the tasks run sequentially (Pull the file, parse the file, then email your manager).

```
# Import the Operator
from airflow.operators.email_operator import EmailOperator

# Define the task
email_manager_task = EmailOperator(
    task_id='email_manager',
    to='manager@datacamp.com',
    subject='Latest sales JSON',
    html_content='Attached is the latest sales JSON file as requested.',
    files='parsedfile.json',
    dag=process_sales_dag
)

# Set the order of tasks
pull_file_task >> parse_file_task >> email_manager_task
```

### Airflow Scheduling

How to schedule and run workflows. 

```
# Update the scheduling arguments as defined
default_args = {
  'owner': 'Engineering',
  'start_date': datetime(2019, 11, 1),
  'email': ['airflowresults@datacamp.com'],
  'email_on_failure': False,
  'email_on_retry': False,
  'retries': 3,
  'retry_delay': timedelta(minutes=20)
}

dag = DAG('update_dataflows', default_args=default_args, schedule_interval='30 12 * * 3')
```



