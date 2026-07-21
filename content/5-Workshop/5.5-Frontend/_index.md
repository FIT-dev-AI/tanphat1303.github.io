---
title: "Frontend Development"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

# Workshop 5.5: Frontend Development

---

## Learning Objectives

By the end of this module, you will:

- Understand the React component architecture
- Implement state management with Zustand
- Create WebSocket client for real-time updates
- Build video player with subtitle overlay
- Implement auto-save functionality
- Design the video editor interface

---

## Prerequisites

- Workshops 5.1-5.4 completed
- React and TypeScript knowledge
- Zustand state management basics
- WebSocket fundamentals

---

## Architecture Context

The frontend orchestrates user interactions and displays real-time processing status:

![Frontend Architecture](/images/5-Workshop/5.5-Frontend/Fr_end.png)

---

## Step 1: Project Structure

### Folder Organization

```
myapporc/src/
├── components/
│   ├── ui/              # Reusable UI components
│   ├── video/           # Video-specific components
│   ├── editor/          # Editor components
│   └── ...
├── hooks/               # Custom React hooks
├── interfaces/          # TypeScript interfaces
├── pages/
│   ├── auth/            # Login, Register pages
│   ├── dashboard/        # Video list page
│   └── editor/          # Video editor page
├── store/               # Zustand stores
├── lib/                 # Utilities
├── App.tsx
└── main.tsx
```

---

## Step 2: State Management with Zustand

### Video Store

```typescript
// src/store/use-video-store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface VideoItem {
  videoId: string;
  title: string;
  fileName: string;
  fileUrl: string;
  videoUrl: string;
  thumbnailUrl: string | null;
  status: VideoStatus;
  processingPercentage: number;
  voiceUsed: string | null;
  createdAt: string;
  updatedAt?: string;
}

type VideoStatus = 
  | 'PROCESSING'
  | 'AWAITING_REVIEW'
  | 'STAGE2_PROCESSING'
  | 'COMPLETED'
  | 'FAILED';

interface VideoStore {
  // State
  videos: VideoItem[];
  currentVideo: VideoItem | null;
  
  // Actions
  setVideos: (videos: VideoItem[]) => void;
  addVideo: (video: VideoItem) => void;
  updateVideo: (videoId: string, updates: Partial<VideoItem>) => void;
  removeVideo: (videoId: string) => void;
  setCurrentVideo: (video: VideoItem | null) => void;
  
  // Progress updates
  updateProgress: (videoId: string, percentage: number, status?: VideoStatus) => void;
}

export const useVideoStore = create<VideoStore>()(
  persist(
    (set, get) => ({
      videos: [],
      currentVideo: null,
      
      setVideos: (videos) => set({ videos }),
      
      addVideo: (video) => set((state) => ({
        videos: [video, ...state.videos]
      })),
      
      updateVideo: (videoId, updates) => set((state) => ({
        videos: state.videos.map((v) =>
          v.videoId === videoId ? { ...v, ...updates } : v
        ),
        currentVideo: state.currentVideo?.videoId === videoId
          ? { ...state.currentVideo, ...updates }
          : state.currentVideo
      })),
      
      removeVideo: (videoId) => set((state) => ({
        videos: state.videos.filter((v) => v.videoId !== videoId),
        currentVideo: state.currentVideo?.videoId === videoId
          ? null
          : state.currentVideo
      })),
      
      setCurrentVideo: (video) => set({ currentVideo: video }),
      
      updateProgress: (videoId, percentage, status) => {
        const updates: Partial<VideoItem> = { processingPercentage: percentage };
        if (status) updates.status = status;
        get().updateVideo(videoId, updates);
      }
    }),
    {
      name: 'video-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({ videos: state.videos })
    }
  )
);
```

### Auth Store

```typescript
// src/store/use-auth-store.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface User {
  id: number;
  email: string;
  username: string;
}

interface AuthStore {
  // State
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  
  // Actions
  setUser: (user: User) => void;
  setToken: (token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      
      setUser: (user) => set({ user, isAuthenticated: true }),
      
      setToken: (token) => set({ token }),
      
      logout: () => set({
        user: null,
        token: null,
        isAuthenticated: false
      })
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        user: state.user,
        token: state.token,
        isAuthenticated: state.isAuthenticated
      })
    }
  )
);
```

### Zustand State Flow

![Zustand State Flow](/images/5-Workshop/5.5-Frontend/Luong_state_Zustand.png)

---

## Step 3: WebSocket Client

### Progress Hook Implementation

```typescript
// src/hooks/useVideoProgress.ts
import { useEffect, useRef, useCallback } from 'react';
import { useVideoStore } from '@/store/use-video-store';

interface ProgressData {
  video_id: string;
  percentage: number;
  status: string;
  message: string;
  stage?: string;
  error_code?: string;
}

const WS_URL = import.meta.env.VITE_WS_URL || 'ws://localhost:8000';
const RECONNECT_DELAY = 3000;
const MAX_RECONNECT_ATTEMPTS = 5;

export function useVideoProgress(videoId: string) {
  const wsRef = useRef<WebSocket | null>(null);
  const reconnectAttempts = useRef(0);
  const reconnectTimeout = useRef<NodeJS.Timeout | null>(null);
  
  const updateProgress = useVideoStore((state) => state.updateProgress);
  
  const connect = useCallback(() => {
    // Skip if already connected
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      return;
    }
    
    const ws = new WebSocket(`${WS_URL}/ws/progress/${videoId}`);
    
    ws.onopen = () => {
      console.log('[WebSocket] Connected to progress stream');
      reconnectAttempts.current = 0;
    };
    
    ws.onmessage = (event) => {
      try {
        const data: ProgressData = JSON.parse(event.data);
        
        // Update store with progress
        updateProgress(
          data.video_id,
          data.percentage,
          data.status as any
        );
        
        console.log(`[Progress] ${data.percentage}% - ${data.message}`);
        
        // Trigger custom event for other components
        window.dispatchEvent(
          new CustomEvent('video-progress', { detail: data })
        );
        
      } catch (error) {
        console.error('[WebSocket] Failed to parse message:', error);
      }
    };
    
    ws.onerror = (error) => {
      console.error('[WebSocket] Error:', error);
    };
    
    ws.onclose = () => {
      console.log('[WebSocket] Connection closed');
      wsRef.current = null;
      
      // Attempt reconnection if not at max attempts
      if (reconnectAttempts.current < MAX_RECONNECT_ATTEMPTS) {
        reconnectAttempts.current++;
        console.log(
          `[WebSocket] Reconnecting in ${RECONNECT_DELAY}ms ` +
          `(attempt ${reconnectAttempts.current}/${MAX_RECONNECT_ATTEMPTS})`
        );
        
        reconnectTimeout.current = setTimeout(() => {
          connect();
        }, RECONNECT_DELAY);
      }
    };
    
    wsRef.current = ws;
  }, [videoId, updateProgress]);
  
  const disconnect = useCallback(() => {
    if (reconnectTimeout.current) {
      clearTimeout(reconnectTimeout.current);
    }
    
    if (wsRef.current) {
      wsRef.current.close();
      wsRef.current = null;
    }
  }, []);
  
  useEffect(() => {
    connect();
    
    return () => {
      disconnect();
    };
  }, [connect, disconnect]);
  
  return {
    disconnect,
    reconnect: connect
  };
}
```

---

## Step 4: Video Player Component

### Video Player with Subtitle Overlay

```typescript
// src/components/VideoPlayer.tsx
import { useRef, useState, useEffect } from 'react';

interface Subtitle {
  index: number;
  start: number;  // seconds
  end: number;    // seconds
  text: string;
}

interface VideoPlayerProps {
  src: string;
  subtitles?: Subtitle[];
  onTimeUpdate?: (currentTime: number) => void;
}

export function VideoPlayer({ 
  src, 
  subtitles = [],
  onTimeUpdate 
}: VideoPlayerProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [currentSubtitle, setCurrentSubtitle] = useState<Subtitle | null>(null);
  
  // Find current subtitle based on playback time
  useEffect(() => {
    const video = videoRef.current;
    if (!video || subtitles.length === 0) return;
    
    const updateSubtitle = () => {
      const currentTime = video.currentTime;
      
      const subtitle = subtitles.find(
        (sub) => currentTime >= sub.start && currentTime <= sub.end
      );
      
      setCurrentSubtitle(subtitle || null);
      
      if (onTimeUpdate) {
        onTimeUpdate(currentTime);
      }
    };
    
    video.addEventListener('timeupdate', updateSubtitle);
    
    return () => {
      video.removeEventListener('timeupdate', updateSubtitle);
    };
  }, [subtitles, onTimeUpdate]);
  
  return (
    <div className="relative w-full aspect-video bg-black rounded-lg overflow-hidden">
      {/* Video Element */}
      <video
        ref={videoRef}
        src={src}
        className="w-full h-full object-contain"
        controls
        playsInline
      />
      
      {/* Subtitle Overlay */}
      {currentSubtitle && (
        <div 
          className="absolute bottom-16 left-0 right-0 text-center pointer-events-none"
          aria-live="polite"
        >
          <span 
            className="inline-block px-4 py-2 bg-black/80 text-white text-xl rounded"
            style={{ maxWidth: '90%' }}
          >
            {currentSubtitle.text}
          </span>
        </div>
      )}
    </div>
  );
}
```

### Video Player with Hard Subtitle Overlay

```typescript
// src/components/VideoPlayerWithOverlay.tsx
import { useState, useRef } from 'react';

interface VideoPlayerWithOverlayProps {
  videoSrc: string;
  subtitleSrc?: string;
}

export function VideoPlayerWithOverlay({
  videoSrc,
  subtitleSrc
}: VideoPlayerWithOverlayProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [currentTime, setCurrentTime] = useState(0);
  const [duration, setDuration] = useState(0);
  const [showOverlay, setShowOverlay] = useState(true);
  
  // Parse SRT and convert to time-based lookup
  const subtitles = useMemo(() => {
    if (!subtitleSrc) return [];
    
    // SRT parsing logic here
    // Return array of { start, end, text } objects
  }, [subtitleSrc]);
  
  const currentSubtitle = useMemo(() => {
    return subtitles.find(
      (sub) => currentTime >= sub.start && currentTime <= sub.end
    );
  }, [subtitles, currentTime]);
  
  const handleTimeUpdate = () => {
    if (videoRef.current) {
      setCurrentTime(videoRef.current.currentTime);
    }
  };
  
  const handleLoadedMetadata = () => {
    if (videoRef.current) {
      setDuration(videoRef.current.duration);
    }
  };
  
  return (
    <div className="relative w-full">
      {/* Video Player */}
      <video
        ref={videoRef}
        src={videoSrc}
        controls
        onTimeUpdate={handleTimeUpdate}
        onLoadedMetadata={handleLoadedMetadata}
        className="w-full rounded-lg"
      />
      
      {/* Subtitle Overlay */}
      {showOverlay && currentSubtitle && (
        <div className="absolute bottom-12 left-0 right-0 flex justify-center">
          <div className="bg-black/80 text-white px-6 py-3 rounded-lg text-center max-w-3xl">
            <p className="text-lg leading-relaxed">
              {currentSubtitle.text}
            </p>
          </div>
        </div>
      )}
      
      {/* Toggle Overlay Button */}
      <button
        onClick={() => setShowOverlay(!showOverlay)}
        className="absolute top-4 right-4 bg-black/60 text-white px-3 py-1 rounded text-sm"
      >
        {showOverlay ? 'Hide Subtitles' : 'Show Subtitles'}
      </button>
    </div>
  );
}
```

---

## Step 5: Subtitle Editor with Auto-Save

### Editor Hook

```typescript
// src/hooks/useSubtitleSync.ts
import { useState, useEffect, useCallback, useRef } from 'react';
import { debounce } from '@/lib/utils';

interface Subtitle {
  id: string;
  start: number;
  end: number;
  text: string;
}

interface UseSubtitleSyncProps {
  subtitles: Subtitle[];
  videoId: string;
  onSave: (subtitles: Subtitle[]) => Promise<void>;
  debounceMs?: number;
}

export function useSubtitleSync({
  subtitles,
  videoId,
  onSave,
  debounceMs = 1000
}: UseSubtitleSyncProps) {
  const [localSubtitles, setLocalSubtitles] = useState<Subtitle[]>(subtitles);
  const [isSaving, setIsSaving] = useState(false);
  const [lastSaved, setLastSaved] = useState<Date | null>(null);
  const [hasChanges, setHasChanges] = useState(false);
  
  // Update local state when props change
  useEffect(() => {
    setLocalSubtitles(subtitles);
  }, [subtitles]);
  
  // Debounced save function
  const debouncedSave = useRef(
    debounce(async (subtitlesToSave: Subtitle[]) => {
      setIsSaving(true);
      try {
        await onSave(subtitlesToSave);
        setLastSaved(new Date());
        setHasChanges(false);
      } catch (error) {
        console.error('Failed to save subtitles:', error);
      } finally {
        setIsSaving(false);
      }
    }, debounceMs)
  ).current;
  
  // Update subtitle text
  const updateSubtitle = useCallback((id: string, text: string) => {
    setLocalSubtitles((prev) =>
      prev.map((sub) =>
        sub.id === id ? { ...sub, text } : sub
      )
    );
    setHasChanges(true);
    debouncedSave(localSubtitles);
  }, [debouncedSave, localSubtitles]);
  
  // Adjust subtitle timing
  const adjustSubtitle = useCallback((
    id: string,
    field: 'start' | 'end',
    delta: number
  ) => {
    setLocalSubtitles((prev) =>
      prev.map((sub) =>
        sub.id === id
          ? { ...sub, [field]: Math.max(0, sub[field] + delta) }
          : sub
      )
    );
    setHasChanges(true);
    debouncedSave(localSubtitles);
  }, [debouncedSave, localSubtitles]);
  
  // Delete subtitle
  const deleteSubtitle = useCallback((id: string) => {
    setLocalSubtitles((prev) => prev.filter((sub) => sub.id !== id));
    setHasChanges(true);
    debouncedSave(localSubtitles);
  }, [debouncedSave, localSubtitles]);
  
  // Add new subtitle
  const addSubtitle = useCallback((afterId: string) => {
    const index = localSubtitles.findIndex((s) => s.id === afterId);
    if (index === -1) return;
    
    const current = localSubtitles[index];
    const next = localSubtitles[index + 1];
    
    const newSubtitle: Subtitle = {
      id: `sub_${Date.now()}`,
      start: current.end,
      end: next ? next.start : current.end + 3,
      text: ''
    };
    
    const newSubtitles = [
      ...localSubtitles.slice(0, index + 1),
      newSubtitle,
      ...localSubtitles.slice(index + 1)
    ];
    
    setLocalSubtitles(newSubtitles);
    setHasChanges(true);
    debouncedSave(newSubtitles);
  }, [localSubtitles, debouncedSave]);
  
  return {
    subtitles: localSubtitles,
    isSaving,
    lastSaved,
    hasChanges,
    updateSubtitle,
    adjustSubtitle,
    deleteSubtitle,
    addSubtitle
  };
}
```

### Subtitle Editor Component

```typescript
// src/components/SubtitleEditor.tsx
import { useState } from 'react';
import { useSubtitleSync } from '@/hooks/useSubtitleSync';
import { formatTime } from '@/lib/utils';

interface SubtitleEditorProps {
  videoId: string;
  subtitles: Subtitle[];
  onSave: (subtitles: Subtitle[]) => Promise<void>;
}

export function SubtitleEditor({ 
  videoId, 
  subtitles,
  onSave 
}: SubtitleEditorProps) {
  const {
    subtitles: localSubtitles,
    isSaving,
    lastSaved,
    hasChanges,
    updateSubtitle,
    adjustSubtitle,
    deleteSubtitle
  } = useSubtitleSync({ subtitles, videoId, onSave });
  
  const [selectedId, setSelectedId] = useState<string | null>(null);
  
  return (
    <div className="flex flex-col h-full">
      {/* Header with save status */}
      <div className="flex items-center justify-between px-4 py-3 border-b">
        <h2 className="text-lg font-semibold">Subtitle Editor</h2>
        
        <div className="flex items-center gap-4">
          {/* Save indicator */}
          <div className="text-sm text-gray-500">
            {isSaving ? (
              <span className="flex items-center gap-2">
                <span className="w-2 h-2 bg-yellow-500 rounded-full animate-pulse" />
                Saving...
              </span>
            ) : lastSaved ? (
              <span className="flex items-center gap-2">
                <span className="w-2 h-2 bg-green-500 rounded-full" />
                Saved {formatTime(lastSaved.getTime())}
              </span>
            ) : hasChanges ? (
              <span className="flex items-center gap-2">
                <span className="w-2 h-2 bg-gray-400 rounded-full" />
                Unsaved changes
              </span>
            ) : null}
          </div>
        </div>
      </div>
      
      {/* Subtitle list */}
      <div className="flex-1 overflow-y-auto">
        {localSubtitles.map((sub, index) => (
          <div
            key={sub.id}
            className={`px-4 py-3 border-b ${
              selectedId === sub.id ? 'bg-blue-50' : 'hover:bg-gray-50'
            }`}
            onClick={() => setSelectedId(sub.id)}
          >
            {/* Time display */}
            <div className="flex items-center gap-2 text-sm text-gray-600 mb-2">
              <span className="font-mono">
                {formatTime(sub.start)} → {formatTime(sub.end)}
              </span>
              <span className="text-gray-400">#{index + 1}</span>
            </div>
            
            {/* Text input */}
            <textarea
              value={sub.text}
              onChange={(e) => updateSubtitle(sub.id, e.target.value)}
              className="w-full px-3 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
              rows={2}
              placeholder="Enter subtitle text..."
            />
            
            {/* Time adjustment buttons */}
            <div className="flex items-center gap-2 mt-2">
              <button
                onClick={(e) => {
                  e.stopPropagation();
                  adjustSubtitle(sub.id, 'start', -0.1);
                }}
                className="px-2 py-1 text-xs bg-gray-100 rounded hover:bg-gray-200"
              >
                -0.1s
              </button>
              <button
                onClick={(e) => {
                  e.stopPropagation();
                  adjustSubtitle(sub.id, 'start', 0.1);
                }}
                className="px-2 py-1 text-xs bg-gray-100 rounded hover:bg-gray-200"
              >
                +0.1s
              </button>
              <button
                onClick={(e) => {
                  e.stopPropagation();
                  deleteSubtitle(sub.id);
                }}
                className="px-2 py-1 text-xs text-red-600 bg-red-50 rounded hover:bg-red-100 ml-auto"
              >
                Delete
              </button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 6: Video Upload Component

```typescript
// src/components/VideoUploader.tsx
import { useState, useCallback } from 'react';
import { useVideoStore } from '@/store/use-video-store';
import { useAuthStore } from '@/store/use-auth-store';
import axios from 'axios';

export function VideoUploader() {
  const [isUploading, setIsUploading] = useState(false);
  const [uploadProgress, setUploadProgress] = useState(0);
  const [dragActive, setDragActive] = useState(false);
  
  const addVideo = useVideoStore((state) => state.addVideo);
  const token = useAuthStore((state) => state.token);
  
  const handleUpload = useCallback(async (file: File) => {
    if (!file.type.startsWith('video/')) {
      alert('Please upload a video file');
      return;
    }
    
    setIsUploading(true);
    setUploadProgress(0);
    
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      const response = await axios.post(
        `${import.meta.env.VITE_API_URL}/api/v1/videos/upload`,
        formData,
        {
          headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'multipart/form-data'
          },
          onUploadProgress: (progressEvent) => {
            const progress = Math.round(
              (progressEvent.loaded * 100) / (progressEvent.total || 1)
            );
            setUploadProgress(progress);
          }
        }
      );
      
      // Add to store
      addVideo({
        videoId: response.data.video_id,
        title: response.data.title,
        fileName: response.data.file_name,
        fileUrl: response.data.file_url,
        videoUrl: response.data.file_url,
        thumbnailUrl: null,
        status: 'PROCESSING',
        processingPercentage: 0,
        voiceUsed: null,
        createdAt: new Date().toISOString()
      });
      
      alert('Video uploaded successfully! Processing will begin shortly.');
      
    } catch (error) {
      console.error('Upload failed:', error);
      alert('Upload failed. Please try again.');
    } finally {
      setIsUploading(false);
    }
  }, [token, addVideo]);
  
  const handleDrop = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    setDragActive(false);
    
    const files = e.dataTransfer.files;
    if (files.length > 0) {
      handleUpload(files[0]);
    }
  }, [handleUpload]);
  
  return (
    <div
      className={`border-2 border-dashed rounded-lg p-8 text-center transition-colors ${
        dragActive ? 'border-blue-500 bg-blue-50' : 'border-gray-300'
      }`}
      onDragOver={(e) => {
        e.preventDefault();
        setDragActive(true);
      }}
      onDragLeave={() => setDragActive(false)}
      onDrop={handleDrop}
    >
      <input
        type="file"
        accept="video/*"
        onChange={(e) => {
          if (e.target.files?.[0]) {
            handleUpload(e.target.files[0]);
          }
        }}
        className="hidden"
        id="video-upload"
      />
      
      <label htmlFor="video-upload" className="cursor-pointer">
        <div className="flex flex-col items-center gap-4">
          <div className="w-16 h-16 bg-blue-100 rounded-full flex items-center justify-center">
            <UploadIcon className="w-8 h-8 text-blue-600" />
          </div>
          
          <div>
            <p className="text-lg font-medium text-gray-700">
              Drop your video here or click to upload
            </p>
            <p className="text-sm text-gray-500 mt-1">
              Supported formats: MP4, MOV, AVI, MKV
            </p>
          </div>
          
          {isUploading && (
            <div className="w-full max-w-xs mt-4">
              <div className="h-2 bg-gray-200 rounded-full overflow-hidden">
                <div
                  className="h-full bg-blue-600 transition-all"
                  style={{ width: `${uploadProgress}%` }}
                />
              </div>
              <p className="text-sm text-gray-500 mt-2">
                Uploading... {uploadProgress}%
              </p>
            </div>
          )}
        </div>
      </label>
    </div>
  );
}
```

---

## Code References

| Component | File | Description |
|-----------|------|-------------|
| Video Store | `src/store/use-video-store.ts` | Video state management |
| Auth Store | `src/store/use-auth-store.ts` | Authentication state |
| Progress Hook | `src/hooks/useVideoProgress.ts` | WebSocket client |
| Subtitle Sync | `src/hooks/useSubtitleSync.ts` | Auto-save logic |
| Video Player | `src/components/VideoPlayer.tsx` | Basic player |
| Player Overlay | `src/components/VideoPlayerWithOverlay.tsx` | Subtitle display |
| Subtitle Editor | `src/components/SubtitleEditor.tsx` | In-browser editing |
| Video Uploader | `src/components/VideoUploader.tsx` | Upload interface |

---

## Summary

In this module, you learned:

- **Zustand Stores**: Video and authentication state management
- **WebSocket Client**: Real-time progress tracking with auto-reconnect
- **Video Player**: Components with subtitle overlay support
- **Auto-Save**: Debounced saving with visual feedback
- **Subtitle Editor**: In-browser editing with timing adjustment
- **Upload Component**: Drag-and-drop video upload

---

## Next Steps

Proceed to [Workshop 5.6: Deployment](5.6-Deployment/) to learn about deploying the application to AWS.

---

## Validation

Test the frontend:

```bash
# Start development server
cd myapporc
npm run dev

# Check for TypeScript errors
npm run type-check

# Build for production
npm run build
```
