# Settings

> Learn how to define and use app settings in MentraOS—supported types, real-time synchronization, access patterns, and best practices with code examples.

MentraOS provides a powerful settings system that allows apps to offer customizable experiences to users. Settings can be configured through the developer console and are automatically synchronized with your app, allowing users to personalize their experience through the MentraOS app.

```typescript
// Example of accessing settings in your app
import {AppServer, AppSession} from "@mentra/sdk"

export class MyAppServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Get a specific setting value
    const language = session.settings.get<string>("transcribe_language", "English")

    // Listen for setting changes
    session.settings.onValueChange("line_width", (newValue, oldValue) => {
      console.log(`Line width changed from ${oldValue} to ${newValue}`)
      this.updateDisplay(newValue)
    })
  }
}
```

## What Are App Settings?

App settings are user-configurable options that control how your application behaves. They are:

* **Persistent**: Settings are stored in the cloud and synchronized across devices
* **Type-safe**: Each setting has a defined type (toggle, text, select, etc.)
* **Real-time**: Changes are immediately synchronized to your running app
* **User-friendly**: Displayed in the MentraOS app with proper UI controls

Settings allow users to customize aspects like:

* Display preferences (langauges, layouts)
* Feature toggles (enable/disable functionality)
* User login or API keys

## Defining Your Settings

Settings are defined in the developer console. Go to [console.mentra.glass/apps](https://console.mentra.glass/apps) and edit your app, then look for the "App Settings" section.

<img src="https://mintlify.s3.us-west-1.amazonaws.com/mentra/img/settings-editor.png" alt="App Settings Section" />

* **`key`**: The unique identifier for the setting, used in your app code
* **`label`**: The human-readable name of the setting shown in the settings UI
* **`defaultValue`**: The default value for the setting
* **`type`**: The type of the setting (toggle, text, select, etc.)
* **`options`**: For select settings, a list of allowed values
  * **`label`**: The human-readable name of the option shown in the settings UI
  * **`value`**: The unique value of the option, used in your app code
* **`min`** and **`max`**: For slider settings, the minimum and maximum values

### Setting Types

MentraOS supports several setting types:

#### Toggle (Boolean)

On/off switches for boolean values.

```json
{
  "type": "toggle",
  "key": "show_subtitles",
  "label": "Show Subtitles",
  "defaultValue": true
}
```

#### Text Input

Text input with a save button, that only updates when the user presses the save button.

```json
{
  "type": "text",
  "key": "user_name",
  "label": "Display Name",
  "defaultValue": "User"
}
```

#### Text (No Save Button)

Multi-line text input without save button. It saves whenever the user makes a change.

```json
{
  "type": "text_no_save_button",
  "key": "notes",
  "label": "Notes",
  "defaultValue": "",
  "maxLines": 5
}
```

#### Select (Dropdown)

Single choice from predefined options. The default value should be a single value matching the value of one of the options.

```json
{
  "type": "select",
  "key": "theme",
  "label": "Color Theme",
  "options": [
    {"label": "Light", "value": "light"},
    {"label": "Dark", "value": "dark"},
    {"label": "Auto", "value": "auto"}
  ],
  "defaultValue": "auto"
}
```

#### Select with Search

Dropdown with search functionality for long lists. The default value should be a single value matching the value of one of the options.

```json
{
  "type": "select_with_search",
  "key": "language",
  "label": "Language",
  "options": [
    {"label": "English", "value": "en"},
    {"label": "Spanish", "value": "es"},
    {"label": "French", "value": "fr"}
  ],
  "defaultValue": "en"
}
```

#### Multiselect

Multiple choices from options. The default value should be an array of values (which may be empty).

```json
{
  "type": "multiselect",
  "key": "enabled_features",
  "label": "Enabled Features",
  "options": [
    {"label": "Auto-save", "value": "autosave"},
    {"label": "Notifications", "value": "notifications"},
    {"label": "Analytics", "value": "analytics"}
  ],
  "defaultValue": ["autosave"]
}
```

#### Slider

Numeric value selection.

```json
{
  "type": "slider",
  "key": "font_size",
  "label": "Font Size",
  "min": 10,
  "max": 30,
  "defaultValue": 16
}
```

#### Group

Organize settings into logical sections. Displays the title as a group seperator. Use this to delieate the start of a new section of settings.

```json
{
  "type": "group",
  "title": "Display Settings"
}
```

#### Title/Value Display

Read-only display of information, shown in the UI as the title (label) and the value.

```json
{
  "type": "titleValue",
  "label": "App Version",
  "value": "1.0.0"
}
```

For additional information on the types, see [Setting Types Reference](/reference/interfaces/setting-types).

## Accessing Settings in Your App

### Using the Settings Manager

The app session provides a `settings` property with methods to access and monitor settings:

```typescript
import {AppServer, AppSession} from "@mentra/sdk"

export class MyAppServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Get a setting value with type safety
    const fontSize = session.settings.get<number>("font_size", 16)
    const theme = session.settings.get<string>("theme", "auto")

    // Check if a setting exists
    if (session.settings.has("show_subtitles")) {
      const showSubtitles = session.settings.get<boolean>("show_subtitles")
      this.toggleSubtitles(showSubtitles)
    }

    // Get all settings
    const allSettings = session.settings.getAll()
    console.log("Current settings:", allSettings)
  }
}
```

### Listening for Setting Changes

React to setting changes in real-time:

```typescript
import {AppServer, AppSession} from "@mentra/sdk"

export class MyAppServer extends AppServer {
  private cleanupHandlers: Array<() => void> = []

  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Listen for any setting change
    const cleanup1 = session.settings.onChange(changes => {
      console.log("Settings changed:", changes)
      // changes is a map of key -> { oldValue, newValue }
      for (const [key, change] of Object.entries(changes)) {
        console.log(`${key}: ${change.oldValue} → ${change.newValue}`)
      }
    })

    // Listen for specific setting changes
    const cleanup2 = session.settings.onValueChange("theme", (newTheme, oldTheme) => {
      console.log(`Theme changed from ${oldTheme} to ${newTheme}`)
      this.applyTheme(newTheme)
    })

    // Store cleanup functions
    this.cleanupHandlers.push(cleanup1, cleanup2)
  }

  protected async onStop(sessionId: string, userId: string, reason: string): Promise<void> {
    // Clean up listeners
    this.cleanupHandlers.forEach(cleanup => cleanup())
  }
}
```

## Common Patterns

### Feature Toggles

```typescript
import {AppServer, AppSession} from "@mentra/sdk"

export class FeatureToggleServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Check initial state
    this.updateFeatures(session)

    // Listen for toggle changes
    session.settings.onValueChange("enable_advanced_mode", enabled => {
      this.updateFeatures(session)
    })
  }

  private updateFeatures(session: AppSession): void {
    const advancedMode = session.settings.get<boolean>("enable_advanced_mode", false)

    if (advancedMode) {
      this.enableAdvancedFeatures()
    } else {
      this.disableAdvancedFeatures()
    }
  }
}
```

### Language Selection

```typescript
import {AppServer, AppSession} from "@mentra/sdk"

export class MultilingualServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    const language = session.settings.get<string>("ui_language", "en")
    this.setLanguage(language)

    // Update subscriptions based on language
    this.updateTranscriptionSubscription(session)

    // Listen for language changes
    session.settings.onValueChange("ui_language", newLang => {
      this.setLanguage(newLang)
      this.updateTranscriptionSubscription(session)
    })
  }

  private updateTranscriptionSubscription(session: AppSession): void {
    const transcribeLang = session.settings.get<string>("transcribe_language", "en-US")

    // Unsubscribe from all transcription streams
    session.events.unsubscribe("transcription")

    // Subscribe to language-specific stream
    session.onTranscriptionForLanguage(transcribeLang, data => {
      this.handleTranscription(data)
    })
  }
}
```

## Best Practices

### 1. Provide Sensible Defaults

Always specify appropriate default values that work for most users:

```typescript
// Good: Sensible defaults
{
  "key": "line_delay",
  "defaultValue": 100,  // default
  "min": 10,
  "max": 300
}

// Avoid: No default or extreme values
{
  "key": "line_delay",
  "min": 1,
  "max": 10000
  // Missing defaultValue
}
```

### 2. Group Related Settings

Use group settings to organize related options:

```typescript
[
  { "type": "group", "title": "Display Settings" },
  { "key": "font_size", "type": "slider", ... },
  { "key": "text_color", "type": "select", ... },
  { "key": "line_width", "type": "slider", ... },

  { "type": "group", "title": "Privacy Settings" },
  { "key": "save_history", "type": "toggle", ... },
  { "key": "analytics_enabled", "type": "toggle", ... }
]
```

### 3. Use Descriptive Labels

Make settings self-explanatory:

```typescript
// Good: Clear and descriptive
{
  "key": "transcribe_language",
  "label": "Transcription Language",
}

// Avoid: Vague or technical
{
  "key": "trans_lang",
  "label": "Trans Lang"
}
```

### 4. Clean Up Listeners

Remove setting change listeners when your app stops:

```typescript
import {AppServer, AppSession} from "@mentra/sdk"

export class CleanupExampleServer extends AppServer {
  private settingsCleanup: Array<() => void> = []

  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Store cleanup functions
    this.settingsCleanup.push(
      session.settings.onValueChange("setting1", this.handleSetting1),
      session.settings.onValueChange("setting2", this.handleSetting2),
      session.settings.onChange(this.handleAnyChange),
    )
  }

  protected async onStop(sessionId: string, userId: string, reason: string): Promise<void> {
    // Clean up all listeners
    this.settingsCleanup.forEach(cleanup => cleanup())
    this.settingsCleanup = []
  }
}
```

## Settings Lifecycle

1. **Definition**: Settings are defined in the developer console
2. **Storage**: Settings are stored in MentraOS Cloud
3. **Synchronization**: When an app starts, settings are automatically loaded
4. **User Changes**: Users modify settings through the MentraOS Mobile App
5. **Real-time Sync**: Changes are immediately pushed to running apps
6. **Persistence**: Settings persist across app restarts and devices

## Related Documentation

* [Setting Types Reference](/reference/interfaces/setting-types) - Detailed type definitions for settings
* [Settings Manager](/reference/managers/settings-manager) - API reference for the settings manager
* [Getting Started with Apps](/getting-started) - Complete guide to building an app
* [Core Concepts](/core-concepts) - Understanding app architecture
