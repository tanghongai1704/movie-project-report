---
title: "Xử lý, tải và xác minh dữ liệu"
date: 2026-07-30
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

Phần này chạy data pipeline cục bộ, kiểm tra tính quyết định của output, đồng bộ artifact lên S3 và xác định rõ khoảng trống nạp dữ liệu vào DynamoDB.

## 1. Chuẩn bị dữ liệu thô

Đặt các file CSV đúng tên trong thư mục raw được khai báo bởi `ml/configs/data_pipeline.yaml`. `ratings.csv` và `links.csv` là nguồn interaction lịch sử chính; các tập `small` chỉ phục vụ profiling.

Không commit dataset vào Git.

## 2. Chạy data pipeline

Tại thư mục `ml`:

```bash
python scripts/run_data_pipeline.py \
  --config configs/data_pipeline.yaml
```

Pipeline thực hiện tuần tự:

1. Profile dữ liệu.
2. Làm sạch dữ liệu.
3. Xây dựng feature.
4. Tạo split và serving export.
5. Chạy validation.

Critical validation failure phải trả exit code khác `0`.

## 3. Quy tắc xử lý dữ liệu

- MovieLens movie ID được ánh xạ sang TMDB ID bằng `links.csv`.
- Metadata trùng lặp giữ record đầy đủ hơn theo deterministic rule.
- Interaction alias trùng giữ timestamp mới nhất.
- Dữ liệu được chia theo từng user: interaction mới nhất vào test, interaction liền trước vào validation.
- JSONL output chỉ chứa các serving field nằm trong allowlist.

Ví dụ minh họa schema, không phải dữ liệu production:

```json
{
  "movie_id": "<MOVIE_ID>",
  "title": "<TITLE>",
  "genres": ["<GENRE>"],
  "poster_path": "<RELATIVE_TMDB_PATH>"
}
```

## 4. Kiểm tra tính quyết định và test

```bash
python scripts/check_determinism.py \
  --config configs/data_pipeline.yaml

python -m pytest -q
```

Kết quả mong đợi:

- Validation không có critical failure.
- Determinism check thành công.
- Các test ML pass trong môi trường dependency tương thích.

## 5. Đồng bộ lên S3

Luôn chạy dry-run trước:

```bash
python scripts/aws_sync.py push --dry-run
```

Sau khi review danh sách object và được phê duyệt:

```bash
python scripts/aws_sync.py push
```

![Các prefix dữ liệu trong S3 bucket](/images/5-Workshop/5.3-Data-layer/5.3.2-load-and-validate-data/s3-bucket-prefixes.png)

*Các vùng dữ liệu `datasets`, `evaluation`, `inference`, `logs`, `models` và `training` trong S3 bucket.*

## 6. Nạp dữ liệu vào DynamoDB

Pipeline tạo các serving JSONL nhưng repository chưa có loader chính thức cho `Movies` và `PopularMovies`.


## 7. Export interaction cho retraining

Exporter quét bảng `UserInteractions`, chuẩn hóa trạng thái reaction/rating và có thể ghi JSONL cục bộ hoặc tải lên interaction export prefix:

```bash
python scripts/export_interactions.py --upload
```

![Luồng phản hồi production từ frontend tới retraining](/images/5-Workshop/5.3-Data-layer/5.3.2-load-and-validate-data/production-feedback-flow.jpg)

*Interaction được ghi vào DynamoDB, export sang S3 rồi trở thành input cho lần retrain tiếp theo.*

