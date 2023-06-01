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