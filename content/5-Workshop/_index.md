---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

<div class="workshop-hero">
  <h1>Video Localization Platform</h1>
  <div class="workshop-hero-subtitle">Automated Video Translation and Dubbing System</div>
</div>

---

## Overview

This workshop walks through the architecture and implementation of a cloud-native video localization platform. The system automates subtitle extraction, translation, voice generation, and final video rendering so multilingual video processing becomes faster, more scalable, and more cost-effective.

The workshop is organized around the actual project structure in this repository, covering the backend services, AI/ML pipeline, frontend application, and deployment flow used to deliver the platform end to end.

## What You Will Build

By following this workshop, you will understand how to build a system that can:

- Upload and validate video files
- Extract subtitles using OCR or Speech-to-Text
- Translate subtitle content with context-aware AI
- Generate dubbed audio using Text-to-Speech providers
- Render final videos with subtitles and/or voice-over
- Track processing progress in real time

## Core Technologies

- **Frontend**: React, TypeScript, Vite, Zustand
- **Backend**: FastAPI, Celery, Redis, MySQL
- **AI/ML**: PaddleOCR, Google Cloud Speech-to-Text, **Amazon Translate (Primary) + Google Gemini (Fallback 1) + Google Cloud Translation v2 (Fallback 2)**, gTTS, ElevenLabs
- **Video Processing**: FFmpeg, MoviePy
- **Infrastructure**: Amazon S3, Amazon RDS, **Amazon Translate**, Amazon SES, Docker

---

## Workshop Content

1. [Video Localization Platform](5.1-Workshop-overview/)
2. [Development Environment Setup](5.2-Environment/)
3. [Backend Development](5.3-Backend/)
4. [AI/ML Pipeline](5.4-AI-Pipeline/)
5. [Frontend Development](5.5-Frontend/)
6. [Deployment](5.6-Deployment/)

---

## Suggested Learning Path

Start with the architecture overview in `5.1`, then move through environment setup, backend, AI pipeline, frontend, and finally deployment. This order matches the way the project is assembled in practice and makes it easier to connect each subsystem to the full workflow.
