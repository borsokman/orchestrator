### orchestrator

This project provisions a microservices architecture using Kubernetes (K3s) and Vagrant. It migrates a multi-tier application into isolated Pods running PostgreSQL, RabbitMQ, and Python/Pika services.

## Architecture Overview

It implements a scalable microservices architecture on a 2-node Kubernetes (K3s) cluster provisioned via Vagrant. The system is designed for high availability, asynchronous processing, and persistent data storage.

# Infrastructure (K3s Cluster)

- **Master Node:** Runs the Kubernetes control plane and API server (`192.168.56.10`). Traefik ingress is disabled to rely on native Service LoadBalancers.
- **Agent Node:** Worker node (`192.168.56.11`) where workloads are scheduled.
- **Networking:** Flannel CNI is explicitly bound to the private host-only network to ensure reliable cross-node pod communication.

# Application Components

- **API Gateway (`api-gateway-app`):**
  - The single entry point exposed to the host machine via a `LoadBalancer` service on port 3000.
  - Deployed as a `Deployment` with a Horizontal Pod Autoscaler (HPA) configured to scale up to 3 replicas when CPU exceeds 60%.
  - Routes synchronous requests to the Inventory Service and publishes asynchronous events to RabbitMQ.
- **Inventory Service (`inventory-app`):**
  - Handles synchronous REST API calls (CRUD operations for movies).
  - Deployed as a `Deployment` with HPA (scales up to 3 replicas at 60% CPU).
- **Billing Service (`billing-app`):**
  - An asynchronous background worker that consumes order messages from RabbitMQ.
  - Deployed as a `StatefulSet` to guarantee strict, ordered processing and stable network identity.

# Data & Messaging Layer

- **PostgreSQL Databases (`inventory-db` & `billing-db`):**
  - Isolated database instances for each service (Database-per-Service pattern).
  - Deployed as `StatefulSets` with `PersistentVolumeClaims` (PVCs).
  - Uses the K3s default `local-path` provisioner to ensure data survives pod restarts.
- **RabbitMQ (`rabbitmq-server`):**
  - Message broker deployed to decouple the API Gateway from the Billing Service.
  - Facilitates reliable, asynchronous inter-service communication.

## Prerequisites

- Vagrant
- VirtualBox
- kubectl CLI tool

## Infrastructure Setup & Management

The entire infrastructure is managed via Vagrant and Kubernetes.

## To build and start the infrastructure:

# Build the Master and Agent VMs and Kubernetes cluster

chmod +x orchestrator.sh
./orchestrator.sh start

# Before the kubectl test commands everytime on a new terminal

export KUBECONFIG=$PWD/k3s.yaml

# Test the cluster

kubectl get nodes

# Verify the HPA (Autoscaling 1%/60%)

kubectl get hpa

## API Testing with Postman

A Postman Collection is included in the repository to automate the audit tests:
Open Postman and click Import.
Select the orchestrator_API_tests.json file.
In the imported collection, go to the Variables tab.
Ensure base_url is set to http://192.168.56.10:3000 (or your VM's IP if testing remotely).
Run the requests to verify Inventory CRUD operations and the asynchronous Billing Queue.

## Autoscalling test

Step 1: Open a terminal and watch the autoscaler live: kubectl get hpa -w

Step 2: Open a second terminal and watch the pods: kubectl get pods -w

Step 3: Open a third terminal and generate massive load Run this infinite loop to spam the API with hundreds of requests per second: while true; do curl -s http://192.168.56.10:3000/api/movies > /dev/null; done

## Project Tree

```
orchestrator
├─ Manifests
│  ├─ api-gateway-app.yaml
│  ├─ billing-app.yaml
│  ├─ billing-db.yaml
│  ├─ inventory-app.yaml
│  ├─ inventory-db.yaml
│  ├─ rabbitmq-server.yaml
│  └─ secrets.yaml
├─ README.md
├─ Vagrantfile
├─ architecture.png
├─ docker-compose.yml
├─ orchestrator.sh
├─ orchestrator_API_tests.json
└─ srcs
   ├─ api-gateway-app
   │  ├─ Dockerfile
   │  ├─ app
   │  │  ├─ __init__.py
   │  │  ├─ config.py
   │  │  └─ routes.py
   │  ├─ requirements.txt
   │  └─ server.py
   ├─ billing-app
   │  ├─ Dockerfile
   │  ├─ app
   │  │  ├─ __init__.py
   │  │  ├─ consumer.py
   │  │  └─ models.py
   │  ├─ requirements.txt
   │  └─ server.py
   ├─ billing-db
   │  ├─ Dockerfile
   │  └─ entrypoint.sh
   ├─ inventory-app
   │  ├─ Dockerfile
   │  ├─ app
   │  │  ├─ __init__.py
   │  │  ├─ models.py
   │  │  └─ routes.py
   │  ├─ requirements.txt
   │  └─ server.py
   ├─ inventory-db
   │  ├─ Dockerfile
   │  └─ entrypoint.sh
   └─ rabbitmq-server
      ├─ Dockerfile
      └─ entrypoint.sh

```
