---
title: "Proposal"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Movie Recommendation System
## User Data-Driven Movie Recommendation System

### 1. Executive Summary
The Movie Recommendation System is a user data-driven movie suggestion platform designed to provide a personalized user experience. The system utilizes a combination of Content-Based Filtering and Collaborative Filtering.

The project is comprehensively deployed on AWS using a Real-time Inference architecture. The model training process is executed periodically in the background via SageMaker Processing Jobs. Meanwhile, delivering instant recommendation results is handled by SageMaker Endpoints operating 24/7. The entire system is connected through a FastAPI Backend along with Amazon S3 and DynamoDB databases, ensuring a smooth and seamless response for the user.

### 2. Problem Statement
_Current Problem_

Currently, online movie streaming services are booming, and the number of available movies and TV shows has reached tens or even hundreds of thousands of titles.

Core issues:
- With too many choices, users often experience information overload and spend an excessive amount of time (averaging 15–20 minutes) just browsing through movie lists without selecting a suitable film.

- Manually searching for movies and shows based on keywords or genres does not accurately reflect the diverse and time-varying preferences of users, leading to a degraded user experience.

- Users can easily become frustrated and abandon the application if they do not find engaging content within the first few minutes.

Therefore, streaming services require a smart recommendation system capable of understanding user behavior and preferences, thereby providing personalized suggestions to help users save time and enhance their viewing experience.

_Solution_

To resolve the aforementioned issues, the platform needs to shift from a **passive content delivery** model to a **proactive content recommendation** model. The recommendation system uses Machine Learning algorithms to mine implicit behavioral data, thereby modeling viewer personas and discovering similarity patterns among thousands of movies. To ensure the system operates efficiently and is scalable when interaction data surges, the solution must leverage cloud computing services. By integrating AWS infrastructure services, the project establishes a complete data flow: from collecting interactions and scheduling periodic model training to delivering personalized results with ultra-low latency.

### 3. Solution Architecture

![Overall Architecture](/images/2-Proposal/diagram.png)

_Utilized AWS Services_

- Amazon VPC & Internet Gateway: Provides a private virtual network and secure communication gateway, routing user requests into the system.
- Amazon EC2: Hosts the web application (Vite + FastAPI) and directly serves the API.
- Amazon DynamoDB: Stores movie metadata and user interaction history in real-time.
- Amazon S3: Stores raw data and trained model artifacts.
- Amazon SageMaker: The Machine Learning processing hub of the system, comprising two components:
  - **SageMaker Processing Job:** Runs periodically to train the recommendation model based on interaction data.
  - **SageMaker Endpoint:** A prediction server maintained 24/7, always loaded with the latest model version to serve real-time predictions.
- AWS IAM & CloudWatch: Manages access control flows and monitors system logs.

_Component Design_
- Presentation Layer: Responsible for rendering the user interface, movie catalog, and directly capturing interaction events (click, rate, watch).
- Application Layer: The central coordinator: receives requests from the Frontend, retrieves data from the Database, and makes API calls to the Machine Learning system.
- Data Layer: Separated into two specialized storage pipelines:
    - Hot Data: Managed by DynamoDB, handling high-speed query requirements such as updating user history and movie details.
    - Cold Storage Data: Managed by S3, acting as a secure repository for massive raw datasets and storing model weight versions (Model Artifacts).
- Machine Learning Layer: Operates independently on Amazon SageMaker with two distinct tasks: model retraining and real-time prediction serving.

### 4. Technical Implementation
_Deployment Phases_

The project consists of 2 parts deployed in parallel: building the movie streaming Web application and building the Machine Learning recommendation model.

1. **Infrastructure Setup & Data Preparation:** Establish the basic foundation for both Web and Machine Learning components, finalize the data schema, and process the raw data source.
    - _Web Component:_
        - Set up the project structure, configure the Docker environment, and establish basic CI/CD.  
        - Build the FastAPI Backend framework, design the DynamoDB schema, and create tables on AWS.  
        - Design basic UI/UX, and build the Register/Login and Onboarding flows on Vite. 
    - _Machine Learning Component:_
        - Pre-process The Movies Dataset.
        - Develop a Data Pipeline to output cleaned data into Train/Validation/Test sets.

2. **Cost Estimation:** Use the AWS Pricing Calculator to estimate and adjust expenses.

3. **Feature Development:** Complete features, build user scenarios on the Web, and successfully train the first recommendation models.
    - _Web Component:_
        - Develop the movie detail page, build APIs to display metadata.
        - Build a complete Interaction Pipeline: capture events (such as `click`, `watch`, `rate`, `like`) from the Frontend and save them to the Interactions table on DynamoDB.
    - _Machine Learning Component:_
        - Build the **Popularity Ranker** model for guest users and the **Content-Based Recommender** for logged-in users.
        - Develop the core **Collaborative Filtering** model, converting interaction events into weighted scores.
        - Integrate the model into the SageMaker Endpoint to serve movie recommendation predictions.

4. **System Integration & Cloud Deployment:** Build a POST API on the Backend to route requests. Package the AI model and deploy it to the SageMaker Endpoint to serve real-time predictions. Set up automation for the periodic model retraining process.
    - _Web Component:_
        - Integrate the Machine Learning model into the Backend process. Build a POST API to route requests from the Frontend to the model.
    - _Machine Learning Component:_
        - Automate the model re-train process.
        - Run tests on Amazon SageMaker.

5. **Testing:** Review the entire system, handle errors, and measure real-world performance. Optimize page load times and DynamoDB query speeds. Monitor AWS costs to ensure no background resources exceed the budget. 

_Technical Requirements_

- _Frontend:_ Use Vite and have an understanding of EC2. Build the movie display interface and manage login states.
- _Backend:_ Build using FastAPI. Accurately handle authentication flows, manage the interaction collection pipeline, and route recommendation scenarios based on user status.
- _Machine Learning:_ Develop in Python using the implicit, scikit-learn, numpy, and pandas libraries. Requires building an Implicit ALS model, along with a Weighted RRF hybrid algorithm.
- _Cloud - AWS:_ Amazon S3 to store raw datasets and model result files. Amazon DynamoDB as the main database for the Web. Use Amazon SageMaker to run the automated Re-train process.

### 5. Roadmap & Milestones
- _Pre-internship (Month 0):_ Planning, dataset preparation.
- _Internship (Months 1-3):_
    - Month 1: General understanding of AWS services.
    - Month 2: Build the movie streaming web and recommendation system.
    - Month 3: System testing and preparation for actual deployment.
- _Post-deployment:_ Monitor performance, optimize models, and expand features.

### 6. Budget Estimate
Costs can be viewed on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=26e628729fcf910d969fbf894cec6b86db18ad4c)

_Infrastructure Costs_

- Amazon EC2: 3.87 USD/month (1 t3.micro, 730 hours, 30 GB storage).
- Amazon SageMaker: 
    - Processing Jobs: 1.41 USD/month (1 ml.m5.xlarge, 30 jobs, 10 minutes/job).
    - Endpoints: 85.56 USD/month (1 ml.m5.xlarge, 730 hours).
- Amazon DynamoDB: 0.37 USD/month (On-demand, 1 GB storage, 100,000 requests).
- Amazon S3 Standard: 0.13 USD/month (5 GB storage, 2000 requests).

_Total:_ 91.34 USD/month, 1,096.08 USD/12 months.

### 7. Risk Assessment

_Risk Matrix_

- **Cloud Cost Explosion - High Impact, Medium Probability:** The `ml.m5.large` prediction server configuration running 24/7 exceeds necessary traffic, or failing to configure lifecycle rules for Amazon S3 causes old weight files to accumulate permanently, resulting in hidden storage fees.
- **Model Quality Degradation - High Impact, Fair Probability:** The periodic Processing Jobs running on noisy interaction datasets generate poor-quality models that automatically overwrite the well-functioning Endpoint.
- **Internal Data Discrepancy - High Impact, Low Probability:** Index mapping files not matching the model matrix will cause the system to return incorrect movie IDs, leading to display errors on the user interface.

_Mitigation Strategies_

- **Budget Guardrails & Resource Optimization:** Set up AWS Budgets to send automated alerts when costs reach 50% and 75% of the monthly budget. Apply S3 Lifecycle Rules to automatically delete old model versions after 30 days.
- **Automated Moderation Gate:** A new model is only allowed to be deployed when it meets 3 conditions:
    - The number of scored users exceeds 1000.
    - The performance metric surpasses the Popularity Baseline.
    - There is no more than a 5% accuracy drop compared to the current version.
- **Cross-Data Validation:** Add verification steps on the Backend to ensure every movie ID suggested by the model actually exists in the DynamoDB catalog.

_Contingency Plans_

- **Safe Fallback Mechanism:** If the Endpoint is overloaded or new users lack sufficient calculation data, the Backend will automatically revert to a safe scenario: Suggesting top-rated popular movies read directly from DynamoDB to ensure a seamless experience.
- **Model Rollback:** When a new model is found to provide inaccurate recommendations, administrators can update the SageMaker Endpoint configuration to point to the old weight file. The Endpoint will automatically reload the stable model without requiring a full Backend system restart.

### 8. Expected Outcomes
_Technical Improvements:_ 

- Successfully deploy the Real-time Inference architecture with SageMaker Endpoints operating 24/7, providing instant personalization and completely eliminating cold-start latency.
- The system is capable of reacting immediately to changes in user preferences by capturing implicit interactions (watch time, shares, likes/dislikes) and transmitting them directly to the prediction server.

_Long-term Value:_ 

- Successfully build a solid architectural foundation that meets production-ready standards within the AWS ecosystem.
- Open up the possibility of flexibly reusing the entire data collection and real-time inference pipeline for e-commerce projects, online education, or other large-scale content platforms in the future.