# null

# RTMP Streaming

MentraOS supports two streaming modes: managed streaming (cloud-orchestrated) and unmanaged streaming (direct RTMP to your endpoint). This document provides a concise reference and guidance for choosing and integrating each mode.

## Types of streaming

### Managed Streaming

Managed streaming delegates ingest and playback to the MentraOS cloud. The SDK requests a stream, and the cloud returns viewer URLs (HLS/DASH, optional WebRTC). Multiple apps can consume the same managed stream concurrently.

* Requires internet connectivity
* Non-exclusive camera access (cooperative with other MentraOS apps)
* Playback URLs become usable when status is "active"

#### Reference

* Method: `session.camera.startManagedStream(options?: ManagedStreamOptions): Promise<ManagedStreamResult>`
* Stop: `session.camera.stopManagedStream(): Promise<void>`
* Status events: `session.camera.onManagedStreamStatus(handler)`
* Status stream type: `StreamType.MANAGED_STREAM_STATUS`

Types:

```typescript
// Options for starting a managed stream
interface ManagedStreamOptions {
  quality?: "720p" | "1080p";
  enableWebRTC?: boolean;
  restreamDestinations?: { url: string; name?: string }[]; // optional RTMP fan-out
}

// Result returned when managed stream URLs are ready
interface ManagedStreamResult {
  hlsUrl: string;
  dashUrl: string;
  webrtcUrl?: string;
  previewUrl?: string;
  thumbnailUrl?: string;
  streamId: string;
}

// Managed stream status messages
// type: CloudToAppMessageType.MANAGED_STREAM_STATUS
// status: "initializing" | "preparing" | "active" | "stopping" | "stopped" | "error"
```

Managed stream URLs:

* hlsUrl: HTTP Live Streaming (HLS) manifest. Broadest player support on web and mobile; recommended default for viewers.
* dashUrl: MPEG-DASH manifest. Alternative adaptive streaming format for DASH-capable players/environments.
* webrtcUrl: Low-latency playback endpoint (available when `enableWebRTC` is true). Use for near real-time viewing with compatible players.
* previewUrl: Hosted player page suitable for quick testing and simple embedding. Useful for rapid validation without integrating a custom player.
* thumbnailUrl: Static/periodic thumbnail image for previews and UI cards; not a video stream.

Usage:

```typescript
// Subscribe to status BEFORE starting
const unsubscribe = session.camera.onManagedStreamStatus((status) => {
  if (status.status === "active") {
    // URLs are now ready for viewers
    console.log("HLS:", status.hlsUrl);
    console.log("DASH:", status.dashUrl);
    if (status.webrtcUrl) console.log("WebRTC:", status.webrtcUrl);
  } else if (status.status === "error") {
    console.error("Managed stream error:", status.message);
  }
});

// Start managed stream
const urls = await session.camera.startManagedStream({
  quality: "720p",
  enableWebRTC: true,
});

// Note: urls are returned immediately, but viewers should connect only after
// a status event reports status === "active".

// Stop when done
await session.camera.stopManagedStream();

// Cleanup status listener
unsubscribe();
```

Status handling guidelines:

* Do not present or share URLs to viewers until a status event with `status === "active"` is received.
* Treat `error` as non-recoverable for the current attempt. You may retry by calling `startManagedStream()` again.

### Unmanaged Streaming

Unmanaged streaming establishes a direct RTMP connection from the device to your RTMP endpoint. You control ingest, transcoding, and distribution.

* Works with local or custom RTMP servers
* Exclusive camera access while active
* You manage endpoint availability, retries, and distribution

#### Reference

* Method: `session.camera.startStream(options: RtmpStreamOptions): Promise<void>`
* Stop: `session.camera.stopStream(): Promise<void>`
* Status events: `session.camera.onStreamStatus(handler)`
* Status stream type: `StreamType.RTMP_STREAM_STATUS`

Types (from SDK):

```typescript
interface RtmpStreamOptions {
  rtmpUrl: string;        // e.g., rtmp://your-server.com/live/key
}
```

Usage:

```typescript
const cleanup = session.camera.onStreamStatus((status) => {
  console.log("RTMP status:", status.status);
  if (status.status === "active") {
    console.log("Streaming to:", session.camera.getCurrentStreamUrl());
  }
  if (status.status === "error") {
    console.error(status.errorDetails);
  }
});

await session.camera.startStream({
  rtmpUrl: "rtmp://your-server.com/live/stream-key",
});

// ... later
await session.camera.stopStream();
cleanup();
```

## When to use each

* Managed streaming: use when you need turnkey ingest and globally accessible playback URLs (HLS/DASH; optional WebRTC), want cooperative camera access, or want cloud-managed resilience and URL lifecycle.
* Unmanaged streaming: use when you must stream to a custom/local RTMP endpoint, require full control of ingest and distribution, or operate without the MentraOS managed pipeline.

Permissions

* All streaming requires the CAMERA permission in your app configuration.

Operational notes

* Only one unmanaged stream can run at a time per device and it blocks managed streams while active.
* Managed streams allow multiple apps to consume the same device feed.

## Checking for existing streams

Use `session.camera.checkExistingStream()` to detect if any stream is already active for the current user (managed or unmanaged). This is useful for reconnection, avoiding duplicate streams, and coordinating across multiple apps.

Signature:

```typescript
async checkExistingStream(): Promise<{
  hasActiveStream: boolean;
  streamInfo?: {
    type: "managed" | "unmanaged";
  streamId: string;
    status: string;
    createdAt: Date;
    // For managed streams
    hlsUrl?: string;
    dashUrl?: string;
    webrtcUrl?: string;
    previewUrl?: string;
    thumbnailUrl?: string;
    activeViewers?: number;
    // For unmanaged streams
    rtmpUrl?: string;
    requestingAppId?: string;
  };
}>
```

Example usage:

```typescript
const result = await session.camera.checkExistingStream();

if (!result.hasActiveStream) {
  // No active stream; safe to start one
  await session.camera.startManagedStream();
  // Or: await session.camera.startStream({ rtmpUrl: "rtmp://..." });
  return;
}

if (result.streamInfo?.type === "managed") {
  // Reuse existing managed stream URLs
  console.log("HLS:", result.streamInfo.hlsUrl);
  console.log("DASH:", result.streamInfo.dashUrl);
  if (result.streamInfo.webrtcUrl) console.log("WebRTC:", result.streamInfo.webrtcUrl);
} else {
  // Unmanaged stream is active; do not start another
  console.log("Active RTMP:", result.streamInfo?.rtmpUrl);
  console.log("Requesting app:", result.streamInfo?.requestingAppId);
}
```

Notes:

* Managed streams support multiple apps; calling `startManagedStream()` will join the existing managed stream for the user.
* Unmanaged streams are exclusive and block other streams until stopped.

## FAQ

* How do I know when managed stream URLs are usable?
  * Subscribe to `onManagedStreamStatus` and wait for `status === "active"` before sharing or embedding.

* Can I re-stream a managed stream to RTMP destinations like YouTube or Twitch?
  * Yes. Provide `restreamDestinations` in `ManagedStreamOptions`.

* How do I check for an existing active stream after an app restart?
  * Call `session.camera.checkExistingStream()` and inspect `streamInfo`.

* Does managed streaming require internet access?
  * Yes. Unmanaged can work on local networks depending on your RTMP server reachability.

* Can I enable low-latency viewing?
  * Set `enableWebRTC: true` in `ManagedStreamOptions`. Use `webrtcUrl` from the result/status for playback.
