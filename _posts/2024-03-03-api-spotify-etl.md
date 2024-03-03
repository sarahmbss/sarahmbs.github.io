---
layout: post
title: ETL process with Spotify API
subtitle: Using the spotify API to build an ETL process with Airflow, Kubernetes and MySQL
tags: [mysq, etl, api, airflow, kubernetes]
author: Sarah Silva
--- 

[IN PROGESS]

# Objective

The main goal with this project is to collect personal data on Spotify - top tracks, favorite artists, playlists - clean it, and add to a local MySQL database for further analysis. The process needs to be executed each week, updating the new favorite artists and songs (in case there are new ones).

In order to achieve this goal, I decided to deploy an instance of Airflow on Minikube, using the Helm package.

## Preparing the environment 

### Minikube
First of all, I installed an instance of Minikube, with the Docker container.

In order to work with k8s, I also installed kubctx and kubens as well, through a tool called **Chocolatey**.
- **kubectx** is a tool to switch between contexts (clusters) on kubectl faster;
- **kubens** is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily.

```powershell
choco install kubectlx
choco install kubens
```

### Chocolatey
Chocolatey aims to automate the entire software lifecycle from install through upgrade and removal on Windows operating systems.

**How to install**
> 1. Open Windows Powershell as an administrator
> 2. Type the command: Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

### Helm package

Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application. This is exactly why I chose to download the Airflow on k8s using the package provided by Helm. To install it, I also used chocolatey:

```powershell
choco install kubernetes-helm
```

## Configuring the helm package

After starting my k8s cluster with the command:

```powershell
minikube start
```
I added airflow to my Helm packages:

```powershell
helm repo add apache-airflow https://airflow.apache.org/
```

By default, airflow points directly to its own DAGs repository. So, we need to copy the helm package to our local machine, instead of just downloading it directly to the cluster. This way, we can change all of the standard configurations to our necessities. This is done by changing the file values.yaml.

```powershell
helm pull apache-airflow/airflow
tar zxvf airflow-1.11.0.tgz
```

So, everytime we install helm, it points to a file called values.yaml. On the case of the airflow, at the end of the file on the part of **gitSync**, it points directly to the git file of the own airflow. We need to alter that part, so that we can create our own DAGs.

As my git repository is public, I just changed the default one for mine. If I had a private one, I would have to create a file named encoding with git's name and token, and update yaml with the following command

```powershell
kubectl apply -f git-secret.yaml -n airflow
```

After changing the first configurations, I installed the airflow helm package on my k8s cluster:

```powershell
helm install airflow airflow -n airflow --create-namespace
```

## Monitor and access the cluster

To monitor the pods on my namespace, I used k9s:

```powershell
k9s -n airflow
```

This allowed me to check how many restarts each pod had, as well as their log file.

After checking if all my pods were ok, I accessed the Airflow page:

```powershell
kubens airflow
kubectl port-forward svc/airflow-webserver 8080:8080
```