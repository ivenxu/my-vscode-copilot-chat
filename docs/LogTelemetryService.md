# LogTelemetryService - Debugging Telemetry in Development

## Overview

`LogTelemetryService` is a development-only telemetry service that logs all telemetry events to the console instead of sending them to Azure Application Insights. This is useful for:

- Debugging telemetry flow and understanding what data is being sent
- Verifying telemetry event names and properties during development
- Testing telemetry integration without affecting production metrics

## Features

- **Color-coded output**: Different colors for event types, destinations, properties, and measurements
- **Detailed logging**: Shows event name, timestamp, destination, properties, and measurements
- **Exception tracking**: Formats error stack traces for easy debugging
- **Zero network requests**: All telemetry is logged locally, nothing is sent to remote servers

## Usage

The service is automatically enabled in development mode (when not in Production). To switch between logging and no-op telemetry:

### Enable Telemetry Logging (Default)
In `src/extension/extension/vscode-node/services.ts`:
```typescript
builder.define(ITelemetryService, new LogTelemetryService());
// builder.define(ITelemetryService, new NullTelemetryService());
```

### Disable Telemetry Logging
In `src/extension/extension/vscode-node/services.ts`:
```typescript
// builder.define(ITelemetryService, new LogTelemetryService());
builder.define(ITelemetryService, new NullTelemetryService());
```

## Console Output Example

When telemetry events are fired, you'll see output like:

```
================================================================================
🔍 LogTelemetryService initialized - All telemetry will be logged to console
================================================================================

📊 MSFT Event
   Time: 2025-12-12T17:05:34.123Z
   Destination: Microsoft (External)
   Event: copilot.chat.request
   Properties:
    chatLocation: panel
    intent: explain
    languageId: typescript
   Measurements:
    duration: 1234.5
    tokenCount: 500
  ------------------------------------------------------------------------------

📊 GitHub Event
   Time: 2025-12-12T17:05:35.456Z
   Destination: GitHub (Standard)
   Event: copilot.panel.message
   Properties:
    messageType: user
    turnId: abc-123-def
  ------------------------------------------------------------------------------

💥 GitHub Exception
   Time: 2025-12-12T17:05:36.789Z
   Destination: GitHub (Standard)
   Origin: chat-participant
   Error:
    Name: Error
    Message: Failed to parse response
    Stack (first 5 lines):
      Error: Failed to parse response
        at Object.parse (/path/to/file.js:123:45)
        at ChatParticipant.handleResponse (/path/to/file.js:234:56)
        ...
  ------------------------------------------------------------------------------
```

## Event Types Logged

The service logs all ITelemetryService methods:

1. **Microsoft Telemetry**
   - `sendInternalMSFTTelemetryEvent()` - Internal Microsoft telemetry (employee-only)
   - `sendMSFTTelemetryEvent()` - External Microsoft telemetry
   - `sendMSFTTelemetryErrorEvent()` - Microsoft error events

2. **GitHub Telemetry**
   - `sendGHTelemetryEvent()` - Standard GitHub telemetry
   - `sendGHTelemetryErrorEvent()` - GitHub error events
   - `sendGHTelemetryException()` - GitHub exceptions with stack traces
   - `sendEnhancedGHTelemetryEvent()` - Enhanced GitHub telemetry (opt-in)
   - `sendEnhancedGHTelemetryErrorEvent()` - Enhanced GitHub error events

3. **Generic Telemetry**
   - `sendTelemetryEvent()` - Generic events with destination routing
   - `sendTelemetryErrorEvent()` - Generic error events

4. **Experimentation**
   - `postEvent()` - Experimentation service events
   - `setSharedProperty()` - Shared property updates
   - `setAdditionalExpAssignments()` - Experiment assignment updates

## Color Coding

- **Green**: Regular telemetry events
- **Red**: Error events and exceptions
- **Blue**: Microsoft telemetry destination
- **Magenta**: GitHub telemetry destination / measurements
- **Cyan**: Property names / shared properties
- **Yellow**: Event names / measurement values
- **Dim gray**: Timestamps and separators

## Property Truncation

Long property values (>100 characters) are automatically truncated with "..." to keep the console output readable. The full value is still captured in the actual telemetry in production.

## Debugging Tips

1. **Search for specific events**: Use your terminal's search functionality (Ctrl+F / Cmd+F) to find specific event names
2. **Filter by destination**: Search for "Microsoft" or "GitHub" to see only events sent to specific destinations
3. **Track event flow**: The timestamp shows when each event was logged, useful for understanding the sequence
4. **Verify properties**: Check that properties and measurements match your expectations before merging

## Performance Note

The logging service has minimal performance impact as it only runs in development mode and uses synchronous console.log. Colors are ANSI escape codes with no external dependencies.

## Disabling Colors

If colors don't render properly in your terminal, you can disable them by setting `enableColors = false` in the LogTelemetryService constructor.
