### orchestrator

This project provisions a microservices architecture using Docker and Docker Compose. It migrates a multi-tier application into isolated Linux containers running PostgreSQL, RabbitMQ, and Python Flask/Pika services.

## Architecture Overview

- **inventory-db**: PostgreSQL database for the inventory service.
- **billing-db**: PostgreSQL database for the billing service.
- **rabbitmq-server**: Message broker for asynchronous order processing.
- **inventory-app**: REST API managing movies (Internal Port 8080).
- **billing-app**: Message consumer processing orders and writing to the database (Internal Port 8080).
- **api-gateway-app**: Reverse proxy routing requests to appropriate services (Exposed Port 3000).

_All containers are built from scratch using `debian:bullseye` as the base image. No pre-built service images are used._

## Prerequisites

- Linux Operating System (Virtual Machine or Bare Metal)
- Docker Engine
- Docker Compose (V2)

Configuration
Before building the infrastructure, you must create a .env file in the root directory. This file is ignored by Git to prevent credential leaks.

Create .env with the following variables:

# Inventory Database

INVENTORY_DB_NAME=movies_db
INVENTORY_DB_USER=movies_user
INVENTORY_DB_PASSWORD=your_secure_password
INVENTORY_DB_HOST=inventory-db
INVENTORY_DB_PORT=5432

# Billing Database

BILLING_DB_NAME=billing_db
BILLING_DB_USER=orders_user
BILLING_DB_PASSWORD=your_secure_password
BILLING_DB_HOST=billing-db
BILLING_DB_PORT=5432

# RabbitMQ Server

RABBITMQ_HOST=rabbitmq-server
RABBITMQ_PORT=5672
RABBITMQ_USER=billing_user
RABBITMQ_PASSWORD=your_secure_password

# API Gateway Routing

INVENTORY_URL=http://inventory-app:8080

## Infrastructure Setup & Management

The entire infrastructure is managed exclusively via Docker Compose.

To build and start the infrastructure:

docker compose up -d --build

To stop and remove the containers:

docker compose down

To view logs for a specific service:

docker compose logs api-gateway-app

## API Testing with Postman

A Postman Collection (Gateway_API_Tests.postman_collection.json) is included in the repository to automate the audit tests.

Open Postman and click Import.
Select the Gateway_API_Tests.postman_collection.json file.
In the imported collection, go to the Variables tab.
Ensure base_url is set to http://localhost:3000 (or your VM's IP if testing remotely).
Run the requests to verify Inventory CRUD operations and the asynchronous Billing Queue.

## Project Tree

```
play-with-containers
├─ README.md
├─ docker-compose.yml
├─ play-with-containers API tests.json
├─ play-with-containers-py.png
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

# 1. Log in to Docker Hub (it will ask for your password)

docker login

# 2. Build and tag all 6 images

docker build -t borsok/inventory-db:v1 ./srcs/inventory-db
docker build -t borsok/billing-db:v1 ./srcs/billing-db
docker build -t borsok/rabbitmq-server:v1 ./srcs/rabbitmq-server
docker build -t borsok/inventory-app:v1 ./srcs/inventory-app
docker build -t borsok/billing-app:v1 ./srcs/billing-app
docker build -t borsok/api-gateway-app:v1 ./srcs/api-gateway-app

# 3. Push them to your public Docker Hub repository

docker push borsok/inventory-db:v1
docker push borsok/billing-db:v1
docker push borsok/rabbitmq-server:v1
docker push borsok/inventory-app:v1
docker push borsok/billing-app:v1
docker push borsok/api-gateway-app:v1
