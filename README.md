# Airflow

Activate Python virtual environment
```bash
source .venv/Scripts/activate
```
Deactivate Python virtual environments (or close the terminal)
```bash
deactivate
```
Install the Python modules defined in [requirements.txt](requirements.txt)
```bash
pip install -r requirements.txt
```

## Docker

Instructions on how to set up Airflow via Docker are derived from [Installation of Airflow® > Using Production Docker Images](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html#using-production-docker-images) > [Docker Image for Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html).

Note that for consistency with the [latest version of Amazon MWAA](https://docs.aws.amazon.com/mwaa/latest/userguide/airflow-versions.html#airflow-versions-official) as of `2026-04-01`, we will be using version `3.0.6` of [DockerHub > apache/airflow](https://hub.docker.com/r/apache/airflow)
```bash
docker pull apache/airflow:3.0.6-python3.12
```
Note that `slim-3.0.6-python3.12` doesn't work with the `docker-compose.yaml`

1. Create a [.env](.env) file with the following content
    ```dotenv
    AIRFLOW_IMAGE_NAME="apache/airflow:3.0.6-python3.12"
    AIRFLOW_UID=50000
    ```
   * The `docker compose` command expects `.env` file or it fails. The file can be empty as long as the relevant environment variables are available to the `docker compose` command.
        ```bash
        export AIRFLOW_IMAGE_NAME="apache/airflow:3.0.6-python3.12"
        export AIRFLOW_UID=50000
        ```
2. Set up the Docker Compose cluster via the [docker-compose.yaml](/docker-compose.yaml)  derived from the [docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/3.1.8/docker-compose.yaml) that's referenced in the documentation
    ```bash
    docker compose up -d
    ```
   * [Airflow UI](https://airflow.apache.org/docs/apache-airflow/3.0.6/ui.html) is accessible via http://localhost:8080 through `airflow` / `airflow` default credentials

3. Manage the Docker Compose cluster
   * Start Docker containers
        ```bash
        docker compose start 
        ```
   * Stop Docker containers
        ```bash
        docker compose stop
        ```
4. Tear down the Docker Compose cluster
    ```bash
    docker compose down --volumes
    ```

## Helm

An alternative approach to deployment is through [Helm](https://helm.sh/docs/intro/quickstart/) chart as documented via [Installation of Airflow® > Using Official Airflow Helm Chart](https://airflow.apache.org/docs/apache-airflow/stable/installation/index.html#using-official-airflow-helm-chart) > [Helm Chart for Apache Airflow](https://airflow.apache.org/docs/helm-chart/stable/index.html). Another reference is [OneUptime Blog > How to Deploy Apache Airflow with Helm for Workflow Orchestration](https://oneuptime.com/blog/post/2026-01-17-helm-apache-airflow-deployment/view).

Note that the instructions aren't complete, but a starting point is to add the Helm repository
```bash
helm repo add apache-airflow https://airflow.apache.org
helm search repo apache-airflow/airflow --versions
helm repo list
```
