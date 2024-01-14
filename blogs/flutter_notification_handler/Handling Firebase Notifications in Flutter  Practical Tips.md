---
title: Handling Firebase Notifications in Flutter: Practical Tips
published: false
description: 
series: Flutter AppImage
tags: flutter, fcm, notifications ,firebase
series: Firebase Notifications in Flutter
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o2bazpxla52vr1m8alfh.png
# published_at: 2024-01-01 18:36 +0000
---

I was assigned the task of handling system notifications for medication reminders. The notifications will include two actions: **Take** and **Skip**. The notification should function in all app states, including **foreground**, **background**, and **terminated**. Let's begin with these basic requirements and add more as we progress.

This blog will serve as a QA form for the challenges I faced while implementing this functionality.

Tools helped me along the way:
- [Mockoon](https://mockoon.com/) - Used as a logging server instead of using `print` in the console.
- [Google Dev Playground](https://developers.google.com/oauthplayground) - Used for obtaining an auth token to use Google APIs for sending notifications.
- [Hoppscotch](https://hoppscotch.io/) - HTTP client used for sending notifications through the Google API.

### What libraries should use to achieve this functionality ?

```yml
firebase_messaging: ^14.6.8
permission_handler: ^10.4.5
flutter_local_notifications: ^15.1.1
```

### How to listen to remote notification in all app states ?

1- Ensure that notification permission is configured and granted
```dart
// in AndroidManifist.xml add this permession
// <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
Permission.notification.request();
```

2- Initialize Firebase in the `main` function.
```dart 
await Firebase.initializeApp();
```

3- Configure notification callbacks.
```dart
FirebaseMessaging.onBackgroundMessage(_onBackgroundMessage);
FirebaseMessaging.onMessage.listen(_onForegroundMessage);
```

4- Implement notification callbacks.
```dart
@pragma('vm:entry-point')
Future<void> _onBackgroundMessage(RemoteMessage message) async {}

void _onForegroundMessage(RemoteMessage message) {}  
```

- `_onBackgroundMessage`: Triggered when the app receives a remote notification in the background and terminated.
- `_onForegroundMessage`: Triggered when the app receives a remote notification in the foreground.

Don't forget `@pragma('vm:entry-point')` for the background callback to work properly. Now we can show notifications with actions in all states.

_**Hint**_: Sometimes, `FirebaseMessaging` callbacks don't work properly without calling `FirebaseMessaging.instance.getToken()` first.
### How to add actions to remote notifications ?

We can't add actions to notification directly using `firebase_messaging` package only, we need use `flutter_local_notifications` to implement this functionally

1- Initialize and configure `flutter_local_notifications`.
```dart
// call this function in main
Future<void> initNotificationConfig() async {
  await FlutterLocalNotificationsPlugin().initialize(
    const InitializationSettings(
      android: AndroidInitializationSettings('@mipmap/ic_launcher'),
    ),
    onDidReceiveNotificationResponse: _onForegroundNotificationResponse,
    onDidReceiveBackgroundNotificationResponse: _onBackgroundNotificationResponse,
  );
}
```

2- Implement action callbacks.
```dart
void _onForegroundNotificationResponse(NotificationResponse details) {}

@pragma('vm:entry-point')
void _onBackgroundNotificationResponse(NotificationResponse details) {}
```

- `_onForegroundNotificationResponse`: Triggered when the app receives a notification action in the foreground.
- `_onBackgroundNotificationResponse`: Triggered when the app receives a notification action in the background and terminated.
Don't forget `@pragma('vm:entry-point')` for background callback to work properly.
Now we can show notification with actions in all states.

3- Receive ***silent remote notifications*** or ***data-only notifications*** from the backend and then show local notifications with actions once the device receives remote notifications.

```json
{
    "message": {
        "token": "",
        "android": {
            "priority": "HIGH"
        },
        "data": {
            "title": "string",
            "body": "string"
        }
    }
}
```

4- Show local notifications with actions on `_onBackgroundMessage` and `_onForegroundMessage` callbacks using the `FlutterLocalNotificationsPlugin.show(...)` method.

### How to redirect the user to a specific page if the app was opened from a notification tap?

Call `FlutterLocalNotificationsPlugin().getNotificationAppLaunchDetails()` in the `main` function to know if the app was opened from a notification or not.
### How to bring app to foreground when app receives remote notification on background or terminated state ?

Using `android_intent_plus` and `package_info_plus` will help us achieve this feature:

1-  Make sure that `SYSTEM_ALERT_WINDOW` permission is granted
```dart
/// <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
// Used to show full app notifcation on lock screen
 Permission.systemAlertWindow.request();
```

2- use android `Intent` to open or bring app to foreground
```dart
Future<void> _bringAppToForeground() async {
  final package = await PackageInfo.fromPlatform();
  final packageName = package.packageName;
  final intent = AndroidIntent(
    action: 'android.intent.action.MAIN',
    flags: [Flag.FLAG_ACTIVITY_NEW_TASK],
    category: 'android.intent.category.LAUNCHER',
    arguments: {'args': 'run flutter app automatically from notification'},
    package: packageName,
    componentName: '$packageName.MainActivity',
  );
  return intent.launch();
}
```

- we need to check `arguments` on app start to check if app was opened automatically from background
- Also if we need to bring app to foreground when screen in locked we need to add `USE_FULL_SCREEN_INTENT` permission to manifest, and config main activity
  
```xml
<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />

<!-- add this keys to activity -->
<activity
	...
	android:showWhenLocked="true"
	android:turnScreenOn="true">
</activity>

```

- To check if app was launched using the android `Intent` we need to check launch `arguments` using `launch_args` package or simply implement native android function and call it using method channel and call it in `main` function
  
```kotlin
private fun getIntentArgs(): Map<String,Any?>?{
     val args = intent.extras
     val map = mutableMapOf<String,Any?>();
     args?.keySet()?.forEach{
         map[it] = args[it]
     }
     return  if(args == null) null else map
}
```

```dart
const _methodChannel = MethodChannel('channel_name');
Future<Map<String, dynamic>?> getIntentArgs() async {
  final argsResult = await _methodChannel.invokeMapMethod('getIntentArgs');
  final args = argsResult?.cast<String, dynamic>();
  return args;
}
```

### How to close app once user responds to notification if app was opened from notification ?

Using the `flutter_exit_app` package to close the app completely whenever we want.
### How to Display App-Specific UI when Receiving Notifications in Foreground or Background?

While reacting to notification events in the foreground has no challenges since all listeners are registered in the _**main isolate**_, the complexity arises when dealing with background events. To address this, a communication mechanism between the _**main isolate**_ and _**other isolates**_ becomes essential.

The `IsolateNameServer` API comes to our rescue in establishing straightforward isolate communication. We achieve this by registering the `UIIsolateCommunicationChannel` in the main thread using the `forceRegister` method. Additionally, the `listen` function facilitates the reception of data through this channel, making use of the `send` method to communicate with the `UIIsolateCommunicationChannel`.

```dart
abstract class UIIsolateCommunicationChannel {
  static final _receivePort = ReceivePort();
  static const name = 'ui_isolate';

  static bool register() => IsolateNameServer.registerPortWithName(
        _receivePort.sendPort,
        name,
      );

  static void forceRegister() {
    final isRegistered = IsolateNameServer.registerPortWithName(
      _receivePort.sendPort,
      name,
    );

    if (isRegistered == false) {
      IsolateNameServer.removePortNameMapping(name);
      IsolateNameServer.registerPortWithName(
        _receivePort.sendPort,
        name,
      );
    }
  }

  static StreamSubscription listen(void Function(dynamic) listener) => _receivePort.listen(listener);

  static void send(dynamic value) => IsolateNameServer.lookupPortByName(name)?.send(value);

  static bool unregister() => IsolateNameServer.removePortNameMapping(name);

  static bool isRegistered() => IsolateNameServer.lookupPortByName(name) != null;
}
```
***Hints***:
- Supported types for communication channel [Link](https://api.dart.dev/stable/3.2.0/dart-isolate/SendPort/send.html). 
- Also, make sure to use primary constructors from `List` and `Map` to pass them through communication channel to work on release mode . [issue reference](https://github.com/flutter/flutter/issues/113183#issuecomment-1285473165)
```dart
UIIsolateCommunicationChannel.send((data.toList())) // fails in release mode ❌
UIIsolateCommunicationChannel.send((List.of(data))) //  ✅
```

**Resources:**
https://firebase.google.com/docs/cloud-messaging/concept-options
https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#Notification
https://pub.dev/packages/flutter_local_notifications
https://api.flutter.dev/flutter/dart-ui/IsolateNameServer-class.html
https://medium.com/flutter/introducing-background-isolate-channels-7a299609cad8