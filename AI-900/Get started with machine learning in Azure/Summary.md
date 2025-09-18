# Get Started with Machine Learning in Azure — Summary

## Introduction
- ML solutions power predictive analytics, personalization, and modern AI apps.  
- Building ML in Azure involves six steps:  
  1. **Define the problem**  
  2. **Get the data**  
  3. **Prepare the data**  
  4. **Train the model**  
  5. **Integrate the model**  
  6. **Monitor the model**  

---

## Step 1 – Define the Problem
- Decide: what to predict, which ML task to use, and how to measure success.  
- Tasks: **Classification**, **Regression**, **Time-series forecasting**, **Computer vision**, **NLP**.  
- Metrics depend on task (accuracy, precision, etc.).  
- Example: diabetes prediction → **classification**.

---

## Step 2 – Get and Prepare Data
- **Data quality and quantity** directly affect accuracy.  
- Identify **data sources** (CRM, SQL, IoT, etc.) and **formats** (structured, semi-structured, unstructured).  
- Use **ETL/ELT pipelines** for ingestion: extract → transform → load.  
- Azure tools: **Synapse Analytics**, **Databricks**, **Azure ML**.  
- Common flow: source → transform (Synapse) → store (Blob Storage) → train (Azure ML).

---

## Step 3 – Train the Model
- Choose service based on control, resources, and language:  
  - **Azure Machine Learning**: full ML lifecycle management.  
  - **Azure Databricks**: big data & Spark-based ML.  
  - **Microsoft Fabric**: integrated analytics (data → ML → Power BI).  
  - **Azure AI Services**: prebuilt models with API access.  
- **Azure ML Features**: centralized data, on-demand compute, AutoML, pipelines, MLflow integration, Responsible AI tools.  
- **Azure ML Studio**: browser-based interface for data, compute, AutoML, pipelines, and deployments.  
- **Compute choices**: CPU (cheap, small data), GPU (images, text, large data), distributed Spark.  
- AutoML: automates algorithm selection and hyperparameter tuning.

---

## Step 4 – Integrate the Model
- Deploy to an **endpoint** for app use.  
- Options:  
  - **Real-time predictions** (instant, always-on compute; e.g., recommendations on a website).  
  - **Batch predictions** (scheduled, cost-efficient; e.g., weekly sales forecast).  
- Decision factors: frequency, latency needs, individual vs. batch, and compute costs.  
  - **Real-time** → ACI or AKS; compute always active (higher cost).  
  - **Batch** → scalable clusters; compute spins up only when needed.

---

## Key Takeaways
- Azure provides a **complete ML lifecycle platform**.  
- Core workflow: define → prepare → train → deploy → monitor.  
- Services like **Azure ML**, **Databricks**, and **Fabric** streamline collaboration.  
- Deployment choice (real-time vs batch) balances **speed vs cost**.  
- AutoML and Responsible AI are built-in to help scale safely.
