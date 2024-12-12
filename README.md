# Assignment #2: Building a Cloud-Native App for Best Buy

## Overview

This project involves designing, building, and deploying a cloud-native application for Best Buy's online store using Kubernetes. The application follow the microservices architecture principles of the Algonquin Pet Store (On Steroids) and incorporate AI-powered product descriptions and image generation using GPT-4 and DALL-E models.

---
## Application Architecture


### Components Overview

| Service              | Description                                                       | Notes                           |
| -------------------- | ----------------------------------------------------------------- | ------------------------------- |
| **Store-Front**      | Customer-facing app for browsing and placing orders.              | -                               |
| **Store-Admin**      | Employee-facing app for managing products and viewing orders.     | -                               |
| **Order-Service**    | Handles order creation and sends data to the managed order queue. | Uses RabbitMQ for messaging.    |
| **Product-Service**  | Handles CRUD operations for product data.                         | -                               |
| **Makeline-Service** | Processes and completes orders by reading from the order queue.   | -                               |
| **AI-Service**       | Generates product descriptions and images.                        | Uses GPT-4 and DALL-E-3 models. |
| **Database**         | MongoDB for persisting order and product data.                    | -                               |

### Architecture Diagram

![diagram](assets/Algonquin%20Pet%20Store%20On%20Steroids.png)

### Application and Architecture Explanation

The application consists of multiple microservices, each handling a specific functionality to support Best Buy's online store:

- **Store-Front:** A Vue.js application that provides customers a user-friendly interface to browse and place orders.
- **Store-Admin:** Another Vue.js application tailored for employees to manage products and track orders.
- **Order-Service:** Built in Node.js, this microservice processes customer orders and queues them in RabbitMQ for further handling.
- **Product-Service:** Developed using Rust, it manages CRUD operations for product data.
- **Makeline-Service:** Written in Go, it processes orders from RabbitMQ and updates the MongoDB database.
- **AI-Service:** A Python-based microservice leveraging GPT-4 for generating product descriptions and DALL-E for creating product images.
- **MongoDB Database:** Centralized storage for persisting product and order data, ensuring data integrity and availability.

The architecture ensures a seamless flow of data between customers, employees, and backend services. It emphasizes scalability, modularity, and real-time updates to meet the high demands of an eCommerce platform.

---
### Deployment Instrunction 

## Step 1: Clone the BestBuy Repository

To begin, clone the [**Best Buy (On Steroids)**]() repository, which contains all necessary deployment files.

 **Review the Deployment Files**:
   - Navigate to the `Deployment Files` folder
   - This folder contains YAML files for deploying all necessary Kubernetes resources, including services, deployments, StatefulSets, ConfigMaps, and Secrets.


## Step 2: Set Up the AKS Cluster
Create an AKS cluster with two worker nodes for this exercise.

1. **Log in to Azure Portal:**
   - Go to [https://portal.azure.com](https://portal.azure.com) and log in with your Azure account.

2. **Create a Resource Group:**
   - In the Azure Portal, search for **Resource Groups** in the search bar.
   - Click **Create** and fill in the following:
     - **Resource group name**: `bestbuy`
     - **Region**: `Canada`.
   - Click **Review + Create** and then **Create**.

3. **Create an AKS Cluster:**
   - In the search bar, type **Kubernetes services** and click on it.
   - Click **Create** and select **Kubernetes cluster**
   - In the `Basics` tap fill in the following details:
     - **Subscription**: Select your subscription.
     - **Resource group**: Choose `bestbuy`.
     - **Cluster preset configuration**: Choose `Dev/Test`.
     - **Kubernetes cluster name**: `BestBuyCluster`.
     - **Region**: Same as your resource group (e.g., `Canada`).
     - **Availability zones**: `None`.
     - **AKS pricing tier**: `Free`.
     - **Kubernetes version**: `Default`.
     - **Automatic upgrade**: `Disabled`.
     - **Automatic upgrade scheduler**: `No schedule`.
     - **Node security channel type**: `None`.
     - **Security channel scheduler**: `No schedule`.
     - **Authentication and Authorization**: `Local accounts with Kubernetes RBAC`.
   - In the `Node pools` tap fill in the following details:
     - Select **agentpool**. Optionally change its name to `masterpool`. This nodes will have the controlplane.
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `1`
        - Click `update`
     - Click on **Add node pool**:
        - **Node pool name**: `workerspool`.
        - **Mode**: `User` 
        - Set **node size** to `D2as_v4`.
        - **Scale method**: `Manual`
        - **Node count**: `2`
        - Click `add`
   - Click **Review + Create**, and then **Create**. The deployment will take a few minutes.

4. **Connect to the AKS Cluster:**
   - Once the AKS cluster is deployed, navigate to the cluster in the Azure Portal.
   - In the overview page, click on **Connect**. 
   - Select **Azure CLI** tap. You will need Azure CLI. If you don't have it: [**Install Azure CLI**](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
   - Login to your azure account using the following command:
      ```
      az login
      ```
   - Set the cluster subscription using the command shown in the portal (it will look something like this):
      ```
      az account set --subscription 'subscribtion-id'
      ```

   - Copy the command shown in the portal for configuring `kubectl` (it will look something like this):
     ```
     az aks get-credentials --resource-group bestbuy --name BestbuyCluster
     ```
      **Understanding the Command:**
      - The command `az aks get-credentials` pulls the necessary configuration files to enable `kubectl` to access your AKS cluster. Here’s a breakdown:
     - `--resource-group` specifies the resource group where your AKS cluster resides.
     - `--name` specifies the name of your AKS cluster.
     - `--overwrite-existing` can be used to overwrite any existing Kubernetes configuration files for the same cluster. This is useful if you’ve connected to the cluster before or if multiple configurations exist for it.
   - Verify Cluster Access:
      - Test your connection to the AKS cluster by listing all nodes:
        ```
        kubectl get nodes
        ```
        You should see details of the nodes in your AKS cluster if the connection is successful.
---

## Step 3: Set Up the AI Backing Services
To enable AI-generated product descriptions and image generation features, you will deploy the required **Azure OpenAI Services** for GPT-4 (text generation) and DALL-E 3 (image generation). This step is essential to configure the **AI Service** component in the Best Buy application.

### Task 1: Create an Azure OpenAI Service Instance

1. **Navigate to Azure Portal**:
   - Go to the [Azure Portal](https://portal.azure.com/).

2. **Create a Resource**:
   - Select **Create a Resource** from the Azure portal dashboard.
   - Search for **Azure OpenAI** in the marketplace.

3. **Set Up the Azure OpenAI Resource**:
   - Choose the **East US** region for deployment to ensure capacity for GPT-4 and DALL-E 3 models.
   - Fill in the required details:
     - Resource group: Use an existing one or create a new group.
     - Pricing tier: Select `Standard`.

4. **Deploy the Resource**:
   - Click **Review + Create** and then **Create** to deploy the Azure OpenAI service.

---

### Task 2: Deploy the GPT-4 and DALL-E 3 Models

1. **Access the Azure OpenAI Resource**:
   - Navigate to the Azure OpenAI resource you just created.

2. **Deploy GPT-4**:
   - Go to the **Model Deployments** section and click **Add Deployment**.
   - Choose **GPT-4** as the model and provide a deployment name (e.g., `gpt-4-deployment`).
   - Set the deployment configuration as required and deploy the model.

3. **Deploy DALL-E 3**:
   - Repeat the same process to deploy **DALL-E 3**.
   - Use a descriptive deployment name (e.g., `dalle-3-deployment`).

4. **Note Configuration Details**:
   - Once deployed, note down the following details for each model:
     - Deployment Name
     - Endpoint URL

---

### Task 3: Retrieve and Configure API Keys

1. **Get API Keys**:
   - Go to the **Keys and Endpoints** section of your Azure OpenAI resource.
   - Copy the **API Key (API key 1)** and **Endpoint URL**.

2. **Base64 Encode the API Key**:
   - Use the following command to Base64 encode your API key:
     ```bash
     echo -n "<your-api-key>" | base64
     ```
   - Replace `<your-api-key>` with your actual API key.

---

### Task 4: Update AI Service Deployment Configuration in the `Deployment Files` folder.
1. **Modify Secretes YAML**:
   - Edit the `secrets.yaml` file.
   - Replace `OPENAI_API_KEY` placeholder with the Base64-encoded value of the `API_KEY`. 
2. **Modify Deployment YAML**:
   - Edit the `aps-all-in-one.yaml` file.
   - Replace the placeholders with the configurations you retrieved:
     - `AZURE_OPENAI_DEPLOYMENT_NAME`: Enter the deployment name for GPT-4.
     - `AZURE_OPENAI_ENDPOINT`: Enter the endpoint URL for the GPT-4 deployment.
     - `AZURE_OPENAI_DALLE_ENDPOINT`: Enter the endpoint URL for the DALL-E 3 deployment.
     - `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME`: Enter the deployment name for DALL-E 3.

   Example configuration in the YAML file:
   ```yaml
   - name: AZURE_OPENAI_API_VERSION
     value: "2024-07-01-preview"
   - name: AZURE_OPENAI_DEPLOYMENT_NAME
     value: "gpt-4-deployment"
   - name: AZURE_OPENAI_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_DEPLOYMENT_NAME
     value: "dalle-3-deployment"
   ```

## Step 4: Deploy the ConfigMaps and Secrets
- Deploy the ConfigMap for RabbitMQ Plugins:
   ```bash
   kubectl apply -f config-maps.yaml
   ```
- Create and Deploy the Secret for OpenAI API:  
   - Make sure that you have replaced Base64-encoded-API-KEY in secrets.yaml with your Base64-encoded OpenAI API key.
   ```bash
   kubectl apply -f secrets.yaml
   ```
- Verify:
   ```bash
   kubectl get configmaps
   kubectl get secrets
   ```

## Step 5: Deploy the Application
   ```bash
   kubectl apply -f aps-all-in-one.yaml
   ```
### Validate the Deployment
- Check Pods and Services:
   ```bash
   kubectl get pods
   kubectl get services
   ```
- Test Frontend Access:
   - Locate the external IPs for store-front and store-admin services:
   ```bash
   kubectl get services
   ```
   - Access the Store Front app at the external IP on port 80.
   - Access the Store Admin app at the external IP on port 80.

## Step 6: Scale and Monitor Services
### Scale Deployments:
- Scale the `order-service` to 3 replicas:
```bash
kubectl scale deployment order-service --replicas=3
```
- Check Scaling:
```bash
kubectl get pods
```
- Monitor Resource Usage:

   - Enable metrics server for resource monitoring.
   - Use kubectl top to monitor pod and node usage:
   ```bash
   kubectl top pods
   kubectl top nodes
   ```

## Step 7: Explore Advanced Features
### AI-Generated Descriptions and Images:
- Use the AI Service for generating product descriptions and images.
- Ensure your OpenAI API key is correctly configured in the deployed secret.
### RabbitMQ Management:
- Access the RabbitMQ management UI:
   ```bash
   kubectl port-forward service/rabbitmq 15672:15672
   ```
   The kubectl port-forward command is used to forward a local port to a port on a Kubernetes resource (e.g., a Pod or Service). This allows you to access the application running in the cluster from your local machine without exposing it externally.


- Login with the default credentials (`username`/`password`).

### MongoDB Shell Access and Database Exploration
In this section, you will use the MongoDB shell to interact with the `orderdb` database, which stores order information for the Best Buy application. Follow the steps below to connect to the MongoDB pod and explore its contents.

#### **1- Access the MongoDB Shell**
Run the following command to connect to the MongoDB shell inside the running MongoDB pod:
```bash
kubectl exec -it <mongodb-pod-name> -- mongo
```
Explanation: This command uses kubectl exec to open an interactive shell (-it) inside the MongoDB pod and starts the MongoDB shell program (mongo).

#### **2- List All Databases**
Once inside the MongoDB shell, run:
```bash
show dbs
```
Explanation: The show dbs command lists all databases available on the MongoDB server. You should see a list that includes the orderdb, which stores order-related data for the application.
#### **3- Switch to the Order Database**
```bash
use orderdb
```
Explanation: The use orderdb command selects the orderdb database, making it the active database for subsequent queries and commands.
#### **4- List Collections in the Database**
Display all collections in the orderdb database:
```bash
show collections
```
Explanation: The show collections command lists all collections (similar to tables in relational databases) in the current database. The orders collection contains the order data.
#### **5- Query the Orders Collection**
Retrieve all documents in the orders collection:
```bash
db.orders.find()
```
Explanation: The db.orders.find() command fetches and displays all documents (records) in the orders collection. This allows you to view the stored order data, including details such as customer information, products, and order status.
---

# Microservices Repository Table

This table lists the GitHub repositories for each of the microservices in the project:

| **Service**        | **Repository Link**                          |
|--------------------|----------------------------------------------|
| Store-Front        | [store-front-bestbuy](https://github.com/Vishal-Vekariya/store-front-bestbuy)  |
| Order-Service      | [order-service-bestbuy](https://github.com/Vishal-Vekariya/order-service-bestbuy) |
| Product-Service    | [product-service-bestbuy](https://github.com/Vishal-Vekariya/product-service-bestbuy) |
| Makeline-Service   | [makeline-service-bestbuy](https://github.com/Vishal-Vekariya/makeline-service-bestbuy) |
| Store-Admin        | [store-admin-bestbuy](https://github.com/Vishal-Vekariya/store-admin-bestbuy) |
| AI-Service        | [store-front-bestbuy](https://github.com/Vishal-Vekariya/store-front-bestbuy) |

# Docker Images Table

The following table provides links to the Docker Hub repositories for each of the microservices' Docker images:

| **Service**        | **Docker Image Link**                                          |
|--------------------|---------------------------------------------------------------|
| Store-Front        | [vishalvekariya/store-front](https://hub.docker.com/r/vishalvekariya/store-front) |
| Order-Service      | [vishalvekariya/order-service](https://hub.docker.com/r/vishalvekariya/order-service) |
| Product-Service    | [vishalvekariya/product-service](https://hub.docker.com/r/vishalvekariya/product-service) |
| Makeline-Service   | [vishalvekariya/makeline-service](https://hub.docker.com/r/vishalvekariya/makeline-service) |
| Store-Admin        | [vishalvekariya/store-admin](https://hub.docker.com/r/vishalvekariya/store-admin) |
| Ai-Service         | [vishalvekariya/ai-service](https://hub.docker.com/r/vishalvekariya/ai-service) |
