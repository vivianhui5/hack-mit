# Dashboard API

# Dashboard API Reference

The Dashboard API provides an interface for displaying real-time information and status updates directly on the user's MentraOS glasses. It allows your app to send content to the dashboard.

## Import

```typescript
import { AppServer, AppSession, DashboardMode } from '@mentra/sdk';

export class MyAppServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
    // Access the dashboard API
    const dashboard = session.dashboard;

    // Send content to the main dashboard
    dashboard.content.writeToMain('🔄 Processing...');
  }
}
```

## Overview

Every [`AppSession`](/reference/app-session) exposes a `dashboard` property that provides access to the Dashboard API:

| Property                    | Type                                                | Purpose                                        |
| :-------------------------- | :-------------------------------------------------- | :--------------------------------------------- |
| `session.dashboard.content` | [`DashboardContentAPI`](#class-dashboardcontentapi) | Interface for writing content to the dashboard |

The SDK automatically converts your method calls into WebSocket messages that the MentraOS cloud routes to the user's glasses. You never have to construct layouts manually—simply provide text content and the SDK renders it in the main dashboard.

## Enum: DashboardMode

The `DashboardMode` enum defines the different display modes available on the MentraOS dashboard.

```typescript
enum DashboardMode {
  MAIN = 'main'
}
```

**Values:**

* `MAIN`: The default dashboard mode that appears as a small overlay

Use these values whenever a method accepts a `mode` parameter or when checking the current dashboard state.

## Class: DashboardContentAPI

The `DashboardContentAPI` class provides methods for sending content to the dashboard. It is automatically instantiated by the `AppSession` and available at `session.dashboard.content`.

### Constructor

The DashboardContentAPI is automatically instantiated by the AppSession. You should not create instances directly.

```typescript
class DashboardContentAPI {
  constructor(
    private wsConnection: WebSocketConnection,
    private packageName: string
  )
}
```

## Methods

### write

Send content to the dashboard.

```typescript
write(content: string): void
```

**Parameters:**

* `content`: The text content to display on the dashboard

**Returns:** void

**Example:**

```typescript
session.dashboard.content.write('✅ Task completed');
```

### writeToMain

Convenience method for sending content specifically to the main dashboard mode.

```typescript
writeToMain(content: string): void
```

**Parameters:**

* `content`: The text content to display in main mode

**Returns:** void

**Example:**

```typescript
// These are equivalent:
session.dashboard.content.writeToMain('🔄 Processing...');
session.dashboard.content.write('🔄 Processing...', [DashboardMode.MAIN]);

// Common use cases
session.dashboard.content.writeToMain('📧 3 new messages');
session.dashboard.content.writeToMain('🎵 Now playing: Song Title');
session.dashboard.content.writeToMain('⚠️ Low battery: 15%');
```

## Interface: DashboardAPI

The main dashboard interface that contains all dashboard-related functionality.

```typescript
interface DashboardAPI {
  content: DashboardContentAPI;
}
```

Every `AppSession` constructs this object and assigns it to `session.dashboard`. Currently, it only contains the `content` API, but future versions may include additional dashboard capabilities.

## Content Guidelines

### Character Limits

To ensure optimal display:

* Keep content under **60 characters** to avoid truncation

```typescript
// Good length
session.dashboard.content.writeToMain('✅ Build complete');

// Too long
session.dashboard.content.writeToMain('✅ Build completed successfully with all tests passing and no errors found');
```

### Content Replacement

The dashboard keeps only the **latest** message per app. Writing a new message automatically replaces your previous one.

```typescript
// First message
session.dashboard.content.writeToMain('🔄 Starting...');

// This replaces the previous message
session.dashboard.content.writeToMain('✅ Complete!');
```

## Message Types (Advanced)

The SDK handles these WebSocket messages automatically, but they are documented here for completeness:

| Message                  | `type` value               | Sent By  | Purpose                       |
| :----------------------- | :------------------------- | :------- | :---------------------------- |
| `DashboardContentUpdate` | `dashboard_content_update` | App      | Send new content to dashboard |
| `DashboardModeChange`    | `dashboard_mode_change`    | MentraOS | Notify of mode transitions    |
| `DashboardModeQuery`     | `dashboard_mode_query`     | App      | Request current mode          |

These correspond to TypeScript interfaces in `@mentra/sdk/src/types/dashboard`.

## Frequently Asked Questions

### Can I send layouts or images?

Not yet. The current release supports **plain text** only. Rich layouts, images, and interactive elements are planned for future releases.

### What happens if I write multiple times in a row?

The dashboard keeps only the **latest** message per app per mode. Each new message replaces the previous one for that specific mode.

```typescript
session.dashboard.content.writeToMain('First message');
session.dashboard.content.writeToMain('Second message'); // Replaces first message
```

### Is there a character limit?

Yes, to ensure optimal display:

* 60 characters maximum

Content exceeding this limit may be truncated. This limit is subject to change in future releases.
