I was handed a task to handle system notification for medication reminder, the notification will contain two action **Take** and **Skip**.
The notification should work in all app states **foreground**, **background** and **terminated**.
Let's start with those basic requirements and add more as we go.

This blog will a QA form for the challenges I faced to implement this functionality

Tools helped me along the way:
- [Mockoon](https://mockoon.com/) - Use as logging server instead of using `print` in console
- [Google dev playground](https://developers.google.com/oauthplayground) - for getting auth token to use Google APIS for sending notifications
- [Hoppscotch](https://hoppscotch.io/) - Http client used for sending notification through Google API
### What libraries should use to achieve this functionality ?

```yml
firebase_messaging: ^14.6.8
permission_handler: ^10.4.5
flutter_local_notifications: ^15.1.1
```

### How to listen to remote notification in all app states ?

1- Make sure that notification permission is configured and granted 
```dart
// in AndroidManifist.xml add this permession
// <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
Permission.notification.request();
```

2- We need init `Firebase` in `main` function
```dart 
await Firebase.initializeApp();
```

3- Config notification callbacks
```dart
FirebaseMessaging.onBackgroundMessage(_onBackgroundMessage);
FirebaseMessaging.onMessage.listen(_onForegroundMessage);
```

4- implement notification callbacks
```dart
@pragma('vm:entry-point')
Future<void> _onBackgroundMessage(RemoteMessage message) async {}

void _onForegroundMessage(RemoteMessage message) {}  
```

- `_onBackgroundMessage` : triggered when app receives remote notification in background and terminated  
- `_onForegroundMessage` : triggered when app receives remote notification in foreground

Don't forget `@pragma('vm:entry-point')` for background callback to work properly.
Now we can show notification with actions in all states.

***Hint***: Sometimes `FirebaseMessaging` callbacks don't work properly without calling `FirebaseMessaging.instance.getToken()` first.

### How to add actions to remote notifications ?

We can't add actions to notification directly using `firebase_messaging` package only, we need use `flutter_local_notifications` to implement this functionally

1- Init and config `flutter_local_notifications`
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

2- Impanelment action callbacks
```dart
void _onForegroundNotificationResponse(NotificationResponse details) {}

@pragma('vm:entry-point')
void _onBackgroundNotificationResponse(NotificationResponse details) {}
```

- `_onForegroundNotificationResponse` triggered when app receives notification action in foreground
- `_onBackgroundNotificationResponse` : triggered when app receives notification action in background and terminated
Don't forget `@pragma('vm:entry-point')` for background callback to work properly.
Now we can get a show notification with actions in all states.

3- We need to receive ***silent remote notification*** or also called ***data only notification*** from the backend and then show local notification with actions once device receives remote notifications

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
}```

4- show local notification with actions on `_onBackgroundMessage` and `_onForegroundMessage` callbacks using  `FlutterLocalNotificationsPlugin.show(...)` method

### How to redirect user to specific UI if app was opened  from notification tap ?

Calling `FlutterLocalNotificationsPlugin().getNotificationAppLaunchDetails()` in `main` function to know if app was opened from notification or not.
### How to bring app to foreground when app receives remote notification on background or terminated state ?

Using `android_intent_plus` and `package_info_plus` will help us achieve this feature:
1-  Make sure that `SYSTEM_ALERT_WINDOW` permission is granted
```dart
/// <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
// Used to show full app notifcation on lock screen
 Permission.systemAlertWindow.request();
```

2- use android `Intent` to open or bring it to foreground
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
- Also if we need to bring app to foreground when screen in locked we need to add `USE_FULL_SCREEN_INTENT` permission to manifest
```xml
<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />
```
- To check if app was launched using android `Intent` we need to check launch `arguments` using `launch_args` package or simply implement native android function and call it using method channel and call in `main` function
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

Using `flutter_exit_app` package to close the app completely whenever we want

### How to show in app specific UI if app receives notification on foreground or background state ?

there is no problem to react to notification events on foreground state because all listeners are registered in ***main isolate*** . the problem when background event was triggered , we need to find a way of communication between ***main isolate*** and ***other isolates***.

Using `IsolateNameServer` API we can achieve simple isolate communication by registering  `UIIsolateCommunicationChannel` in the main thread by calling  `forceRegister` in main function and using `listen` function to listen to data sent through this channel using `send` to the `UIIsolateCommunicationChannel`.

```dart
abstract class UIIsolateCommunicationChannel {
  static final _receivePort = ReceivePort();
  static const name = 'ui_isolate';

  static void register() => IsolateNameServer.registerPortWithName(
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