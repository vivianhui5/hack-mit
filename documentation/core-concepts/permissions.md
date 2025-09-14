# Permissions

> Learn how MentraOS permissions work, the available permission types, how to declare them for your app, and how enforcement and best practices ensure user privacy and transparency.

# Permissions

MentraOS uses a permissions system to control which device data and system resources your app can access. This system ensures user privacy and transparency while allowing you to build powerful applications with access to the data streams you need.

## Overview

When you build an MentraOS app, you must declare which permissions your app requires to function. These permissions map directly to sensitive device capabilities like microphone access, location services, and notification data. Users can see exactly what permissions your app requires before installing it, and the system automatically enforces these permissions at runtime.

```typescript
// Example: Your app automatically receives permission information in session
protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
  // Subscribe to transcription events (requires MICROPHONE permission)
  session.events.onTranscription((data) => {
    console.log('User said:', data.text);
  });

  // Subscribe to location updates (requires LOCATION permission)
  session.events.onLocationUpdate((data) => {
    console.log('User location:', data.lat, data.lng);
  });
}
```

## How Permissions Work

The permissions system operates at two levels:

1. **Declaration Level**: You declare which permissions your app needs in the [MentraOS Developer Console](https://console.mentra.glass/)
2. **Runtime Level**: The system automatically enforces these permissions when your app tries to access protected data streams

When a user starts your app:

* The system checks if your app has declared the necessary permissions for the data streams it's trying to access
* If device-level permissions (like microphone access) are disabled, your app receives error messages
* Users see exactly what data your app can access based on your permission declarations

## Declaring Permissions

### Using the Developer Console

To declare permissions for your app:

1. Log in to the [MentraOS Developer Console](https://console.mentra.glass/)
2. Navigate to your app's settings page
3. Scroll to the **Required Permissions** section
4. Click **Add Permission** to add a new permission
5. Select the permission type from the dropdown
6. Add a description explaining why your app needs this permission.  This is shown to the user when they install your app.
7. Save your changes

## Available Permission Types

MentraOS defines five core permission types that map to device-level capabilities:

### 1. **MICROPHONE**

* **Purpose**: Access to audio recording and speech processing

* **Data Streams Enabled**:
  * [`TRANSCRIPTION`](/reference/enums#streamtype) - Real-time speech-to-text conversion
  * [`TRANSLATION`](/reference/enums#streamtype) - Language translation of speech
  * [`AUDIO_CHUNK`](/reference/enums#streamtype) - Raw audio data for advanced processing
  * [`VAD`](/reference/enums#streamtype) - Voice activity detection
  * Language-specific streams (e.g., `transcription:en`, `translation:fr`)

* **Common Use Cases**: Voice assistants, transcription apps, language translation, voice commands

```typescript
// Example: Using microphone permission
session.events.onTranscription((data) => {
  if (data.isFinal) {
    console.log('User said:', data.text);
    // Process the transcribed text
  }
});
```

### 2. **LOCATION**

* **Purpose**: Access to device location services

* **Data Streams Enabled**:
  * [`LOCATION_UPDATE`](/reference/enums#streamtype) - Real-time GPS coordinates

* **Common Use Cases**: Navigation apps, location-based reminders, weather apps, local search

```typescript
// Example: Using location permission
session.location.subscribeToStream({ accuracy: 'high' }, (data) => {
  console.log(`User is at: ${data.lat}, ${data.lng}`);
  // Update location-based features
});
```

### 3. **CALENDAR**

* **Purpose**: Access to device calendar events

* **Data Streams Enabled**:
  * [`CALENDAR_EVENT`](/reference/enums#streamtype) - Calendar event information

* **Common Use Cases**: Schedule management, meeting reminders, productivity apps

```typescript
// Example: Using calendar permission
session.events.on<CalendarEvent>(StreamType.CALENDAR_EVENT, (data) => {
  console.log('Upcoming event:', data.title);
  // Display calendar information
});
```

### 4. **CAMERA**

* **Purpose**: Access to camera for photo capture and video streaming

* **Data Streams Enabled**:
  * [`PHOTO_RESPONSE`](/reference/enums#streamtype) - Photo capture responses
  * [`PHOTO_TAKEN`](/reference/enums#streamtype) - Photo capture events
  * [`RTMP_STREAM_STATUS`](/reference/enums#streamtype) - RTMP streaming status updates
  * [`MANAGED_STREAM_STATUS`](/reference/enums#streamtype) - Managed streaming status updates

* **Common Use Cases**: Photo documentation, live streaming, video recording, security monitoring

```typescript
// Example: Using camera permission
session.camera.requestPhoto({ saveToGallery: true }).then((photo) => {
  console.log('Photo captured:', photo.filename);
  // Process the captured photo
});

// Start streaming
session.camera.startStream({
  rtmpUrl: 'rtmp://example.com/live/stream-key'
});
```

### 5. **READ\_NOTIFICATIONS**

* **Purpose**: Access to device notifications

* **Data Streams Enabled**:
  * [`PHONE_NOTIFICATION`](/reference/enums#streamtype) - Incoming notification data
  * [`NOTIFICATION_DISMISSED`](/reference/enums#streamtype) - Notification dismissal events

* **Common Use Cases**: Notification filtering, smart alerts, communication apps

```typescript
// Example: Using notifications permission
session.events.onPhoneNotification((data) => {
  console.log(`New notification from ${data.app}: ${data.title}`);
  // Process or filter notifications
});
```

## Permission Enforcement

### Automatic Stream Filtering

When your app subscribes to data streams, the system automatically checks:

1. **Permission Declaration**: Does your app declare the required permission?
2. **Device Permission Status**: Is the corresponding device permission enabled?
3. **Access Decision**: Grant or deny access based on both criteria

```typescript
// If your app hasn't declared MICROPHONE permission, this will fail
session.events.onTranscription((data) => {
  // This handler will never be called without MICROPHONE permission
});
```

### Error Handling

When permission requirements aren't met, you'll receive error messages:

```typescript
// Listen for permission-related errors
session.events.on('permission_denied', (data) => {
  console.log(`Cannot access ${data.stream} - ${data.permission} permission required`);
  // Provide alternative functionality or user guidance
});
```

## Best Practices

### 1. **Minimize Permissions**

Only declare permissions that are essential for your app's core functionality:

```typescript
// ✅ Good: Only request what you need
{
  "permissions": [
    { "type": "MICROPHONE", "description": "For voice commands" }
  ]
}
```

### 2. **Provide Clear Descriptions**

Help users understand why your app needs each permission:

```typescript
// ✅ Good: Clear, specific description
{
  "type": "LOCATION",
  "description": "Used to provide weather information for your current location"
}

// ❌ Avoid: Vague or missing description
{
  "type": "LOCATION"
}
```

### 3. **Respect User Privacy**

* Only access the data you actually use
* Process sensitive data securely
* Be transparent about data usage in your app description
