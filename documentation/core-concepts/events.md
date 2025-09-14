# Events

> Subscribe to MentraOS Cloud real-time events (transcription, head position, button presses, notifications, battery, and more) using the EventManager. Learn how to subscribe/unsubscribe and review all event types with their data structures.

# MentraOS Cloud Events

MentraOS Cloud uses an event-driven architecture to communicate real-time data from the smartglasses to your MentraOS app. Your app can *subscribe* to specific events and receive data as it becomes available. This document describes how to subscribe to events, the available event types, and the data structure for each event.

## Subscribing to Events

The [`AppSession`](/reference/app-session) class in the `@mentra/sdk` provides several ways to subscribe to events.  You'll typically do this within the [`onSession`](/reference/app-server#onsession-protected) method of your [`AppServer`](/reference/app-server):

```typescript
protected async onSession(session: AppSession, sessionId: string, userId: string): Promise<void> {
  // ... your session setup ...

  // Subscribe to transcription events
  const unsubscribe = session.events.onTranscription((data) => {
    // Handle transcription data
    console.log(data.text);
  });

  // ... other event subscriptions ...

  // **Important:** Unsubscribe when the session ends.
  this.addCleanupHandler(unsubscribe);
}
```

`session.events` is an [`EventManager`](/reference/managers/event-manager) instance that provides a consistent interface for all event subscriptions.  All subscription methods return an `unsubscribe` function, which you *must* call when you no longer need the subscription (typically when the session ends).

### Subscription Methods

You can subscribe to events using the [`EventManager`](/reference/managers/event-manager). These methods are type-safe for each event type.

```typescript
session.events.onTranscription((data: TranscriptionData) => { ... });
session.events.onHeadPosition((data: HeadPosition) => { ... });
session.events.onButtonPress((data: ButtonPress) => { ... });
// ... other methods ...
```

### Unsubscribing from Events

Event subscription functions return a function that can be called to unsubscribe from the event.  Call this when you no longer need a subscription, for example to stop listening to transcription events when the user requests turning off the microphone.

```typescript
const unsubscribe = session.events.onTranscription((data) => {
  // ... handle transcription ...
});

// Later, when you no longer need the subscription:
unsubscribe();
```

## Available Events

The following table lists the available event types, their descriptions, and the corresponding data types.  You can find the full list with more details in the [`EventManager`](/reference/managers/event-manager) reference.

| Event Type                     | Description                                            | Data Type                                                                            |
| :----------------------------- | :----------------------------------------------------- | :----------------------------------------------------------------------------------- |
| Transcription                  | Real-time speech-to-text transcription.                | [`TranscriptionData`](/reference/interfaces/event-types#transcriptiondata)           |
| Translation                    | Real-time translation of transcribed text.             | [`TranslationData`](/reference/interfaces/event-types#translationdata)               |
| Head Position                  | User's head position ('up' or 'down').                 | [`HeadPosition`](/reference/interfaces/event-types#headposition)                     |
| Button Press                   | Hardware button press on the glasses.                  | [`ButtonPress`](/reference/interfaces/event-types#buttonpress)                       |
| Phone Notification             | Notification received from the user's connected phone. | [`PhoneNotification`](/reference/interfaces/event-types#phonenotification)           |
| Glasses Battery Update         | Battery level update from the glasses.                 | [`GlassesBatteryUpdate`](/reference/interfaces/event-types#glassesbatteryupdate)     |
| Phone Battery Update           | Battery level update from the phone.                   | [`PhoneBatteryUpdate`](/reference/interfaces/event-types#phonebatteryupdate)         |
| Glasses Connection State       | Connection status of the glasses.                      | [`GlassesConnectionState`](/reference/interfaces/event-types#glassesconnectionstate) |
| Voice Activity Detection (VAD) | Indicates whether voice activity is detected.          | [`Vad`](/reference/interfaces/event-types#vad-voice-activity-detection)              |
| Notification Dismissed         | User dismissed a notification.                         | [`NotificationDismissed`](/reference/interfaces/event-types#notificationdismissed)   |
| Audio Chunk                    | Raw audio data (for advanced use cases).               | [`ArrayBuffer`](/reference/interfaces/event-types#audiochunk)                        |
