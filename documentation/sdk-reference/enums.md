# Enums

# Enums

This page documents all the enumeration types available in the MentraOS SDK. Enums are used to define sets of named constants that represent distinct values throughout the SDK.

## AppType

Defines the different types or roles an app can have within the MentraOS system.

```typescript
enum AppType {
  SYSTEM_DASHBOARD = 'system_dashboard', // Special UI placement, system functionality
  SYSTEM_APPSTORE = 'system_appstore',   // System app store functionality
  BACKGROUND = 'background',             // Runs without primary UI, can temporarily display content
  STANDARD = 'standard'                  // Regular app with standard lifecycle and UI interaction (default)
}
```

## AppState

Represents the lifecycle states of an app on the user's device.

```typescript
enum AppState {
  NOT_INSTALLED = 'not_installed', // App is not installed
  INSTALLED = 'installed',         // App is installed but not running
  BOOTING = 'booting',             // App is in the process of starting
  RUNNING = 'running',             // App is currently active
  STOPPED = 'stopped',             // App has been stopped (by user or system)
  ERROR = 'error'                  // App encountered an error
}
```

## Language

Defines supported language codes (BCP 47 format).

```typescript
enum Language {
  EN = "en", // English
  ES = "es", // Spanish
  FR = "fr"  // French
  // Additional languages may be added as supported
}
```

## LayoutType

Identifies the different predefined UI layout structures available for display.

```typescript
enum LayoutType {
  TEXT_WALL = 'text_wall',           // Single block of text
  DOUBLE_TEXT_WALL = 'double_text_wall', // Two blocks of text (top/bottom)
  DASHBOARD_CARD = 'dashboard_card',   // Key-value pair style card (left/right text)
  REFERENCE_CARD = 'reference_card',   // Card with a title and content text
  BITMAP_VIEW = 'bitmap_view'          // Displays a bitmap image
}
```

## ViewType

Specifies the target area on the AR display where a layout should appear.

```typescript
enum ViewType {
  DASHBOARD = 'dashboard', // The persistent dashboard area
  MAIN = 'main'            // The main, typically temporary, display area
}
```

## DashboardMode

Defines the different display modes available on the MentraOS dashboard.

```typescript
enum DashboardMode {
  MAIN = 'main',           // Standard compact dashboard (default)
  EXPANDED = 'expanded'    // Larger dashboard when user opens it explicitly
  // ALWAYS_ON = 'always_on'  // Compact overlay (coming soon)
}
```

**Values:**

* `MAIN`: The default compact dashboard mode that appears as a small overlay
* `EXPANDED`: The full-screen dashboard mode that users can open for detailed information
* `ALWAYS_ON`: *(Coming soon)* A persistent compact overlay mode

## AppSettingType

Defines the types of interactive settings that can be configured for an app in the MentraOS settings UI. Used within the `AppConfig` interface.

```typescript
enum AppSettingType {
  TOGGLE = 'toggle', // A boolean switch
  TEXT = 'text',     // A text input field
  SELECT = 'select'  // A dropdown selection list
}
```

## StreamType

Identifies the different types of real-time data streams and events Apps can subscribe to.

```typescript
enum StreamType {
  // Hardware streams
  BUTTON_PRESS = 'button_press',             // Hardware button pressed on glasses
  HEAD_POSITION = 'head_position',           // Head movement detected (e.g., up, down)
  GLASSES_BATTERY_UPDATE = 'glasses_battery_update', // Glasses battery level and status
  PHONE_BATTERY_UPDATE = 'phone_battery_update',   // Connected phone battery level and status
  GLASSES_CONNECTION_STATE = 'glasses_connection_state', // Glasses connection info (e.g., model)
  LOCATION_UPDATE = 'location_update',       // GPS location update

  // Audio streams
  TRANSCRIPTION = 'transcription',           // Real-time speech-to-text results
  TRANSLATION = 'translation',               // Real-time speech translation results
  VAD = 'VAD',                               // Voice Activity Detection status (speaking/not speaking)
  AUDIO_CHUNK = 'audio_chunk',               // Raw audio data chunks (requires explicit subscription)

  // Phone streams
  PHONE_NOTIFICATION = 'phone_notification', // Notification received on the connected phone
  NOTIFICATION_DISMISSED = 'notification_dismissed', // Notification dismissed on the glasses/phone
  CALENDAR_EVENT = 'calendar_event',         // Calendar event from the connected phone

  // System streams / Control actions originating from glasses
  START_APP = 'start_app',                   // User requested to start an app
  STOP_APP = 'stop_app',                     // User requested to stop an app
  OPEN_DASHBOARD = 'open_dashboard',         // User requested to open the dashboard

  // Video streams
  VIDEO = 'video',                           // Video stream data (if available/supported)

  // Special subscription types (for cloud internal use primarily)
  ALL = 'all',                               // Subscribe to all possible streams
  WILDCARD = '*'                             // Wildcard subscription (similar to ALL)
}
```

## WebhookRequestType

Identifies the type of request being sent from MentraOS Cloud to an app server's webhook endpoint.

```typescript
enum WebhookRequestType {
  SESSION_REQUEST = 'session_request',         // Request to start a new app session for a user
  STOP_REQUEST = 'stop_request',               // Request to stop an existing app session
  SERVER_REGISTRATION = 'server_registration', // Confirmation that the app server is registered
  SERVER_HEARTBEAT = 'server_heartbeat',       // Periodic check to ensure the app server is responsive
  SESSION_RECOVERY = 'session_recovery'        // Request to recover a session (e.g., after cloud restart)
}
```

## Message Type Enums

These enums are used to identify the types of messages exchanged between different components of the MentraOS system.

### GlassesToCloudMessageType

Message types sent FROM glasses TO cloud.

```typescript
enum GlassesToCloudMessageType {
  CONNECTION_INIT = 'connection_init',
  START_APP = 'start_app',
  STOP_APP = 'stop_app',
  DASHBOARD_STATE = 'dashboard_state',
  OPEN_DASHBOARD = 'open_dashboard',
  BUTTON_PRESS = 'button_press',
  HEAD_POSITION = 'head_position',
  GLASSES_BATTERY_UPDATE = 'glasses_battery_update',
  PHONE_BATTERY_UPDATE = 'phone_battery_update',
  GLASSES_CONNECTION_STATE = 'glasses_connection_state',
  LOCATION_UPDATE = 'location_update',
  VAD = 'VAD',
  PHONE_NOTIFICATION = 'phone_notification',
  NOTIFICATION_DISMISSED = 'notification_dismissed',
  CALENDAR_EVENT = 'calendar_event'
}
```

### CloudToGlassesMessageType

Message types sent FROM cloud TO glasses.

```typescript
enum CloudToGlassesMessageType {
  CONNECTION_ACK = 'connection_ack',
  CONNECTION_ERROR = 'connection_error',
  AUTH_ERROR = 'auth_error',
  DISPLAY_EVENT = 'display_event',
  APP_STATE_CHANGE = 'app_state_change',
  MICROPHONE_STATE_CHANGE = 'microphone_state_change',
  WEBSOCKET_ERROR = 'websocket_error'
}
```

### AppToCloudMessageType

Message types sent FROM App TO cloud.

```typescript
enum AppToCloudMessageType {
  CONNECTION_INIT = 'tpa_connection_init',
  SUBSCRIPTION_UPDATE = 'subscription_update',
  DISPLAY_REQUEST = 'display_event', // Note: Reuses 'display_event' type string
  DASHBOARD_CONTENT_UPDATE = 'dashboard_content_update'
}
```

### CloudToAppMessageType

Message types sent FROM cloud TO App.

```typescript
enum CloudToAppMessageType {
  CONNECTION_ACK = 'tpa_connection_ack',
  CONNECTION_ERROR = 'tpa_connection_error',
  APP_STOPPED = 'app_stopped',
  SETTINGS_UPDATE = 'settings_update',
  DATA_STREAM = 'data_stream', // Wrapper for stream data
  DASHBOARD_MODE_CHANGED = 'dashboard_mode_changed',
  DASHBOARD_ALWAYS_ON_CHANGED = 'dashboard_always_on_changed',
  WEBSOCKET_ERROR = 'websocket_error'
  // Note: Specific stream data like TranscriptionData uses StreamType directly as its type identifier.
}
```
