---
title: "Event 1"
date: 2026-05-23
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Bài thu hoạch: “AWS Vietnam Community Day 2026”

### Mục Đích Của Sự Kiện

- Chia sẻ kiến thức thực tế về Cloud Computing, Generative AI và kiến trúc phần mềm hiện đại.
- Giới thiệu các mô hình Enterprise AI và Multi-Agent Architecture.
- Trình bày cách các dịch vụ AWS hỗ trợ bảo mật, mở rộng hệ thống, tự động hóa và tối ưu hiệu năng.
- Chia sẻ kinh nghiệm thực tế từ các dự án AI, hackathon và hệ thống doanh nghiệp.
- Giúp sinh viên và developer hiểu rõ hơn về xu hướng Cloud và AI hiện nay.

### Danh Sách Diễn Giả

- **Tinh Truong** – Platform Engineer, GoTymeX  
  Chủ đề: *Context Is Everything*

- **Pham Ng Hai Anh** – AWS Community Builder, G-AsiaPacific Vietnam  
  Chủ đề: *Friendly AI Assistant with Amazon Quick*

- **Nguyen Tuan Thinh** – DevOps Engineer, First Cloud AI Journey  
  Chủ đề: *From Edge to Origin: CloudFront as Your Foundation*

- **Team VIB**  
  Chủ đề: *36 Hours with LotusHacks – Building UTMorpho from Idea to Reality*

- **Duc Dao** – Solution Architect, Cloud Kinetics  
  Chủ đề: *Deep Dive Talk: How LLM Actually Works?*

- **Vy Lam** – Senior Business Systems Analyst, VPBank  
  Chủ đề: *Enterprise-Grade Multi-Agent System: The Case of Startup Credit Scoring*

### Nội Dung Nổi Bật

#### Context Engineering và GenAIOps

Một trong những nội dung quan trọng của sự kiện là vai trò của context khi làm việc với Generative AI. Diễn giả giải thích rằng nhiều câu trả lời chưa tốt của AI không phải do mô hình yếu, mà do người dùng cung cấp context chưa đầy đủ hoặc chưa rõ ràng.

Các ý chính bao gồm:

- Mục tiêu rõ ràng giúp AI hiểu nhiệm vụ tốt hơn.
- Context tốt sẽ tạo ra kết quả tốt hơn.
- Nhiều context không đồng nghĩa với context chất lượng.
- Hệ thống AI đang phát triển từ prompt đơn giản sang context và memory.
- Mô hình “Second AI Brain” có thể lưu trữ, truy xuất, tạo câu trả lời và học từ thông tin hữu ích.

#### Friendly AI Assistant với Amazon Quick

Session này giới thiệu cách Amazon Quick Suite hỗ trợ business user tự động hóa các công việc lặp lại và phân tích thông tin từ nhiều nguồn dữ liệu khác nhau.

Các ý chính bao gồm:

- AI assistant có thể hỗ trợ tổng hợp thông tin.
- Automation flows giúp giảm bớt các thao tác thủ công.
- Người dùng doanh nghiệp có thể dùng AI để tạo meeting notes, gửi email và lên lịch họp.
- Amazon Quick Suite kết hợp data connectors, Bedrock models, web search, actions, governance và guardrails.

#### CloudFront as a Foundation

Session về Amazon CloudFront giúp em hiểu rằng CloudFront không chỉ là một dịch vụ CDN, mà còn là nền tảng hỗ trợ bảo mật, hiệu năng, độ tin cậy và tối ưu chi phí.

Các nội dung chính bao gồm:

- Edge caching và request collapsing.
- AWS WAF và DDoS protection.
- HTTP/3 và QUIC giúp tăng hiệu năng.
- Signed URLs và mTLS hỗ trợ bảo mật.
- Origin failover và stale cache giúp tăng độ tin cậy.
- Flat-rate pricing giúp giảm rủi ro bill tăng đột biến.

#### Hành Trình Hackathon UTMorpho

Team VIB chia sẻ kinh nghiệm xây dựng UTMorpho trong vòng 36 giờ tại LotusHacks. UTMorpho là một công cụ AI hỗ trợ tạo giao diện và cho phép người dùng chỉnh sửa trực tiếp trên canvas.

Những bài học nổi bật bao gồm:

- Sự khó chịu trong thực tế có thể trở thành ý tưởng sản phẩm.
- Sự phối hợp trong team rất quan trọng khi tham gia hackathon.
- AI có thể được xem như một teammate, không chỉ là một công cụ.
- Chi phí token và sự nhất quán trong thiết kế là yếu tố quan trọng khi xây dựng sản phẩm AI.

#### How LLM Actually Works

Session này giải thích cách Large Language Models tạo ra văn bản và lý do vì sao kết quả có thể khác nhau ngay cả khi temperature được đặt bằng 0.

Các ý chính bao gồm:

- LLM tạo văn bản từng token một.
- Temperature ảnh hưởng đến mức độ ngẫu nhiên khi chọn token.
- Temperature 0 không đảm bảo output luôn giống nhau hoàn toàn.
- Tính toán floating-point trên GPU và inference batching có thể tạo ra khác biệt nhỏ.
- Developer cần thiết kế hệ thống AI với tư duy chấp nhận variance.

#### Enterprise-Grade Multi-Agent System

Session này giới thiệu hệ thống multi-agent dùng cho bài toán chấm điểm tín dụng startup. Thay vì sử dụng một AI agent duy nhất, hệ thống dùng nhiều agent chuyên biệt để phân tích các khía cạnh khác nhau của startup.

Các agent bao gồm:

- Financial Analyst Agent
- Market Analyst Agent
- Team Evaluator Agent
- Risk Assessor Agent
- Compliance Agent

Kiến trúc này giúp cải thiện khả năng audit, mở rộng hệ thống, tăng độ tin cậy và nâng cao chất lượng quyết định.

### Những Gì Học Được

#### Kiến Thức Chuyên Môn

- Context engineering rất quan trọng khi làm việc với Generative AI.
- Hệ thống AI nên được thiết kế với memory, retrieval và constraints rõ ràng.
- CloudFront có thể cải thiện bảo mật, hiệu năng, độ tin cậy và kiểm soát chi phí.
- Output của LLM có tính xác suất và không nên xem là hoàn toàn deterministic.
- Multi-agent architecture phù hợp với các bài toán enterprise phức tạp.
- AWS cung cấp nhiều dịch vụ hỗ trợ AI, automation, security và cloud infrastructure.

#### Tư Duy Kỹ Thuật

- Bảo mật và compliance nên được xem xét ngay từ đầu.
- Hệ thống cloud cần được thiết kế để có khả năng mở rộng và độ tin cậy cao.
- Sản phẩm AI nên giải quyết vấn đề thực tế của người dùng.
- Developer cần hiểu cả kiến trúc kỹ thuật và giá trị kinh doanh.
- Teamwork và khả năng iterate nhanh rất quan trọng trong phát triển sản phẩm thực tế.

### Ứng Dụng Vào Công Việc Và Học Tập

Những kiến thức học được từ sự kiện có thể áp dụng vào quá trình học tập và các dự án cá nhân của em như:

- Sử dụng prompt và context rõ ràng hơn khi làm việc với AI tools.
- Áp dụng các khái niệm cloud security và performance khi xây dựng web application.
- Tìm hiểu thêm về CloudFront, AWS WAF và edge computing.
- Phát triển các dự án AI assistant như PDF chat app, study assistant hoặc personal knowledge base.
- Nghiên cứu multi-agent systems cho các bài toán automation thông minh.
- Cải thiện tư duy thiết kế backend system theo hướng scalable, reliable và cost-aware.

### Trải Nghiệm Trong Event

Tham gia **AWS Vietnam Community Day 2026** là một trải nghiệm rất giá trị đối với em. Sự kiện giúp em hiểu rõ hơn về nhiều xu hướng công nghệ hiện đại, đặc biệt là Cloud Computing và Generative AI.

Em ấn tượng với sự đa dạng của các chủ đề, từ context engineering, cách hoạt động của LLM, hạ tầng CloudFront cho đến hệ thống enterprise multi-agent. Các diễn giả đã chia sẻ nhiều ví dụ thực tế, giúp em kết nối kiến thức lý thuyết với ứng dụng trong doanh nghiệp.

Là sinh viên ngành Software Engineering và có định hướng theo Backend Development, sự kiện này giúp em có thêm động lực để tiếp tục học AWS, AI và kiến trúc hệ thống hiện đại.

#### Một số hình ảnh khi tham gia sự kiện

![Tổng quan sự kiện](/images/4-EventParticipated/4.1-Event1/event_overview.png)

![Hình ảnh nhóm 1](/images/4-EventParticipated/4.1-Event1/team_photo1.png)

![Hình ảnh nhóm 2](/images/4-EventParticipated/4.1-Event1/team_photo2.png)

![Hình ảnh nhóm 3](/images/4-EventParticipated/4.1-Event1/team_photo3.png)

> Tổng thể, sự kiện đã mang lại cho em nhiều kiến thức kỹ thuật bổ ích, kinh nghiệm thực tế và định hướng rõ ràng hơn cho quá trình phát triển nghề nghiệp trong tương lai.