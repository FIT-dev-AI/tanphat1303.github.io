---
title: "AI/ML Pipeline"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Workshop 5.4: AI/ML Pipeline

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Hiểu subtitle extraction sử dụng PaddleOCR
- Implement Speech-to-Text với Google Cloud
- Cấu hình multi-provider translation pipeline (Amazon Translate → Gemini → GCP Translate v2)
- Generate voice-over với gTTS và ElevenLabs
- Handle video format validation và transcoding
- Implement error handling và retry mechanisms

---

## Điều kiện tiên quyết

- Hoàn thành Workshops 5.1-5.3
- Understanding của OCR concepts
- Basic knowledge của video codecs
- API keys đã configured

---

## Kiến trúc Pipeline

![Pipeline AI-ML](/images/5-Workshop/5.4-AI-Pipeline/Pipeline-AI-ML.png)

---

## Step 1: Video Format Validation

### FFprobe Analysis

```python
# app/utils/video_validator.py
def analyze_video(video_path: str) -> Dict:
    """
    Analyze video file using FFprobe.
    """
    cmd = [
        'ffprobe', '-v', 'quiet',
        '-print_format', 'json',
        '-show_format', '-show_streams',
        video_path
    ]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return json.loads(result.stdout)
```

### Format Compatibility Check

```python
def is_format_compatible(analysis: Dict) -> Tuple[bool, str]:
    """Check if video format is compatible."""
    video_codec = analysis.get('video_codec', '').lower()
    
    if 'h264' in video_codec or 'avc' in video_codec:
        return True, "H.264/AAC MP4 - fully compatible"
    
    if 'hevc' in video_codec or 'h265' in video_codec:
        return False, "HEVC/H.265 not supported, needs transcoding"
    
    return False, f"Unknown codec: {video_codec}"
```

---

## Step 2: OCR Subtitle Extraction

### PaddleOCR Configuration

```python
# app/modules/video_process.py
_ocr_instances_cache = {}
_default_lang = 'latin'

def get_ocr_instance(
    lang: str = None,
    det_db_thresh: float = 0.2,
    det_db_box_thresh: float = 0.5,
    use_angle_cls: bool = False,
    use_gpu: bool = False
):
    """Get or create OCR instance for the specified language."""
    global _ocr_instances_cache
    
    if lang is None:
        lang = _default_lang
    
    lang_map = {
        'en': 'latin', 'vi': 'latin',
        'zh': 'ch', 'ja': 'japan', 'ko': 'korean'
    }
    lang = lang_map.get(lang, lang)
    
    cache_key = f"{lang}_{det_db_thresh}_{det_db_box_thresh}"
    
    if cache_key not in _ocr_instances_cache:
        _ocr_instances_cache[cache_key] = PaddleOCR(
            use_angle_cls=use_angle_cls,
            lang=lang,
            det_db_thresh=det_db_thresh,
            det_db_box_thresh=det_db_box_thresh,
            use_gpu=use_gpu,
            show_log=False
        )
    
    return _ocr_instances_cache[cache_key]
```

### Video Frame Extraction

```python
def extract_subtitles_from_video(
    video_path: str,
    target_lang: str = None,
    fps_sample_rate: float = 1.0
) -> str:
    """Extract subtitles from video frames using OCR."""
    import cv2
    
    cap = cv2.VideoCapture(video_path)
    fps = cap.get(cv2.CAP_PROP_FPS)
    frame_interval = int(fps * fps_sample_rate)
    
    ocr = get_ocr_instance(lang=target_lang or 'latin')
    subtitles = []
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        
        frame_count = cap.get(cv2.CAP_PROP_POS_FRAMES)
        
        if frame_count % frame_interval != 0:
            continue
        
        result = ocr.ocr(frame, cls=False)
        
        if result and result[0]:
            texts = [
                line[1][0] 
                for line in result[0] 
                if line[1][1] > 0.5 and len(line[1][0].strip()) > 2
            ]
            if texts:
                subtitles.append({
                    'start': cap.get(cv2.CAP_PROP_POS_MSEC) / 1000.0,
                    'end': cap.get(cv2.CAP_PROP_POS_MSEC) / 1000.0 + 2.0,
                    'text': ' '.join(texts)
                })
    
    cap.release()
    return subtitles_to_srt(subtitles)
```

### OCR Process Flow

![OCR](/images/5-Workshop/5.4-AI-Pipeline/OCR.png)

---

## Step 3: Speech-to-Text

### Google Cloud STT Integration

```python
# app/modules/module/module_speech_to_text.py
from google.cloud import speech_v1

def process_audio_to_subtitles(
    video_path: str,
    target_language: str = 'vi-VN'
) -> str:
    """Extract subtitles using Google Cloud Speech-to-Text."""
    
    # Extract audio from video
    audio_path = extract_audio_from_video(video_path)
    
    client = speech_v1.SpeechClient()
    
    with open(audio_path, 'rb') as f:
        content = f.read()
    
    audio = speech_v1.RecognitionAudio(content=content)
    
    config = speech_v1.RecognitionConfig(
        encoding=enums.RecognitionConfig.AudioEncoding.LINEAR16,
        sample_rate_hertz=16000,
        language_code=target_language,
        enable_automatic_punctuation=True,
        model='latest_long'
    )
    
    operation = client.long_running_recognize(config=config, audio=audio)
    response = operation.result(timeout=600)
    
    return convert_to_srt(response.results)
```

---

## Step 4: Translation Pipeline (Đa nhà cung cấp)

Hệ thống dịch phụ đề qua một **translation pipeline đa nhà cung cấp** để đảm bảo resilience và chất lượng. Mỗi segment được định tuyến qua các provider theo thứ tự ưu tiên; nếu một provider lỗi, orchestrator tự động failover sang provider tiếp theo. Cách này loại bỏ điểm lỗi duy nhất từng tồn tại khi chỉ phụ thuộc Gemini.

| Ưu tiên | Provider | Vai trò | Khi nào dùng |
|---------|----------|---------|--------------|
| 1 (Primary) | **Amazon Translate** | Lần thử đầu tiên | Cặp ngôn ngữ AWS-eligible (en↔vi, ko↔vi, ja↔vi, zh↔vi, …) |
| 2 (Fallback 1) | **Google Gemini API** | Fallback nhận biết ngữ cảnh | Amazon không đủ điều kiện hoặc Amazon lỗi (retryable category) |
| 3 (Fallback 2) | **Google Cloud Translation v2** | Last-resort shared batch | Cả Amazon và Gemini đều fail |

### Kiến trúc: Tại sao cần pipeline thay vì một provider

Phụ thuộc vào Gemini duy nhất từng là điểm lỗi đơn lẻ — quota exhaustion, regional outage, hoặc upstream rate-limit có thể chặn toàn bộ Stage-1. Pipeline xếp 3 provider với routing xác định và failure semantics rõ ràng:

- **Provider ownership contract** — Mỗi adapter chỉ thực hiện đúng MỘT provider operation. Không nested fallback. Không silent source-text return.
- **Routing logic** — Segment AWS-eligible thử Amazon trước, rồi Gemini. Segment AWS-ineligible (mixed/unknown/null source) bỏ qua Amazon, đi thẳng tới Gemini.
- **Ranh giới retry duy nhất** — botocore hidden retry bị disable qua `Config(retries={'max_attempts': 1})`; Tenacity sở hữu retry (3 attempts, exponential backoff 1–10s) với policy classification-driven từ `provider_failure.classify_failure`.
- **Observability** — Mỗi attempt được log với `provider`, `status` (ATTEMPTED / SUCCESS / FAILED), latency, và chuỗi `attempted_providers` để truy vết end-to-end.

### Primary Adapter: Amazon Translate

```python
# app/utils/aws_translate.py
import boto3
from botocore.config import Config
from tenacity import retry, stop_after_attempt, wait_exponential

LANGUAGE_CODE_MAP = {
    'ko': 'ko', 'ko-KR': 'ko',
    'en': 'en', 'en-US': 'en',
    'cmn': 'zh', 'cmn-Hans-CN': 'zh', 'zh': 'zh',
    'ja': 'ja', 'vi': 'vi', 'auto': 'auto',
}

def normalize_language_code(lang: str) -> str:
    if not lang:
        return 'auto'
    return LANGUAGE_CODE_MAP.get(lang, lang)

def get_translate_client():
    return boto3.client(
        'translate',
        region_name=settings.AWS_TRANSLATE_REGION,
        config=Config(retries={'max_attempts': 1}, connect_timeout=60, read_timeout=60),
    )

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=10),
    reraise=True,
)
def translate_text(text: str, source_lang: str = 'auto', target_lang: str = 'vi') -> str:
    """Dịch một đoạn text bằng Amazon Translate (primary)."""
    client = get_translate_client()
    response = client.translate_text(
        Text=text,
        SourceLanguageCode=normalize_language_code(source_lang),
        TargetLanguageCode=normalize_language_code(target_lang),
    )
    return response['TranslatedText']

def translate_batch(texts: list, source_lang: str = 'auto', target_lang: str = 'vi') -> list:
    """Amazon Translate không có batch API native — xử lý tuần tự."""
    results = []
    for text in texts:
        if not text or not text.strip():
            results.append("")
            continue
        try:
            results.append(translate_text(text, source_lang, target_lang))
        except Exception:
            raise  # Caller (orchestrator) sẽ quyết định có failover sang Gemini hay không
    return results
```

**Cấu hình:**

```bash
# .env — Amazon Translate dùng chung credentials với S3/SES
AWS_REGION=ap-southeast-2
AWS_TRANSLATE_REGION=ap-southeast-2   # thường trùng với region chính
```

### Orchestrator: AWS-eligible trước, Gemini fallback, GCP v2 last resort

```python
# app/modules/module/module_speech_to_text.py (trích đoạn)
from app.utils.aws_translate import translate_text

def _call_amazon_translate(text: str, aws_source_code: str, target_language: str) -> str:
    """Amazon Translate adapter — chỉ MỘT provider operation."""
    return translate_text(text, source_lang=aws_source_code, target_lang=target_language)

def _call_gemini_translate(text: str, target_language: str = 'vi') -> str:
    """Gemini adapter — chỉ MỘT provider operation."""
    model = genai.GenerativeModel(os.getenv('API_MODEL', 'gemini-2.0-flash'))
    response = model.generate_content(
        f"Translate to {target_language}. Output only the translation:\n\n{text}"
    )
    return response.text.strip()

def _call_google_translate_batch(texts: list, target_language: str) -> list:
    """Google Cloud Translation v2 — một batched call duy nhất, last resort."""
    if translate_client is None:
        raise RuntimeError("google-cloud-translate chưa được cấu hình")
    response = translate_client.translate(texts, target_language=target_language)
    return [t['translatedText'] for t in response]

def translate_stt_segment(text: str, source_language: str, target_language: str) -> str:
    """Orchestration theo segment: Amazon → Gemini → GCP v2."""
    attempted = []

    # STEP 1: AMAZON (chỉ khi cặp ngôn ngữ AWS-eligible)
    if is_aws_eligible(source_language, target_language):
        attempted.append('AMAZON_TRANSLATE')
        try:
            return _call_amazon_translate(text, source_language, target_language)
        except Exception:
            logger.warning("[STT_TRANSLATION] AMAZON failed, falling back to GEMINI")

    # STEP 2: GEMINI (context-aware fallback)
    attempted.append('GEMINI')
    try:
        return _call_gemini_translate(text, target_language)
    except Exception:
        logger.warning("[STT_TRANSLATION] GEMINI failed, falling back to GCP v2")

    # STEP 3: GCP v2 (last resort — caller wrap trong batch call site)
    raise TranslationFailure(
        f"All translation providers exhausted. attempted={attempted}",
        attempted_providers=attempted,
    )
```

**Retry classification (TASK 9):**

```python
# app/utils/provider_failure.py
RETRYABLE_CATEGORIES = {
    'TIMEOUT', 'RATE_LIMIT', 'QUOTA_EXCEEDED',
    'SERVICE_UNAVAILABLE', 'INTERNAL_ERROR',
    'CONNECTION_RESET', 'DNS', 'TLS',
    'BAD_RESPONSE', 'MALFORMED_RESPONSE',
}

def _is_retryable_exception(exc: BaseException) -> bool:
    if isinstance(exc, TranslationFailure):
        return False  # không retry một deterministic failure
    return is_retryable(classify_failure(exc))
```

### Batch Translation (kèm shared GCP v2 fallback)

```python
def translate_stt_segments(segments: list, target_language: str) -> list:
    """Dịch một batch STT segment, fallback GCP v2 cho segment fail cả 2 provider trên."""
    results = [None] * len(segments)
    gcp_v2_pending = []

    for idx, seg in enumerate(segments):
        try:
            results[idx] = translate_stt_segment(
                seg.text, seg.source_language, target_language
            )
        except TranslationFailure:
            gcp_v2_pending.append(idx)

    if gcp_v2_pending:
        pending_texts = [segments[i].text for i in gcp_v2_pending]
        gcp_results = _call_google_translate_batch(pending_texts, target_language)
        for i, translated in zip(gcp_v2_pending, gcp_results):
            results[i] = translated

    return results
```

### Biến môi trường

```bash
# .env — translation configuration
# Amazon Translate (primary) — dùng chung AWS credentials
AWS_REGION=ap-southeast-2
AWS_TRANSLATE_REGION=ap-southeast-2

# Google Gemini (fallback 1)
API_KEY=your_gemini_api_key
API_MODEL=gemini-2.0-flash

# Google Cloud Translation v2 (fallback 2, last resort)
GOOGLE_APPLICATION_CREDENTIALS=/path/to/gcp-service-account.json
```

---

## Step 5: Text-to-Speech

### TTS Module

```python
# app/modules/module/module_text_to_speech_v4_parallel.py
def generate_audio_from_srt(
    srt_content: str,
    output_path: str,
    provider: str = 'gtts',
    voice_id: str = None
) -> str:
    """Generate TTS audio from SRT content."""
    import pysrt
    
    subs = pysrt.open(io.StringIO(srt_content))
    audio_segments = []
    
    for sub in subs:
        segment_audio = generate_segment_audio(
            text=sub.text,
            provider=provider,
            voice_id=voice_id
        )
        audio_segments.append({
            'audio': segment_audio,
            'start': sub.start.ordinal,
            'end': sub.end.ordinal
        })
    
    final_audio = concatenate_audio_segments(audio_segments)
    final_audio.export(output_path, format='mp3')
    
    return output_path
```

### Complete Data Flow

![Data flow](/images/5-Workshop/5.4-AI-Pipeline/Data-flow.png)

---

## Step 6: Video Rendering

```python
# app/modules/module/module_meger_video_v2.py
def process_video_with_sync(
    video_path: str,
    srt_path: str,
    audio_path: str = None,
    output_path: str = None
) -> str:
    """Merge video with subtitles and optional audio."""
    
    if audio_path:
        cmd = [
            'ffmpeg', '-i', video_path, '-i', audio_path,
            '-vf', f"subtitles='{srt_path}'",
            '-map', '0:v', '-map', '1:a',
            '-c:v', 'libx264', '-c:a', 'aac',
            '-shortest', '-y', output_path
        ]
    else:
        cmd = [
            'ffmpeg', '-i', video_path,
            '-vf', f"subtitles='{srt_path}'",
            '-c:v', 'libx264', '-c:a', 'copy',
            '-y', output_path
        ]
    
    subprocess.run(cmd, capture_output=True)
    return output_path
```

---

## Step 7: Error Handling

```python
# app/utils/provider_failure.py
def exponential_backoff(
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0
):
    """Decorator for exponential backoff retry."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    delay = min(base_delay * (2 ** attempt), max_delay)
                    print(f"Retry {attempt + 1} in {delay}s: {e}")
                    time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator
```

---

## Code References

| Component | File |
|-----------|------|
| Video Validation | `app/utils/video_validator.py` |
| OCR Engine | `app/modules/video_process.py` |
| STT Module | `app/modules/module/module_speech_to_text.py` |
| Translation | `app/modules/video_process.py` |
| TTS Module | `app/modules/module/module_text_to_speech_v4_parallel.py` |
| Video Render | `app/modules/module/module_meger_video_v2.py` |

---

## Tóm tắt

Trong module này, bạn đã học:

- **Video Validation**: FFprobe analysis và FFmpeg transcoding
- **OCR Extraction**: PaddleOCR với singleton pattern
- **Speech-to-Text**: Google Cloud STT cho audio transcription
- **Translation Pipeline**: Amazon Translate (primary) → Gemini (fallback 1) → GCP Translate v2 (fallback 2), retry policy classification-driven
- **Text-to-Speech**: gTTS và ElevenLabs integration
- **Video Rendering**: FFmpeg subtitle và audio composition
- **Error Handling**: Exponential backoff retry mechanism

---

## Bước tiếp theo

Tiếp tục đến [Workshop 5.5: Frontend Development](5.5-Frontend/) để hiểu React frontend implementation.
