# NativeScript Raven TraceWriter

This plug-in provides a custom NativeScript TraceWriter that will log messages to Sentry.io using the [Raven.js JavaScript library](https://www.npmjs.com/package/raven-js).

## Getting started

To use this plug-in, you must first obtain a "DSN" key from [Sentry.io](https://sentry.io/welcome/). This key is used to initialize the plug-in and send logs to your specific Sentry.io account.

To add the plug-in to your project:

`tns plugin add nativescript-trace-raven`

## Usage

Once the plug-in is installed, you need to simply initialize the new TraceWriter. This can be done in different ways, but for easy global usage, setup the new TraceWriter when your app starts:

**app.ts**
```typescript
import * as app from 'application';
import * as trace from 'trace';
import { TraceRaven } from 'nativescript-trace-raven';

app.on(app.launchEvent, (args: app.ApplicationEventData) => {
    let sentryDsn = "[YOUR SENTRY DSN KEY]";

    trace.setCategories(trace.categories.concat(trace.categories.Error, trace.categories.Debug));
    trace.addWriter(new TraceRaven(sentryDsn));
    trace.enable();
});
```
Then, in your app, just use trace as normal and the output will be sent to Sentry.io.

**Example:**
```typescript
trace.write("Something happened in the app", trace.categories.Error, trace.messageType.error);
```

In addition to your trace message, this plug-in will auto-log these additional details to Sentry:

- NativeScript runtime version
- Device Platform (iOS/Android)
- Device OS Version
- Device Type (phone/tablet/etc)
- Device Model (iPhone, Galaxy S3, etc)
- Device Language (en-US, etc)
- Device UUID
- Device Orientation (portrait, landscape)
- Battery level (0 - 100, if available)
- App version name
- Environment name (default "Debug")
- User IP Address

### Log level
Sentry.io provides three levels for classifying logs: info, warning and error.

When logging using the TraceWriter and `trace` API, the `trace.messageType` is mapped to Sentry log levels:

- `trace.messageType.log` === Log type: `info`
- `trace.messageType.info` === Log type: `info`
- `trace.messageType.warn` === Log type: `warning`
- `trace.messageType.error` === Log type: `error`

If `trace.messageType` is omitted, the default log level is `error`.

## Extended use

When initializing Raven, you can optionally provide an `environment` string to describe where the app is running when sending log messages. By default, this string is set to `debug`. If you want to specify your own environment string, just add it when initializing with your DSN key:

```typescript
trace.addWriter(new TraceRaven(sentryDsn, "production"));
```

### Additional Raven APIs
The default TraceWriter API only provides a `write` method, but the Raven.js library provides additional capabilities such as logging exception detail. To use these additional APIs, you can directly use the Raven library. As long as you've initialized `TraceRaven` on app startup, all Raven configuration will be set. To learn more about the Raven APIs, [visit the JavaScript docs on Sentry.io](https://docs.sentry.io/clients/javascript/usage/).

#### Logging Exceptions
Unlike using `trace` to write record an Error, the `captureException` API will also attempt to include Stack Trace information with the log.
```typescript
import Raven = require("raven-js");

try { 
  throw new Error("This is an example error with stack trace");
} catch (err) {
  Raven.captureException(err);
}
```

#### Adding Breadcrumb
You can manually create "breadcrumbs" that will be included with Sentry logs. Breadcrumbs are intended to show the path of actions that lead to an exception, app crash or log message. For example, to add a crumb when a button is tapped:

```typescript
public buttonTap(args: EventData) {
  let btn = <Button>args.object;
  Raven.captureBreadcrumb({
    message: `Button tapped`,
    category: "action",
    data: {
      id: btn.id,
      text: btn.text
    },
    level: "info"
  });
}
```

#### Last EventId
The `EventId` is a globally unique string generated by Sentry for all logs. Raven provides the ability to get the `EventId` for the most recent log so that you can present it to users and use for customer service reports. To get the most recent `EventId` with Raven:

```typescript
let eventId = Raven.lastEventId();
```

## Considerations
Sentry.io provides a generous free tier for logging events, but does eventually charge by logging volume. As a result, you want to be careful to log only events that are helpful for troubleshooting in production.

**That means you do not want to use `trace.categories.All` when logging to Sentry**

This verbose logging will likely generate far more logs than you need, and quickly run-up your Sentry.io bill.

Best practices:

1. Only log to Sentry in production
2. Minimize the `trace` categories logged (minimum: `trace.categories.Error`)

### Native Errors
Since this plug-in is running in the NativeScript JavaScript layer, it may not capture all native iOS or Android errors. This is generally okay as you will get errors that relate to your app code, but if you need logging at the native iOS/Android level, you'll need to use a different plug-in.

## Auto Breadcrumbs
In addition to providing the custom TraceWriter, this plugin will automatically wire-up automatic breadcrumbs for these global `Page` events:

- `onNavigatedTo`
- `onLoaded`
- `onShownModally`

Whenever one of these events occurs, a new breadcrumb will get added to the history. To disable this behavior, initialize `TraceWriter` with an additional parameter:

```typescript
new TraceWriter("[YOUR DSN KEY]", "production", false)
```

The last parameter will enable/disable auto-breadcrumbs created by this plug-in. Default is `true` (enabled).

## Using the Demo
If you would just like to try the demo for this plug-in, simply follow these steps:

1. Get a DSN key from Sentry.io
2. Clone this repo
3. Navigate to the `demo` folder and open `app.ts` in your code editor
4. Replace the `sentryDsn` string with your DSN key
5. Navigate back to the root of the cloned repo
6. Run `npm run demo.ios` or `npm run demo.android`

If you do not add your DSN key before running the demo, the app will crash on launch.

## Known Issues

- Stack Trace detail for exceptions does not get sent from Android emulator (not sure about devices)
- No way in JavaScript API with current Raven-JS plug-in to disable auto-breadcrumbs for console, xhr, etc.

## Contributing
Want to help make this plug-in better? Report issues in GitHub:
https://github.com/toddanglin/nativescript-trace-raven/pulls

Pull requests welcome.