# Apache Airflow Task SDK

[Apache Airflow Task SDK](https://airflow.apache.org/docs/task-sdk/1.0.6/index.html) was introduced as part of Airflow 2.X to Airflow 3.X to "decouple Dag authoring from Airflow internals (Scheduler, API Server, etc.), providing a forward-compatible, stable interface for writing and maintaining Dags across Airflow versions".

> The Apache Airflow Task SDK provides python-native interfaces for defining [DAGs](concepts.md#dag), executing [tasks](concepts.md#task) in isolated subprocesses, and interacting with Airflow resources (e.g., Connections, Variables, XComs, Metrics, Logs, and OpenLineage events) at runtime. It also includes core execution-time components to manage communication between the worker and the Airflow scheduler/backend.
>
> The goal of task-sdk is to decouple Dag authoring from Airflow internals (Scheduler, API Server, etc.), providing a forward-compatible, stable interface for writing and maintaining DAGs across Airflow versions. This approach reduces boilerplate and keeps your DAG definitions concise and readable.

**Note:** `apache-airflow-task-sdk==1.0.6` is the version that is associated with Apache Airflow 3.0.6 (which in turn is the [latest version supported by Amazon MWAA](https://docs.aws.amazon.com/mwaa/latest/userguide/airflow-versions.html#airflow-versions-official) as of 2026-04-05) 

The key imports for the Task SDK are at the `airflow.sdk` package are divided into:
* Classes, typically represented in Pascal case where the first letter of every word is capitalized
* Decorators and Helper Functions in snake case where all the letters of every word are lower case and space is handled by underscore

The basic principles of how to leverage the Task SDK are covered by the [Examples](https://airflow.apache.org/docs/task-sdk/1.0.6/examples.html) section with additional sections:
* [Dynamic Task Mapping with Task SDK](https://airflow.apache.org/docs/task-sdk/1.0.6/dynamic-task-mapping.html) section covering how to leverage [dynamic task mapping](concepts.md#dynamic-task-mapping)
* [airflow.sdk API Reference](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html) section that's further broken down by the sections below
* [Concepts](https://airflow.apache.org/docs/task-sdk/1.0.6/concepts.html) section that summarizes some terminology, task lifecycles, and components relevant to the Task SDK

## airflow.sdk.dag

**Related Concept(s):** [DAG](concepts.md#dag)

[airflow.sdk.dag](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.dag) is a [decorator](https://docs.python.org/3/reference/compound_stmts.html#grammar-token-python-grammar-decorators) has the following return type:
> [Callable](https://typing.python.org/en/latest/spec/callables.html)[[Callable], Callable[[Ellipsis](https://docs.python.org/3/library/typing.html#annotating-callable-objects), [DAG](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.DAG)]]

The decorator converts the Python function into an Airflow DAG represented by the underlying [airflow.sdk.DAG](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.DAG) class:
> A dag (directed acyclic graph) is a collection of tasks with directional dependencies.
> 
> A dag also has a schedule, a start date, and an end date (optional). For each schedule, (say daily or hourly), the DAG needs to run each individual tasks as their dependencies are met. Certain tasks have the property of depending on their own past, meaning that they can’t run until their previous schedule (and upstream tasks) are completed.
> 
> DAGs essentially act as namespaces for tasks. A task_id can only be added once to a DAG.
>
> Note that if you plan to use time zones all the dates provided should be pendulum dates. See [Time zone aware dags](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/timezone.html#timezone-aware-dags).

All nested calls to functions decorated by [airflow.sdk.task](#airflowsdktask) within the primary function decorated by `airflow.sdk.dag` become Airflow Tasks in the DAG. 

## airflow.sdk.task_group

[airflow.sdk.task_group](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.task_group) is a decorator to group [tasks](#airflowsdktask) into logical [TaskGroups](concepts.md#taskgroup).

## airflow.sdk.task

**Related Concept(s):** [Task](concepts.md#task)

[airflow.sdk.task](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.task) is a decorator "to wrap Python callables as tasks and leverage [dynamic task mapping](concepts.md#dynamic-task-mapping) with the [.expand()](#airflowsdkbasedecorator_taskdecoratorexpand) method" and communicate through [airflow.sdk.XComArg](#airflowsdkxcomarg). The decorator references `airfllow.sdk.base.decorators.TaskDecorator` underneath the covers so the associated methods like [expand()](#airflowsdkbasedecorator_taskdecoratorexpand) and [partial()](#airflowsdkbasedecorator_taskdecoratorpartial) are organized under this section.

**Note:** The documentation mentions "[f]or traditional operators and sensors, import classes like [airflow.sdk.BaseOperator](#airflowsdkbaseoperator) or `airflow.sdk.Sensor`". There doesn't seem to be `airflow.sdk.Sensor` so it may be referring to [airflow.sdk.BaseSensorOperator](#airflowsdkbasesensoroperator) instead.

### airflow.sdk.base.decorator._TaskDecorator.expand

The underlying `expand()` method for a task decorator allows a keyword argument to be passed to "generate a variable number of task instances at runtime based on upstream data". The `expand()` method "provid[es] a way to parallelize execution without knowing the number of tasks ahead of time".

Static parameters are defined by the [partial()](#airflowsdkbasedecorator_taskdecoratorpartial) method whereas `expand()` handles the dynamic parameters.

### airflow.sdk.base.decorator._TaskDecorator.partial

The underlying `partial()` method for a task decorator allows the static parameters to be defined so the [expand()](#airflowsdkbasedecorator_taskdecoratorexpand) method can pass the dynamic parameters.

## airflow.sdk.setup

[airflow.sdk.setup](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.setup) is a decorator for a setup hook function encapsulated by either [airflow.sdk.task_group](#airflowsdktask_group) or [airflow.sdk.dag](#airflowsdkdag).

## airflow.sdk.teardown

[airflow.sdk.teardown](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.teardown) is a decorator for a teardown hook function encapsulated by either [airflow.sdk.task_group](#airflowsdktask_group) or [airflow.sdk.dag](#airflowsdkdag).

## airflow.sdk.asset

**Related Concept(s):** [Asset](concepts.md#asset)

[airflow.sdk.asset](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.asset) is a decorator for the materialization function for an asset, represented by [airflow.sdk.Asset](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.Asset). An asset is "[a] representation of data asset dependencies between workflows".

## airflow.sdk.BaseOperator

[airflow.sdk.BaseOperator](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.BaseOperator) is the "[a]bstract base class for all operators" and "create objects that become nodes in the DAG". Note that all [tasks](concepts.md#task) are subclasses of this abstract base class internally, but the documentation recommends thinking of task and operator as separate concepts. 

## airflow.sdk.BaseSensorOperator

[airflow.sdk.BaseSensorOperator](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.BaseSensorOperator) is a class that "keep[s] executing at a time interval and succeed when a criteria is met and fail if and when they time out". This class is a subclass of the [airflow.sdk.BaseOperator](#airflowsdkbaseoperator) and what the [sensors](concepts.md#sensors) are subclasses of.

## airflow.sdk.XComArg

[airflow.sdk.XComArg](https://airflow.apache.org/docs/task-sdk/1.0.6/api.html#airflow.sdk.XComArg) is a class that acts as "reference to an XCom value pushed from another [operator](#airflowsdkbaseoperator)".
