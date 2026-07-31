---
title: "Recommendation Model Testing"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.5. </b> "
---

{{% notice info %}}
Our movie streaming website does not yet have much historical user interaction data such as `like`, `watch progress`, `share`, etc. Therefore, testing the recommendation model based on historical user data will be conducted by generating mock data for `user_id`s in the available dataset. These `user_id`s already have a history of `rating` interactions with movies.
{{% /notice %}}

## Features to Test
- Recommend 5 similar movies based on the movie entered by the user - **For all user types**.
- Recommend **Top-Rated** movies - **For unauthenticated/guest users**.
- Recommend movies based on the user's historical interactions with movies - **For authenticated users with interaction history**.
- Recommend movies for newly registered users by surveying their favorite genres - **For newly authenticated users without interaction history**.

![test model 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/test-model-1.png)

> This is a demo testing the movie recommendation system

When logging in with a `user_id`, there are 3 types of users:

- **Unauthenticated user - Guest:** Leave blank, do not enter a `user_id`.  
- **Newly authenticated user without interaction history - New user:** Enter a `user_id` from id 270897 onwards. This is because there is existing historical rating data for the previous 270896 users.
- **Authenticated user with interaction history - Returning user:** Enter a `user_id` from id 1 to 270896. This is because there is existing historical rating data for these 270896 users. 

## Guest
### Recommend similar movies based on user input

![guest 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-1.png)

The user enters the name of the movie they want to search for, and the system displays a list of search results. The user then selects a specific movie from the list for the system to recommend 5 similar movies.

![guest 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-2.png)

{{% notice note %}} 
Although this is a guest user, they can still utilize the `returning_user` scenario. Because the system has selected the **Content-based** model (since the user is a guest), it does not require historical user interaction data and relies solely on the content and title of the searched movie.
{{% /notice %}}

### Recommend **Top-Rated** movies

![guest 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/guest-3.png)

The recommendation model relies on **Non-personalized recommendations** to suggest **Top-Rated** movies to unauthenticated users. The system uses precomputed movie rankings. Specifically, the **Top-Rated** list is built based on an IMDb-style weighted rating algorithm.

## New user

{{% notice note %}}
For newly authenticated users, there is also the feature to [**Recommend similar movies based on user input**](#recommend-similar-movies-based-on-user-input) just like guest users.
{{% /notice %}}

### Recommend movies by favorite genres

![new user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-1.png)

Newly authenticated users will be surveyed for their favorite genres. After selecting genres, the system will recommend movies belonging to those categories.

**Example:** selecting the genres `Music`, `Romance`, `Family`

![new user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/new-user-2.png)

{{% notice note %}} 
Since the user has selected their favorite genres, the **Recommend movies based on historical interactions** feature will yield results similar to **Recommend movies by favorite genres**. 
{{% /notice %}}

## Returning user

{{% notice note %}}
For authenticated users, there is also the feature to [**Recommend similar movies based on user input**](#recommend-similar-movies-based-on-user-input) just like guest users.
{{% /notice %}}

### Recommend movies based on historical interactions

**Example:** `user_id = 1`. First, let's review the available `rating` interaction history for this user.

![return user 1](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-1.png)

Next, use the **Recommend movies based on historical interactions** feature to suggest movies based on the aforementioned `rating` history.

![return user 2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-2.png)

The recommended movie list contains many popular genres such as `Action`, `Adventure`, `Fantasy`, `Family`. Let's check if the list changes when the user interacts heavily with movies in the `Music` and `Romance` genres.

We proceed to load a mock dataset containing the latest interactions of this user. Specifically, the user performed high-weight actions (`rating: 5.0`, `watch: 1.0`) on movies in the `Music` and `Romance` genres, and simultaneously made one `click` on the movie **Harry Potter and the Half-Blood Prince** (belonging to the `Adventure`, `Fantasy` genres) which they had watched in the past.

![return user 3](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-3.png)

The movie list after the model generates recommendations:

![return user 4](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.5-model-testing/return-user-4.png)

- **Preserving long-term preferences:** The Top 1-4 positions all belong to the **Harry Potter** series. The system identifies the connection between the current short-term click and the 5-star rating history from the past, thereby pushing movies from the same series to the top. This proves the system does not lose its memory when the user develops new interests.

- **Compromising with short-term interactions:** In the subsequent positions, the model begins to insert movies featuring `Music` and `Romance` characteristics. The system successfully expands the recommendation space, instantly meeting the user's newly arisen needs without waiting to retrain the entire model.

## Model Evaluation

To measure the actual performance of the recommendation algorithms, the quantitative testing process is conducted on the `test` dataset. One movie that each user genuinely likes is hidden, and then compared to see if the model's predicted list contains that specific movie.

### 1. Testing Configuration

- **Evaluation set:** Sample of 5,000 users.
- **Relevance threshold:** The hidden movie must have a rating `>= 4.0`.
- **Recommendation list size:** Top 20 movies.
- **Baseline:** Uses the `popularity_train` model (recommending the most popular movies system-wide). The baseline is reconstructed entirely from the training set to prevent data leakage.

{{% notice note %}}
Due to the specific nature where each user has exactly one hidden item, the `Recall@K` and `Precision@K` metrics do not carry independent information. Therefore, the evaluation will focus on two core metrics: **Hit Rate** and **NDCG - Ranking Quality**.
{{% /notice %}}

### 2. Evaluation Results

The table below summarizes the performance of the 4 deployed model pipelines:

| Model | HitRate@10 | NDCG@10 | Distinct Movies (Diversity) | Coverage |
| :--- | :---: | :---: | :---: | :---: |
| **Popularity - Baseline** | 0.0332 | 0.0201 | 128 | 0.28% |
| **Collaborative - ALS** | 0.1115 | 0.0537 | 2,411 | 5.31% |
| **Content-based**| 0.0051 | 0.0035 | 17,530 | 38.59% |
| **Hybrid - RRF** | 0.0818 | 0.0393 | 8,110 | 17.85% |

### 3. Analysis and Evaluation

Based on the obtained metrics, we can draw important assessments regarding the characteristics of each algorithm:

- **Popularity Model:** Only achieves a HitRate@10 of 0.0332 and an extremely narrow coverage (only 0.28% with 128 movies recommended). The reason is that this model applies the same **Top-Rated** movie list universally to all users, resulting in zero personalization.
- **Collaborative Filtering Model:** Serves as the most accurate prediction algorithm in the system. ALS completely outperforms the Baseline, achieving a HitRate@10 of 0.1115 (an increase of **+235.8%**) and the highest NDCG@10 (0.0537). However, the fatal flaw of this method became apparent: 57 users fell into a **Cold-start** state, meaning the system could not generate any recommendations for them.
- **Content-based Model:** This method uses the user's 3 earliest highly-rated movies as anchors to find similar movies. Despite having the lowest accuracy (84.8% worse than the Baseline), it brings a massive discovery space with a coverage of up to **38.59%** (17,530 distinct movies).
- **Hybrid Model - Combined via Weighted RRF:** This is the most optimal achievement of the system, demonstrating a harmonious combination between the algorithms. The hybrid model maintains a very high accuracy, with a HitRate@10 reaching 0.0818 (exceeding the Baseline by **+146.4%**). Notably, the combined algorithm successfully handled all 57 users suffering from the ALS network's "Cold-start" error through a global Fallback layer, while simultaneously boosting the system's coverage to a safe level and diversifying the catalog to 8,110 movies (17.85%).

**Conclusion:** The Hybrid model has excellently fulfilled its design objectives: maintaining high accuracy from historical data, ensuring content diversity, and completely resolving the issue of lacking historical data.