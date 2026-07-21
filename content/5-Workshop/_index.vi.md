---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

<div class="workshop-hero">
  <h1>Video Localization Platform</h1>
  <div class="workshop-hero-subtitle">Hệ thống Tự động hóa Dịch thuật và Lồng tiếng Video</div>
</div>

---

## Tổng quan

Workshop này hướng dẫn toàn bộ kiến trúc và cách triển khai một nền tảng video localization trên cloud. Hệ thống tự động hóa các bước trích xuất phụ đề, dịch thuật, tạo giọng nói và render video cuối cùng, từ đó giúp quy trình xử lý video đa ngôn ngữ nhanh hơn, dễ mở rộng hơn và tiết kiệm chi phí hơn.

Nội dung workshop được tổ chức bám sát cấu trúc thật của project trong repository này, bao gồm backend service, AI/ML pipeline, frontend application và quy trình deployment hoàn chỉnh.

## Bạn sẽ xây dựng gì

Khi đi hết workshop, bạn sẽ hiểu cách xây dựng một hệ thống có thể:

- Upload và kiểm tra tính hợp lệ của file video
- Trích xuất phụ đề bằng OCR hoặc Speech-to-Text
- Dịch nội dung phụ đề bằng AI có nhận biết ngữ cảnh
- Tạo audio lồng tiếng bằng các dịch vụ Text-to-Speech
- Render video đầu ra với phụ đề và/hoặc voice-over
- Theo dõi tiến độ xử lý theo thời gian thực

## Công nghệ cốt lõi

- **Frontend**: React, TypeScript, Vite, Zustand
- **Backend**: FastAPI, Celery, Redis, MySQL
- **AI/ML**: PaddleOCR, Google Cloud Speech-to-Text, **Amazon Translate (Primary) + Google Gemini (Fallback 1) + Google Cloud Translation v2 (Fallback 2)**, gTTS, ElevenLabs
- **Xử lý video**: FFmpeg, MoviePy
- **Hạ tầng**: Amazon S3, Amazon RDS, **Amazon Translate**, Amazon SES, Docker

---

## Nội dung workshop

1. [Video Localization Platform](5.1-Workshop-overview/)
2. [Thiết lập môi trường phát triển](5.2-Environment/)
3. [Phát triển Backend](5.3-Backend/)
4. [AI/ML Pipeline](5.4-AI-Pipeline/)
5. [Phát triển Frontend](5.5-Frontend/)
6. [Triển khai hệ thống](5.6-Deployment/)

---

## Lộ trình học gợi ý

Bạn nên bắt đầu từ phần tổng quan kiến trúc ở `5.1`, sau đó đi lần lượt qua thiết lập môi trường, backend, AI pipeline, frontend và cuối cùng là deployment. Thứ tự này bám sát cách project được ghép thành một hệ thống hoàn chỉnh trong thực tế.
