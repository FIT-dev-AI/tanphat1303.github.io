---
title: "Xây dựng Modern Data Lakehouse (Serverless) trên AWS"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


### XỬ LÝ BIG DATA VÀ TỐI ƯU 80% CHI PHÍ TRUY VẤN VỚI S3, GLUE VÀ ATHENA

![Modern Data Lakehouse Architecture](/images/3-BlogsPosted/3.1-Blog1/blog1.png)

Khi làm việc với Big Data (Log hệ thống, lịch sử giao dịch, user clickstream...), thách thức lớn nhất không chỉ là lưu trữ mà là làm sao để truy vấn nhanh với chi phí tối ưu nhất. Nếu tiếp tục duy trì các cụm Relational Database lớn hoặc các Data Warehouse truyền thống, chi phí hạ tầng sẽ tỷ lệ thuận với độ phình của dữ liệu.

Kiến trúc Modern Data Lakehouse hoàn toàn Serverless sử dụng bộ ba: **Amazon S3**, **AWS Glue** và **Amazon Athena** giúp tách biệt hoàn toàn giữa Storage (Lưu trữ) và Compute (Tính toán), mang lại khả năng mở rộng vô hạn với chi phí tối ưu.

---

### Sơ đồ luồng dữ liệu (Data Pipeline Architecture)

Kiến trúc Data Lakehouse của chúng ta sẽ được chia thành các phân vùng (Zones) chuẩn trên S3 để quản lý dữ liệu một cách khoa học:

* **Raw Zone (S3):** Nơi tiếp nhận dữ liệu thô đầu vào (CSV, JSON, Logs...) được đẩy về liên tục từ các nguồn.
* **Cataloging (AWS Glue Crawler):** Quét qua Raw Zone để tự động trích xuất Schema, tạo lập định nghĩa bảng (Metadata) và lưu vào AWS Glue Data Catalog.
* **Processing (AWS Glue Job - PySpark):** Chuyển đổi định dạng dữ liệu (ETL) từ dữ liệu thô sang định dạng cột Apache Parquet, đồng thời thực hiện Partitioning (Phân vùng theo Ngày/Tháng/Năm) rồi ghi xuống Processed Zone (S3).
* **Analytics (Amazon Athena):** Sử dụng câu lệnh SQL tiêu chuẩn để truy vấn trực tiếp trên Processed Zone thông qua Metadata từ Data Catalog mà không cần load dữ liệu vào bất kỳ database nào.

---

### Tại sao kiến trúc này tối ưu được chi phí?

Amazon Athena tính phí dựa trên dung lượng dữ liệu bị quét (Data Scanned) khi thực hiện lệnh SQL ($5/TB$). Nếu bạn chạy `SELECT count(*)` trên 100GB file CSV thô, Athena sẽ quét toàn bộ 100GB dữ liệu đó.

Để giải quyết bài toán tối ưu chi phí này, 2 kỹ thuật cốt lõi trong Glue Job bắt buộc phải áp dụng:

1. **Chuyển đổi sang Columnar Storage (Parquet):** Parquet lưu trữ dữ liệu theo dạng cột. Khi bạn chạy câu lệnh chỉ lấy `user_id` và `total_amount`, Athena chỉ quét đúng dữ liệu của 2 cột đó và bỏ qua toàn bộ các cột còn lại. Lượng dữ liệu cần quét giảm ngay 60% - 80%.
2. **Data Partitioning (Phân vùng dữ liệu):** Dữ liệu trên S3 sẽ được cấu trúc theo dạng: `s3://processed-zone/year=2026/month=07/day=03/`. Khi bạn viết câu lệnh SQL có điều kiện `WHERE year = '2026' AND month = '07'`, Athena sẽ nhảy thẳng vào folder của tháng đó và bỏ qua toàn bộ dữ liệu của các năm/tháng khác.

---

### Các bước triển khai thực nghiệm (Hands-on)

* **Bước 1 (Storage):** Khởi tạo 2 bucket S3 đại diện cho `raw-zone` và `processed-zone`. Upload file dữ liệu mẫu (ví dụ: `sales_data.csv`).
* **Bước 2 (Metadata):** Cấu hình một AWS Glue Crawler trỏ vào `raw-zone`. Sau khi chạy, Crawler sẽ tự động tạo ra một table trong Glue Data Catalog với đầy đủ các cột và kiểu dữ liệu (String, Int, Datetime...) mà không cần cấu hình thủ công.
* **Bước 3 (ETL với PySpark):** Tạo một Glue Job (có thể dùng giao diện kéo thả Glue Studio hoặc viết code PySpark). Nhiệm vụ của Job là đọc dữ liệu từ bảng Raw, convert định dạng sang `.parquet`, cấu hình partition theo cột thời gian và ghi xuống `processed-zone`.
* **Bước 4 (Query):** Mở giao diện Amazon Athena, chọn Database từ Glue Catalog và chạy thử truy vấn bằng câu lệnh SQL tiêu chuẩn.

### Kết luận

Kiến trúc Modern Data Lakehouse Serverless này mang lại 3 lợi ích lớn: Không cần quản lý server (Zero Management), Tự động mở rộng (Auto-scaling) và Tiết kiệm chi phí tối đa. Đây là nền tảng cực kỳ vững chắc để tích hợp thêm các công cụ BI (Amazon QuickSight) hoặc làm nguồn cấp dữ liệu cho các mô hình Machine Learning (Amazon SageMaker).

---

### Nguồn tham khảo

* [AWS Lakehouse Architecture Guide](https://vntechies.dev/.../aws-lakehouse-architecture-guide)
* [Modern Data Lake House on AWS](https://www.linkedin.com/.../modern-data-lake-house.../)
* [Xây dựng Modern Data Lakehouse (Serverless) trên AWS - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2203105570454403)
