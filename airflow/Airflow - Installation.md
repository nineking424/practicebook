# Airflow - Installation

## Reference
- artifact hub : https://artifacthub.io/packages/helm/bitnami/airflow
- git : https://github.com/bitnami/charts/tree/master/bitnami/airflow

## About
- helm 배포 툴 : kubeapps
- helm repo : https://charts.bitnami.com/bitnami
- helm chart : bitnami/airflow

## Installation

### Install via helm chart
```bash
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm fetch bitnami/airflow
$ tar zxvf airflow.tar.gz && cd airflow
$ vi values.yaml
$ helm install airflow -n airflow .
```

### Changes - values.yaml
- service.type : LoadBalancer
- auth.username : admin
- auth.password : ****
