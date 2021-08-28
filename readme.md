# Airflow Helm Chart : Quick start for Beginners

# Introduction

The official Helm chart for Apache Airflow is available 🥳

- No need to choose between different Helm charts
- Built and supported by the community, released by the Airflow PMC Members.

The Helm Chart makes it easy to deploy Airflow on Kubernetes.

**Use cases:**

- Deploy Airflow on Kubernetes in local to make tests
- Deploy and configure Airflow in a Kubernetes environment quickly
- Easy to implement awesome features such as KEDA or the CeleryKubernetesExecutor

# Requirements

- Kubectl
- Helm
- Docker
- KinD

[https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

# Materials

[kind-airflow.zip](media/kind-airflow.zip)

# Set up

```bash
# Create a kubernetes cluster of 1 control plane and 3 worker nodes
kind create cluster --name airflow-cluster --config kind-cluster.yaml

# Check the cluster info
kubectl cluster-info

# Check the nodes with kubectl
kubectl get nodes -o wide

# Add the official repository of the Airflow Helm Chart
helm repo add apache-airflow https://airflow.apache.org

# Update the repo
helm repo update

# Create namespace airflow
kubectl create namespace airflow

# Check the namespace 
kubectl get namespaces

# Install the Airflow Helm Chart
helm install airflow apache-airflow/airflow --namespace airflow --debug

# Get pods
kubectl get pods -n airflow

# Check release
helm ls -n airflow

# If for some reasons the release is stuck in pending-install or timed out
# Resinstall the chart
helm delete airflow --namespace airflow
helm install airflow apache-airflow/airflow --namespace airflow --debug —timeout 10m0s

# Port forward 8080:8080
kubectl port-forward svc/airflow-webserver 8080:8080 -n airflow --context kind-airflow-cluster
```

## Jobs

**airflow-run-airflow-migrations** performs airflow db upgrade (initiliaze the metadata database)

`kubectl exec --stdin --tty <pod> -n <namespace> -- /bin/bash` is really helpful to run a shell in a container

# Debug

If for some reasons during the helm install you got a timeout

```yaml
# 1. Delete the Helm release
helm delete airflow --namespace airflow

# 2. Check your PODs
kubectl get pods -n airflow

# 3. If airflow-migrations is in ContainerCreating state delete it
kubectl get jobs -n airflow
kubectl delete jobs <pods_name_of_airflow_migrations> -n airflow

# 4. Install the chart again
helm install airflow apache-airflow/airflow --namespace airflow --debug --timeout 10m0s
```

# Values.yaml

Configuration file of the Helm chart and so, of your Airflow deployment

```python
# Get the Chart values
helm show values apache-airflow/airflow > values.yaml
```

To update your Airflow deployment

```python
# Check the current revision
helm ls -n airflow

# Upgrade the chart
helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml --debug

# Check after
helm ls -n airflow
```

To configure your Airflow instance

```yaml
extraEnv: |
	- name: AIRFLOW__CORE__LOAD_EXAMPLES
		value: 'True'
```

# Clean everything

```python
kind delete cluster --name airflow-cluster
```