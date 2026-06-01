# ☁️ AWS Cost Optimizer Dashboard

An AWS cost optimization dashboard powered by Streamlit, Terraform, Lambda, API Gateway, boto3, and an AI advisor. Monitor cloud spend, inspect active AWS resources, and trigger cost-saving actions from a clean web interface.

## 🌟 Features

- **Cost Analytics Dashboard**: View recent AWS spend, service-level breakdowns, and forecast-style charts
- **AWS Resource Inventory**: List EC2, RDS, S3, Lambda, ECS, ElastiCache, and NAT Gateway resources
- **Actionable Controls**: Stop/start EC2 and RDS, delete Lambda functions, S3 buckets, ECS clusters, ElastiCache clusters, and NAT Gateways
- **AI Cost Advisor**: Uses LangChain and OpenAI to explain optimization opportunities in plain English
- **Serverless Backend**: Deploys Python Lambda behind API Gateway using Terraform
- **Interactive UI**: Streamlit frontend with Plotly visualizations and confirmation prompts for destructive actions

## 🛠️ Tech Stack

- **Frontend**: Streamlit
- **Backend**: AWS Lambda + API Gateway
- **Infrastructure**: Terraform
- **AWS SDK**: boto3 / botocore
- **AI Framework**: LangChain + OpenAI
- **Charts**: Plotly
- **Language**: Python 3.10+

## 📁 Project Structure

```text
AWS-Cost-Optimizer-Dashboard/
├── config/                 # Application settings loaded from environment variables
│   └── settings.py
├── dashboard/              # Streamlit frontend
│   ├── app.py              # Dashboard home page
│   ├── pages/
│   │   ├── 2_Services.py   # AWS service inventory and actions
│   │   └── 3_AI_Advisor.py # AI advisor chat page
│   └── utils/
│       ├── ai_agent.py     # LangChain-based cost advisor
│       ├── aws_client.py   # API Gateway client wrapper
│       └── cost_utils.py   # UI formatting helpers
├── lambda_/                # Lambda backend source code
│   ├── cost_explorer.py    # AWS Cost Explorer queries
│   ├── handler.py          # Lambda request router
│   ├── service_actions.py  # Stop, start, terminate, and delete actions
│   └── services_inventory.py
├── terraform/              # AWS infrastructure as code
│   ├── main.tf
│   ├── outputs.tf
│   └── variables.tf
├── requirements.txt
└── README.md
```

## 🚀 Getting Started

### Prerequisites

- Python 3.10 or higher
- AWS CLI configured with credentials that can deploy Terraform resources
- Terraform 1.5 or higher
- OpenAI API key for the AI Advisor

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/deep1305/AWS_Cost_Optimizer_Dashboard.git
   cd AWS_Cost_Optimizer_Dashboard
   ```

2. **Create and activate a virtual environment**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **Deploy the backend**
   ```bash
   cd terraform
   terraform init
   terraform apply
   ```

5. **Copy the API Gateway URL**
   ```bash
   terraform output api_gateway_url
   ```

6. **Set up environment variables**

   Create a `.env` file in the project root:
   ```env
   OPENAI_API_KEY=sk-your-openai-api-key-here
   OPENAI_MODEL=gpt-4o
   AWS_REGION=us-east-1
   APP_TITLE=AWS Cost Optimizer
   COST_LOOKBACK_DAYS=30
   API_GATEWAY_URL=https://your-api-id.execute-api.us-east-1.amazonaws.com/dev/optimize
   ```

7. **Run the dashboard**
   ```bash
   streamlit run dashboard/app.py
   ```

8. **Open your browser** to `http://localhost:8501`

## 💡 Usage

Use the sidebar to choose an AWS region and refresh dashboard data.

- Open the **Home** page to review total spend, service cost breakdowns, and trend charts
- Open the **Services** page to inspect AWS resources and run approved stop/delete actions
- Open the **AI Advisor** page to ask questions like *"Where can I reduce my AWS bill this month?"*
- Use resource-specific advice buttons to get short recommendations for individual services

## 🏗️ Terraform Deployment

Terraform provisions the serverless backend used by the dashboard:

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

The deployment creates:

- IAM role and policy for the Lambda backend
- Lambda function packaged from the `lambda_/` folder
- API Gateway REST endpoint
- CloudWatch log group

After deployment, add the `api_gateway_url` output to your `.env` file as `API_GATEWAY_URL`.

## 📊 How It Works

1. **Dashboard Request**: Streamlit calls the API Gateway endpoint through `dashboard/utils/aws_client.py`
2. **Lambda Routing**: `lambda_/handler.py` receives the action and routes it to the correct backend function
3. **AWS Querying**: boto3 queries Cost Explorer, EC2, RDS, S3, Lambda, ECS, ElastiCache, and NAT Gateway APIs
4. **Resource Actions**: Approved actions call AWS APIs to stop, start, terminate, or delete selected resources
5. **AI Analysis**: LangChain sends selected cost/resource context to OpenAI for optimization guidance

## ⚠️ Safety Notes

This project can perform real actions in your AWS account.

- **EC2 terminate** permanently deletes an instance
- **S3 delete** empties bucket contents and deletes the bucket
- **Lambda, ECS, ElastiCache, and NAT Gateway delete** actions remove live resources
- Always review the confirmation prompt before running destructive actions
- Use a sandbox AWS account while testing the dashboard

## 🙏 Acknowledgments

- Built with Streamlit, boto3, Terraform, LangChain, and OpenAI
- Inspired by practical cloud cost optimization workflows
- Designed as an end-to-end AWS, serverless, and AI engineering project

---

**Made for cloud engineers who want cleaner AWS bills**
