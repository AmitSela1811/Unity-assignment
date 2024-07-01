# Project Overview

This project consists of two Python web applications, **customer-mgmt** and **web-service**, both developed using FastAPI. Each application has its own Dockerfile, Helm chart for deployment.

## Directory Structure
```csharp
/
├── README.md
├── customer-mgmt-service
│   ├── app
│   │   └── main.py
│   ├── chart
        ├── customer-mgmt
          ├── templates
            ├── deployment.yaml
            ├── service
            ├── _helpers.tpl
            ├── NOTES.txt
│   │     ├── Chart.yaml
│   │     └── values.yaml
│   ├── Dockerfile
│   └── requirements.txt
├── customer-web-service
│   ├── app
│   │   └── main.py
│   ├── chart
        ├── web-service
          ├── templates
            ├── deployment.yaml
            ├── ingress.yaml
            ├── service.yaml
            ├── _helpers.tpl
            ├── NOTES.txt
│   │     ├── Chart.yaml
│   │     └── values.yaml
│   ├── Dockerfile
│   └── requirements.txt
```


## Components

### Applications
  #### customer-mgmt
- **overview**: A FastAPI-based application that handles customer purchase data. It interacts with MongoDB for data storage and Kafka for consuming purchase messages. The service exposes endpoints to create and retrieve customer purchases.
- **features**: 
  - Create Purchase: Accepts customer purchase data and stores it in MongoDB.
  - Retrieve Purchases: Retrieves all customer purchase records from MongoDB.
  - Kafka Integration: Consumes purchase messages from a Kafka topic and stores them in MongoDB.
 
- **Architecture**:
  - FastAPI: Provides the web framework for the service.
  - MongoDB: Used for storing customer purchase data.
  - Kafka: Acts as a message broker for purchase data.
  - Docker & Kubernetes: The service is containerized and deployed on a Kubernetes cluster. 
        
  #### customer-mgmt
- **overview**:A FastAPI-based web application that handles customer purchase requests. It interacts with Kafka to publish purchase messages and communicates with another service to retrieve customer purchases.
- **features**: 
  - Create Purchase Request: Accepts customer purchase data and publishes it to a Kafka topic.
  - Retrieve All Purchases: Fetches all customer purchase records from the Customer Management Service.
 
- **Architecture**:
  - FastAPI: Provides the web framework for the service.
  - Kafka: Acts as a message broker for purchase data.
  - Docker & Kubernetes: The service is containerized and deployed on a Kubernetes cluster.
 
### infrastructure 
  #### minikube k8s cluster
  #### kafka cluster
  - kafka cluster deployed as Statefullset on k8s cluster, 3 kafka broker pods and 3 zookeeper pods.
  - mongodb deployed as Statefullset on k8s cluster 3 mongodb pods and 1 arbiter pods.
        
    
### Helm Charts
- Both applications include a Helm chart for deployment.
- kafka cluster uses bitnami/kafka Helm chart for deployment.
- mongodb uses bitnami/mongodb Helm chart for deployment.

### Dockerfiles
- Each application has its own Dockerfile for containerization based on `python:3.9-slim` image.

## Setup Instructions

### 1. Deploy Kubernetes Cluster

Deploy a Kubernetes cluster on your preferred platform. For local development, you can use MiniKube. 
```bash
brew install minikube
minikube start
```
Ensure that the Minikube Ingress addon is enabled using
```bash
minikube addons enable ingress
```

### 2. Clone the Repository

Clone this repository to your local machine:

```bash
git clone git@github.com:AmitSela1811/unity-home-assignment.git
```

### 3. Deploy infrastructure On Minikube

1. Deploy kafka cluster:

    ```bash
    helm upgrade --install kafka-cluster bitnami/kafka --version 21.0.1 -n kafka-cluster  --create-namespace --set replicaCount=3 --set zookeeper.replicaCount=3
    ```
    This chart will deploy:
    - kafka: 2 kafka services, 3 pods representing kafka brokers and 1 statefulset.
    - zookeeper: 2 zookeeper services, 3 pods representing zookeeper brokers and 1 statefulset.

2. Deploy mongodb:

    ```bash
    helm upgrade --install mongodb bitnami/mongodb -n mongodb --create-namespace --set architecture=replicaset,auth.rootPassword=secretpassword,auth.username=my-user,auth.password=my- 
    password,auth.database=my-database --set replicaCount=3
    ```
    This chart will deploy:
    - 3 mongodb pods,Each pod runs a MongoDB instance that is part of the replica set.
    - 2 services and 2 statfullsets.


### 3. Deploy Microservices On Minikube

1. Deploy customer-mgmt:

    ```bash
    helm upgrade --install customer-mgmt src/microservices/customer-mgmt-service/chart/customer-mgmt -n customer-mgmt --create-namespace
    ```
    This chart will deploy:
    - 1 pod runing the application.
    - 2 services and 1 deployment.
  
2. Deploy web-service:

    ```bash
    helm upgrade --install web-service src/microservices/customer-web-service/chart/web-service -n web-service --create-namespace
    ```
    This chart will deploy:
    - 1 pod runing the application.
    - 1 ingress resouce.
    - 2 services and 1 deployment.
    ```

    This chart will deploy a service, a deployment and an ingress.
    `API_URL=http://numbers-api.apps.svc.cluster.local:8080`, so when reaching microservices, it will redirect the request to `numbers-api` service using internal k8s communication.

### 4. Accessing Service on Minikube (Mac)

1. Edit the hosts file to map the Ingress IP address to your desired hostname:

    ```bash
    sudo vim /etc/hosts
    ```

    Add the following lines:

    ```bash
     127.0.0.1 web-service.local
    ```

2. Start Minikube tunnel:

    ```bash
     minikube tunnel
    ```
3. open a terminal and do the following curl to generate a "buy" with any parmeters you like:
   ```bash
    curl -X POST \ 
          http://web-service.local/buy \
          -H 'Content-Type: application/json' \
          -d '{
            "username": "john_doe",
            "userid": 123,
            "price": 99.99,
            "timestamp": "2024-07-01T12:00:00Z"
    }'
    ```
