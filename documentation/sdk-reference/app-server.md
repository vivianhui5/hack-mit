# AppServer

> Base class for building MentraOS app servers that handle webhooks from Mentra Cloud to start/stop sessions, process tool calls, expose an Express app, and generate authentication tokens.

# AppServer

`AppServer` (also known as `TpaServer` in older versions) is the base class for creating Third Party Application (app) servers that handle webhook requests from MentraOS Cloud to manage app sessions.

```typescript
import { AppServer } from '@mentra/sdk';
```

## Constructor

```typescript
constructor(config: AppServerConfig)
```

**Parameters:**

* `config`: [Configuration](#configuration) options for the app server

## Methods

### getExpressApp()

Exposes the internal Express app instance for adding custom routes or middleware.

```typescript
getExpressApp(): express.Express
```

**Returns:** The Express application instance.

### onSession() *\[protected]*

Override this method to handle the initiation of a new app session when a user starts your app. Implement your app's core logic here (e.g., setting up event listeners).

```typescript
protected onSession(
  session: AppSession,
  sessionId: string,
  userId: string
): Promise<void>
```

**Parameters:**

* `session`: The [`AppSession`](/reference/app-session) instance for this specific user session
* `sessionId`: The unique identifier for this session
* `userId`: The unique identifier for the user

**Returns:** A Promise that resolves when session initialization is complete

### onStop() *\[protected]*

Override this method to handle cleanup when an app session is stopped by the user or system.

```typescript
protected onStop(
  sessionId: string,
  userId: string,
  reason: string
): Promise<void>
```

**Parameters:**

* `sessionId`: The unique identifier for the session being stopped
* `userId`: The unique identifier for the user
* `reason`: The reason the session was stopped ('user\_disabled', 'system\_stop', 'error')

**Returns:** A Promise that resolves when session cleanup is complete

### onToolCall() *\[protected]*

Override this method to handle tool calls from Mira AI. This is where you implement your app's tool functionality.

For a comprehensive guide on implementing AI tools, see [AI Tools](/tools).

```typescript
protected onToolCall(
  toolCall: ToolCall
): Promise<string | undefined>
```

**Parameters:**

* `toolCall`: A [ToolCall](/reference/interfaces/tool-types#toolcall) object containing the tool ID, parameters, user ID, and timestamp

**Returns:** A Promise that resolves to an optional string response that will be sent back to Mira

**Example:**

```typescript
protected async onToolCall(toolCall: ToolCall): Promise<string | undefined> {
  console.log(`Tool called: ${toolCall.toolId}`);
  console.log(`Tool call timestamp: ${toolCall.timestamp}`);
  console.log(`Tool call userId: ${toolCall.userId}`);

  if (toolCall.toolParameters && Object.keys(toolCall.toolParameters).length > 0) {
    console.log("Tool call parameter values:", toolCall.toolParameters);
  }

  if (toolCall.toolId === "add_todo") {
    const reminder = addReminder(
      toolCall.userId,
      toolCall.toolParameters.todo_item as string,
      toolCall.toolParameters.due_date as string | undefined
    );
    return `Added reminder: ${reminder.text}`;
  }

  return undefined;
}
```

### start()

Starts the app server, making it listen for incoming webhook requests.

```typescript
start(): Promise<void>
```

**Returns:** A promise that resolves when the server has successfully started.

### stop()

Gracefully shuts down the app server, cleaning up all active sessions and resources.

```typescript
stop(): void
```

### generateToken() *\[protected]*

Generates a JWT token suitable for app authentication, typically used for webviews. See [Token Utilities](/reference/token-utils) for more details.

```typescript
protected generateToken(
  userId: string,
  sessionId: string,
  secretKey: string
): string
```

**Parameters:**

* `userId`: The user's identifier
* `sessionId`: The session identifier
* `secretKey`: Your app's secret key (should match the one configured in MentraOS Cloud)

**Returns:** The generated JWT token string

### addCleanupHandler() *\[protected]*

Registers a function to be executed during the server's graceful shutdown process.

```typescript
protected addCleanupHandler(handler: () => void): void
```

**Parameters:**

* `handler`: The cleanup function to add

## Configuration

```typescript
interface AppServerConfig {
  /** Your unique app identifier (e.g., 'org.company.appname'). Must match console.mentra.glass. */
  packageName: string;

  /** Your API key obtained from console.mentra.glass for authentication. */
  apiKey: string;

  /** The port number the app server will listen on. Defaults to 3000. */
  port?: number;

  /** Path to a directory for serving static files (e.g., images, logos). Set to `false` to disable. Defaults to `false`. */
  publicDir?: string | false;

  /** Whether to enable the `/health` endpoint for status checks. Defaults to `true`. */
  healthCheck?: boolean;
}
```
