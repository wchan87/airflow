# Airflow Components

This section is based on [Core Concepts > Architecture Overview](https://airflow.apache.org/docs/apache-airflow/3.0.6/core-concepts/overview.html) with each component summarized for someone to cross-reference if necessary.

The diagram below is based on the [separate DAG processing architecture](https://airflow.apache.org/docs/apache-airflow/3.0.6/core-concepts/overview.html#separate-dag-processing-architecture) to illustrate what components to expect and how they interact with each other.
```mermaid
flowchart LR
    dag_files@{ shape: documents, label: "DAG files" }
    plugin_folder@{ shape: documents, label: "Plugin folder & installed packages" }
    subgraph Execution
        workers@{ shape: processes, label: "Worker(s)" }
        triggerers@{ shape: processes, label: "Triggerer(s)" }
    end
    subgraph Parsing
        dag_processors@{ shape: processes, label: "DAG Processor(s)" }
    end
    metadata_db@{ shape: database, label: "Metadata DB" }
    subgraph Scheduling
        schedulers@{ shape: processes, label: "Scheduler(s)" }
    end
    subgraph UI
        api_servers@{ shape: processes, label: "API Server(s)" }
    end

    dag_files -->|sync| workers
    dag_files -->|sync| triggerers
    dag_files -->|sync| dag_processors
    linkStyle 0,1,2 stroke:darkred;

    plugin_folder -->|install| workers
    plugin_folder -->|install| triggerers
    plugin_folder -->|install| dag_processors
    plugin_folder -->|install| schedulers
    plugin_folder -->|install| api_servers
    linkStyle 3,4,5,6,7 stroke:blue;

    workers <-.-> metadata_db
    triggerers <-.-> metadata_db
    dag_processors <-.-> metadata_db
    schedulers <-.-> metadata_db
    api_servers <-.-> metadata_db
    linkStyle 8,9,10,11,12 stroke:red;

    schedulers -.-|via Executor| workers
```

## Scheduler

From [Administration and Deployment > Scheduler](https://airflow.apache.org/docs/apache-airflow/3.0.6/administration-and-deployment/scheduler.html),
> The Airflow scheduler monitors all tasks and dags, then triggers the [task instances](concepts.md#task-instance) once their dependencies are complete. Behind the scenes, the scheduler spins up a subprocess, which monitors and stays in sync with all dags in the specified DAG directory. Once per minute, by default, the scheduler collects DAG parsing results and checks whether any active tasks can be triggered.

## Worker

Within the worker, there are two additional processes, [Supervisor](#supervisor) and [Task Runner](#task-runner). See the related [task lifecycle](concepts.md#task-lifecycle) for more information on the order of execution.

### Supervisor

As summarized in [Supervisor & Task Runner](https://airflow.apache.org/docs/task-sdk/1.0.6/concepts.html#supervisor-task-runner):
> Within an Airflow [worker](#worker), a Supervisor process manages the execution of [task instances](concepts.md#task-instance):
> * Spawns isolated subprocesses ([Task Runners](#task-runner)) for each [task](concepts.md#task), following a parent–child model.
> * Establishes dedicated STDIN, STDOUT, and log pipes to communicate with each subprocess.
> * Proxies Execution API calls: forwards subprocess requests (e.g., variables, connections, XCom operations, state transitions) to the API server and relays responses.
> * Monitors subprocess liveness via heartbeats and marks tasks as failed if heartbeats are missed.
> * Generates and refreshes JWT tokens on behalf of subprocesses through heartbeat responses to ensure authenticated API calls.

### Task Runner

As summarized in [Supervisor & Task Runner](https://airflow.apache.org/docs/task-sdk/1.0.6/concepts.html#supervisor-task-runner):
> A Task Runner subprocess provides a sandboxed environment where user task code runs:
> * Receives startup messages (run parameters) via STDIN from the [Supervisor](#supervisor).
> * Executes the Python function or operator code in isolation.
> * Emits logs through STDOUT and communicates runtime events (heartbeats, XCom messages) via the Supervisor.
> * Performs final state transitions by sending authenticated API calls through the Supervisor.

## Triggerer

From [Authoring and Scheduling > Deferrable Operators & Triggers](https://airflow.apache.org/docs/apache-airflow/3.0.6/authoring-and-scheduling/deferring.html),
> This is where Deferrable Operators can be used. When it has nothing to do but wait, an operator can suspend itself and free up the worker for other processes by deferring. When an operator defers, execution moves to the triggerer, where the trigger specified by the operator will run. The trigger can do the polling or waiting required by the operator. Then, when the trigger finishes polling or waiting, it sends a signal for the operator to resume its execution. During the deferred phase of execution, since work has been offloaded to the triggerer, the task no longer occupies a worker slot, and you have more free workload capacity. By default, tasks in a deferred state don’t occupy pool slots. If you would like them to, you can change this by editing the pool in question.

The triggerer appears to be a special type of [worker](#worker) used to handle triggers that deferrable operators wait for.

## DAG Processor

From [Administration and Deployment > DAG File Processing](https://airflow.apache.org/docs/apache-airflow/3.0.6/administration-and-deployment/dagfile-processing.html),
> DAG File Processing refers to the process of reading the python files that define your dags and storing them such that the scheduler can schedule them.
> 
> There are two primary components involved in DAG file processing. The `DagFileProcessorManager` is a process executing an infinite loop that determines which files need to be processed, and the `DagFileProcessorProcess` is a separate process that is started to convert an individual file into one or more DAG objects.

## Webserver

From the documentation, the webserver primarily refers to the [Airflow UI](https://airflow.apache.org/docs/apache-airflow/3.0.6/ui.html), but it can sometimes encapsulate the [Airflow REST API](https://airflow.apache.org/docs/apache-airflow/3.0.6/stable-rest-api-ref.html) as well.
