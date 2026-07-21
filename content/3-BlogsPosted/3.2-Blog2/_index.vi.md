---
title: "Tự động hóa Security Scan trên Amazon EKS"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---


### TỰ ĐỘNG HÓA SECURITY SCAN TRÊN AMAZON EKS VỚI KUBESCAPE, AWS CODEBUILD VÀ AWS CODEPIPELINE

![Security Scan Architecture](/images/3-BlogsPosted/3.2-Blog2/blog2.png)

Khi triển khai ứng dụng trên Amazon EKS, việc đảm bảo bảo mật và tuân thủ các tiêu chuẩn (Compliance) là một yêu cầu rất quan trọng. Tuy nhiên, nếu thực hiện kiểm tra thủ công sẽ mất nhiều thời gian, dễ bỏ sót các lỗi cấu hình và khó mở rộng khi hệ thống ngày càng lớn.

Giải pháp sử dụng **Kubescape**, **AWS CodeBuild** và **AWS CodePipeline** giúp tự động hóa toàn bộ quá trình quét bảo mật trong quy trình CI/CD, giúp phát hiện sớm các rủi ro trước khi ứng dụng được triển khai lên môi trường production.

---

### Những thách thức trong việc kiểm tra bảo mật

Một số vấn đề thường gặp khi vận hành Kubernetes trên Amazon EKS:

* Cluster Kubernetes có thể bị cấu hình sai dẫn đến các lỗ hổng bảo mật.
* Workload có thể được cấp quyền quá cao, không tuân theo nguyên tắc Least Privilege.
* Việc kiểm tra thủ công theo các tiêu chuẩn như CIS Benchmark, NSA hoặc MITRE tốn nhiều thời gian.
* Security Scan chưa được tích hợp vào quy trình CI/CD.
* Khó theo dõi và quản lý kết quả sau khi phát hiện lỗi bảo mật.

---

### Tổng quan kiến trúc giải pháp

Giải pháp sử dụng Kubescape để quét bảo mật cho Amazon EKS và kết hợp với AWS CodeBuild cùng AWS CodePipeline để tự động hóa toàn bộ quy trình.

Luồng hoạt động gồm các bước:

* Developer hoặc End-user kích hoạt Pipeline.
* AWS CodePipeline bắt đầu quy trình CI/CD.
* AWS CodeBuild thực hiện Build Job.
* CodeBuild cài đặt và chạy Kubescape.
* Kubescape quét Amazon EKS theo các framework như CIS Benchmark, NSA và MITRE.
* Kết quả quét được lưu trên Amazon S3.
* Đội ngũ DevOps hoặc Security phân tích kết quả và xử lý các vấn đề bảo mật.

---

### Các dịch vụ AWS được sử dụng

* **Amazon EKS:** Triển khai Kubernetes Cluster và các ứng dụng Container.
* **Kubescape:** Công cụ mã nguồn mở dùng để quét Security và Compliance.
* **AWS CodePipeline:** Tự động hóa quy trình CI/CD.
* **AWS CodeBuild:** Thực hiện Build Job và chạy Kubescape Scan.
* **Amazon S3:** Lưu trữ báo cáo kết quả Security Scan.
* **IAM:** Cấp quyền truy cập giữa các dịch vụ AWS.
* **Amazon CloudWatch Logs:** Theo dõi log của toàn bộ quá trình Build và Scan.

---

### Các chế độ quét của Kubescape

Kubescape hỗ trợ nhiều framework bảo mật phổ biến giúp đánh giá mức độ an toàn của Kubernetes Cluster:

* **CIS Benchmark Scan:** Kiểm tra cấu hình Kubernetes theo tiêu chuẩn CIS.
* **NSA Framework Scan:** Đánh giá khả năng Hardening của hệ thống.
* **MITRE Framework Scan:** Phát hiện các điểm yếu liên quan đến các kỹ thuật tấn công phổ biến.
* **Namespace Scan:** Chỉ quét một Namespace cụ thể thay vì toàn bộ Cluster.

---

### Lợi ích của giải pháp

Việc tích hợp Kubescape vào quy trình CI/CD mang lại nhiều lợi ích:

* Tự động hóa quá trình kiểm tra bảo mật trên Amazon EKS.
* Giảm thao tác thủ công và hạn chế bỏ sót lỗi.
* Phát hiện sớm các vấn đề bảo mật trước khi triển khai Production.
* Kiểm tra Compliance theo CIS Benchmark, NSA và MITRE.
* Lưu trữ kết quả tập trung trên Amazon S3 để dễ theo dõi.
* Có thể chặn Deployment nếu phát hiện lỗi nghiêm trọng.
* Hỗ trợ tích hợp với các hệ thống quản lý công việc như Jira để theo dõi quá trình xử lý.

---

### Một số Best Practices

Để tăng cường bảo mật cho Amazon EKS, nên kết hợp thêm các giải pháp sau:

* Áp dụng RBAC để giới hạn quyền truy cập.
* Sử dụng IAM Roles for Service Accounts (IRSA) theo nguyên tắc Least Privilege.
* Quản lý thông tin nhạy cảm bằng AWS Secrets Manager.
* Kích hoạt Amazon GuardDuty để phát hiện các mối đe dọa.
* Áp dụng Pod Security Standards nhằm kiểm soát Workload.
* Sử dụng Kyverno để kiểm tra và thực thi Security Policy trên Kubernetes.

---

### Kết luận

Việc tích hợp Security Scan trực tiếp vào CI/CD giúp phát hiện các lỗ hổng ngay từ giai đoạn phát triển thay vì chờ đến khi triển khai Production.

Sự kết hợp giữa **Kubescape**, **AWS CodeBuild** và **AWS CodePipeline** giúp xây dựng quy trình DevSecOps tự động, hỗ trợ kiểm tra Security và Compliance liên tục cho Amazon EKS. Đây là giải pháp phù hợp cho các hệ thống Cloud-Native hiện đại, giúp nâng cao mức độ an toàn, giảm rủi ro và tối ưu quá trình vận hành.

---

### Nguồn tham khảo

* AWS Blog - Automating Security Scans for Amazon EKS using Kubescape
* Kubescape Documentation: https://kubescape.io/
* AWS CodePipeline Documentation: https://docs.aws.amazon.com/codepipeline/
* AWS CodeBuild Documentation: https://docs.aws.amazon.com/codebuild/
* Amazon EKS Documentation: https://docs.aws.amazon.com/eks/
* [Tự động hóa Security Scan trên Amazon EKS - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2205463036885323)
