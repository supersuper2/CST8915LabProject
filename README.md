# CST8915LabProject

## Application Architecture

![cst8915final2 drawio](https://github.com/user-attachments/assets/70a436f1-dc39-49a1-9d68-7effe449fb2d)

## Application and Architecture Explanation

Customers browse products and make purchases through the store-front application. The store-front retrieves product details from the product-service and displays them to users. Once an order is confirmed, the store-front sends the order information to the order-service, which encrypts the data and publishes it to Azure Service Bus. The makeline-service listens for messages on the Service Bus and stores the latest order in MongoDB. The order-service will send messages via the Service Bus. The store-admin service accesses this order data via the makeline-service and also utilizes the ai-service to generate product descriptions using gpt-4 and create product images with dall-e-3.

## Deployment Instructions

## Step 1: Clone the BestBuy Demo Repository

  aps-all-in-one.yaml is for the application version that includes the connection to the service bus
  
  secrets.yaml stores the Openai Service key
  
  config-maps.yaml stores the configuration
  
## Step 2: Create an Azure Kubernetes Cluster (AKS)

1. **Log in to Azure Portal:**

   - Go to [https://portal.azure.com](https://portal.azure.com/) and log in with your Azure account.

2. **Create a Resource Group:**

   - In the Azure Portal, search for **Resource Groups** in the search bar.
   - Create a resource group for this deployment.

3. **Create an AKS Cluster:**

   - In the search bar, type **Kubernetes services** and click on it.
   - Click **Create** and select **Kubernetes cluster**
   - In the `Basics` tap fill in the following details:
     - **Subscription**: Select your subscription.
     - **Resource group**: Choose the one that you just created.
     - **Cluster preset configuration**: Choose `Dev/Test`.
     - **Region**: Same as your resource group (e.g., `Canada Central`).
     - **Availability zones**: `None`.
     - **AKS pricing tier**: `Free`.
     - **Kubernetes version**: `Default`.
     - **Automatic upgrade**: `Disabled`.
     - **Automatic upgrade scheduler**: `No schedule`.
     - **Node security channel type**: `None`.
     - **Security channel scheduler**: `No schedule`.
     - **Authentication and Authorization**: `Local accounts with Kubernetes RBAC`.
   - In the Node pools, create a masterpool and some worker pools.
     - Set **node size** to `D2as_v4`.

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
     az aks get-credentials --resource-group <your resource group> --name <your cluster name>
     ```

   - Verify Cluster Access:

     - Test your connection to the AKS cluster by listing all nodes:

       ```
       kubectl get nodes
       ```

       You should see details of the nodes in your AKS cluster if the connection is successful.

## Step 3: Set Up the AI Backing Services

To enable AI-generated product descriptions and image generation features, you will deploy the required **Azure OpenAI Services** for GPT-4 (text generation) and DALL-E 3 (image generation). This step is essential to configure the **AI Service** component in the Bestbuy Demo application.

### 1: Create an Azure OpenAI Service Instance

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

------

### 2: Deploy the GPT-4 and DALL-E 3 Models

1. **Access the Azure OpenAI Resource**:
   - Navigate to the Azure OpenAI resource you just created.
2. **Deploy GPT-4**:
   - Go to the **Model Deployments** section and click **Add Deployment**.
   - Choose **GPT-4** as the model and provide a deployment name (e.g., `gpt-4`).
   - Set the deployment configuration as required and deploy the model.
3. **Deploy DALL-E 3**:
   - Repeat the same process to deploy **DALL-E 3**.
   - Use a descriptive deployment name (e.g., `dalle-3`).
4. **Note Configuration Details**:
   - Once deployed, note down the following details for each model:
     - Deployment Name
     - Endpoint URL

------

### 3: Retrieve and Configure API Keys

1. **Get API Keys**:

   - Go to the **Keys and Endpoints** section of your Azure OpenAI resource.
   - Copy the **API Key (API key 1)** and **Endpoint URL**.

2. **Base64 Encode the API Key**:

   - Use the following command to Base64 encode your API key:

     ```
     echo -n "<your-api-key>" | base64
     ```

   - Replace `<your-api-key>` with your actual API key.

------

### Task 4: Update AI Service Deployment Configuration in the `Deployment Files` folder.

1. **Modify Secretes YAML**:

   - Edit the `secrets.yaml` file.
   - Replace `OPENAI_API_KEY` placeholder with the Base64-encoded value of the `API_KEY`.

2. **Modify Deployment YAML**:

   - Edit the `all-in-one.yaml` file.
   - Replace the placeholders with the configurations you retrieved:
     - `AZURE_OPENAI_DEPLOYMENT_NAME`: Enter the deployment name for GPT-4.
     - `AZURE_OPENAI_ENDPOINT`: Enter the endpoint URL for the GPT-4 deployment.
     - `AZURE_OPENAI_DALLE_ENDPOINT`: Enter the endpoint URL for the DALL-E 3 deployment.
     - `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME`: Enter the deployment name for DALL-E 3.

   Example configuration in the YAML file:

   ```
   - name: AZURE_OPENAI_API_VERSION
     value: "2024-07-01-preview"
   - name: AZURE_OPENAI_DEPLOYMENT_NAME
     value: "gpt-4"
   - name: AZURE_OPENAI_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_ENDPOINT
     value: "https://<your-openai-resource-name>.openai.azure.com/"
   - name: AZURE_OPENAI_DALLE_DEPLOYMENT_NAME
     value: "dalle-3"
   ```

## Step 4: Deploy the Secrets

- Create and Deploy the Secret for OpenAI API:

  - Make sure that you have replaced Base64-encoded-API-KEY in secrets.yaml with your Base64-encoded OpenAI API key.

  ```
  kubectl apply -f secrets.yaml
  ```

- Verify:

  ```
  kubectl get configmaps
  kubectl get secrets
  ```

## Step 4: Deploy the Bestbuy Demo Application

1. **Apply the YAML file to the AKS cluster:**

   - In this step, use the K8s deployment YAML file provided: `aps-all-in-one.yaml`.

   - Open the terminal and navigate to the file directory.

   - Run the following command to apply the YAML configuration and deploy the application to AKS:

     ```
     kubectl apply -f aps-all-in-one.yaml
     ```

2. **Verify the deployment:**

   - After the command executes, verify that the pods are running by using the following command:

     ```
     kubectl get pods
     ```

3. **Check services:**

   - Confirm that all services are up and running:

     ```
     kubectl get services
     ```

   - using `EXTERNAL-IP`:80  of store-front and store-admin services to visit those pages.

## Table of Microservice Repositories

| **Service**      | **Docker Image Link**                                    |
| ---------------- | -------------------------------------------------------- |
| Store-Front      | `https://github.com/supersuper2/store-front-L8`      |
| Order-Service    | `https://github.com/supersuper2/order-service-L8`    |
| Store-Admin      | `https://github.com/supersuper2/store-admin-L8`      |
| Product-Service  | `https://github.com/supersuper2/product-service-L8`  |
| Virtual-Worker   | `https://github.com/supersuper2/virtual-worker-L8`   |
| Virtual-Customer | `https://github.com/supersuper2/virtual-customer-L8` |
| Ai-Service       | `https://github.com/supersuper2/ai-service-L8`       |
| Makeline-Service | `https://github.com/supersuper2/makeline-service-L8` |

## Table of Docker Images

| **Service**      | **Docker Image Link**                                        |
| ---------------- | ------------------------------------------------------------ |
| Store-Front      | `https://hub.docker.com/r/sing1883/store-front-a2` |
| Order-Service    | `https://hub.docker.com/r/sing1883/order-service-a2` |
| Store-Admin      | `https://hub.docker.com/r/sing1883/store-admin-a2` |
| Product-Service  | `https://hub.docker.com/r/sing1883/product-service-a2` |
| Virtual-Worker   | `https://hub.docker.com/r/sing1883/virtual-worker-a2` |
| Virtual-Customer | `https://hub.docker.com/r/sing1883/virtual-customer-a2` |
| Ai-Service       | `https://hub.docker.com/r/sing1883/ai-service-a2` |
| Makeline-Service | `https://hub.docker.com/r/sing1883/makeline-service-a2` |

## Demo Video
https://youtu.be/KxmestPgGY8

## Limitations and Issues 
1. The first limitation that had occured was the I creating the azure openAI service I was already "Insufficient quota", Which meant I was unable to use the AI services. To get around this, I had just changed the region to East US 2, and everything seemed to be working again.

![errorcst8915](https://github.com/user-attachments/assets/b1fd1ba8-aa84-4664-b009-f5a41e1b1f43)

2. The second issue that had occured was that the product description was not showing up. The page had shown an error, but when I was looking at the logs, there was no error. But when generating the product image, I was successful in doing so, so this leads me to believe that either something in the backend/frontend was not programmed properly or that a problem from the azures side, since the product image was generate. Inputting the AI services keys and endpoints, much have been done correctly. However the product description was not generated.

![issue2](https://github.com/user-attachments/assets/e7be4cab-ae6d-46c6-a1f9-42af5f92a2a3)











