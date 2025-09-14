# Camera Module

> Capture photos and stream video from MentraOS smart glasses. Learn managed vs unmanaged streaming, quick starts for photo capture and RTMP, permissions, and API references.

# Camera Module Documentation

The Camera Module provides comprehensive camera functionality for MentraOS apps, including photo capture and video streaming capabilities.

## Available Features

### [Photo Capture](./photo-capture.md)

Take high-quality photos from smart glasses with options for gallery saving and raw buffer access.

**Key capabilities:**

* Request individual photos on demand
* Save to device gallery
* Access raw photo data for processing
* Handle timeouts and errors gracefully

### [RTMP Streaming](./rtmp-streaming.md)

Stream live video from smart glasses with two powerful options:

**Managed Streaming (Recommended)**

* Zero infrastructure required
* Automatic HLS/DASH URL generation
* Multi-app support - multiple apps can access the same stream
* Optional low-latency WebRTC URL
* Optional re-streaming fan-out to RTMP destinations (YouTube, X, Twitch, etc.)

**Unmanaged Streaming**

* Full control over RTMP endpoints
* Exclusive camera access
* Custom server integration
* Ultra-low latency options

## Quick Start

### Taking a Photo

```typescript
const photo = await session.camera.requestPhoto({ saveToGallery: true });
console.log(`Photo captured: ${photo.mimeType}, ${photo.size} bytes`);
```

### Starting a Managed Stream (Easy Mode)

```typescript
// Start streaming with zero configuration
const result = await session.camera.startManagedStream();
console.log('Share this URL with viewers:', result.hlsUrl);
```

### Starting an Unmanaged Stream (Full Control)

```typescript
await session.camera.startStream({
  rtmpUrl: 'rtmp://your-server.com/live/stream-key'
});
```

### Re-streaming to social platforms (YouTube, X, Twitch)

Managed streaming can fan-out to RTMP destinations you specify. Provide one or more `restreamDestinations` with platform RTMP ingest URLs (including your stream keys).

```typescript
// Example: Start a managed stream and re-stream to YouTube and Twitch
const result = await session.camera.startManagedStream({
  restreamDestinations: [
    { url: 'rtmp://a.rtmp.youtube.com/live2/YOUR_YOUTUBE_STREAM_KEY', name: 'YouTube' },
    { url: 'rtmp://live.twitch.tv/app/YOUR_TWITCH_STREAM_KEY', name: 'Twitch' }
  ]
});

// Wait for managed status 'active' before sharing URLs
session.camera.onManagedStreamStatus((status) => {
  if (status.status === 'active') {
    console.log('Viewer URLs:', status.hlsUrl, status.dashUrl, status.webrtcUrl);
  }
});
```

Notes:

* Each platform provides an RTMP ingest URL and stream key; combine them into the `url` value.
* Managed playback URLs (HLS/DASH/WebRTC) are separate from platform destinations and should be shared only after the managed stream status is `active`.

## Documentation Structure

* **[Photo Capture Guide](./photo-capture.md)** - Complete guide for taking photos
* **[RTMP Streaming Guide](./rtmp-streaming.md)** - Comprehensive streaming documentation covering both managed and unmanaged options
* **[API Reference](/reference/managers/camera)** - Detailed API documentation for all camera methods

## Common Use Cases

### Social Media Streaming

Use managed streaming for easy integration with platforms like YouTube Live, X (Twitter), and Twitch:

```typescript
const stream = await session.camera.startManagedStream({
  restreamDestinations: [
    { url: 'rtmp://a.rtmp.youtube.com/live2/YOUR_KEY' }
  ]
});
```

### Security Camera App

Use unmanaged streaming for full control and local network streaming:

```typescript
await session.camera.startStream({
  rtmpUrl: 'rtmp://192.168.1.100/security/cam1'
});
```

### Photo Documentation App

Capture and save photos for documentation:

```typescript
const photo = await session.camera.requestPhoto({
  saveToGallery: true
});
await uploadToCloudStorage(photo.buffer);
```

## Important Notes

* **Permissions**: Camera access requires the `CAMERA` permission in your app manifest. See [Permissions Guide](/permissions) for setup instructions.
* **Hardware**: Only available on camera-equipped glasses (e.g., Mentra Live)
* **Battery**: Extended streaming can drain battery quickly
* **Privacy**: Always notify users when camera is active

## See Also

* [Permissions Guide](/permissions) - Setting up camera permissions
* [Events Documentation](/events) - Handling camera-related events
* [API Reference](/reference/managers/camera) - Complete API documentation
