# AWS Cost Optimizer: Step-by-Step Creation Guide

This guide explains the exact sequence to create the files and folders for the AWS Cost Optimizer project. Follow this order to build the project logically from the ground up.

---

## 1. Environment & Dependencies

First, we set up the environment variables and install the required packages so our project has everything it needs to run.

### `/.env`
*   **Create this file first.** 
*   **Purpose:** This file stores your private secrets, primarily your `OPENAI_API_KEY`, and sets your default `AWS_REGION`. 
*   **Why it's first:** You must never hardcode secrets into your main code. Setting this up ensures you start with secure practices from minute one.

### `/requirements.txt`
*   **Create this file second.** 
*   **Purpose:** Lists every Python package the project needs, such as `streamlit`, `boto3`, `langchain`, and `pandas`.
*   **Action:** Run `pip install -r requirements.txt` immediately after creating this to ensure your local environment is ready for the Python code we're about to write.

---

## 2. The `config/` Folder

Now we create the central hub for our app's settings.

### `/config/settings.py`
*   **Create this file third.**
*   **Purpose:** This Python file uses the `dotenv` library to read the `.env` file you just made. It defines constants (like `OPENAI_API_KEY` and `APP_TITLE`) that any other file in the project can import.
*   **Why it's here in the sequence:** Any future file that needs a setting or API key will simply write `from config.settings import OPENAI_API_KEY`. It prevents us from reading the `.env` file multiple times.

---

## 3. The `lambda_/` Folder (AWS Backend)

Before we can build a dashboard, we need the code that actually interacts with AWS to fetch costs and manage resources. Since this will eventually run in the cloud, we group it in the `lambda_/` folder.

### `/lambda_/cost_explorer.py`
*   **Create this file fourth.**
*   **Purpose:** Uses `boto3` to connect to the AWS Cost Explorer API. It contains functions to get your total spend over the last month and your top expensive services.

### `/lambda_/services_inventory.py`
*   **Create this file fifth.**
*   **Purpose:** Uses `boto3` to fetch a list of your running resources. It connects to EC2, RDS, S3, and standard AWS endpoints to gather data on what is actively running.

### `/lambda_/service_actions.py`
*   **Create this file sixth.**
*   **Purpose:** The "action" file. It uses `boto3` to perform destructive or state-changing actions, like stopping an EC2 instance or deleting an S3 bucket.

### `/lambda_/handler.py`
*   **Create this file seventh.**
*   **Purpose:** This is the entry point for AWS Lambda. It takes an incoming internet request, decides if it's asking for cost data or service inventory, routes the request to one of the three files above, and returns the result as JSON.

---

## 4. The `terraform/` Folder (Infrastructure Deployment)

Now that our backend Python code is written in the `lambda_/` folder, we need to deploy it to AWS. We use Terraform to create the cloud resources.

### `/terraform/variables.tf`
*   **Create this file eighth.**
*   **Purpose:** Defines the input variables Terraform needs, such as the `aws_region` and `project_name`. It acts as the settings file for Terraform.

### `/terraform/main.tf`
*   **Create this file ninth.**
*   **Purpose:** The core blueprint. This file tells AWS to create an IAM Security Role, zip up our `lambda_/` folder, deploy it as an AWS Lambda function, and attach an API Gateway so our dashboard can securely talk to it.

### `/terraform/outputs.tf`
*   **Create this file tenth.**
*   **Purpose:** After Terraform finishes building the infrastructure, this file prints out important information, like the newly created `api_gateway_url`. *You will need this URL later!*

*(At this stage, you would run `terraform init` and `terraform apply` in your terminal to deploy your backend.)*

---

## 5. The `dashboard/` Folder (Streamlit UI & AI)

With the backend deployed, we finally build the frontend that the user sees.

### `/dashboard/utils/aws_client.py`
*   **Create this file eleventh.**
*   **Purpose:** A helper class that acts as a bridge. Whenever the dashboard wants to fetch data or stop an EC2 instance, it uses this client to securely talk to the underlying AWS Python SDK (`boto3`). 

### `/dashboard/utils/cost_utils.py`
*   **Create this file twelfth.**
*   **Purpose:** A small helper file for formatting data. For example, it contains a function that turns the number `150` into the string `"$150.00"`.

### `/dashboard/utils/ai_agent.py`
*   **Create this file thirteenth.**
*   **Purpose:** The AI brain. This file imports `LangChain` and connects to OpenAI. It defines special `@tool` functions that allow the AI to look at your AWS resources and offer cost-saving advice.

### `/dashboard/app.py`
*   **Create this file fourteenth.**
*   **Purpose:** The Home Page of your Streamlit app. It sets up the main visualization dashboard, drawing charts (using Plotly) based on the cost data fetched by our AWS client.

### `/dashboard/pages/2_Services.py`
*   **Create this file fifteenth.**
*   **Purpose:** The management page. It displays tabs for EC2, RDS, etc. It provides the UI buttons ("Stop", "Terminate") that trigger the actions we wrote earlier, and it uses `ai_agent.py` to add "Analyze with AI" functionality to each resource.

### `/dashboard/pages/3_AI_Advisor.py`
*   **Create this file sixteenth.**
*   **Purpose:** The final piece. A dedicated chat interface page where the user can type free-form questions to the AI about their AWS bill, relying heavily on the logic we built in `ai_agent.py`.
