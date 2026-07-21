---
title: "Frontend Development"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Workshop 5.5: Frontend Development

---

## Mục tiêu học tập

Sau khi hoàn thành module này, bạn sẽ:

- Hiểu React component architecture
- Implement state management với Zustand
- Create WebSocket client cho real-time updates
- Build video player với subtitle overlay
- Implement auto-save functionality
- Design video editor interface

---

## Điều kiện tiên quyết

- Hoàn thành Workshops 5.1-5.4
- React và TypeScript knowledge
- Zustand state management basics
- WebSocket fundamentals

---

## Kiến trúc Frontend

![Frontend Architecture](/images/5-Workshop/5.5-Frontend/Fr_end.png)

---

## Step 1: Video Store với Zustand

```typescript
// src/store/use-video-store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface VideoItem {
  videoId: string;
  title: string;
  status: VideoStatus;
  processingPercentage: number;
}

type VideoStatus = 
  | 'PROCESSING'
  | 'AWAITING_REVIEW'
  | 'STAGE2_PROCESSING'
  | 'COMPLETED'
  | 'FAILED';

interface VideoStore {
  videos: VideoItem[];
  addVideo: (video: VideoItem) => void;
  updateVideo: (videoId: string, updates: Partial<VideoItem>) => void;
  updateProgress: (videoId: string, percentage: number, status?: VideoStatus) => void;
}

export const useVideoStore = create<VideoStore>()(
  persist(
    (set, get) => ({
      videos: [],
      
      addVideo: (video) => set((state) => ({
        videos: [video, ...state.videos]
      })),
      
      updateVideo: (videoId, updates) => set((state) => ({
        videos: state.videos.map((v) =>
          v.videoId === videoId ? { ...v, ...updates } : v
        )
      })),
      
      updateProgress: (videoId, percentage, status) => {
        const updates: Partial<VideoItem> = { processingPercentage: percentage };
        if (status) updates.status = status;
        get().updateVideo(videoId, updates);
      }
    }),
    {
      name: 'video-storage',
      storage: createJSONStorage(() => localStorage)
    }
  )
);
```

### Zustand State Flow

![Zustand State Flow](/images/5-Workshop/5.5-Frontend/Luong_state_Zustand.png)

---

## Step 2: WebSocket Client

```typescript
// src/hooks/useVideoProgress.ts
export function useVideoProgress(videoId: string) {
  const wsRef = useRef<WebSocket | null>(null);
  const updateProgress = useVideoStore((state) => state.updateProgress);
  
  useEffect(() => {
    const ws = new WebSocket(`${WS_URL}/ws/progress/${videoId}`);
    
    ws.onopen = () => {
      console.log('[WebSocket] Connected');
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      updateProgress(data.video_id, data.percentage, data.status);
    };
    
    ws.onclose = () => {
      // Reconnect logic here
    };
    
    wsRef.current = ws;
    
    return () => ws.close();
  }, [videoId, updateProgress]);
  
  return { disconnect: () => wsRef.current?.close() };
}
```

---

## Step 3: Video Player Component

```typescript
// src/components/VideoPlayer.tsx
interface VideoPlayerProps {
  src: string;
  subtitles?: Subtitle[];
}

export function VideoPlayer({ src, subtitles = [] }: VideoPlayerProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [currentSubtitle, setCurrentSubtitle] = useState<Subtitle | null>(null);
  
  useEffect(() => {
    const video = videoRef.current;
    if (!video || subtitles.length === 0) return;
    
    const updateSubtitle = () => {
      const currentTime = video.currentTime;
      const subtitle = subtitles.find(
        (sub) => currentTime >= sub.start && currentTime <= sub.end
      );
      setCurrentSubtitle(subtitle || null);
    };
    
    video.addEventListener('timeupdate', updateSubtitle);
    return () => video.removeEventListener('timeupdate', updateSubtitle);
  }, [subtitles]);
  
  return (
    <div className="relative w-full aspect-video bg-black rounded-lg">
      <video ref={videoRef} src={src} controls className="w-full h-full" />
      
      {currentSubtitle && (
        <div className="absolute bottom-16 left-0 right-0 text-center">
          <span className="inline-block px-4 py-2 bg-black/80 text-white rounded">
            {currentSubtitle.text}
          </span>
        </div>
      )}
    </div>
  );
}
```

---

## Step 4: Subtitle Editor với Auto-Save

```typescript
// src/hooks/useSubtitleSync.ts
export function useSubtitleSync({
  subtitles,
  onSave,
  debounceMs = 1000
}: UseSubtitleSyncProps) {
  const [localSubtitles, setLocalSubtitles] = useState(subtitles);
  const [isSaving, setIsSaving] = useState(false);
  const [lastSaved, setLastSaved] = useState<Date | null>(null);
  
  const debouncedSave = useRef(
    debounce(async (subs: Subtitle[]) => {
      setIsSaving(true);
      try {
        await onSave(subs);
        setLastSaved(new Date());
      } finally {
        setIsSaving(false);
      }
    }, debounceMs)
  ).current;
  
  const updateSubtitle = (id: string, text: string) => {
    setLocalSubtitles((prev) =>
      prev.map((sub) => (sub.id === id ? { ...sub, text } : sub))
    );
    debouncedSave(localSubtitles);
  };
  
  return { subtitles: localSubtitles, isSaving, lastSaved, updateSubtitle };
}
```

---

## Code References

| Component | File | Description |
|-----------|------|-------------|
| Video Store | `src/store/use-video-store.ts` | Video state management |
| Progress Hook | `src/hooks/useVideoProgress.ts` | WebSocket client |
| Video Player | `src/components/VideoPlayer.tsx` | Player với overlay |
| Subtitle Editor | `src/components/SubtitleEditor.tsx` | In-browser editing |

---

## Tóm tắt

Trong module này, bạn đã học:

- **Zustand Stores**: Video và authentication state management
- **WebSocket Client**: Real-time progress tracking với auto-reconnect
- **Video Player**: Components với subtitle overlay support
- **Auto-Save**: Debounced saving với visual feedback
- **Subtitle Editor**: In-browser editing với timing adjustment

---

## Bước tiếp theo

Tiếp tục đến [Workshop 5.6: Deployment](5.6-Deployment/) để learn about deploying the application to AWS.
