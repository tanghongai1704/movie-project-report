---
title: "Evaluation and Future Directions"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

This section summarizes the current maturity of the movie recommendation workshop and proposes a practical roadmap for further development. The assessment is based on the available source code, configuration, AWS Console evidence, and test scenarios; it does not represent a production SLA certification.

## Workshop Evaluation

| Area | Current maturity | Assessment |
|---|---|---|
| Data layer | Documented with deployment evidence | S3 stores datasets, model artifacts, and reports; DynamoDB supports catalog, interaction, cache, and user-state access patterns. |
| Recommendation pipeline | Complete at workshop level | Popularity, content-based, implicit ALS, hybrid ranking, offline evaluation, and a promotion gate are covered. |
| SageMaker | Operational evidence available | Processing Job and Endpoint evidence is recorded; model packaging and release automation still require completion. |
| Application integration | Documented and partially tested | The FastAPI provider supports caching, fallback, and endpoint invocation; EC2 operations and load testing need additional evidence. |
| IAM and security | Baseline established | Trust policies, least privilege, and required permissions are identified; automated policy review and continuous monitoring remain future work. |
| End-to-end testing | Primary workflows covered | Guest, new-user, returning-user, interaction, and endpoint scenarios are included but do not replace a production test suite. |

## Strengths

- Clear separation between the S3 batch path and the DynamoDB request-time path.
- Multiple recommendation strategies for different user states.
- Offline evaluation and a promotion gate before model release.
- Caching, fallback behavior, and `reason_code` values for resilience and explainability.
- Explicit IAM boundaries and cleanup ordering to reduce security and cost risks.

## Current Limitations

- No unified Infrastructure as Code package recreates S3, DynamoDB, SageMaker, IAM, and EC2 resources.
- The audited source does not fully demonstrate the build and deployment path for the SageMaker serving container.
- Much of the deployment and test evidence is still collected manually.
- Offline metrics alone cannot establish real user satisfaction.
- Data drift, model drift, fairness, privacy, and long-term data quality have not been fully assessed.
- Concurrency, P95/P99 latency, recovery behavior, and cost-per-request results are incomplete.

## Future Roadmap

### Short Term

1. Standardize infrastructure with AWS CDK, CloudFormation, or Terraform and separate configuration by environment.
2. Automate data validation, unit, integration, and end-to-end tests in CI/CD.
3. Add CloudWatch dashboards, alarms, log retention, cost budgets, and pipeline failure alerts.
4. Complete deployment, rollback, backup, restore, and cleanup runbooks.
5. Add source code, repository, and demo video links to **8. References**.

### Medium Term

1. Orchestrate training, evaluation, and promotion with SageMaker Pipelines or an equivalent stateful workflow.
2. Manage model versions and approvals in a model registry with canary or blue/green deployment.
3. Schedule interaction exports and retraining while monitoring data and model drift.
4. Run A/B tests comparing promoted models with the current baseline before expanding traffic.
5. Tune caching, autoscaling, and batch inference using measured production demand.

### Long Term

1. Evaluate two-tower, sequential recommendation, or learning-to-rank models when data volume supports them.
2. Introduce a feature store or another mechanism that keeps training and inference features consistent.
3. Build a near-real-time interaction event pipeline if recommendation freshness requirements increase.
4. Complete data governance, personal-data protection, audit trails, and access approval processes.
5. Design disaster recovery and resilience tests against approved RTO and RPO objectives.

## Recommended Success Metrics

- **Model quality:** Recall@K, NDCG@K, MAP@K, coverage, diversity, and novelty.
- **User experience:** Click-through rate, playback-start rate, watch time, and completion rate.
- **Operations:** P50/P95/P99 latency, error rate, fallback rate, cache hit rate, and endpoint availability.
- **Pipeline:** Job success rate, training duration, promotion lead time, and model freshness.
- **Cost and security:** Cost per request or active user, idle resources, IAM findings, and anomalous access events.

{{% notice note %}}
Treat an improvement as complete only when it has an owner, measurable pass/fail criteria, test evidence, and a corresponding rollback plan.
{{% /notice %}}
