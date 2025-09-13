# AppSession

> AppSession manages a WebSocket session between your app and MentraOS Cloud, providing events, layouts, settings, dashboard access, capabilities, and logging.

# AppSession

`AppSession` (also known as `TpaSession` in older versions) manages an active WebSocket connection (session) between an app instance and MentraOS Cloud. It handles event subscriptions, layout display, and connection management for a single user session.

```typescript
import { AppSession } from '@mentra/sdk';
```

## Constructor

```typescript
constructor(config: AppSessionConfig)
```

**Parameters:**

* `config`: [Configuration](#configuration) options for the app session

## Properties

### events

Provides access to the [`EventManager`](/reference/managers/event-manager) for subscribing to real-time events.

```typescript
readonly events: EventManager
```

### layouts

Provides access to the [`LayoutManager`](/reference/managers/layout-manager) for controlling the AR display.

```typescript
readonly layouts: LayoutManager
```

### settings

Provides access to the [`SettingsManager`](/reference/managers/settings-manager) for reading and monitoring app settings.

```typescript
readonly settings: SettingsManager
```

### dashboard

Provides access to the [`DashboardAPI`](/reference/dashboard-api#interface-dashboardapi) for sending content to the user's dashboard.

```typescript
readonly dashboard: DashboardAPI
```

The dashboard is a persistent UI surface that appears when users look up, allowing your app to display status updates and information even when other apps are active. See the [Dashboard Tutorial](/dashboard) for a quick start guide and the [Dashboard API Reference](/reference/dashboard-api) for complete documentation.

### capabilities

Provides access to the device capabilities of the connected smart glasses.

```typescript
readonly capabilities: Capabilities | null
```

The capabilities object contains information about what hardware features are available on the connected device, allowing your app to adapt its behavior accordingly.

See the [Device Capabilities Guide](/capabilities) for usage examples and the [Capabilities Reference](/reference/interfaces/capabilities) for complete type documentation.

### logger

Provides access to a pre-configured [Pino](https://getpino.io) logger instance for session-specific logging.

```typescript
readonly logger: Logger
```

The logger is automatically configured with session context including:

* `userId`: The current user's identifier
* `packageName`: Your app's package name
* `sessionId`: The current session identifier
* `service`: Set to 'app-session'

**Example:**

```typescript
protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
  // The logger automatically includes session context
  session.logger.info('Session started successfully');
  session.logger.debug('Detailed debug information', { additionalData: 'value' });

  session.events.onTranscription((data) => {
    session.logger.debug('Received transcription', { text: data.text, isFinal: data.isFinal });

    if (data.text.includes('error')) {
      session.logger.warn('Potential error detected in transcription', { text: data.text });
    }
  });

  try {
    // Some operation that might fail
    await someRiskyOperation();
  } catch (error) {
    session.logger.error(error, 'Failed to perform operation');
    // Handle error appropriately
  }
}
```

**Log Levels:**

* `session.logger.debug()`: Detailed debugging information
* `session.logger.info()`: General information about app operation
* `session.logger.warn()`: Warning conditions that don't stop execution
* `session.logger.error()`: Error conditions that should be investigated

**Structured Logging:**

```typescript
session.logger.info('User performed action', {
  action: 'button_press',
  buttonId: 'main',
  timestamp: new Date(),
  metadata: { /* additional context */ }
});
```

## Event Handling Methods

### onTranscription()

Registers a handler for real-time speech transcription events.

```typescript
onTranscription(handler: (data: TranscriptionData) => void): () => void
```

**Parameters:**

* `handler`: Callback function to process [`TranscriptionData`](/reference/interfaces/event-types#transcriptiondata)

**Returns:** An unsubscribe function to remove the handler

### onHeadPosition()

Registers a handler for head position change events (e.g., 'up', 'down').

```typescript
onHeadPosition(handler: (data: HeadPosition) => void): () => void
```

**Parameters:**

* `handler`: Callback function to process [`HeadPosition`](/reference/interfaces/event-types#headposition) data

**Returns:** An unsubscribe function to remove the handler

### onButtonPress()

Registers a handler for hardware button press events on the glasses.

```typescript
onButtonPress(handler: (data: ButtonPress) => void): () => void
```

**Parameters:**

* `handler`: Callback function to process [`ButtonPress`](/reference/interfaces/event-types#buttonpress) data

**Returns:** An unsubscribe function to remove the handler

### onPhoneNotifications()

Registers a handler for notifications received from the connected phone.

```typescript
onPhoneNotifications(handler: (data: PhoneNotification) => void): () => void
```

**Parameters:**

* `handler`: Callback function to process [`PhoneNotification`](/reference/interfaces/event-types#phonenotification) data

**Returns:** An unsubscribe function to remove the handler

## Subscription Methods

### subscribe()

Informs the MentraOS Cloud that this app session wants to receive events of the specified type.

```typescript
subscribe(type: StreamType): void
```

**Parameters:**

* `type`: The [`StreamType`](/reference/enums#streamtype) to subscribe to

### on()

Generic method to subscribe to any data stream type. Use specific `on<EventType>` methods where available.

```typescript
on<T extends StreamType>(
  event: T,
  handler: (data: StreamDataTypes[T]) => void
): () => void
```

**Parameters:**

* `event`: The [`StreamType`](/reference/enums#streamtype) to listen for
* `handler`: Callback function to process the data associated with the stream type

**Returns:** An unsubscribe function to remove the handler

## Connection Methods

### connect()

Establishes the WebSocket connection to MentraOS Cloud for this session.

```typescript
connect(sessionId: string): Promise<void>
```

**Parameters:**

* `sessionId`: The unique identifier for this session (provided by the [`SESSION_REQUEST`](/reference/interfaces/webhook-types#sessionwebhookrequest) webhook)

**Returns:** A promise that resolves upon successful connection and authentication, or rejects on failure

### disconnect()

Gracefully closes the WebSocket connection and cleans up resources for this session.

```typescript
disconnect(): void
```

## Settings Methods

### getSettings()

Retrieves all current application settings for this user session.

```typescript
getSettings(): AppSettings
```

**Returns:** A copy of the current [`AppSettings`](/reference/interfaces/config-types#appsettings)

### getSetting()

Retrieves the value of a specific application setting by its key.

```typescript
getSetting<T>(key: string): T | undefined
```

**Parameters:**

* `key`: The key of the setting to retrieve

**Returns:** The value of the setting, or undefined if not found or not set

### setSubscriptionSettings()

Configures the app session to automatically manage subscriptions based on changes to specific settings.

```typescript
setSubscriptionSettings(options: {
  updateOnChange: string[];
  handler: (settings: AppSettings) => StreamType[];
}): void
```

**Parameters:**

* `options`: Configuration object
  * `options.updateOnChange`: An array of setting keys that should trigger a subscription update when their value changes
  * `options.handler`: A function that takes the current [`AppSettings`](/reference/interfaces/config-types#appsettings) and returns an array of [`StreamType`](/reference/enums#streamtype) subscriptions that should be active

## Configuration

```typescript
interface AppSessionConfig {
  /** Your unique app identifier (e.g., 'org.company.appname'). */
  packageName: string;

  /** Your API key for authentication. */
  apiKey: string;

  /** The WebSocket URL provided by MentraOS Cloud. Defaults to 'ws://localhost:8002/app-ws'. */
  mentraOSWebsocketUrl?: string;

  /** Whether the session should automatically attempt to reconnect if the connection drops. Defaults to `false`. */
  autoReconnect?: boolean;

  /** Maximum number of reconnection attempts if `autoReconnect` is true. Default: 0 (no limit). */
  maxReconnectAttempts?: number;

  /** Initial delay (in ms) before the first reconnection attempt. Delay increases exponentially. Defaults to 1000. */
  reconnectDelay?: number;
}
```
