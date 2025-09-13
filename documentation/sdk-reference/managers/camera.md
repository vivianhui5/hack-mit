# null

# Camera Module Reference

The Camera Module provides comprehensive camera functionality, including photo capture and both managed and unmanaged RTMP streaming capabilities. It allows apps to request photos from connected smart glasses and stream live camera feeds using two different approaches.

## Overview

The Camera Module is part of the App Session and provides three main capabilities:

* **Photo Capture**: Request individual photos from smart glasses
* **Managed Streaming**: Zero-infrastructure streaming with automatic HLS/DASH URLs
* **Unmanaged Streaming**: Full-control streaming to custom RTMP endpoints

Access the camera module through your app session:

```typescript
const photo = await session.camera.requestPhoto();
const stream = await session.camera.startManagedStream();
await session.camera.startStream({ rtmpUrl: 'rtmp://example.com/live/key' });
```

## Photo Functionality

### requestPhoto()

Request a photo from the connected smart glasses.

```typescript
async requestPhoto(options?: PhotoRequestOptions): Promise<PhotoData>
```

#### Parameters

| Parameter | Type                  | Description                                  |
| --------- | --------------------- | -------------------------------------------- |
| `options` | `PhotoRequestOptions` | Optional configuration for the photo request |

#### PhotoRequestOptions

```typescript
interface PhotoRequestOptions {
  /** Whether to save the photo to the device gallery */
  saveToGallery?: boolean;
}
```

#### Returns

Returns a `Promise<PhotoData>` that resolves with the captured photo data.

#### PhotoData Interface

```typescript
interface PhotoData {
  /** The actual photo file as a Buffer */
  buffer: Buffer;
  /** MIME type of the photo (e.g., 'image/jpeg') */
  mimeType: string;
  /** Original filename from the camera */
  filename: string;
  /** Unique request ID that correlates to the original request */
  requestId: string;
  /** Size of the photo in bytes */
  size: number;
  /** Timestamp when the photo was captured */
  timestamp: Date;
}
```

#### Example

```typescript
// Basic photo request
const photo = await session.camera.requestPhoto();
console.log(`Photo taken at timestamp: ${photo.timestamp}`);
console.log(`MIME type: ${photo.mimeType}, size: ${photo.size} bytes`);

// Access raw photo data
const photoBuffer = photo.buffer;
const base64Photo = photo.buffer.toString('base64');

// Save to gallery
const photoWithSave = await session.camera.requestPhoto({
  saveToGallery: true
});
```

## Managed Streaming Functionality (Recommended)

Managed streaming provides zero-infrastructure streaming where the cloud handles all RTMP endpoints and returns HLS/DASH URLs for viewing. Multiple apps can access the same managed stream simultaneously.

### startManagedStream()

Start a managed stream with automatic URL generation.

```typescript
async startManagedStream(options?: ManagedStreamOptions): Promise<ManagedStreamResult>
```

#### Parameters

| Parameter | Type                   | Description                                   |
| --------- | ---------------------- | --------------------------------------------- |
| `options` | `ManagedStreamOptions` | Optional configuration for the managed stream |

#### ManagedStreamOptions

```typescript
interface ManagedStreamOptions {
  /** Stream quality preset */
  quality?: '720p' | '1080p';
  /** Enable WebRTC for ultra-low latency viewing */
  enableWebRTC?: boolean;
  /** Optional video configuration settings */
  video?: VideoConfig;
  /** Optional audio configuration settings */
  audio?: AudioConfig;
  /** Optional stream configuration settings */
  stream?: StreamConfig;
  /** Optional RTMP destinations to re-stream to (e.g., YouTube, Twitch) */
  restreamDestinations?: { url: string; name?: string }[];
}
```

#### ManagedStreamResult

```typescript
interface ManagedStreamResult {
  /** HLS URL for viewing the stream */
  hlsUrl: string;
  /** DASH URL for viewing the stream */
  dashUrl: string;
  /** WebRTC URL if enabled */
  webrtcUrl?: string;
  /** Internal stream ID */
  streamId: string;
  /** Cloud-hosted preview/player URL (for quick embedding and testing) */
  previewUrl?: string;
  /** Thumbnail image URL for previews */
  thumbnailUrl?: string;
}
```

#### Example

```typescript
// Basic managed streaming
const result = await session.camera.startManagedStream();
console.log('HLS URL:', result.hlsUrl);
console.log('DASH URL:', result.dashUrl);

// Advanced configuration with WebRTC
const result = await session.camera.startManagedStream({
  quality: '1080p',
  enableWebRTC: true,
  video: { frameRate: 30 },
  audio: { sampleRate: 48000 }
});
console.log('WebRTC URL:', result.webrtcUrl); // Low latency option
```

### stopManagedStream()

Stop the current managed stream.

```typescript
async stopManagedStream(): Promise<void>
```

#### Example

```typescript
await session.camera.stopManagedStream();
```

### checkExistingStream()

Check if there's an existing active stream (managed or unmanaged) for the current user.

```typescript
async checkExistingStream(): Promise<StreamCheckResult>
```

#### Returns

Returns a `Promise<StreamCheckResult>` with information about any active stream.

#### StreamCheckResult Interface

```typescript
interface StreamCheckResult {
  /** Whether any stream is currently active */
  hasActiveStream: boolean;
  /** Details about the active stream (if any) */
  streamInfo?: {
    /** Type of stream - either 'managed' or 'unmanaged' */
    type: 'managed' | 'unmanaged';
    /** Unique identifier for the stream */
    streamId: string;
    /** Current status of the stream */
    status: string;
    /** When the stream was created */
    createdAt: Date;
    // For managed streams only:
    /** HLS playback URL */
    hlsUrl?: string;
    /** DASH playback URL */
    dashUrl?: string;
    /** WebRTC playback URL (if enabled) */
    webrtcUrl?: string;
    /** Preview/embed URL */
    previewUrl?: string;
    /** Thumbnail image URL */
    thumbnailUrl?: string;
    /** Number of active viewers */
    activeViewers?: number;
    // For unmanaged streams only:
    /** RTMP destination URL */
    rtmpUrl?: string;
    /** ID of the app that requested the stream */
    requestingAppId?: string;
  };
}
```

#### Example

```typescript
// Check for existing streams before starting a new one
const streamCheck = await session.camera.checkExistingStream();

if (streamCheck.hasActiveStream) {
  console.log(`Found existing ${streamCheck.streamInfo?.type} stream`);

  if (streamCheck.streamInfo?.type === 'managed') {
    // Managed stream is active - can display URLs or join it
    console.log('HLS URL:', streamCheck.streamInfo.hlsUrl);
    console.log('Active viewers:', streamCheck.streamInfo.activeViewers);

    // Option 1: Display the existing stream info
    session.layouts.showTextWall(`Stream active.\nHLS: ${streamCheck.streamInfo.hlsUrl}`);

    // Option 2: Join the existing managed stream
    // (Managed streams support multiple apps)
    const result = await session.camera.startManagedStream();
  } else {
    // Unmanaged stream is active - cannot start another
    console.log('RTMP URL:', streamCheck.streamInfo.rtmpUrl);
    console.log('Requested by:', streamCheck.streamInfo.requestingAppId);

    session.layouts.showTextWall('⚠️ Another app is currently streaming');
  }
} else {
  // No active stream, safe to start a new one
  console.log('No existing stream, starting new...');
  const result = await session.camera.startManagedStream();
}
```

#### Use Cases

* **Reconnection**: Apps can check for and reconnect to existing streams after restart
* **Multi-app coordination**: Detect if another app is already streaming
* **Resource management**: Avoid creating duplicate streams
* **User experience**: Inform users about existing streams before starting new ones

### Managed Stream Status Monitoring

#### onManagedStreamStatus()

Subscribe to managed stream status updates.

```typescript
onManagedStreamStatus(handler: (status: ManagedStreamStatus) => void): () => void
```

#### ManagedStreamStatus

```typescript
interface ManagedStreamStatus {
  type: string;
  status: 'initializing' | 'preparing' | 'active' | 'stopping' | 'stopped' | 'error';
  hlsUrl?: string;
  dashUrl?: string;
  webrtcUrl?: string;
  message?: string;
  streamId?: string;
  timestamp: Date;
}
```

#### Example

```typescript
// Monitor managed stream status
const unsubscribe = session.camera.onManagedStreamStatus((status) => {
  console.log(`Managed stream status: ${status.status}`);

  if (status.status === 'active') {
    console.log('Stream is live!');
    console.log(`HLS URL: ${status.hlsUrl}`);
    console.log(`DASH URL: ${status.dashUrl}`);
  } else if (status.status === 'error') {
    console.error(`Stream error: ${status.message}`);
  }
});

// Later, unsubscribe
unsubscribe();
```

## Unmanaged RTMP Streaming Functionality

Unmanaged streaming provides full control over RTMP endpoints but requires your own streaming infrastructure. Only one unmanaged stream can be active at a time, and it blocks other apps from using the camera.

### startStream()

Start an unmanaged RTMP stream to a specific URL.

```typescript
async startStream(options: RtmpStreamOptions): Promise<void>
```

#### Parameters

| Parameter | Type                | Description                          |
| --------- | ------------------- | ------------------------------------ |
| `options` | `RtmpStreamOptions` | Configuration options for the stream |

#### RtmpStreamOptions

```typescript
interface RtmpStreamOptions {
  /** The RTMP URL to stream to (e.g., rtmp://server.example.com/live/stream-key) */
  rtmpUrl: string;
  /** Optional video configuration settings */
  video?: VideoConfig;
  /** Optional audio configuration settings */
  audio?: AudioConfig;
  /** Optional stream configuration settings */
  stream?: StreamConfig;
}
```

#### VideoConfig

```typescript
interface VideoConfig {
  /** Optional width in pixels (e.g., 1280) */
  width?: number;
  /** Optional height in pixels (e.g., 720) */
  height?: number;
  /** Optional bitrate in bits per second (e.g., 2000000 for 2 Mbps) */
  bitrate?: number;
  /** Optional frame rate in frames per second (e.g., 30) */
  frameRate?: number;
}
```

#### AudioConfig

```typescript
interface AudioConfig {
  /** Optional audio bitrate in bits per second (e.g., 128000 for 128 kbps) */
  bitrate?: number;
  /** Optional audio sample rate in Hz (e.g., 44100) */
  sampleRate?: number;
  /** Optional flag to enable echo cancellation */
  echoCancellation?: boolean;
  /** Optional flag to enable noise suppression */
  noiseSuppression?: boolean;
}
```

#### StreamConfig

```typescript
interface StreamConfig {
  /** Optional maximum duration in seconds (e.g., 1800 for 30 minutes) */
  durationLimit?: number;
}
```

#### Example

```typescript
// Basic streaming
await session.camera.startStream({
  rtmpUrl: 'rtmp://live.example.com/stream/key'
});

// Advanced configuration
await session.camera.startStream({
  rtmpUrl: 'rtmp://live.example.com/stream/key',
  video: {
    width: 1920,
    height: 1080,
    bitrate: 5000000,
    frameRate: 30
  },
  audio: {
    bitrate: 128000,
    sampleRate: 44100,
    echoCancellation: true,
    noiseSuppression: true
  },
  stream: {
    durationLimit: 1800 // 30 minutes
  }
});
```

### stopStream()

Stop the current unmanaged RTMP stream.

```typescript
async stopStream(): Promise<void>
```

#### Example

```typescript
await session.camera.stopStream();
```

### Unmanaged Stream Status Monitoring

#### onStreamStatus()

Subscribe to unmanaged stream status updates.

```typescript
onStreamStatus(handler: StreamStatusHandler): () => void
```

#### StreamStatusHandler

```typescript
type StreamStatusHandler = (status: RtmpStreamStatus) => void;
```

#### RtmpStreamStatus

```typescript
interface RtmpStreamStatus {
  type: string;
  streamId?: string;
  status: 'initializing' | 'connecting' | 'reconnecting' | 'streaming' | 'error' | 'stopped' | 'active' | 'stopping' | 'disconnected' | 'timeout';
  errorDetails?: string;
  appId?: string;
  stats?: {
    bitrate: number;
    fps: number;
    droppedFrames: number;
    duration: number;
  };
  timestamp: Date;
}
```

#### Example

```typescript
// Monitor unmanaged stream status
const unsubscribe = session.camera.onStreamStatus((status) => {
  console.log(`Stream status: ${status.status}`);

  if (status.status === 'active') {
    console.log('Stream is live!');
    if (status.stats) {
      console.log(`Bitrate: ${status.stats.bitrate} bps`);
      console.log(`FPS: ${status.stats.fps}`);
      console.log(`Duration: ${status.stats.duration}s`);
      console.log(`Dropped frames: ${status.stats.droppedFrames}`);
    }
  } else if (status.status === 'error') {
    console.error(`Stream error: ${status.errorDetails}`);
  }
});

// Later, unsubscribe
unsubscribe();
```

### Unmanaged Stream Utility Methods

#### isCurrentlyStreaming()

Check if currently streaming (unmanaged).

```typescript
isCurrentlyStreaming(): boolean
```

#### getCurrentStreamUrl()

Get the URL of the current unmanaged stream.

```typescript
getCurrentStreamUrl(): string | undefined
```

#### getStreamStatus()

Get the current unmanaged stream status.

```typescript
getStreamStatus(): RtmpStreamStatus | undefined
```

## Streaming Comparison

| Feature                      | Managed Streaming                             | Unmanaged Streaming              |
| ---------------------------- | --------------------------------------------- | -------------------------------- |
| **Infrastructure Required**  | None                                          | RTMP Server                      |
| **Multiple Apps Can Stream** | ✅ Yes                                         | ❌ No (Exclusive)                 |
| **Blocks Other Apps**        | ❌ No                                          | ✅ Yes                            |
| **Viewer URLs Provided**     | ✅ HLS/DASH/WebRTC                             | ❌ You manage                     |
| **Best For**                 | Social media, prototypes, multi-app scenarios | Custom servers, exclusive access |
| **Requires Internet**        | Yes                                           | No                               |

## Error Handling

### Photo Errors

* **Timeout**: Photo requests timeout after 30 seconds
* **Cancellation**: Requests can be cancelled manually or during session cleanup
* **Device Errors**: Camera unavailable or hardware issues

### Unmanaged Stream Errors

* **Already Streaming**: Cannot start while any stream (managed or unmanaged) is active
* **Invalid URL**: RTMP URL validation failures
* **Network Issues**: Connection problems to RTMP endpoint
* **Device Limitations**: Hardware doesn't support requested configuration
