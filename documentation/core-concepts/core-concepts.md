# Core Concepts

> Learn the fundamental concepts you need to understand to build MentraOS apps for smart glasses.

# Core Concepts

This section explains the fundamental concepts you need to understand to build MentraOS apps for smart glasses.

## 1. MentraOS Apps

MentraOS apps are the core of extending MentraOS. They are *server-side* applications that you build to provide functionality to the smart glasses. Key characteristics of MentraOS apps:

* **Independent:** Apps run in their own process, separate from the MentraOS Cloud and the glasses.
* **Event-Driven:** Apps primarily interact with the system by responding to events (like transcriptions, button presses, etc.).
* **Real-Time:** Apps communicate with the glasses via WebSockets for low-latency interaction.
* **Server-Side:** All app logic runs on your server, not on the glasses themselves. This allows for more complex processing and integration with external services.

## 2. Sessions

A *session* represents the active connection between a user's smart glasses and your app.

* **Unique ID:** Each session has a unique `sessionId` (a UUID string) assigned by MentraOS Cloud.
* **User Association:** A session is associated with a specific `userId`.
* **App Association:** When a user starts your app, a new session is created specifically for that user and your app.
* **Lifecycle:** Sessions are created when a user starts your app and are terminated when the user stops the app, the glasses disconnect, or an error occurs.

The [`AppSession`](/reference/app-session) class in the SDK provides methods for interacting with a session.

## 3. WebSockets

WebSockets are the primary communication mechanism between:

* Your app and MentraOS Cloud.
* MentraOS Cloud and the smart glasses.

The SDK handles the complexities of WebSocket connections for you. You primarily interact with the [`AppSession`](/reference/app-session) object, which provides methods for sending and receiving messages.

## 4. Events and Data Streams

MentraOS Cloud sends real-time data to your app as *events*. These events represent:

* **User Input:** [Button presses](/reference/interfaces/event-types#buttonpress), [head movements](/reference/interfaces/event-types#headposition).
* **Sensor Data:** [Location](/reference/interfaces/event-types#locationupdate), [battery level](/reference/interfaces/event-types#glassesbatteryupdate).
* **System Events:** Connection status, [settings changes](/reference/interfaces/config-types#appsettings).
* **Processed Data:** [Speech transcription](/reference/interfaces/event-types#transcriptiondata), [phone notifications](/reference/interfaces/event-types#phonenotification).

Your app *subscribes* to the events it needs. The [`EventManager`](/reference/managers/event-manager) class (accessible through `session.events`) provides methods for subscribing to and handling events.

See the [Events](./events) section for a complete list of available events.

## 5. The App Lifecycle

A typical MentraOS app lifecycle looks like this:

1. **Webhook Request:** When a user starts your app on their glasses, MentraOS Cloud sends an HTTP POST request (a "webhook") to your app's pre-defined webhook URL. This request includes a unique `sessionId` and `userId`.
2. **WebSocket Connection:** Your app receives the webhook and uses the `sessionId` to establish a WebSocket connection to MentraOS Cloud.
3. **Subscription:** Your app subscribes to the events it needs (e.g., [transcription](/reference/interfaces/event-types#transcriptiondata), [head position](/reference/interfaces/event-types#headposition)).
4. **Event Handling:** Your app receives events from MentraOS Cloud and processes them. This often involves updating the display using the [`LayoutManager`](/reference/managers/layout-manager).
5. **Session Termination:** The session ends when the user stops the app, the glasses disconnect, or an error occurs.

See [App Lifecycle](./app-lifecycle) for a more detailed explanation.

## 6. Permissions

MentraOS uses a permissions system to control which device data and system resources your app can access. This system ensures user privacy and transparency while allowing you to build powerful applications with access to the data streams you need.

Your app must declare which permissions it needs to access device capabilities.

See the [Permissions](./permissions) section for detailed information about declaring and using permissions.

## 7. Layouts

Layouts control what is displayed on the smart glasses' screen. The SDK provides several pre-defined layout types:

* [`TextWall`](/reference/interfaces/layout-types#textwall): Displays a single block of text.
* [`DoubleTextWall`](/reference/interfaces/layout-types#doubletextwall): Displays two blocks of text (top and bottom).
* [`ReferenceCard`](/reference/interfaces/layout-types#referencecard): Displays a card with a title and content.

You use the [`LayoutManager`](/reference/managers/layout-manager) (accessible through `session.layouts`) to display layouts.

See the [Layouts](./layouts) section for more details.

## 8. Dashboard

The dashboard is a persistent UI surface that appears when users look up on their smart glasses. Unlike regular layouts that are temporary, the dashboard provides a always-available space where your app can display status updates, notifications, and contextual information even when other apps are active.

* **Persistent:** The dashboard remains visible across different app states and can show content from multiple apps simultaneously.
* **Mode-based:** Supports different display modes (main, expanded) to accommodate varying levels of detail.
* **Lightweight:** Designed for quick glances and brief status updates rather than complex interactions.

You use the [`DashboardAPI`](/reference/dashboard-api) (accessible through `session.dashboard`) to send content to the dashboard.

See the [Dashboard Tutorial](/dashboard) for a quick start guide and the [Dashboard API Reference](/reference/dashboard-api) for complete documentation.

## 9. User Authentication

MentraOS provides mechanisms for identifying and authenticating users:

1. **Session-based identification**: Each webhook includes a `userId` that identifies the current user.
2. **Webview authentication**: If your app provides a companion web interface, and users access it through the MentraOS Mobile App, you can authenticate them automatically using a token exchange system.

This allows you to provide personalized experiences and maintain user data across sessions without requiring separate login flows.

See the [Webview Authentication](/webview-auth-overview) section for more details about the webview authentication flow.

## 10. AI Tools

Your app can provide tools to the Mira AI assistant:

Your app defines tools in the [MentraOS Developer Console](https://console.mentra.glass/apps) under the "AI Tools" section. When a user asks the AI to perform a task related to your app, the AI calls your tools via the `onToolCall` event. Your app executes the requested action and returns results to the AI.

This allows users to interact with your app through natural language via the AI assistant. Tool calls don't happen in the context of a session, and your app does not need to be running to respond to a tool call.

See the [Tools](/tools) section for more details.

## 11. Settings

MentraOS provides a comprehensive settings system that allows users to customize how your app behaves.

Settings support various types including toggles, text inputs, dropdowns, sliders, and more. This allows you to offer rich customization options without building your own settings UI.

See the [Settings](/settings) section for more details.

## 12. Device Capabilities

Different smart glasses models have different hardware features. Some have cameras, others don't. Some have displays, others are audio-only. MentraOS provides a capabilities system that lets your app discover what hardware is available on the connected device and adapt accordingly.

The [`AppSession`](/reference/app-session) exposes device capabilities through `session.capabilities`. See the [Device Capabilities](/capabilities) guide for detailed usage examples and the [Capabilities Reference](/reference/interfaces/capabilities) for complete type documentation.

## 13. The MentraOS Cloud

The MentraOS Cloud acts as a central hub, managing:

* User sessions.
* App connections.
* Data stream routing.
* Display management.
* Communication with external services (speech-to-text, etc).

Your app interacts with the cloud, but you don't need to worry about the internal details of how the cloud operates. The SDK abstracts away these complexities.

## 14. Logging

MentraOS provides structured logging capabilities using [Pino](https://getpino.io/), a fast and low-overhead logging library. Every [`AppSession`](/reference/app-session) includes a pre-configured logger that automatically includes session context.

### Session Logger

Each session has a dedicated logger accessible via `session.logger`:

```typescript
protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
  // Logger automatically includes session context
  session.logger.info('Session started', { timestamp: new Date() });

  session.events.onTranscription((data) => {
    session.logger.debug('Transcription received', {
      text: data.text,
      isFinal: data.isFinal
    });
  });
}
```

### Log Levels

Use appropriate log levels for different types of information:

```typescript
// Debug: Detailed information for troubleshooting
session.logger.debug("Processing user input", {inputType: "voice"})
// Info: General information about app operation
session.logger.info("User completed onboarding")
// Warn: Something unexpected happened but app continues
session.logger.warn("Transcription quality low", {confidence: 0.3})
// Error: Something went wrong that needs attention
session.logger.error(error, "Failed to save user preferences")
```

See the [App Session Logger documentation](/reference/app-session#logger) for complete API details.

## 15. Playing Audio

MentraOS provides comprehensive audio functionality through the [`AudioManager`](/reference/managers/audio-manager) (accessible via `session.audio`):

* **Audio Playback:** Play audio files from URLs directly through the smart glasses speakers or phone
* **Text-to-Speech:** Convert text to natural-sounding speech using ElevenLabs TTS with customizable voice settings

Audio routes through your phone by default. To route audio through smart glasses with speakers, connect them via Bluetooth like any other headphones.

See the [Audio Tutorial](/audio) for detailed usage examples.

## 16. Camera

The camera module enables visual functionality on camera-equipped smart glasses through the [`CameraManager`](/reference/managers/camera) (accessible via `session.camera`):

* **Photo Capture:** Take individual photos from the smart glasses camera with options for gallery saving
* **RTMP Streaming:** Stream live video using managed streaming (zero infrastructure) or unmanaged streaming (full control)

Camera functionality requires smart glasses with camera hardware (e.g., Mentra Live) and appropriate permissions.

See the [Camera Documentation](/camera/) for complete guides and examples.

## 17. Location

The location module provides battery‑efficient access to device location via the [`LocationManager`](/reference/managers/location-manager) (accessible through `session.location`). Your app can subscribe to continuous updates at different accuracy tiers or request a single fix with `getLatestLocation`. Location data is delivered as [`LocationUpdate`](/reference/interfaces/event-types#locationupdate) events and requires appropriate location permissions.

***

By understanding these core concepts, you'll be well-equipped to start building powerful and engaging apps for MentraOS smart glasses.
