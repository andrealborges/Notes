# Get Started with Machine Learning in Azure – 10 Question Test

1. What is the first step in designing a machine learning solution? 

- [ ] a) Train the model  
- [ ] b) Define the problem  
- [ ] c) Prepare the data  
- [ ] d) Monitor the model  

Answer: B
* The first step is to clearly define what the model should predict, the type of ML task, and what success looks like. This sets the foundation for all later steps.

2. Which of the following is not listed as a machine learning task type?  

- [ ] a) Classification  
- [ ] b) Regression  
- [ ] c) Reinforcement learning  
- [ ] d) Time-series forecasting  

Answer: C
* The module lists classification, regression, time-series forecasting, computer vision, and NLP as core ML tasks. Reinforcement learning isn’t covered here.

3. In the diabetes prediction example, which machine learning task is being used?  

- [ ] a) Regression  
- [ ] b) Classification  
- [ ] c) Clustering  
- [ ] d) Computer vision  

Answer: B
* The diabetes scenario predicts whether a patient has or does not have diabetes, which is a categorical output — a classification problem.

4. What is the purpose of splitting data into training and test sets?  

- [ ] a) To reduce dataset size  
- [ ] b) To make model training faster  
- [ ] c) To evaluate model performance and avoid overfitting  
- [ ] d) To anonymize sensitive data

Answer: C
* By holding out test data, we can check how well the model generalizes to unseen data, ensuring it doesn’t just memorize training examples.

5. Which Azure service provides distributed Spark compute for large-scale data processing and training?  

- [ ] a) Azure Machine Learning  
- [ ] b) Azure AI Services  
- [ ] c) Azure Databricks  
- [ ] d) Microsoft Fabric  

Answer: C
* Databricks is Azure’s distributed Spark-based analytics platform, used for large-scale data engineering and model training.

6. Which compute option is typically more efficient for training models on images or text data?  

- [ ] a) CPU  
- [ ] b) GPU  
- [ ] c) Memory optimized CPU  
- [ ] d) General purpose CPU  

Answer: B
* GPUs are optimized for parallel computations, making them far more efficient for unstructured data like images and text compared to CPUs.

7. What does Azure Automated Machine Learning (AutoML) primarily help with?  

- [ ] a) Automating the deployment of models to Kubernetes  
- [ ] b) Automating iterative model training and hyperparameter tuning  
- [ ] c) Storing datasets in Azure Blob Storage  
- [ ] d) Providing only prebuilt models for image recognition  

Answer: B
* Azure AutoML handles the repetitive tasks of testing different algorithms and hyperparameters to find the best model without manual coding.

8. Which deployment option is best suited when predictions must be generated instantly for a mobile app or website?  

- [ ] a) Batch deployment  
- [ ] b) Real-time deployment  
- [ ] c) Offline scoring  
- [ ] d) Quarterly scoring

Answer: B
* Real-time endpoints are used when applications (like websites or mobile apps) require predictions instantly, e.g., product recommendations during browsing.

9. Why can batch predictions be more cost-effective than real-time predictions?  

- [ ] a) They don’t require any compute power  
- [ ] b) Compute is provisioned only when batch jobs are triggered, then scales down to zero when idle  
- [ ] c) They never require GPUs  
- [ ] d) Batch jobs always run faster than real-time jobs  

Answer: B
* Batch predictions save costs by only using compute when scoring data in bulk, instead of keeping resources always on like real-time models.

10. Which Azure resources are automatically created along with an Azure Machine Learning workspace?  

- [ ] a) Storage accounts, container registries, and virtual machines  
- [ ] b) SQL databases, Power BI dashboards, and Logic Apps  
- [ ] c) Only storage accounts  
- [ ] d) None, they must be provisioned manually  

Answer: A
* When you create an Azure Machine Learning workspace, Azure automatically provisions supporting resources like storage, compute, and container registries to enable end-to-end ML workflows.