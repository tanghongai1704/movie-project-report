---
title: "End-to-End Testing"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

This section tests the guest, authentication, interaction, cache, and endpoint flows separately. Use a dedicated test user and a real `movie_id` from the test environment.

## 1. Test the Guest Path

```bash
curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movie/<MOVIE_ID>"
```

Pass criteria:

- The request does not require a JWT.
- The list is read from `PopularMovies` and enriched with metadata from `Movies`.
- A guest request does not create an interaction.

![Movie catalog displayed in the user interface](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-catalog.png)

*The frontend displays the movie catalog from data returned by the backend.*

![Simulated movie playback page](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-playback.png)

*Movie detail page with a playback area simulated by poster artwork.*

## 2. Test Registration and Onboarding

1. Register a dedicated acceptance-test user.
2. Store the JWT temporarily outside the report.
3. Select between one and three onboarding genres.
4. Read the user state again.

Pass criterion: the state changes from first login to returning user according to the API semantics.

## 3. Test Interaction and Idempotency

Submit a rating or reaction with an `Idempotency-Key`, and then resubmit the same header and body.

Pass criteria:

- The response retains the same `event_id` or `interaction_key`.
- DynamoDB contains only one corresponding item.
- The rating/reaction state can be read back.

![Movie information and interaction controls](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-detail-interactions.png)

*The movie detail page displays metadata together with rating, reaction, and sharing actions.*

## 4. Test the Personalized Path

```bash
curl \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/recommend/<CURRENT_USER_ID>"
```

### Cache Hit

The backend returns the result from `RecommendationCache` without invoking the SageMaker endpoint.

### Cache Miss

The backend:

1. Builds the model request.
2. Invokes the SageMaker Endpoint.
3. Validates the response.
4. Enriches the result with metadata from `Movies`.
5. Writes the cache entry on a best-effort basis.

Do not assert a specific movie because ranking depends on the artifacts and interaction history.

## 5. Test the SageMaker Endpoint

From the `backend` directory:

```bash
python scripts/test_sagemaker_endpoint.py --describe

python scripts/test_sagemaker_endpoint.py \
  --invoke \
  --scenario onboarding_user \
  --genre "<GENRE>"
```

![SageMaker Endpoint test result from the AWS CLI](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/sagemaker-endpoint-cli.jpg)

*The AWS CLI confirms that the `movie-rec-endpoint` endpoint is in the `InService` state.*

## 6. AWS Verification Points

- **DynamoDB:** the interaction item exists.
- **RecommendationCache:** contains `movie_id`, `score`, `reason_code`, model version, and expiry.
- **S3:** an interaction export appears only after the exporter runs; an API write does not write directly to S3.
- **SageMaker:** the provider log contains the request ID, scenario, and result count when the endpoint is invoked.

## 7. Test Authentication Failure

```bash
curl -i \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: workshop-negative-0001" \
  -d '{"interaction_type":"click","interaction_action":"record","movie_id":"<MOVIE_ID>","interaction_value":1,"timestamp":"<ISO_8601_UTC>","session_id":"workshop-session"}' \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/users/me/interactions"
```

Expected result: HTTP `401`.

## 8. Failure-Path Matrix

| Scenario | Expected result |
|---|---|
| Interaction without a JWT | `401` |
| Recommendation requested before onboarding | `403` |
| User ID differs from the JWT subject | `403` |
| Endpoint unavailable | `503` |
| Endpoint timeout | `504` |
| Invalid model response | `502` |
| Some movie IDs cannot be resolved | Skip the failed items |
| No movie IDs can be resolved | `502` |
| Empty request or invalid schema | `400` or `422` |
