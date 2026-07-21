---
title: "Hiện đại hóa ứng dụng với Strands Agents và Amazon Bedrock AgentCore"
date: 2026-07-10
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---


# TỰ ĐỘNG HÓA HIỆN ĐẠI HÓA ỨNG DỤNG VỚI STRANDS AGENTS, AWS TRANSFORM CUSTOM VÀ AMAZON BEDROCK AGENTCORE

![Agentic AI Architecture](/images/3-BlogsPosted/3.3-Blog3/blog3.png)

Chào mọi người, trong bài viết này, mình sẽ chia sẻ về một giải pháp được giới thiệu trên AWS DevOps & Developer Productivity Blog: hiện đại hóa ứng dụng ở quy mô lớn bằng Generative AI Agents, Strands Agents, AWS Transform Custom và Amazon Bedrock AgentCore.

Bài viết này phù hợp với những bạn đang quan tâm đến Generative AI, Agentic AI, DevOps, Cloud Migration và Application Modernization.

## 1. Vấn đề thực tế

Trong quá trình chuyển đổi lên cloud, doanh nghiệp thường cần hiện đại hóa rất nhiều ứng dụng cũ. Những công việc này có thể bao gồm:

- Nâng cấp phiên bản runtime của ứng dụng.
- Migration SDK cũ sang SDK mới.
- Refactor framework.
- Chuẩn hóa codebase giữa nhiều repository khác nhau.
- Kiểm tra dependency và xác định những phần cần thay đổi.

Nếu chỉ làm thủ công, mỗi repository phải được phân tích, chỉnh sửa, kiểm thử và triển khai riêng lẻ. Khi số lượng ứng dụng lên đến hàng trăm repository, quy trình này có thể kéo dài từ vài tháng đến vài năm và dễ dẫn đến sự không nhất quán giữa các team.

Vì vậy, bài toán đặt ra là làm sao có thể tự động hóa việc phân tích, tạo transformation và thực thi thay đổi trên nhiều repository một cách nhất quán, có kiểm soát và có khả năng mở rộng.

## 2. Tổng quan giải pháp và cách hoạt động

Giải pháp sử dụng kiến trúc agentic AI, trong đó nhiều AI agent phối hợp để tự động hóa quá trình hiện đại hóa ứng dụng.

Người dùng có thể tương tác với hệ thống thông qua giao diện React hoặc API. Từ đó, người dùng gửi vào một repository riêng lẻ hoặc một danh sách nhiều repository bằng file CSV. Request được xử lý bất đồng bộ qua API layer và chuyển đến orchestrator agent chạy trên Amazon Bedrock AgentCore.

Luồng hoạt động của giải pháp có thể tóm gọn như sau:

1. Người dùng gửi repository hoặc file CSV.
2. API tiếp nhận request.
3. Orchestrator Agent xử lý trên Amazon Bedrock AgentCore.
4. Agent phân tích codebase và xác định transformation phù hợp.
5. Nếu chưa có transformation thì tạo mới bằng AI.
6. Execution Agent chạy transformation bằng AWS Batch.
7. Kết quả được lưu lại và hiển thị trên giao diện cho người dùng theo dõi.

Chi tiết hơn, hệ thống bắt đầu bằng việc phân tích repository để xác định ngôn ngữ lập trình, dependency và nhu cầu nâng cấp như runtime version hoặc migration SDK. Nếu đã có transformation phù hợp, hệ thống sẽ sử dụng lại transformation đó. Nếu chưa có, creation agent sẽ tự động tạo transformation mới từ yêu cầu bằng ngôn ngữ tự nhiên và đưa vào registry để tái sử dụng cho các lần sau.

Sau khi transformation được xác định hoặc tạo mới, execution agent sẽ chạy quá trình chuyển đổi ở quy mô lớn bằng AWS Batch jobs. Mỗi job thực thi AWS Transform Custom CLI, giúp xử lý song song nhiều repository thay vì chạy tuần tự từng ứng dụng.

## 3. Các dịch vụ và công cụ chính

- AWS Transform Custom: dùng để tạo và chạy các transformation có thể tái sử dụng, phục vụ việc nâng cấp runtime, SDK hoặc framework.
- Strands Agents: framework dùng để xây dựng hệ thống multi-agent, giúp điều phối các workflow phức tạp.
- Amazon Bedrock AgentCore: cung cấp runtime được quản lý, memory và observability để vận hành AI agents đáng tin cậy.
- Amazon Bedrock: cung cấp nền tảng Generative AI để hỗ trợ agent phân tích và tạo transformation.
- AWS Batch: chạy các transformation job song song trên nhiều repository.
- Amazon S3: lưu trữ frontend, dữ liệu đầu vào hoặc kết quả xử lý.
- Amazon CloudFront: phân phối giao diện web cho người dùng.
- AWS Lambda: hỗ trợ xử lý bất đồng bộ và kết nối giữa API với AgentCore runtime.
- Amazon DynamoDB: lưu trạng thái job và thông tin xử lý.
- AWS CDK và AWS SAM: dùng để triển khai hạ tầng và agent runtime.

## 4. Điểm nổi bật của giải pháp

Điểm nổi bật nhất của giải pháp này là việc sử dụng nhiều AI agent để tự động hóa quá trình hiện đại hóa ứng dụng.

Thay vì developer phải tự đọc từng repository, tự xác định cần nâng cấp gì và tự viết logic chuyển đổi, hệ thống có thể tự động thực hiện các bước quan trọng như:

- Phân tích repository.
- Xác định ngôn ngữ, dependency và framework đang sử dụng.
- Xác định transformation phù hợp.
- Tạo transformation mới nếu chưa có.
- Thực thi transformation trên nhiều repository.
- Theo dõi trạng thái và kết quả xử lý.

Một điểm đáng chú ý khác là hệ thống có khả năng học và mở rộng theo thời gian. Khi creation agent tạo ra một transformation mới, transformation này sẽ được publish vào registry để tái sử dụng cho các repository khác trong tương lai.

## 5. Các flow chính trong ứng dụng

### Flow 1: Tạo custom transformation bằng ngôn ngữ tự nhiên

Người dùng mở tab Create Custom, nhập yêu cầu bằng tiếng Anh tự nhiên, ví dụ: "Upgrade Spring Boot 2 applications to Spring Boot 3". Sau đó, creation agent sẽ phân tích repository tham chiếu, tạo transformation definition và cho phép người dùng review trước khi publish vào registry.

### Flow 2: Chạy transformation cho nhiều repository bằng CSV

Người dùng upload file CSV chứa danh sách repository URL và target transformation. Sau khi submit, mỗi dòng trong CSV sẽ trở thành một AWS Batch job riêng biệt. Các job này có thể chạy song song, giúp xử lý nhiều repository nhanh hơn.

### Flow 3: Theo dõi trạng thái job

Người dùng có thể theo dõi tiến trình xử lý thông qua giao diện Jobs tab. Tại đây, hệ thống hiển thị trạng thái các job, kết quả transformation và output để người dùng kiểm tra.

## 6. Lợi ích của giải pháp

Giải pháp này mang lại nhiều lợi ích cho doanh nghiệp và đội ngũ kỹ thuật:

- Giảm công sức thủ công khi hiện đại hóa ứng dụng.
- Tự động phân tích repository và xác định nhu cầu chuyển đổi.
- Có thể tạo transformation mới bằng Generative AI.
- Tái sử dụng transformation cho nhiều repository khác nhau.
- Xử lý song song nhiều repository bằng AWS Batch.
- Tăng tính nhất quán giữa các team khi modernization.
- Hỗ trợ theo dõi trạng thái, quản lý workflow và kiểm soát kết quả.
- Có thể mở rộng cho nhiều bài toán phân tích code và tự động hóa khác.

## 7. Một số lưu ý khi triển khai

Khi triển khai giải pháp này trong thực tế, cần lưu ý một số điểm:

- Cần kiểm soát quyền IAM cho các thành phần như Lambda, AWS Batch, S3, DynamoDB và Bedrock AgentCore.
- Cần kiểm tra kỹ transformation trước khi áp dụng trên repository thật.
- Nên có bước review trước khi publish transformation vào registry.
- Cần theo dõi chi phí khi chạy nhiều AWS Batch jobs hoặc sử dụng model trên Amazon Bedrock.
- Cần bảo mật thông tin repository, đặc biệt nếu codebase chứa dữ liệu nhạy cảm.
- Nên tích hợp thêm kiểm thử tự động để xác nhận code sau transformation vẫn hoạt động đúng.

## 8. Hướng mở rộng

Kiến trúc này không chỉ dùng cho application modernization mà còn có thể mở rộng cho nhiều workflow khác như:

- Phân tích codebase ở quy mô lớn.
- Tự động nâng cấp framework.
- Migration SDK giữa các phiên bản.
- Chuẩn hóa coding standard giữa nhiều repository.
- Tích hợp vào CI/CD pipeline để modernization liên tục.
- Hỗ trợ cloud migration bằng cách tự động xử lý các thay đổi trong source code.

## 9. Lời kết

Qua bài viết này, mình thấy Generative AI không chỉ dùng để tạo chatbot hay hỗ trợ trả lời câu hỏi, mà còn có thể tham gia trực tiếp vào các workflow kỹ thuật phức tạp như hiện đại hóa ứng dụng.

Sự kết hợp giữa Strands Agents, AWS Transform Custom và Amazon Bedrock AgentCore giúp xây dựng một hệ thống agentic AI có khả năng phân tích repository, xác định thay đổi cần thiết, tạo transformation mới và thực thi chuyển đổi ở quy mô lớn.

Đây là một ví dụ thực tế về cách AWS đang ứng dụng Generative AI vào DevOps và Application Modernization, giúp doanh nghiệp giảm thời gian nâng cấp hệ thống, tăng tính nhất quán và đẩy nhanh quá trình chuyển đổi lên cloud.

### Nguồn tham khảo

- [AWS Blog - Use generative AI agents for application modernization at scale with Strands, Amazon Transform Custom, and Amazon Bedrock AgentCore](https://aws.amazon.com/vi/blogs/devops/use-generative-ai-agents-for-application-modernization-at-scale-with-strands-amazon-transform-custom-and-amazon-bedrock-agentcore/)
- [Tự động hóa hiện đại hóa ứng dụng với Strands Agents, AWS Transform Custom và Amazon Bedrock AgentCore - Facebook Discussion](https://www.facebook.com/groups/awsstudygroupfcj/posts/2210139623084331)
