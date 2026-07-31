---
title: "Kiểm thử đầu cuối"
date: 2026-07-30
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

Phần này kiểm tra riêng từng luồng guest, authentication, interaction, cache và endpoint. Sử dụng một test user riêng và một `movie_id` có thật trong môi trường thử nghiệm.

## 1. Kiểm tra guest path

```bash
curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movie/<MOVIE_ID>"
```

Tiêu chí đạt:

- Request không cần JWT.
- Danh sách được lấy từ `PopularMovies` và bổ sung metadata từ `Movies`.
- Guest request không tạo interaction.

![Danh mục phim được hiển thị trên giao diện](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-catalog.png)

*Danh mục phim được frontend hiển thị từ dữ liệu trả về bởi backend.*

![Trang phát phim mô phỏng](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-playback.png)

*Trang movie detail với khu vực phát phim mô phỏng bằng poster artwork.*

## 2. Kiểm tra đăng ký và onboarding

1. Đăng ký một acceptance test user riêng.
2. Lưu JWT tạm thời ngoài report.
3. Chọn từ một đến ba thể loại onboarding.
4. Đọc lại user state.

Tiêu chí đạt: trạng thái chuyển từ first login sang returning user theo API semantics.

## 3. Kiểm tra interaction và idempotency

Gửi một rating hoặc reaction với `Idempotency-Key`, sau đó gửi lại cùng header và body.

Tiêu chí đạt:

- Response giữ cùng `event_id` hoặc `interaction_key`.
- DynamoDB chỉ có một item tương ứng.
- Rating/reaction state có thể đọc lại.

![Thông tin phim và các điều khiển tương tác](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/ui-movie-detail-interactions.png)

*Trang chi tiết phim hiển thị metadata cùng các thao tác rating, reaction và share.*

## 4. Kiểm tra personalized path

```bash
curl \
  -H "Authorization: Bearer <ACCESS_TOKEN>" \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/recommend/<CURRENT_USER_ID>"
```

### Cache hit

Backend trả kết quả từ `RecommendationCache` mà không gọi SageMaker endpoint.

### Cache miss

Backend:

1. Dựng model request.
2. Gọi SageMaker Endpoint.
3. Kiểm tra response.
4. Bổ sung metadata từ `Movies`.
5. Ghi cache theo cơ chế best effort.

Không assert một movie cụ thể vì ranking phụ thuộc artifact và lịch sử tương tác.

## 5. Kiểm tra SageMaker endpoint

Tại thư mục `backend`:

```bash
python scripts/test_sagemaker_endpoint.py --describe

python scripts/test_sagemaker_endpoint.py \
  --invoke \
  --scenario onboarding_user \
  --genre "<GENRE>"
```

![Kết quả kiểm tra SageMaker endpoint bằng AWS CLI](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.4-end-to-end-testing/sagemaker-endpoint-cli.jpg)

*AWS CLI xác nhận endpoint `movie-rec-endpoint` đang ở trạng thái `InService`.*


## 6. Các điểm kiểm tra trên AWS

- **DynamoDB:** interaction item tồn tại.
- **RecommendationCache:** có `movie_id`, `score`, `reason_code`, model version và expiry.
- **S3:** interaction export chỉ xuất hiện sau khi exporter chạy; API write không tự ghi S3.
- **SageMaker:** provider log có request ID, scenario và result count khi endpoint được gọi.

## 7. Kiểm tra lỗi xác thực

```bash
curl -i \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: workshop-negative-0001" \
  -d '{"interaction_type":"click","interaction_action":"record","movie_id":"<MOVIE_ID>","interaction_value":1,"timestamp":"<ISO_8601_UTC>","session_id":"workshop-session"}' \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/users/me/interactions"
```

Kết quả mong đợi: HTTP `401`.

## 8. Ma trận failure path

| Trường hợp | Kết quả mong đợi |
|---|---|
| Interaction không có JWT | `401` |
| Chưa onboarding nhưng gọi recommendation | `403` |
| User ID khác JWT subject | `403` |
| Endpoint không khả dụng | `503` |
| Endpoint timeout | `504` |
| Model response không hợp lệ | `502` |
| Một số movie ID không resolve được | Bỏ qua item lỗi |
| Toàn bộ movie ID không resolve được | `502` |
| Request rỗng hoặc sai schema | `400` hoặc `422` |

