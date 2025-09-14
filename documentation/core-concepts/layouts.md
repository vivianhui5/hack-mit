# Layouts

> Use pre-defined layouts with the LayoutManager to render text, cards, and bitmaps on smart glasses in MentraOS Cloud. Includes APIs, options like duration and view, utilities, and best practices.

MentraOS Cloud provides a set of pre-defined layouts for displaying content on the smart glasses' AR view. These layouts allow you to create visually consistent and user-friendly interfaces without needing to deal with low-level graphics programming. You interact with layouts through the [`LayoutManager`](/reference/managers/layout-manager), which is a property of the [`AppSession`](/reference/app-session) object.

```typescript
// Example of accessing the LayoutManager
session.layouts.showTextWall("Hello, World!")
```

## Available Layout Types

The SDK currently supports the following layout types:

### 1. `TextWall`

* **Purpose:** Displays a single block of text. This is the most basic layout and is suitable for simple messages, status updates, and short notifications.

* **Type:** [`LayoutType.TEXT_WALL`](/reference/enums#layouttype)

* **Properties:**
  * `text`: The text content to display (string).

* **Example:**

  ```typescript
  import {AppSession, LayoutType, ViewType, AppToCloudMessageType} from "@mentra/sdk"

  // Assuming you have a AppSession instance called 'session'
  session.layouts.showTextWall("Connected to MentraOS Cloud")

  // Example of a complete DisplayRequest for a TextWall.
  // You don't normally construct this directly; the LayoutManager does it.
  const displayRequest: DisplayRequest = {
    type: AppToCloudMessageType.DISPLAY_REQUEST,
    view: ViewType.MAIN,
    packageName: "com.example.myapp",
    sessionId: "some-session-id",
    layout: {
      layoutType: LayoutType.TEXT_WALL,
      text: "Hello, TextWall!",
    },
    timestamp: new Date(),
    durationMs: 5000, // Optional duration
  }
  ```

### 2. `DoubleTextWall`

* **Purpose:** Displays two blocks of text, one above the other. This is useful for showing two related pieces of information, such as a title and a subtitle, or a before/after comparison.

* **Type:** [`LayoutType.DOUBLE_TEXT_WALL`](/reference/enums#layouttype)

* **Properties:**
  * `topText`: The text for the top section.
  * `bottomText`: The text for the bottom section.

* **Example:**

  ```typescript
  session.layouts.showDoubleTextWall("Original Text:", "Translated Text:")
  ```

### 3. `ReferenceCard`

* **Purpose:** Displays a card with a title and content text. This is a more structured layout, suitable for displaying important information, notifications with titles, or key data points.

* **Type:** [`LayoutType.REFERENCE_CARD`](/reference/enums#layouttype)

* **Properties:**
  * `title`: The title of the card (string).
  * `text`: The main content of the card (string).

* **Example:**

  ```typescript
  session.layouts.showReferenceCard("Meeting Reminder", "Team Standup in 5 minutes")
  ```

### 4. `DashboardCard`

* **Purpose:** Displays a card with left-aligned and right-aligned text. This layout is ideal for key-value pairs, metrics, and dashboard-style displays.

* **Type:** [`LayoutType.DASHBOARD_CARD`](/reference/enums#layouttype)

* **Properties:**
  * `leftText`: Text for the left side (often a label or key).
  * `rightText`: Text for the right side (often a value or status).

* **Example:**

  ```typescript
  session.layouts.showDashboardCard("Battery Level", "85%")
  ```

### 5. `BitmapView`

* **Purpose:** Displays bitmap images on the AR glasses. Supports monochrome bitmaps at 526x100 (576×135 when padded) pixel resolution.
  > Only use 526x100 images unless you know what you're doing! (the file size should be \~7-10kb)

* **Type:** [`LayoutType.BITMAP_VIEW`](/reference/enums#layouttype)

* **Properties:**
  * `data`: Bitmap data as hex or base64 encoded string.

* **Example:**

  ```typescript
  // Display a single bitmap image
  const bitmapData = "424d461a00000000000036000000..." // hex encoded BMP data
  session.layouts.showBitmapView(bitmapData)
  ```

### 6. `ClearView`

* **Purpose:** Clears the current display, returning to a blank state. Useful for resetting the view before showing new content.

* **Type:** [`LayoutType.CLEAR_VIEW`](/reference/enums#layouttype)

* **Properties:** None.

* **Example:**

  ```typescript
  // Clear the display
  session.layouts.clearView()
  // Clear specific view
  session.layouts.clearView({view: ViewType.DASHBOARD})
  ```

## Bitmap Animation

The LayoutManager provides animation support for bitmap sequences:

```typescript
/**
 * Shows an animated sequence of bitmap images
 * @param bitmapDataArray - Array of bitmap data strings (hex encoded)
 * @param intervalMs - Time between frames in milliseconds
 * @param repeat - Whether to loop the animation continuously
 * @param options - Optional parameters (view)
 * @returns Animation controller object with stop() method
 */
session.layouts.showBitmapAnimation(
  bitmapDataArray: string[],
  intervalMs?: number,
  repeat?: boolean,
  options?: { view?: ViewType }
): { stop: () => void }
```

**Example:**

```typescript
// Simple 3-frame loading animation
const loadingFrames = [
  "424d461a000000...", // Frame 1 hex data
  "424d461a000001...", // Frame 2 hex data
  "424d461a000002...", // Frame 3 hex data
]
session.layouts.showBitmapAnimation(loadingFrames, 1650, false)
// Continuous loop animation
const animation = session.layouts.showBitmapAnimation(frames, 1650, true)
// Stop the animation
setTimeout(() => animation.stop(), 10000)
```

## Bitmap Utilities

The SDK provides utility classes for working with bitmap images:

### BitmapUtils

```typescript
import {BitmapUtils} from "@mentra/sdk"
// Load a single BMP file
const bmpHex = await BitmapUtils.loadBmpAsHex("./assets/icon.bmp")
session.layouts.showBitmapView(bmpHex)
// Load multiple animation frames
const frames = await BitmapUtils.loadBmpFrames("./animations", 10)
session.layouts.showBitmapAnimation(frames, 1650, true)
// Validate bitmap data
const validation = BitmapUtils.validateBmpHex(bmpHex)
if (!validation.isValid) {
  console.error("Invalid bitmap:", validation.errors)
}
```

### AnimationUtils

```typescript
import {AnimationUtils} from "@mentra/sdk"
// Create animation from files
const animation = await AnimationUtils.createBitmapAnimation(
  session,
  "./animations", // Base path
  10, // Frame count
  {
    intervalMs: 1650, // Hardware-optimized timing
    repeat: true,
    onFrame: (frame, total) => console.log(`Frame ${frame}/${total}`),
  },
)
// Use optimized settings for hardware
const config = AnimationUtils.getOptimizedConfig("even-realities-g1")
const optimizedAnimation = await AnimationUtils.createBitmapAnimation(session, "./frames", 8, config)
```

## Display Duration (`durationMs`)

All layout methods accept an optional `options` object, which can include a `durationMs` property. This property controls how long the layout is displayed on the glasses, in milliseconds.

* **`durationMs: number` (optional):** The duration, in milliseconds, to display the layout.
* **`durationMs: -1`:** Indicates that the layout should be displayed persistently, until it is explicitly replaced or cleared. This is useful for content that should remain visible until further notice.
* **Omitting `durationMs`:** If you omit `durationMs`, the behavior depends on whether you are sending interim transcription.
  If it's provided, the layout will remain until replaced or cleared. If the duration is omitted the current `DisplayManager` implementation will display for 20 seconds. This behavior should not be relied upon. Always provide `durationMs` in production code.

```typescript
// Display for 5 seconds
session.layouts.showTextWall("This will disappear in 5 seconds", {durationMs: 5000})

// Display persistently (until replaced or cleared)
session.layouts.showReferenceCard("Important", "This will stay on screen", {durationMs: -1})
```

## View Type

The `options` object in the layout methods also allows you to specify a `view` property:

```typescript
interface LayoutOptions {
  view?: ViewType
  durationMs?: number
}
```

* **[`ViewType.MAIN`](/reference/enums#viewtype) (default):** The main AR view that the user sees most of the time.
* **[`ViewType.DASHBOARD`](/reference/enums#viewtype):** A special dashboard view. Currently, the user looks up to access the dashboard.

```typescript
// Display in the main view (default)
session.layouts.showTextWall("Hello in main view")

// Display in the dashboard view
session.layouts.showTextWall("Dashboard Message", {view: ViewType.DASHBOARD})
```

## Layout Interfaces

The `@mentra/sdk` provides TypeScript interfaces for each layout type. These interfaces ensure type safety and provide autocompletion in your IDE. You can import these directly:

```typescript
import {TextWall, DoubleTextWall, ReferenceCard, LayoutType, ViewType} from "@mentra/sdk"

const textWallLayout: TextWall = {
  layoutType: LayoutType.TEXT_WALL,
  text: "Hello, TextWall!",
}
```

## Best Practices

* **Keep it Concise:** The glasses display has limited space. Use short, clear text. Avoid long paragraphs or complex layouts.
* **Prioritize Information:** Display the most important information prominently.
* **Use Appropriate Layouts:** Choose the layout type that best suits the content you're displaying.
* **Consider Head Position:** The [`head_position`](/reference/interfaces/event-types#headposition) event lets you know if the user is looking up (at the dashboard) or down (at the main view). You can use this to tailor the displayed content.
* **Avoid Flicker:** Rapidly changing the display can be visually jarring. Use appropriate durations and consider debouncing or throttling updates if necessary.
* **Text Wrapping:** The glasses have limited width, so it is important to handle text that is longer than can fit on one line. The `@mentraos/utils` package provides a `wrapText` utility function to handle this.
