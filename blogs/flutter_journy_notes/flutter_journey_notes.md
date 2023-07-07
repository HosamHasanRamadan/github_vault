# Flutter journey notes

As a flutter developer, I take so many micro decisions while coding is this blog I will add stuff that I like or don't like about flutter also I will add thoughts and ideas , you can say that this blog is ***Flutter Thoughts Dump***.

### Flutter scrolling seems off from native scrolling
Every time I use native IOS app and Flutter I immediately feel the difference in app feeling, regarding native UI components on IOS feels so much superior to IOS components in flutter.
I know it is really hard to match each platform user experience, that's the trade off that flutter toke.


### `BuildContext` IS YOUR FRIEND, DON'T FIGHT IT  🤗

### I don't like Textfield UX at all
Since flutter tries to deliver ui components that needs to fit all platforms ui/ux textfield is one of the hardest challenges that tries to solve. But I still don't like it 😁. and also TextEditingController is not declarative. Butting this aside I don't know yet if two way binding or declarative api in better for textfield , what I am sure about that forms with two way binding is way easier to handle . I don't know  any forms' package that has 100% declarative api. [Reactive Forms](https://pub.dev/packages/reactive_forms) uses two-way binding and it can handle very complex forms. 


### Text selection doesn't feel natural
Specially on web, text selection is far from native web ux.

### Material and Cupertino should be separate packages
Material and Cupertino should be decoupled from flutter like [macos_ui](https://pub.dev/packages/macos_ui), [fluent_ui](https://pub.dev/packages/fluent_ui) .
We need to have rich [widgets](https://api.flutter.dev/flutter/widgets/widgets-library.html) lib that provide the basic ui components and every design system should rely on it.
Flutter team is working on something like this , the project called [Flutter Blank Canvas](https://docs.google.com/document/u/0/d/1rS_RO2DQ_d4_roc3taAB6vXFjv7-9hJP7pyZ9NhPOdA/mobilebasic?resourcekey=0-VBzTPoqLwsruo0j9dokuOg)

### Official support for more native apis 
I know flutter is a portable ui framework, but building apps isn't about UI only . But honestly, flutter team and the community try to provide tools to smooth the integration between flutter and the native platform, like :
 - Dart FFI [ffiegen](https://pub.dev/packages/ffigen) [ffi](https://pub.dev/packages/ffi)
 - JNI [Link](https://pub.dev/packages/jni)
 - JS [Link](https://pub.dev/packages/js)
 - Flutter rust bridge [Link](https://pub.dev/packages/flutter_rust_bridge)
 - Tizen interop [Link](https://pub.dev/packages/tizen_interop)
 - Pigeon [Link](https://pub.dev/packages/pigeon)
 - Native views [Link](https://docs.flutter.dev/platform-integration/android/platform-views)
 
### Some packages should be adopted by flutter team
Package like Provider should be adopted by flutter since Remi will be focused on Riverpod and Provider won't get that much attention from him sooner or later.
Technically, Provider is just InheritedWidget for humans , it would be no-brainer to be part of the flutter SDK. other packages like:
 - Gap
 - Nested 
 - Flutter portal

### Flutter form is very basic and got really messy easily with slightly complex form
[Reactive Froms](https://pub.dev/packages/reactive_forms) to the rescue, you're welcome 

### Isolate support on flutter web

### You have to be updated, unless your code will break
Flutter moves really fast and breaks really fast, so you have to be always up-to-date.

### Static extensions are useless

### Showing dialogs using navigation stack
Using the navigation stack to show dialog is a crime , thanks god flutter_portal and stack is here.

### Material without flex scheme is just broken
Material [Mike](https://twitter.com/RydMike) , that's it.

### Reverting native package configuration is a nightmare
Just try to remove Firebase from your project , it's not a piece of cake.
The simplest way is to delete the native file and call `flutter create .` again 

### Why `mainAxis` and `crossAxis` for `Flex`
Every time I use `Row` or `Column` I feel like my brain glitched , What is wrong with `Vertical` and `Horizintal` 

### I miss `Checked Exception`
I really miss checked exception and also catching error in flutter is really weird, it should be forbidden. 
we have `@alwaysThrows` but is not enough and no one uses it .

### Elevated and Raised button
when flutter wanted to change button api they introduced  new buttons with  new names . so then if flutter wants to change the api one more time they will change it back to Raised button . closed loop 😅.

### `useMaterial3` is a software engineering failure

### No direct solution for conditional nesting 
- conditional nesting ( nested ,parameters )
- check [Container](https://github.com/flutter/flutter/blob/d3d8effc68/packages/flutter/lib/src/widgets/container.dart#L240) implementation

### Expand tap area
Change hit area without changing widget size . flutter_portal to the rescue.

### Using flutter without native experience is a huge fallacy

### Sign out reset state ways 
-  restart app (flutter_phonix)
-  Invalidate state - (riverpod)
-  Event bus -  Simple api like https://pub.dev/packages/notification_dispatcher
-  Scopes - (getit scopes , ProviderContainer)

### They tried to fix nav 1 with nav 2 which no one uses directly 😁

### Restart app doesn't like global keys

### Analysis server , Restart 🚀

### Riverpod , why I can't implement custom provider

### Riverpod doesn't have `ValueNotifer` provider or `ValueListenable` provider

### Why we have two `async` and `collection` libs

### Use generated data models 
Using freezed , built_value will help you in serialization , copyWith ..etc . And also will avoid mistakes when model scheme changed and forget to apply those changes to copyWith or to/from map.

### Observable pattern using push or pull data when notified
I started to feel that pull data after notification is  is better than pushing it . ChangNotifier vs ValueNotifier , Fight 🤼‍♀️

### You have to have a unified global place to handle platform permissions

### Global Event Stream is awesome for whom who concerned 


### I hate `try catch.
Returning result with value and error is better, but it should in the core of the lang like rust. Exception propagation is a huge question that need to be taken in consideration before starting the app

### You should know when function have to fail 
function should fail in a situation of doing so many checks where they aren't its responsibility
example:
```dart
void do(){
	if(checkA()) return;
	if(checkB()) return ;
	if(checkC()) return ;
}
```
it is better to fail and let checks which isn't function responsibility  to others , like:
```dart
void do(){
	if(checkA()) return;
	if(checkB()) return ;
	/// Fails because `C` isn't valid not theri responsibility check 
	/// rest of function
}
```

### Forked packages names 
package _x , package_plus , package_2 , package_fixes_x ..... Craaaaap 😠

### Dart disposable and serializable interfaces should be part of Dart SDK


### Ideas are good implementation is a hell
Developing and maintaining big apps is a pain  in the a** , specially when you have limitation from the Flutter SDK and other lib you are using inside the app , is freaking hell
- https://github.com/flutter/flutter/issues/65538
  I need to clean up some services on app close , which apparently I can't do
- https://github.com/rrousselGit/riverpod/issues/2645
  The app behaves very weird when screen is locked while running


### I hate the nature of lazy loaded providers in Riverpod 😡

### Business logic name consistency
For example if you want  change football to soccer in your app that's a huge change in terms of backend and frontend . I don't know how to overcome this problem without producing a lot of changes and shitty workarounds.
Also, it is very crucial between the teams' communication.

#### Where this piece of  update should live ?! 🤷‍♂️

### Rethink flutter native communication
When I see SDKs like [Expo](https://docs.expo.dev/versions/latest/?redirected) and [Compose](https://www.jetbrains.com/lp/compose-multiplatform/) multi platform. 
**Expo**: Has very rich native api with very good support from the Expo team
**Compose**: has seamless interoperability between Kotlin and native .
I need to explore more in **Compose** it seems very good competitor to Flutter 

### Cross Platform Solutions:
- Flutter - Dart
- Compose Multiplateform - Kotlin
- Xamarin - C#
- Avalonia UI - C#
- Uno - C#
- React Native -  JS/TS
- Tauri - JS/TS - Rust
- Expo - JS/TS
- Cordove - JS/TS
- PhoneGap - JS/TS
- Ionic - JS/TS
- NativeScript - JS/TS  

How to choose between them:
- Community support
- Architecture 
- Native communication
- Performance 
- Language personal pref

### We need easy bridge for native features
I was trying to use overlay windows in android which is being implanted by community package, and it's not reliable . Most of the time you want to use part of native APIs and I want  to implement it my self, time proved me using community plugin packages is pain in the a** .