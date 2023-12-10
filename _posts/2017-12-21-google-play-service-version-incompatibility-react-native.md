---
layout: post
title: Resolving Google Play services Incompatibility in React-Native(Android)
---
In react-native when we use any external library where `play-services` are required like `react-native-maps` for google map there is an android project into this library where play-services v8.x.x included into gradle and in your project you are using play-services v9.x.x for another reason

![react-native-android-ios](https://cdn57.androidauthority.net/wp-content/uploads/2016/09/react-native-logo.jpg)

We are using `react-native` for our mobile application where have to build and deploy both android and ios from same branch. Last week our frontend (mobile) team were facing an issue with `play-services` version incompatibility and `multidex` issue. That MultiDex is a common issue in native android for different version of google-play-services and we can solve it by two way, either we can make a common version for all types play-services dependencies or in gradle we can set `multidex : true`. So it's easy for native.

But in react-native when we use any external library where `play-services` are required like `react-native-maps` for google map there is an android project into this library where `play-services v8.x.x` included into `gradle` and in your project you are using play-services v9.x.x for another reason. You will get an error for inconsistency. How you will resolve this? Obviously it's not the way that editing the external module's `android/gradle.build` you have to make your own play-service version by following that external libs dependency, right ? ok solved, it's fine.

 ** What if!,**  you have to keep your version for any other reason or you have to include another external library like `react-native-firebase` where also using `google-play-services v10.x.x.` Now what you will do?

 That's the scenario our team were facing and they are modifying external libs `gradle` for making both version same as interim solution.

 As I was an Android Engineer, I know some gradle syntax, So I figured it out. This in an incompatibility between Google Play Services 8.4 and Firebase 9.0 (ideally, they should probably be kept on the same version). The hack I did that, to stop including `play services` from external libs fucked them both by exclude and I've declared my own play services into my gradle what they both needed.

My gradle ended up looking like this, though you will most likely need to understand what is going on in the below snippet to adapt it for your purposes.

```
dependencies {
    compile (project(':react-native-maps')) {
        // exclude version 8.+
        exclude group: 'com.google.android.gms', module: 'play-services-base'
        exclude group: 'com.google.android.gms', module: 'play-services-maps'
    }

    compile(project(':react-native-firebase')) {
        // exclude version 10.+
        transitive = false
        exclude group: 'com.google.android.gms', module: 'play-services-base'
        exclude group: 'com.google.firebase', module: 'firebase-core'
        exclude group: 'com.google.firebase', module: 'firebase-config'
        exclude group: 'com.google.firebase', module: 'firebase-auth'
        exclude group: 'com.google.firebase', module: 'firebase-database'
        exclude group: 'com.google.firebase', module: 'firebase-storage'
        exclude group: 'com.google.firebase', module: 'firebase-messaging'
        exclude group: 'com.google.firebase', module: 'firebase-crash'
        exclude group: 'com.google.firebase', module: 'firebase-perf'
        exclude group: 'com.google.firebase', module: 'firebase-ads'
        exclude group: 'com.google.firebase', module: 'firebase-firestore'
        exclude group: 'com.google.firebase', module: 'firebase-invites'
    }

    // include whatever firebase packages we want:
    compile 'com.google.firebase:firebase-core:9.0.0'
    // Now re-include an up-to-date version of the packages we excluded above.
    // I believe these basically need to match what we do with firebase, to make things work again
    compile 'com.google.android.gms:play-services-base:9.0.0'
    compile 'com.google.android.gms:play-services-maps:9.0.0'
    ........
    ......

}

```
> Note: No need to share it I'm not using Adsense but if you think that will help other guys then you can share.

Thanks for reading :)