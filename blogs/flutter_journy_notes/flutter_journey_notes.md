# Flutter journey notes

As a flutter developer, I take so many micro decisions while coding is this blog I will add stuff that I like or don't like about flutter also I will add thoughts and ideas , you can say that this blog is ***Flutter Thoughts Dump***.

- ### Flutter scrolling seems off from native scrolling
Every time I use native IOS app and Flutter I immediately feel the differance in app feeling, regarding native UI components on IOS feels so much superior to IOS componts in flutter.
I know it is really hard to match each platform user experience, that's the trade off that flutter toke.

- ### I don't like Textfield ux at all
Since flutter tries to deliver ui components that needs to fit all platforms ui/ux textfield is one of the hardest challenges that tries to solve. But I still don't like it 😁.


- ### Text selection doesn't feel natural
Specially on web, text selection is far from native web ux.

- ### Material and Cupertino should be separate packages
Material and Cupertino should be decoupled from flutter, [macos_ui](https://pub.dev/packages/macos_ui), [fluent_ui](https://pub.dev/packages/fluent_ui) .
We need to have rich [widgets](https://api.flutter.dev/flutter/widgets/widgets-library.html) lib that provide the basic ui components and every design system should rely on it.
Flutter team is working on something like this , the project called [Flutter Blank Canvas](https://docs.google.com/document/u/0/d/1rS_RO2DQ_d4_roc3taAB6vXFjv7-9hJP7pyZ9NhPOdA/mobilebasic?resourcekey=0-VBzTPoqLwsruo0j9dokuOg)

- ### Official support for more native apis 
I know flutter is a portable ui framework, but building apps isn't about UI only . But honestly flutter team and the community tries provide tools to smooth the intergration between flutter and the native platform. like :
 - Dart FFI [ffiegen](https://pub.dev/packages/ffigen) [ffi](https://pub.dev/packages/ffi)
 - JNI [Link](https://pub.dev/packages/jni)
 - JS [Link](https://pub.dev/packages/js)
 - Flutter rust bridge [Link](https://pub.dev/packages/flutter_rust_bridge)
 - Tizen interop [Link](https://pub.dev/packages/tizen_interop)
 - Pigeon [Link](https://pub.dev/packages/pigeon)
 - Native views [Link](https://docs.flutter.dev/platform-integration/android/platform-views)
 
 - ### Some packages should be adopted by flutter team
Package like Provider should be adopted by flutter since Remi will be focused on Riverpod and Provider won't get that much attention from the author sooner or later.
Technically Provider is just InheritedWidget for humans , it would be no brainer to be part of the flutter SDK. other packages like:
 - Gap
 - Nested 
 - Flutter portal