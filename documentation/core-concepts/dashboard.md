# Dashboard

> Use the MentraOS Dashboard (a persistent, glanceable surface) to show app status and content. Write content via session.dashboard.content.writeToMain().

# Dashboard Tutorial

## What is the Dashboard?

The **dashboard** is a persistent UI surface that MentraOS renders on the smart-glasses when the user looks up.  It can show system information (time, battery, status) and content contributed by apps.  Your app can treat the dashboard as an additional, lightweight display surface that remains visible even when other apps are in the foreground.

MentraOS exposes a high-level *Dashboard API* through `AppSession`.  You do not need to manage WebSocket messages or layouts manually—the SDK takes care of that.  All you have to do is call a few convenience methods on `session.dashboard.content`.

## Prerequisites

1. MentraOS SDK installed in your project.
2. A working app server with a standard `onSession` implementation (see the [Quickstart](/quickstart) guide).

## Write to the Dashboard

```typescript
session.dashboard.content.writeToMain('👋 Hello from my app!');
```

This will show the message in the main dashboard.  If your app had previous dashboard content, it will be replaced.

## Full Example

```typescript title="packages/apps/hello-dashboard/src/index.ts"
import { AppServer, AppSession } from '@mentra/sdk';

class HelloDashboardServer extends AppServer {
  protected async onSession(session: AppSession, sessionId: string, userId: string) {
    // Write a welcome message to the dashboard
    session.dashboard.content.writeToMain('👋 Hello from my app!');
  }
}
```

That's all you need, no subscriptions, no manual layout construction.  The SDK handles the heavy lifting.

## Best Practices

1. **Be concise** – Users glance at the dashboard; keep messages short.
2. **Aim for ≤60 characters** – Longer messages may be truncated.
3. **Respect user settings** – Provide toggles or frequency controls so users can decide how often content appears.

## Next Steps

* Read the [Dashboard API reference](/reference/dashboard-api) for detailed method signatures, types, and enums.
* Join our Discord community if you have questions or feedback.
