# Audio

> Learn how to play audio from URLs, synthesize speech with ElevenLabs, and manage playback in MentraOS apps.

# Audio Tutorial

Learn how to build **MentraOS Apps** that can:

1. **Play audio files** from URLs on connected smart glasses or phone
2. **Convert text to speech** and play it through the glasses speakers or phone
3. **Stop audio playback** when needed

Audio routes through your phone by default.  To route auto through the Mentra Live, connect to it through your phone’s settings like any other bluetooth headphones.  This is separate from the pairing process to MentraOS.

***

## Prerequisites

1. **MentraOS SDK ≥ `2.1.2`** installed in your project
2. A local development environment configured as described in [Getting Started](/getting-started)

***

## 1 - Set up the Project

Copy the basic project structure from the [Quickstart](/quickstart) if you haven't already. We'll focus on the contents of `src/index.ts`.

***

## 2 - Playing Audio from URLs

The most straightforward way to play audio is from a publicly accessible URL.

```typescript title="src/index.ts"
import { AppServer, AppSession } from "@mentra/sdk";

class AudioDemoServer extends AppServer {
  protected async onSession(
    session: AppSession,
    sessionId: string,
    userId: string,
  ): Promise<void> {
    session.logger.info(`Audio demo session started for ${userId}`);

    // Example: Play a notification sound
    try {
      const result = await session.audio.playAudio({
        audioUrl: "https://okgodoit.com/cool.mp3"
      });

      if (result.success) {
        session.logger.info(`Audio played successfully`);
        if (result.duration) {
          session.logger.info(`Duration: ${result.duration} ms`);
        }
      } else {
        session.logger.error(`❌ Audio playback failed: ${result.error}`);
      }
    } catch (error) {
      session.logger.error(`Exception during audio playback: ${error}`);
    }
  }
}

// Bootstrap the server
new AudioDemoServer({
  packageName: process.env.PACKAGE_NAME ?? "com.example.audio",
  apiKey: process.env.MENTRAOS_API_KEY!,
  port: Number(process.env.PORT ?? "3000"),
}).start();
```

***

## 3 - Text-to-Speech (TTS)

Convert any text to natural-sounding speech using ElevenLabs and play it on the glasses.

```typescript title="src/index.ts"
import { AppServer, AppSession } from "@mentra/sdk";

class TTSServer extends AppServer {
  protected async onSession(
    session: AppSession,
    sessionId: string,
    userId: string,
  ): Promise<void> {
    session.logger.info(`TTS demo session started`);

    // Basic text-to-speech
    try {
      const result = await session.audio.speak("Welcome to Mentra OS! This is your audio assistant.");

      if (result.success) {
        session.logger.info("Speech synthesis successful");
      } else {
        session.logger.error(`❌ TTS failed: ${result.error}`);
      }
    } catch (error) {
      session.logger.error(`Exception during TTS: ${error}`);
    }

    // Advanced TTS with custom voice settings
    try {
      const result = await session.audio.speak(
        "This message uses custom voice settings for a different sound.",
        {
          voice_id: "your_elevenlabs_voice_id", // Optional: specific ElevenLabs voice
          model_id: "eleven_flash_v2_5", // Optional: specific model
          voice_settings: {      // each setting is optional
            stability: 0.7,      // Voice consistency (0.0-1.0)
            similarity_boost: 0.8, // Voice similarity (0.0-1.0)
            style: 0.3,          // Speaking style (0.0-1.0)
            speed: 0.9           // Speaking speed (0.25-4.0)
          }
        }
      );

      if (result.success) {
        session.logger.info("Advanced TTS successful");
      }
    } catch (error) {
      session.logger.error(`Exception during advanced TTS: ${error}`);
    }
  }
}
```

### TTS Configuration Options

| Option                             | Type    | Default             | Description                                                                                                                              |
| ---------------------------------- | ------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `voice_id`                         | string  | Server default      | ElevenLabs voice ID                                                                                                                      |
| `model_id`                         | string  | `eleven_flash_v2_5` | TTS model to use (see models below)                                                                                                      |
| `voice_settings.stability`         | number  | 0.5                 | Voice stability and randomness `(0.0-1.0)`. Lower values introduce broader emotional range, higher values can result in monotonous voice |
| `voice_settings.similarity_boost`  | number  | 0.75                | How closely AI adheres to original voice `(0.0-1.0)`                                                                                     |
| `voice_settings.style`             | number  | 0.0                 | Style exaggeration of the voice `(0.0-1.0)`. Amplifies original speaker's style but increases latency                                    |
| `voice_settings.use_speaker_boost` | boolean | false               | Boosts similarity to original speaker. Increases computational load and latency                                                          |
| `voice_settings.speed`             | number  | 1.0                 | Playback speed. 1.0 = normal, `<1.0` = slower, `>1.0` = faster                                                                           |

### Available TTS Models

| Model                    | Description                                  | Languages                                                                                                           | Latency     |
| ------------------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------- |
| `eleven_v3`              | Human-like and expressive speech generation  | 70+ languages                                                                                                       | Standard    |
| `eleven_flash_v2_5`      | Ultra-fast model optimized for real-time use | All multilingual\_v2 languages + hu, no, vi                                                                         | \~75ms      |
| `eleven_flash_v2`        | Ultra-fast model (English only)              | en                                                                                                                  | \~75ms      |
| `eleven_turbo_v2_5`      | High quality, low-latency with good balance  | Same as flash\_v2\_5                                                                                                | \~250-300ms |
| `eleven_turbo_v2`        | High quality, low-latency (English only)     | en                                                                                                                  | \~250-300ms |
| `eleven_multilingual_v2` | Most lifelike with rich emotional expression | en, ja, zh, de, hi, fr, ko, pt, it, es, id, nl, tr, fil, pl, sv, bg, ro, ar, cs, el, fi, hr, ms, sk, da, ta, uk, ru | Standard    |

***

## 4 - Interactive Audio App

Here's a complete example that combines voice activation with audio responses:

```typescript title="src/index.ts"
import { AppServer, AppSession } from "@mentra/sdk";

class InteractiveAudioApp extends AppServer {
  protected async onSession(
    session: AppSession,
    sessionId: string,
    userId: string,
  ): Promise<void> {
    session.logger.info(`🎤 Interactive audio app started`);

    // Welcome message
    await session.audio.speak("Welcome to the interactive audio demo. Say 'play music' or 'tell me a joke'.");

    // Listen for voice commands
    const unsubscribe = session.events.onTranscription(async (data) => {
      if (data.text.toLowerCase().includes("stop")) {
        await session.audio.stopAudio();
        return;
      }

      if (!data.isFinal) return;

      const command = data.text.toLowerCase().trim();
      session.logger.info(`Heard: "${command}"`);

      if (command.includes("play music")) {
        await this.playMusic(session);
      } else if (command.includes("tell me a joke") || command.includes("joke")) {
        await this.tellJoke(session);
      } else if (command.includes("hello")) {
        await session.audio.speak("Hello there! How can I help you today?");
      }
    });

    // Clean up listener when session ends
    this.addCleanupHandler(unsubscribe);
  }

  private async playMusic(session: AppSession): Promise<void> {
    session.layouts.showTextWall("🎵 Playing music...");

    try {
      const result = await session.audio.playAudio({
        audioUrl: "https://example.com/background-music.mp3",
        volume: 0.6
      });

      if (result.success) {
        await session.audio.speak("Hope you enjoyed the music!");
      } else {
        await session.audio.speak("Sorry, I couldn't play the music right now.");
      }
    } catch (error) {
      session.logger.error(`Music playback error: ${error}`);
      await session.audio.speak("There was an error playing music.");
    }
  }

  private async tellJoke(session: AppSession): Promise<void> {
    const jokes = [
      "What do you call a pair of glasses that can see the future? ... Pre-scription glasses!",
      "Why did the augmented reality app break up with the virtual reality app? ... It said the relationship wasn't real enough!",
      "Why did the phone pair to the smart glasses? ... Because it lost its contacts!",
    ];

    const joke = jokes[Math.floor(Math.random() * jokes.length)];
    session.layouts.showTextWall("Telling a joke...");

    await session.audio.speak(joke, {
      voice_settings: {
        style: 0.8, // More expressive for jokes
        speed: 0.9  // Slightly slower for comedy timing
      }
    });
  }
}

// Bootstrap the server
new InteractiveAudioApp({
  packageName: process.env.PACKAGE_NAME ?? "com.example.interactiveaudio",
  apiKey: process.env.MENTRAOS_API_KEY!,
  port: Number(process.env.PORT ?? "3000"),
}).start();
```

***

## 5 - Audio Management

### Stopping Audio

```typescript
// Stop all currently playing audio
session.audio.stopAudio();
```

### Error Handling

```typescript
const result = await session.audio.playAudio({
  audioUrl: "https://example.com/sound.mp3"
});

if (!result.success) {
  // This could be because the audio was interrupted by session.audio.stopAudio(), or there could be an error

  if (result.error) {
    // Handle specific audio errors
    session.logger.error(`Audio error: ${result.error}`);

    // Provide user feedback
    await session.audio.speak("Sorry, I couldn't play that audio file.");
  }
}
```

***

## Next Steps

* See the detailed [Audio Manager](/reference/managers/audio-manager) documentation
* Explore [Device Capabilities](/capabilities) to adapt audio features based on hardware
* Learn about [Events](/events) to create voice-activated audio experiences
* Review [Permissions](/permissions) for any audio-related permissions your app might need
