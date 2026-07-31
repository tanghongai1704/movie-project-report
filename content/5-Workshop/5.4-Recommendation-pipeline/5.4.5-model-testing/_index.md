---
title: "Recommendation Model Testing"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.5. </b> "
---

{{% notice info %}}
The team's movie website does not yet have a large amount of historical user interaction data such as `like`, `watch progress`, `share`, and so on. Therefore, recommendation testing based on user history is performed by creating simulated data for `user_id` values in the existing dataset. These `user_id` values already have a history of movie `rating` interactions.
{{% /notice %}}

## Features to Test

- Recommend five similar movies based on the movie entered by the user — **for all user types**.
- Recommend **Top-Rated** movies — **for unauthenticated users**.
- Recommend movies based on a user's historical movie interactions — **for authenticated users with interaction history**.
- Recommend movies to newly authenticated users by surveying their preferred genres — **for new users without interaction history**.

![Model test 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/test-model-1.png)

> This is a demonstration of the movie recommendation system test.

When signing in with a `user_id`, there are three user types:

- **Unauthenticated user — Guest:** Leave the field blank and do not enter a `user_id`.
- **Newly authenticated user without interaction history — New user:** Enter a `user_id` of 270897 or higher because historical movie rating data is already available for the preceding 270896 users.
- **Authenticated user with interaction history — Returning user:** Enter a `user_id` from 1 through 270896 because historical movie rating data is already available for those 270896 users.

## Guest

### Recommend Similar Movies Based on User Input

![Guest 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-1.png)

The user enters the name of a movie to search for, and the system displays a list of search results. The user then selects a specific movie from the list so the system can recommend five similar movies.

![Guest 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-2.png)

{{% notice note %}} 
Although this is a guest user, the `returning_user` scenario can still be used. The system selects the **content-based** model because the user is a guest; this model does not require historical user interaction data and relies only on the content and the searched movie title.
{{% /notice %}}

### Recommend **Top-Rated** Movies

![Guest 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-3.png)

The system uses **non-personalized recommendations** to recommend **Top-Rated** movies to unauthenticated users. It relies on precomputed movie rankings. Specifically, the **Top-Rated** list is built with an IMDb-style weighted ranking algorithm.

## New User

{{% notice note %}}
Newly authenticated users can also use [**Recommend similar movies based on user input**](#recommend-similar-movies-based-on-user-input), just like guest users.
{{% /notice %}}

### Recommend Movies by Preferred Genre

![New user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-1.png)

Newly authenticated users are asked to select their preferred genres. After the genres are selected, the system recommends movies that belong to those genres.

**Example:** select the `Music`, `Romance`, and `Family` genres.

![New user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-2.png)

{{% notice note %}} 
Because the user has selected preferred genres, the **recommend movies based on interaction history** feature returns results similar to **recommend movies by preferred genre**.
{{% /notice %}}

## Returning User

{{% notice note %}}
Authenticated users can also use [**Recommend similar movies based on user input**](#recommend-similar-movies-based-on-user-input), just like guest users.
{{% /notice %}}

### Recommend Movies Based on Interaction History

**Example:** `user_id = 1`. First, review this user's existing `rating` interaction history.

![Returning user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-1.png)

Then use **recommend movies based on interaction history** to recommend movies based on the `rating` history above.

![Returning user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-2.png)

The recommendation list includes popular genres such as `Action`, `Adventure`, `Fantasy`, and `Family`. The next test checks whether the list changes when the user interacts heavily with `Music` and `Romance` movies.

Load a simulated dataset containing this user's latest interactions. Specifically, the user performs the high-weight actions `rating: 5.0` and `watch: 1.0` on movies in the `Music` and `Romance` genres and also clicks **Harry Potter and the Half-Blood Prince** (in the `Adventure` and `Fantasy` genres), which the user watched previously.

![Returning user 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-3.png)

The movie list returned by the model is shown below:

![Returning user 4](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-4.png)

- **Preserving long-term preferences:** Positions 1–4 are all occupied by **Harry Potter** movies. The system recognizes the relationship between the current short-term click and the historical five-star ratings, moving movies from the same series to the top. This shows that the system does not forget established preferences when a user develops new interests.

- **Balancing short-term interactions:** In the following positions, the model begins to include movies with `Music` and `Romance` characteristics. The system successfully expands the recommendation space and responds immediately to the user's emerging needs without waiting for a complete model retraining run.

## Model Evaluation

To measure the real-world performance of the recommendation algorithms, quantitative testing is performed on the `test` dataset. One movie that each user genuinely likes is hidden, and the predicted recommendation list is checked to determine whether it contains that movie.

### 1. Test Configuration

- **Evaluation set:** a sample of 5,000 users.
- **Preference threshold:** the hidden movie must have `rating >= 4.0`.
- **Recommendation list size:** Top 20 movies.
- **Baseline:** the `popularity_train` model, which recommends the most popular movies in the entire system. The baseline is rebuilt entirely from the training set to prevent data leakage.

### 2. Evaluation Results

The table below summarizes the performance of the four implemented model flows:

| Model | HitRate@10 | NDCG@10 | Unique movies | Coverage |
| :--- | :---: | :---: | :---: | :---: |
| **Popularity — Baseline** | 0.0332 | 0.0201 | 128 | 0.28% |
| **Collaborative — ALS** | 0.1115 | 0.0537 | 2,411 | 5.31% |
| **Content-based** | 0.0051 | 0.0035 | 17,530 | 38.59% |
| **Hybrid — RRF** | 0.0818 | 0.0393 | 8,110 | 17.85% |

### 3. Analysis and Evaluation

The measured values support several important conclusions about the characteristics of each algorithm:

- **Popularity model:** It achieves only 0.0332 HitRate@10 and extremely limited coverage (only 0.28%, with 128 recommended movies). This is because the model applies the same **Top-Rated** list to every user, providing no personalization.
- **Collaborative Filtering model:** This is the system's most accurate prediction algorithm. ALS significantly outperforms the baseline with a HitRate@10 of 0.1115 (**+235.8%**) and the highest NDCG@10 (0.0537). However, the method's critical weakness also appears: 57 users encounter a **cold-start** state, so the system cannot generate recommendations for them.
- **Content-based model:** This method uses each user's three earliest highly rated movies as anchors to find similar movies. Although it has the lowest accuracy (84.8% below the baseline), it provides a very large discovery space, with **38.59%** coverage (17,530 unique movies).
- **Hybrid model — weighted RRF combination:** This is the system's best overall result and demonstrates a balanced combination of algorithms. The hybrid model retains high accuracy, reaching a HitRate@10 of 0.0818 (**+146.4%** above the baseline). Notably, the combined algorithm successfully handles all 57 users affected by the ALS cold-start problem through the global fallback layer, while increasing coverage to a safer, more diverse catalog of 8,110 movies (17.85%).

**Conclusion:** The hybrid model meets the design objective by maintaining high accuracy from historical data, preserving content diversity, and fully addressing the absence of historical user data.
