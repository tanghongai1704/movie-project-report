---
title: "End-to-End Testing"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

This section covers independent testing of guest workflows, authentication, user interactions, recommendation caching, and SageMaker endpoint invocations using a dedicated test user and a verified `movie_id` in the test environment.

## 1. Guest Workflow Verification

```bash
curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movie/<MOVIE_ID>"
```

Pass criteria:

- Requests do not require JWT authentication headers.
- Movie rankings are retrieved from `PopularMovies` and enriched with metadata from `Movies`.
- Guest requests generate zero interaction records in DynamoDB.

![Movie catalog displayed in the frontend](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-catalog.png)

*The frontend movie catalog rendered from data returned by the backend.*

![Simulated movie playback page](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-playback.png)

*The movie detail page with simulated playback using poster artwork.*

## 2. Registration and Onboarding Verification

1. Register a dedicated acceptance test user.
2. Store JWT tokens securely outside report documentation.
3. Select between one and three onboarding genres.
4. Re-query user state via the API.

Pass criteria: user state transitions cleanly from first login to returning user per API semantics.

## 3. User Interaction and Idempotency Verification

Submit a rating or reaction payload with an `Idempotency-Key` header, then resubmit the exact same header and body payload.

Pass criteria:

- The response retains identical `event_id` or `interaction_key` identifiers.
- DynamoDB contains exactly one corresponding item.
- Rating and reaction state can be queried and verified via read endpoints.

![Movie details and interaction controls](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-detail-interactions.png)

*The movie detail page displaying metadata and rating, reaction, and sharing controls.*

## 4. Personalized Recommendation Workflow Verification

```bash
curl \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/recommend/<CURRENT_USER_ID>"
```

### Cache Hit

The backend returns recommendations from `RecommendationCache` without calling the SageMaker endpoint.

### Cache Miss

The backend:

1. Constructs the model payload context.
2. Invokes SageMaker Endpoint.
3. Validates model response structures.
4. Enriches results with metadata from `Movies`.
5. Writes recommendations to cache on a best-effort basis.

Do not assert static movie rankings, as ranking order depends on model artifacts and interaction history.

## 5. SageMaker Endpoint Verification

Inside the `backend` directory:

```bash
python scripts/test_sagemaker_endpoint.py --describe

python scripts/test_sagemaker_endpoint.py \
  --invoke \
  --scenario onboarding_user \
  --genre "<GENRE>"
```

![SageMaker endpoint verification through AWS CLI](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/sagemaker-endpoint-cli.jpg)

*AWS CLI confirms that `movie-rec-endpoint` is in `InService` status.*

{{% notice warning %}}
Endpoint test paths pass only when the target environment has a compatible serving package. The current repository does not include code to build or deploy this endpoint container.
{{% /notice %}}

## 6. AWS Service Inspection Points

- **DynamoDB:** Verify interaction item presence.
- **RecommendationCache:** Confirm presence of `movie_id`, `score`, `reason_code`, model version, and expiration fields.
- **S3:** Interaction export objects appear only after running the exporter script; direct write APIs do not write directly to S3.
- **SageMaker:** Provider logs contain request ID, scenario, and result counts when endpoints are invoked.

## 7. Authentication Error Verification

```bash
curl -i \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: workshop-negative-0001" \
  -d '{"interaction_type":"click","interaction_action":"record","movie_id":"<MOVIE_ID>","interaction_value":1,"timestamp":"<ISO_8601_UTC>","session_id":"workshop-session"}' \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/users/me/interactions"
```

Expected result: HTTP `401 Unauthorized`.

## 8. Failure Path Matrix

| Scenario | Expected Result |
|---|---|
| Interaction request without JWT | `401` |
| Recommendation request before onboarding | `403` |
| User ID mismatch with JWT subject | `403` |
| Endpoint unavailable | `503` |
| Endpoint timeout | `504` |
| Invalid model response payload | `502` |
| Partial movie ID resolution failure | Skip unresolvable items |
| Complete movie ID resolution failure | `502` |
| Empty or schema-violating request body | `400` or `422` |

<!-- IMAGE-5.4.4-01: End-to-end test execution logs with JWT tokens and user details redacted. -->

{{% notice warning %}}
Acceptance tests record live data to `Users`, `UserInteractions`, and `RecommendationCache`. Use isolated test identities and execute cleanup steps only after approval.
{{% /notice %}}

## Completion Criteria

- [ ] Guest browsing succeeds.
- [ ] Protected endpoints without JWT headers return `401`.
- [ ] Idempotent retries succeed.
- [ ] Rating and reaction readbacks match submitted values.
- [ ] Logs distinguish between cache hits and cache misses.
- [ ] Endpoint tests are marked pass only when serving contracts exist.
- [ ] Do not claim frontend UI renders personalized recommendations; recommendation hooks are not currently wired into `Home`.

**Reference Sources:** FastAPI routes, `frontend/src/services/interactionService.ts`, `backend/app/services/recommendation_service.py`, and `backend/app/services/sagemaker_recommendation_provider.py`.
