I was handed a task to handle system notification for medication reminder, the notification will contain two action **Take** and **Skip**.
The notification should work in all app states **foreground**, **background** and **terminated**.
Let's start with those basic requirements and add more as we go.

This is tested on flutter 3.3.10
google apis token : https://developers.google.com/oauthplayground
http client : https://hoppscotch.io/
I will use Mockoon to as mocked local server and I will use it for logging
Required Dependencies:
```
firebase_messaging: ^14.6.8
android_intent_plus: ^4.0.2
dio: ^4.0.6
permission_handler: ^10.4.5
flutter_local_notifications: ^15.1.1
flutter_exit_app: ^1.1.2
launch_args: ^2.0.1+2
```

I will log to local server instead of `print` using `dio` and Mockoon server:

```dart
void logToServer(Map<String, dynamic> payload) {
  final localServer = 'http://192.168.1.42:3000/log';
  Dio().post(
    localServer,
    data: payload,
  );
}
```

We will start with Android implementation first:
in android if want to show notification with action button you need to receive a data only using  `firebase_messaging` then show local notification with action using `flutter_local_notifications`.
We need send silent high priority notification from the backend to the app After some trials with notification payload, this payload was enough to my use case:
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

1- make sure that notification permission is granted, put in a proper place in your app
```dart
// <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
Permission.notification.request();
```

2- we need to configure `flutter_local_notifications` and `firebase_messaging` with their callbacks

```dart
Future<void> initNotificationConfig() async {
  await FlutterLocalNotificationsPlugin().initialize(
    const InitializationSettings(
      android: AndroidInitializationSettings('@mipmap/ic_launcher'),
      iOS: DarwinInitializationSettings(),
    ),
    onDidReceiveNotificationResponse: _onForegroundNotificationResponse,
    onDidReceiveBackgroundNotificationResponse: _onBackgroundNotificationResponse,
  );
  FirebaseMessaging.onBackgroundMessage(_onBackgroundMessage);
  FirebaseMessaging.onMessage.listen(_onForegroundMessage);
}
```

**Hint**: Sometimes `FirebaseMessaging` callbacks don't work properly without calling `FirebaseMessaging.instance.getToken()` first.

3- Implement notification callbacks
```dart
@pragma('vm:entry-point')
Future<void> _onBackgroundMessage(RemoteMessage message) async {}

void _onForegroundMessage(RemoteMessage message) {}  

void _onForegroundNotificationResponse(NotificationResponse details) {}

@pragma('vm:entry-point')
void _onBackgroundNotificationResponse(NotificationResponse details) {}
```

- `_onBackgroundMessage` : triggered when app receives remote notification in background and terminated  
- `_onForegroundMessage` : triggered when app receives remote notification in foreground
- `_onForegroundNotificationResponse` triggered when app receives notification action in foreground
- `_onBackgroundNotificationResponse` : triggered when app receives notification action in background and terminated
don't forget `@pragma('vm:entry-point')` for background callback to work properly.
Now we can get a show notification with actions in all states.

If user opened the app from the notification we to redirect the user to a specific page on app start and then close the once the user responds to the reminder, in this case we need to know if the user opened the from notification. `FlutterLocalNotificationsPlugin().getNotificationAppLaunchDetails()` 

4- **New Requirement :** in-app notification
We need to show in-app notification in some cases when receive system notification.

There is no problem when receive remote notification in foreground because the callbacks are handled in the main isolate.

The problem is with background callbacks because they are running in different isolate.

So we need to build a communication channel between main isolate and other isolates.

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

5- **New Requirement :** open app when receive remote notification
 app should be opened automatically when receives remote notification when terminated or in background. `systemAlertWindow` is required for this feature to work.
 
 ```dart
// <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
 Permission.systemAlertWindow.request();
```

To open the app from background/terminated, we need to use `android_intent_plus`:

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
   
- **Terminated**
  When app is terminated, we should open the app only on in-app notification page and once user answers the in-app notification, the app should be closed automatically.
  1- We need to launch app with `arguments` and check args on `main` function using `launch_args` package, or you can write android native function and call it using method channel
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
  
  2- We close the using `flutter_exit_app` package once the user answers the question.




  
Resources:
https://firebase.google.com/docs/cloud-messaging/concept-options
https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages#Notification
https://pub.dev/packages/flutter_local_notifications
https://api.flutter.dev/flutter/dart-ui/IsolateNameServer-class.html
https://medium.com/flutter/introducing-background-isolate-channels-7a299609cad8