---
title: "AI/ML Pipeline"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

# Workshop 5.4: AI/ML Pipeline

---

## Learning Objectives

By the end of this module, you will:

- Understand subtitle extraction using PaddleOCR
- Implement Speech-to-Text with Google Cloud
- Configure the multi-provider translation pipeline (Amazon Translate → Gemini → GCP Translate v2)
- Generate voice-over with gTTS and ElevenLabs
- Handle video format validation and transcoding
- Implement error handling and retry mechanisms

---

## Prerequisites

- Workshops 5.1-5.3 completed
- Understanding of OCR concepts
- Basic knowledge of video codecs
- API keys configured

---

## Architecture Context

The AI/ML Pipeline is the core processing engine:

![Pipeline AI-ML](/images/5-Workshop/5.4-AI-Pipeline/Pipeline-AI-ML.png)

---

## Step 1: Video Format Validation

### Why Validation Matters

Before processing, videos must be validated to ensure:
- Compatible codecs for FFmpeg processing
- Audio tracks exist for STT
- Video is not corrupted

### FFprobe Analysis

```python
# app/utils/video_validator.py
import subprocess
import json
from typing import Dict, Tuple

def analyze_video(video_path: str) -> Dict:
    """
    Analyze video file using FFprobe.
    
    Returns dictionary with:
    - container_format
    - format_name
    - duration
    - video_codec
    - audio_codec
    - has_audio
    - resolution
    - bitrate
    """
    cmd = [
        'ffprobe',
        '-v', 'quiet',
        '-print_format', 'json',
        '-show_format',
        '-show_streams',
        video_path
    ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    data = json.loads(result.stdout)
    
    # Extract relevant information
    analysis = {
        'container_format': data.get('format', {}).get('format_name'),
        'duration': float(data.get('format', {}).get('duration', 0)),
        'size': int(data.get('format', {}).get('size', 0)),
    }
    
    # Find video and audio streams
    for stream in data.get('streams', []):
        if stream['codec_type'] == 'video':
            analysis['video_codec'] = stream.get('codec_name')
            analysis['resolution'] = f"{stream.get('width')}x{stream.get('height')}"
            analysis['fps'] = eval(stream.get('r_frame_rate', '0'))
        elif stream['codec_type'] == 'audio':
            analysis['audio_codec'] = stream.get('codec_name')
            analysis['has_audio'] = True
    
    if 'has_audio' not in analysis:
        analysis['has_audio'] = False
    
    return analysis

def is_format_compatible(analysis: Dict) -> Tuple[bool, str]:
    """
    Check if video format is compatible with our processing pipeline.
    
    Returns (is_compatible, reason)
    """
    video_codec = analysis.get('video_codec', '').lower()
    audio_codec = analysis.get('audio_codec', '').lower()
    format_name = analysis.get('container_format', '').lower()
    
    # Check for compatible formats
    if 'h264' in video_codec or 'avc' in video_codec:
        if 'aac' in audio_codec or not analysis.get('has_audio'):
            return True, "H.264/AAC MP4 - fully compatible"
        return False, f"Video codec compatible but audio ({audio_codec}) needs conversion"
    
    if 'hevc' in video_codec or 'h265' in video_codec:
        return False, "HEVC/H.265 not supported, needs transcoding"
    
    if 'vp9' in video_codec:
        return False, "VP9 not supported, needs transcoding"
    
    if 'prores' in video_codec:
        return False, "ProRes not supported, needs transcoding"
    
    return False, f"Unknown codec: {video_codec}"
```

### Auto-Transcode Implementation

```python
# app/utils/video_validator.py (continued)
def transcode_to_mp4_h264(
    input_path: str, 
    output_path: str,
    force_transcode: bool = False
) -> bool:
    """
    Transcode video to H.264/AAC MP4 format.
    
    Args:
        input_path: Path to input video
        output_path: Path for output video
        force_transcode: If True, always transcode. If False, use fast copy when possible.
    
    Returns:
        True if successful, False otherwise
    """
    import os
    
    # Ensure output directory exists
    os.makedirs(os.path.dirname(output_path) or '.', exist_ok=True)
    
    # Build FFmpeg command
    cmd = [
        'ffmpeg',
        '-i', input_path,
        '-c:v', 'libx264',      # H.264 video codec
        '-preset', 'medium',     # Encoding speed/quality tradeoff
        '-crf', '23',            # Constant quality (lower = better)
        '-c:a', 'aac',          # AAC audio codec
        '-b:a', '192k',         # Audio bitrate
        '-movflags', '+faststart',  # Enable streaming
        '-y',                    # Overwrite output
        output_path
    ]
    
    result = subprocess.run(
        cmd,
        capture_output=True,
        text=True
    )
    
    return result.returncode == 0 and os.path.exists(output_path)
```

---

## Step 2: OCR Subtitle Extraction

### PaddleOCR Configuration

The system uses a singleton pattern for efficient OCR model management:

```python
# app/modules/video_process.py
from paddleocr import PaddleOCR
import numpy as np

# Singleton cache for OCR models
_ocr_instances_cache = {}
_default_lang = 'latin'  # Default for English, Vietnamese

def get_ocr_instance(
    lang: str = None,
    det_db_thresh: float = 0.2,
    det_db_box_thresh: float = 0.5,
    use_angle_cls: bool = False,
    use_gpu: bool = False
):
    """
    Get or create OCR instance for the specified language.
    
    Supported languages:
    - 'ch': Chinese
    - 'latin': English, Vietnamese, etc.
    - 'korean': Korean
    - 'japan': Japanese
    - 'arabic': Arabic
    """
    global _ocr_instances_cache
    
    if lang is None:
        lang = _default_lang
    
    # Normalize language code
    lang_map = {
        'en': 'latin', 'vi': 'latin',
        'zh': 'ch', 'ja': 'japan', 'ko': 'korean', 'ar': 'arabic'
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

### Video Frame Extraction and OCR

```python
def extract_subtitles_from_video(
    video_path: str,
    target_lang: str = None,
    fps_sample_rate: float = 1.0
) -> str:
    """
    Extract subtitles from video frames using OCR.
    
    Args:
        video_path: Path to video file
        target_lang: Target language for OCR model selection
        fps_sample_rate: Extract one frame per N seconds (1.0 = 1 FPS)
    
    Returns:
        SRT formatted subtitle string
    """
    import cv2
    import pysrt
    
    # Open video
    cap = cv2.VideoCapture(video_path)
    fps = cap.get(cv2.CAP_PROP_FPS)
    frame_interval = int(fps * fps_sample_rate)
    
    # Get OCR instance
    ocr = get_ocr_instance(lang=target_lang or 'latin')
    
    subtitles = []
    frame_count = 0
    current_subtitle = None
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        
        frame_count += 1
        
        # Only process every N frames
        if frame_count % frame_interval != 0:
            continue
        
        # Calculate timestamp
        timestamp = cap.get(cv2.CAP_PROP_POS_MSEC) / 1000.0
        
        # Run OCR on frame
        try:
            result = ocr.ocr(frame, cls=False)
            
            if result and result[0]:
                # Extract text from OCR results
                texts = []
                for line in result[0]:
                    if line and len(line) >= 2:
                        text = line[1][0]  # Text content
                        confidence = line[1][1]  # Confidence score
                        
                        # Filter low-confidence results
                        if confidence > 0.5 and len(text.strip()) > 2:
                            texts.append(text.strip())
                
                if texts:
                    full_text = ' '.join(texts)
                    
                    # Merge with previous subtitle if close enough
                    if (current_subtitle and 
                        timestamp - current_subtitle['end'] < 0.5):
                        current_subtitle['text'] += ' ' + full_text
                        current_subtitle['end'] = timestamp
                    else:
                        if current_subtitle:
                            subtitles.append(current_subtitle)
                        current_subtitle = {
                            'start': timestamp,
                            'end': timestamp + 2.0,  # Default 2 second duration
                            'text': full_text
                        }
        except Exception as e:
            print(f"OCR error at frame {frame_count}: {e}")
            continue
    
    cap.release()
    
    # Add last subtitle
    if current_subtitle:
        subtitles.append(current_subtitle)
    
    # Convert to SRT format
    return subtitles_to_srt(subtitles)

def subtitles_to_srt(subtitles: list) -> str:
    """Convert subtitle list to SRT format."""
    import pysrt
    
    srt_items = []
    for i, sub in enumerate(subtitles, 1):
        item = pysrt.SubtitleItem(
            index=i,
            start=pysrt.SubRipTime(seconds=sub['start']),
            end=pysrt.SubRipTime(seconds=sub['end']),
            text=sub['text']
        )
        srt_items.append(item)
    
    # Create SRT file
    subs = pysrt.SubtitleFile(items=srt_items)
    return str(subs)
```

### OCR Process Flow

![OCR](/images/5-Workshop/5.4-AI-Pipeline/OCR.png)

---

## Step 3: Speech-to-Text

### Google Cloud STT Integration

```python
# app/modules/module/module_speech_to_text.py
from google.cloud import speech_v1
from google.cloud.speech_v1 import enums
import os

def process_audio_to_subtitles(
    video_path: str,
    target_language: str = 'vi-VN'
) -> str:
    """
    Extract subtitles from video using Google Cloud Speech-to-Text.
    
    Args:
        video_path: Path to video file
        target_language: BCP-47 language tag
    
    Returns:
        SRT formatted subtitle string
    """
    # Extract audio from video
    audio_path = extract_audio_from_video(video_path)
    
    # Configure STT client
    client = speech_v1.SpeechClient()
    
    # Read audio file
    with io.open(audio_path, 'rb') as f:
        content = f.read()
    
    audio = speech_v1.RecognitionAudio(content=content)
    
    # Language code mapping
    language_map = {
        'en': 'en-US',
        'vi': 'vi-VN',
        'zh': 'zh-CN',
        'ja': 'ja-JP',
        'ko': 'ko-KR'
    }
    
    config = speech_v1.RecognitionConfig(
        encoding=enums.RecognitionConfig.AudioEncoding.LINEAR16,
        sample_rate_hertz=16000,
        language_code=language_map.get(target_language, 'en-US'),
        enable_automatic_punctuation=True,
        model='latest_long',
        use_enhanced=True
    )
    
    # Perform transcription
    operation = client.long_running_recognize(config=config, audio=audio)
    response = operation.result(timeout=600)
    
    # Convert to SRT
    subtitles = []
    for result in response.results:
        alternative = result.alternatives[0]
        words = alternative.words
        
        # Group words into subtitle segments
        # (simplified - real implementation is more complex)
        for word_info in words:
            # Create subtitle entries based on word timing
            pass
    
    return srt_content

def extract_audio_from_video(video_path: str) -> str:
    """Extract audio track from video using FFmpeg."""
    import subprocess
    
    audio_path = video_path.replace('.mp4', '.wav')
    
    cmd = [
        'ffmpeg', '-i', video_path,
        '-vn',           # No video
        '-acodec', 'pcm_s16le',  # PCM 16-bit
        '-ar', '16000',  # 16kHz sample rate
        '-ac', '1',      # Mono
        '-y',            # Overwrite
        audio_path
    ]
    
    subprocess.run(cmd, capture_output=True)
    return audio_path
```

---

## Step 4: Translation Pipeline (Multi-Provider)

The platform translates subtitles through a **multi-provider translation pipeline** for resilience and quality. Each subtitle segment is routed through providers in priority order; if one fails, the orchestrator transparently fails over to the next. This avoids the historical problem where a single Gemini outage would block every Stage-1 job.

| Priority | Provider | Role | Trigger |
|----------|----------|------|---------|
| 1 (Primary) | **Amazon Translate** | First attempt | AWS-eligible language pairs (en↔vi, ko↔vi, ja↔vi, zh↔vi, …) |
| 2 (Fallback 1) | **Google Gemini API** | Context-aware fallback | Amazon ineligible OR Amazon errored (retryable category) |
| 3 (Fallback 2) | **Google Cloud Translation v2** | Last-resort shared batch | Both Amazon and Gemini failed for the segment |

### Architecture: Why a pipeline, not a single provider

A single Gemini dependency became a single point of failure — quota exhaustion, regional outages, or upstream rate-limits would abort translation entirely. The pipeline layers three providers with deterministic routing and explicit failure semantics:

- **Provider ownership contract** — Each adapter performs exactly ONE provider operation. No nested fallback inside an adapter. No silent source-text return.
- **Routing logic** — AWS-eligible segments try Amazon first, then Gemini. AWS-ineligible segments (mixed / unknown / null source language) skip Amazon and go straight to Gemini.
- **Single retry boundary** — botocore's hidden retry is disabled via `Config(retries={'max_attempts': 1})`; Tenacity owns retry (3 attempts, exponential backoff 1–10s) with classification-driven policy from `provider_failure.classify_failure`.
- **Observability** — Every attempt is logged with `provider`, `status` (ATTEMPTED / SUCCESS / FAILED), latency, and the `attempted_providers` chain for end-to-end traceability.

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
    """Translate a single text using Amazon Translate (primary)."""
    client = get_translate_client()
    response = client.translate_text(
        Text=text,
        SourceLanguageCode=normalize_language_code(source_lang),
        TargetLanguageCode=normalize_language_code(target_lang),
    )
    return response['TranslatedText']

def translate_batch(texts: list, source_lang: str = 'auto', target_lang: str = 'vi') -> list:
    """Amazon Translate has no native batch API — process sequentially."""
    results = []
    for text in texts:
        if not text or not text.strip():
            results.append("")
            continue
        try:
            results.append(translate_text(text, source_lang, target_lang))
        except Exception as e:
            # Caller (orchestrator) decides whether to fail over to Gemini
            raise
    return results
```

**Configuration:**

```bash
# .env — Amazon Translate uses the same AWS credentials as S3/SES
AWS_REGION=ap-southeast-2
AWS_TRANSLATE_REGION=ap-southeast-2   # typically same as main region
```

### Orchestrator: AWS-eligible first, Gemini as fallback, GCP v2 as last resort

```python
# app/modules/module/module_speech_to_text.py (excerpt)
from app.utils.aws_translate import translate_text

def _call_amazon_translate(text: str, aws_source_code: str, target_language: str) -> str:
    """Amazon Translate adapter — ONE provider operation only."""
    return translate_text(text, source_lang=aws_source_code, target_lang=target_language)

def _call_gemini_translate(text: str, target_language: str = 'vi') -> str:
    """Gemini adapter — ONE provider operation only."""
    model = genai.GenerativeModel(os.getenv('API_MODEL', 'gemini-2.0-flash'))
    response = model.generate_content(
        f"Translate to {target_language}. Output only the translation:\n\n{text}"
    )
    return response.text.strip()

def _call_google_translate_batch(texts: list, target_language: str) -> list:
    """Google Cloud Translation v2 — single batched call, last resort."""
    if translate_client is None:
        raise RuntimeError("google-cloud-translate not configured")
    response = translate_client.translate(texts, target_language=target_language)
    return [t['translatedText'] for t in response]

def translate_stt_segment(text: str, source_language: str, target_language: str) -> str:
    """
    Per-segment orchestration: Amazon → Gemini → GCP v2.
    Returns translated text on the first provider that succeeds.
    """
    attempted = []

    # STEP 1: AMAZON (only if language pair is AWS-eligible)
    if is_aws_eligible(source_language, target_language):
        attempted.append('AMAZON_TRANSLATE')
        try:
            return _call_amazon_translate(text, source_language, target_language)
        except Exception:
            logger.warning(f"[STT_TRANSLATION] AMAZON failed, falling back to GEMINI")

    # STEP 2: GEMINI (context-aware fallback)
    attempted.append('GEMINI')
    try:
        return _call_gemini_translate(text, target_language)
    except Exception:
        logger.warning(f"[STT_TRANSLATION] GEMINI failed, falling back to GCP v2")

    # STEP 3: GCP v2 (last resort — caller wraps in batch call site)
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
        return False  # never retry a deterministic failure
    return is_retryable(classify_failure(exc))
```

### Batch Translation (with shared GCP v2 fallback)

```python
def translate_stt_segments(segments: list, target_language: str) -> list:
    """
    Translate a batch of STT segments. Falls back to a single
    shared Google Translate v2 call for segments that exhausted
    both Amazon and Gemini.
    """
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

### Environment Variables

```bash
# .env — translation configuration
# Amazon Translate (primary) — uses existing AWS credentials
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
import asyncio
from concurrent.futures import ThreadPoolExecutor

def generate_audio_from_srt(
    srt_content: str,
    output_path: str,
    provider: str = 'gtts',
    voice_id: str = None,
    speed: float = 1.0
) -> str:
    """
    Generate TTS audio from SRT content.
    
    Args:
        srt_content: SRT subtitle content
        output_path: Output audio file path
        provider: 'gtts' or 'elevenlabs'
        voice_id: Voice ID for ElevenLabs
        speed: Speech speed multiplier
    
    Returns:
        Path to generated audio file
    """
    import pysrt
    
    # Parse SRT
    subs = pysrt.open(io.StringIO(srt_content))
    
    # Generate audio for each subtitle segment
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
    
    # Concatenate all segments
    final_audio = concatenate_audio_segments(audio_segments)
    
    # Save final audio
    final_audio.export(output_path, format='mp3')
    
    return output_path

def generate_segment_audio(
    text: str,
    provider: str,
    voice_id: str = None
) -> str:
    """Generate audio for a single segment."""
    
    if provider == 'gtts':
        from gtts import gTTS
        
        tts = gTTS(text=text, lang='vi')
        temp_path = f'/tmp/tts_{uuid.uuid4()}.mp3'
        tts.save(temp_path)
        return temp_path
    
    elif provider == 'elevenlabs':
        import requests
        
        # ElevenLabs API call
        url = f"https://api.elevenlabs.io/v1/text-to-speech/{voice_id}"
        
        headers = {
            "Accept": "audio/mpeg",
            "Content-Type": "application/json",
            "xi-api-key": os.getenv('ELEVENLABS_API_KEY')
        }
        
        data = {
            "text": text,
            "voice_settings": {
                "stability": 0.5,
                "similarity_boost": 0.5
            }
        }
        
        response = requests.post(url, json=data, headers=headers)
        
        temp_path = f'/tmp/tts_{uuid.uuid4()}.mp3'
        with open(temp_path, 'wb') as f:
            f.write(response.content)
        
        return temp_path
    
    else:
        raise ValueError(f"Unknown TTS provider: {provider}")
```

### Complete Data Flow

![Data flow](/images/5-Workshop/5.4-AI-Pipeline/Data-flow.png)

---

## Step 6: Video Rendering

### FFmpeg Video Composition

```python
# app/modules/module/module_meger_video_v2.py
def process_video_with_sync(
    video_path: str,
    srt_path: str,
    audio_path: str = None,
    output_path: str = None
) -> str:
    """
    Merge video with subtitles and optional audio track.
    
    Args:
        video_path: Input video path
        srt_path: SRT subtitle file path
        audio_path: Optional TTS audio path
        output_path: Output video path
    
    Returns:
        Path to rendered video
    """
    if output_path is None:
        output_path = video_path.replace('.mp4', '_rendered.mp4')
    
    if audio_path:
        # Render with audio replacement
        cmd = [
            'ffmpeg',
            '-i', video_path,
            '-i', audio_path,
            '-vf', f"subtitles='{srt_path}'",
            '-map', '0:v',
            '-map', '1:a',
            '-c:v', 'libx264',
            '-c:a', 'aac',
            '-shortest',
            '-y',
            output_path
        ]
    else:
        # Render with soft subtitles (displayed but not burned in)
        cmd = [
            'ffmpeg',
            '-i', video_path,
            '-vf', f"subtitles='{srt_path}'",
            '-c:v', 'libx264',
            '-c:a', 'copy',
            '-y',
            output_path
        ]
    
    result = subprocess.run(cmd, capture_output=True, text=True)
    
    if result.returncode != 0:
        raise Exception(f"Video rendering failed: {result.stderr}")
    
    return output_path
```

---

## Step 7: Error Handling and Retry

### Exponential Backoff Implementation

```python
# app/utils/provider_failure.py
import time
import random
from functools import wraps

def exponential_backoff(
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0,
    exponential_base: float = 2.0
):
    """
    Decorator for exponential backoff retry logic.
    
    Usage:
        @exponential_backoff(max_retries=3)
        def my_api_call():
            ...
    """
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    
                    # Calculate delay with jitter
                    delay = min(
                        base_delay * (exponential_base ** attempt),
                        max_delay
                    )
                    jitter = random.uniform(0, 0.1 * delay)
                    delay += jitter
                    
                    print(f"Attempt {attempt + 1} failed: {e}")
                    print(f"Retrying in {delay:.2f} seconds...")
                    
                    time.sleep(delay)
            
            # All retries exhausted
            raise last_exception
        
        return wrapper
    return decorator

# Usage with translation API
@exponential_backoff(max_retries=3, base_delay=2.0)
def translate_with_retry(text: str, target_lang: str) -> str:
    return translate_srt(text, target_lang)
```

---

## Code References

| Component | File | Description |
|-----------|------|-------------|
| Video Validation | `app/utils/video_validator.py` | FFprobe analysis, transcoding |
| OCR Engine | `app/modules/video_process.py` | PaddleOCR integration |
| STT Module | `app/modules/module/module_speech_to_text.py` | Google Cloud STT |
| Translation Pipeline | `app/utils/aws_translate.py` (Amazon) + `app/modules/module/module_speech_to_text.py` (orchestrator) | Amazon Translate primary, Gemini & GCP v2 fallbacks |
| TTS Module | `app/modules/module/module_text_to_speech_v4_parallel.py` | gTTS/ElevenLabs |
| Video Render | `app/modules/module/module_meger_video_v2.py` | FFmpeg composition |
| Error Handling | `app/utils/provider_failure.py` | Exponential backoff |

---

## Summary

In this module, you learned:

- **Video Validation**: FFprobe analysis and FFmpeg transcoding
- **OCR Extraction**: PaddleOCR with singleton pattern
- **Speech-to-Text**: Google Cloud STT for audio transcription
- **Translation Pipeline**: Amazon Translate (primary) → Gemini (fallback 1) → GCP Translate v2 (fallback 2), classification-driven retry policy
- **Text-to-Speech**: gTTS and ElevenLabs integration
- **Video Rendering**: FFmpeg subtitle and audio composition
- **Error Handling**: Exponential backoff retry mechanism

---

## Next Steps

Proceed to [Workshop 5.5: Frontend Development](5.5-Frontend/) to understand the React frontend implementation.

---

## Validation

Test individual components:

```bash
# Test video validation
python -c "
from app.utils.video_validator import analyze_video, is_format_compatible
analysis = analyze_video('test_video.mp4')
print(analysis)
print(is_format_compatible(analysis))
"

# Test OCR extraction
python -c "
from app.modules.video_process import extract_subtitles_from_video
srt = extract_subtitles_from_video('test_video.mp4')
print(srt[:500])
"
```
