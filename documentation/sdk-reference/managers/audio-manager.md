# null

# AudioManager API Reference

The `AudioManager` class provides comprehensive audio functionality for MentraOS Apps, including URL-based audio playback, text-to-speech synthesis, and audio control. It automatically handles request tracking, timeouts, and provides real-time status monitoring.

## Import

```typescript
import { AppServer, AppSession } from '@mentra/sdk';

export class MyAppServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Play audio from URL
    const result = await session.audio.playAudio({
      audioUrl: 'https://example.com/sound.mp3'
    });

    // Convert text to speech
    await session.audio.speak('Hello, world!');
  }
}
```

## Class: AudioManager

The AudioManager is automatically instantiated by the AppSession. You should not create instances directly.

## Audio Playback Methods

### playAudio

Play audio from a URL on the connected glasses (or the phone).

```typescript
async playAudio(options: AudioPlayOptions): Promise<AudioPlayResult>
```

**Parameters:**

* `options`: Audio playback configuration

**Returns:** Promise that resolves with playback result

**Example:**

```typescript
// Basic audio playback
const result = await session.audio.playAudio({
  audioUrl: 'https://example.com/notification.mp3'
});

// Handle the result
if (result.success) {
  console.log(`Audio played successfully`);
} else {
  console.error(`Playback failed: ${result.error}`);
}
```

### speak

Convert text to speech and play it on the connected glasses.

```typescript
async speak(text: string, options: SpeakOptions = {}): Promise<AudioPlayResult>
```

**Parameters:**

* `text`: Text to convert to speech (required)
* `options`: Text-to-speech configuration (optional)

**Returns:** Promise that resolves with playback result

**Example:**

```typescript
// Basic text-to-speech
const result = await session.audio.speak('Welcome to Mentra OS!');

// Advanced TTS with custom voice settings for expressive speech
const result = await session.audio.speak(
  'This uses a custom voice configuration for more expressive delivery.',
  {
    voice_id: 'your_elevenlabs_voice_id',
    model_id: 'eleven_turbo_v2_5',
    voice_settings: {
      stability: 0.4,           // Lower stability for more emotional range
      similarity_boost: 0.85,   // High adherence to original voice
      style: 0.6,              // Amplify speaker's style
      use_speaker_boost: true,  // Boost similarity (increases latency)
      speed: 0.95              // Slightly slower for clarity
    }
  }
);

// Handle multilingual content with appropriate model
await session.audio.speak('Bonjour! Comment allez-vous?', {
  voice_id: 'french_voice_id',
  model_id: 'eleven_turbo_v2_5'  // Supports French
});

// Real-time application with ultra-low latency
await session.audio.speak('Quick notification message', {
  model_id: 'eleven_flash_v2_5',
  voice_settings: {
    stability: 0.8,  // Higher stability for consistent quick responses
    speed: 1.1       // Slightly faster
  }
});
```

### stopAudio

Stop all audio playback on the connected glasses.

```typescript
stopAudio(): void
```

**Example:**

```typescript
// Stop all currently playing audio
session.audio.stopAudio();

// Combine with user feedback
session.audio.stopAudio();
session.layouts.showTextWall('Audio stopped');
```

## Request Management Methods

### hasPendingRequest

Check if there are pending audio requests.

```typescript
hasPendingRequest(requestId?: string): boolean
```

**Parameters:**

* `requestId`: Optional specific request ID to check

**Returns:** True if there are pending requests (or specific request exists)

**Example:**

```typescript
// Check for any pending requests
if (session.audio.hasPendingRequest()) {
  console.log('Audio requests are still processing...');
}

// Check for a specific request
const requestId = 'audio_req_12345';
if (session.audio.hasPendingRequest(requestId)) {
  console.log(`Request ${requestId} is still pending`);
}
```

## Interfaces

### AudioPlayOptions

Options for audio playback from URLs.

```typescript
interface AudioPlayOptions {
  /** URL to audio file for download and play */
  audioUrl: string;
  /** Volume level `0.0-1.0`, defaults to 1.0 */
  volume?: number;
}
```

### SpeakOptions

Options for text-to-speech synthesis.

```typescript
interface SpeakOptions {
  /** Voice ID to use (optional) */
  voice_id?: string;
  /** Model ID to use (optional, defaults to eleven_flash_v2_5) */
  model_id?: string;
  /** Voice settings object (optional) */
  voice_settings?: {
    /** Voice stability and randomness `(0.0-1.0)`. Lower values introduce broader emotional range, higher values result in monotonous voice */
    stability?: number;
    /** How closely AI should adhere to original voice `(0.0-1.0)` */
    similarity_boost?: number;
    /** Style exaggeration of the voice `(0.0-1.0)`. Amplifies original speaker's style but increases latency */
    style?: number;
    /** Boosts similarity to original speaker. Increases computational load and latency */
    use_speaker_boost?: boolean;
    /** Playback speed. 1.0 = normal, `<1.0` = slower, `>1.0` = faster */
    speed?: number;
  };
}
```

### AudioPlayResult

Result of an audio playback attempt.

```typescript
interface AudioPlayResult {
  /** Whether the audio playback was successful (false if audio was stopped, for example) */
  success: boolean;
  /** Error message if playback failed */
  error?: string;
  /** Duration of the audio file in ms (if available) */
  duration?: number;
  }
```

## Available ElevenLabs TTS Models

| Model                    | Description                                  | Languages                                                                                                           | Latency     |
| ------------------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------- |
| `eleven_v3`              | Human-like and expressive speech generation  | 70+ languages                                                                                                       | Standard    |
| `eleven_flash_v2_5`      | Ultra-fast model optimized for real-time use | All multilingual\_v2 languages + hu, no, vi                                                                         | \~75ms      |
| `eleven_flash_v2`        | Ultra-fast model (English only)              | en                                                                                                                  | \~75ms      |
| `eleven_turbo_v2_5`      | High quality, low-latency with good balance  | Same as flash\_v2\_5                                                                                                | \~250-300ms |
| `eleven_turbo_v2`        | High quality, low-latency (English only)     | en                                                                                                                  | \~250-300ms |
| `eleven_multilingual_v2` | Most lifelike with rich emotional expression | en, ja, zh, de, hi, fr, ko, pt, it, es, id, nl, tr, fil, pl, sv, bg, ro, ar, cs, el, fi, hr, ms, sk, da, ta, uk, ru | Standard    |

**Model Selection Guidelines:**

* Use `eleven_flash_v2_5` for real-time applications requiring ultra-low latency
* Use `eleven_turbo_v2_5` for a good balance of quality and speed
* Use `eleven_multilingual_v2` for the highest quality emotional expression
* Use `eleven_v3` for maximum language support

## Best Practices

### 1. Check Device Capabilities

```typescript
protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
  // check if the device supports audio
  // if not, audio plays back through the phone
  if (!session.capabilities?.hasSpeaker) {
    session.layouts.showTextWall('⚠️ Audio not supported on this device, will play through phone');
  }

  // Adapt behavior based on speaker type
  const isPrivate = session.capabilities.speaker?.isPrivate;
  const defaultVolume = isPrivate ? 0.8 : 0.6;

  await session.audio.speak('Audio test', { volume: defaultVolume });
}
```

### 2. Handle Errors Gracefully

```typescript
async function playNotificationSound(session: AppSession): Promise<void> {
  try {
    const result = await session.audio.playAudio({
      audioUrl: 'https://example.com/notification.mp3',
      volume: 0.7
    });

    if (!result.success) {
      // Fallback to TTS if audio file fails
      await session.audio.speak('Notification received');
    }
  } catch (error) {
    // Handle network or system errors
    session.logger.error(`Audio error: ${error}`);
    session.layouts.showTextWall('🔔 Notification');
  }
}
```

### 3. Chain Multiple Audio Requests

```typescript
const response = await session.audio.speak("You said: " + data.text)
if (response.success) {
    await session.audio.playAudio({ audioUrl: "https://okgodoit.com/cool.mp3" })
}
```

### 4. Allow the User to Interrupt the Audio

```typescript
session.events.onTranscription(async (data) => {
    if (data.text.toLowerCase().includes("stop")) {
        await session.audio.stopAudio();
        return;
    }
}
```

## Related Documentation

* [Audio Tutorial](/audio) - Step-by-step guide to audio functionality
* [Device Capabilities](/capabilities) - Check audio hardware support
* [App Session](/reference/app-session) - AppSession class reference
* [Events](/events) - Voice activation and audio events
