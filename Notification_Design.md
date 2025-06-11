# Notification Management

The following is the design outline for a "Notification" bounded context using DDD principles for a microservice in an event-driven architecture.

## Bounded Context: Notification

This context is solely responsible for managing and dispatching notifications. It doesn't care *why* a notification needs to be sent (that's the concern of other bounded contexts), only *that* it needs to be sent, *what* to send, *to whom*, and *how*.

## DDD Elements:

1.  **Aggregates:**
    *   **`NotificationRequest` (Aggregate Root)**: Represents the intent to send a notification.
        *   **Entity:** `Recipient` (could be a list of recipients if a single request can go to multiple users/devices). Each recipient might have channel-specific addresses.
        *   **Value Objects:**
            *   `NotificationRequestId` (Unique ID for the request)
            *   `MessagePayload` (Content of the notification: title, body, rich content, data payload for push)
            *   `NotificationType` (e.g., "SystemAlert", "NewComment", "PasswordReset")
            *   `ChannelPreferences` (List of preferred channels like "Email", "SMS", "Push", with potential order of preference)
            *   `Urgency` (e.g., "High", "Medium", "Low")
            *   `ScheduledTime` (Optional, for deferred notifications)
            *   `CorrelationId` (To link this notification to an originating business process or event)
            *   `Status` (e.g., "Pending", "Processing", "FailedToQueue")
        *   **Behavior:**
            *   `CreateNotificationRequest()`
            *   `MarkAsProcessing()`
            *   `MarkAsFailedToQueue()`

    *   **`SentNotificationLog` (Aggregate Root)**: Represents a notification that has been attempted or successfully sent through a specific channel.
        *   **Entity:** `DeliveryAttempt` (Tracks each attempt to send via a channel, including status and timestamp).
        *   **Value Objects:**
            *   `SentNotificationLogId` (Unique ID)
            *   `OriginalNotificationRequestId` (Links back to the `NotificationRequest`)
            *   `ChannelType` (e.g., "Email", "SMS", "Push")
            *   `RecipientAddress` (The actual address used, e.g., email address, phone number, device token)
            *   `SentTimestamp`
            *   `DeliveryStatus` (e.g., "QueuedForDispatch", "Sent", "Failed", "Delivered", "Read")
            *   `FailureReason` (If applicable)
        *   **Behavior:**
            *   `RecordDispatchAttempt()`
            *   `UpdateDeliveryStatus()`

    *   **`NotificationTemplate` (Aggregate Root)**: Manages notification message templates.
        *   **Value Objects:**
            *   `TemplateId` (Unique ID)
            *   `TemplateName`
            *   `ChannelType` (e.g., "Email", "SMS", "Push")
            *   `SubjectTemplate` (For email)
            *   `BodyTemplate` (Can contain placeholders)
            *   `Version`
        *   **Behavior:**
            *   `CreateTemplate()`
            *   `UpdateTemplate()`

    *   **(Optional) `UserNotificationPreferences` (Aggregate Root)**: If users can customize their notification settings within this microservice.
        *   **Value Objects:**
            *   `UserId`
            *   `NotificationTypePreference` (A list of: `NotificationType`, `ChannelType`, `IsEnabled`)
        *   **Behavior:**
            *   `UpdatePreferences()`

2.  **Entities (within Aggregates, as listed above):**
    *   `Recipient` (within `NotificationRequest`)
    *   `DeliveryAttempt` (within `SentNotificationLog`)

3.  **Value Objects (examples, many listed within Aggregates):**
    *   `EmailAddress`, `PhoneNumber`, `DeviceToken`
    *   `MessageContent` (Title, Body)
    *   `StatusTypes` (e.g., `NotificationStatus`, `DeliveryStatus`)
    *   `Channel` (Enum: Email, SMS, Push, InApp)

4.  **Domain Events:**
    *   `NotificationRequestedEvent` (Published when a `NotificationRequest` is successfully created and validated)
    *   `NotificationDispatchFailedEvent` (Published when a notification could not be sent to a channel gateway after retries)
    *   `NotificationSentToChannelEvent` (Published when a notification is successfully handed off to a channel gateway)
    *   `NotificationDeliveryConfirmedEvent` (If the channel provides delivery receipts)
    *   `NotificationReadEvent` (If the channel/client supports read receipts)
    *   `NotificationTemplateCreatedEvent`
    *   `NotificationTemplateUpdatedEvent`
    *   `UserNotificationPreferencesUpdatedEvent`

5.  **Domain Services:**
    *   **`NotificationDispatcherService`**:
        *   Takes a processed `NotificationRequest` (or a specific message for a channel).
        *   Interacts with external gateway services (Email, SMS, Push Notification Services).
        *   Handles channel-specific formatting or protocols.
        *   Manages retries for sending to gateways.
    *   **`NotificationTemplatingService`**:
        *   Retrieves `NotificationTemplate`.
        *   Merges template with data from the `NotificationRequest` or event payload to produce final `MessagePayload`.
    *   **`NotificationPolicyService`**:
        *   Checks user preferences (`UserNotificationPreferences`) before dispatching.
        *   Enforces rules like do-not-disturb, frequency capping, or channel fallback logic.

6.  **Repositories (Interfaces defined in Domain, Implementation in Infrastructure):**
    *   `INotificationRequestRepository`
    *   `ISentNotificationLogRepository`
    *   `INotificationTemplateRepository`
    *   `IUserNotificationPreferencesRepository` (if applicable)

## Event-Driven Flow Example:

1.  **External Bounded Context (e.g., "Ordering") publishes an event:** `OrderConfirmedEvent { OrderId, UserId, CustomerEmail, ... }`.
2.  **Notification Microservice's Application Service (Event Handler) subscribes to `OrderConfirmedEvent`**.
3.  The handler:
    *   Transforms `OrderConfirmedEvent` into a `CreateNotificationCommand`.
    *   Invokes an **Application Service** (e.g., `RequestNotificationHandler`).
4.  `RequestNotificationHandler`:
    *   Creates a `NotificationRequest` aggregate instance.
    *   Uses `NotificationTemplatingService` to get the appropriate template (e.g., "OrderConfirmationEmail") and render the message.
    *   Saves the `NotificationRequest` via its repository.
    *   The `NotificationRequest` aggregate publishes `NotificationRequestedEvent`.
5.  **Another Application Service (Event Handler) subscribes to `NotificationRequestedEvent`**.
6.  This handler:
    *   Retrieves the `NotificationRequest`.
    *   Uses `NotificationPolicyService` to check user preferences and policies.
    *   For each chosen channel:
        *   Invokes `NotificationDispatcherService` to send the notification.
        *   `NotificationDispatcherService` interacts with the external channel gateway.
    *   Based on the outcome from `NotificationDispatcherService`:
        *   Creates/updates `SentNotificationLog` aggregate.
        *   Publishes `NotificationSentToChannelEvent` or `NotificationDispatchFailedEvent`.

This design promotes loose coupling, as the Notification microservice reacts to events and uses its internal domain logic to handle the complexities of notifications without needing to know the specifics of the upstream business processes.
