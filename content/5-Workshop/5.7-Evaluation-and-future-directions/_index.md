---
title: "Evaluation and Future Directions"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

This section summarizes the current completeness of the movie recommendation workshop and proposes a future development roadmap. The evaluation is based on the source code, configuration, AWS Console screenshots, and existing test scenarios; it is not an SLA certification for a production environment.

## Workshop Outcome Evaluation

| Area | Current level | Assessment |
|---|---|---|
| Data layer | Documented with deployment evidence | S3 stores datasets, model artifacts, and reports; DynamoDB serves the catalog, interactions, cache, and user state. |
| Recommendation pipeline | Complete at workshop level | Includes popularity, content-based, implicit ALS, hybrid ranking, offline evaluation, and a promotion gate. |
| SageMaker | Operational evidence available | The workshop records a Processing Job and Endpoint; the model packaging and release process still requires full automation. |
| Application integration | Documented and partially tested | The FastAPI provider supports caching, fallback, and endpoint invocation; the EC2 deployment needs additional operational evidence and load testing. |
| IAM and security | Baseline established | Trust policies, least privilege, and required permissions have been identified; automated policy review and continuous monitoring are still required. |
| End-to-end testing | Main flows covered | Guest, new-user, returning-user, interaction, and endpoint checks are covered; this does not replace a production test suite. |

## Strengths

- Clearly separates the batch path on S3 from the request-time path on DynamoDB.
- Supports multiple recommendation strategies for different user states.
- Includes offline evaluation and a promotion gate before model release.
- Uses caching, fallback, and `reason_code` to improve explainability and resilience.
- Identifies IAM boundaries and cleanup order to reduce security and cost risks.

## Current Limitations

- There is no unified Infrastructure as Code package to reproduce S3, DynamoDB, SageMaker, IAM, and EC2.
- The build and deployment process for the SageMaker Endpoint serving container is not fully represented in the reviewed source.
- Most deployment and testing evidence is still collected manually.
- Offline metrics alone are insufficient to determine actual user satisfaction.
- Data drift, model drift, fairness, privacy, and long-term data quality have not been evaluated fully.
- Complete measurements for concurrent load, P95/P99 latency, resilience, and cost per request are not yet available.

## Future Development

### Short Term

1. Standardize infrastructure with AWS CDK, CloudFormation, or Terraform, and separate configuration by environment.
2. Automate data validation, unit tests, integration tests, and end-to-end tests in CI/CD.
3. Add CloudWatch dashboards, alarms, log retention, cost budgets, and pipeline failure alerts.
4. Complete the deployment, rollback, backup, restore, and cleanup runbooks.
5. Add source code, repository, and demo video links to **8. References**.

### Medium Term

1. Orchestrate training, evaluation, and promotion with SageMaker Pipelines or an equivalent stateful workflow.
2. Manage model versions and approvals with a model registry; implement canary or blue/green deployment.
3. Schedule interaction exports and retraining while monitoring data drift and model drift.
4. Run A/B tests to compare a new model with the baseline before expanding traffic.
5. Optimize caching, autoscaling, and batch inference based on actual load.

### Long Term

1. Experiment with two-tower models, sequential recommendation, or learning-to-rank when enough data is available.
2. Add a feature store or another mechanism that manages features consistently between training and inference.
3. Build a near-real-time event pipeline for interactions if recommendation update requirements increase.
4. Complete data governance, personal information protection, audit trails, and access approval processes.
5. Design disaster recovery and test resilience against approved RTO/RPO targets.

## Metrics to Monitor

- **Model quality:** Recall@K, NDCG@K, MAP@K, coverage, diversity, and novelty.
- **User experience:** click-through rate, playback start rate, watch time, and completion rate.
- **Operations:** P50/P95/P99 latency, error rate, fallback rate, cache hit rate, and endpoint availability.
- **Pipeline:** job success rate, training time, promotion time, and model freshness.
- **Cost and security:** cost per request or user, idle resources, IAM findings, and the number of anomalous access events.
