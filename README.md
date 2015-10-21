# OkBuck
[ ![Download](https://img.shields.io/bintray/v/piasy/maven/OkBuck.svg) ](https://bintray.com/piasy/maven/OkBuck/_latestVersion)
[![Master branch build status](https://travis-ci.org/Piasy/OkBuck.svg?branch=master)](https://travis-ci.org/Piasy/OkBuck)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-OkBuck-green.svg?style=flat)](https://android-arsenal.com/details/1/2593)

OkBuck is a gradle plugin, aiming to help developers utilize the super fast build system: BUCK, based on the existing project with Android Studio + gradle, and keep both build systems work, with less to only **10 lines of configuration**.

[Wiki](https://github.com/Piasy/OkBuck/wiki), [中文版](README-zh.md)

## Why OkBuck?
Android Studio + Gradle has already been many Android developers' option, and to migrate to buck, there are many works to be done, which are difficult and buggy. OkBuck aims to provide a gradle plugin, which will do these buggy job for you automaticlly after several lines configuration.

Further more, you can still use OkBuck to maintain your BUCK build system when your gradle configurations changes, OkBuck let you even needn't write one line of the magic BUCK script! 

## What will OkBuck do for you?
+  Generate `.buckconfig`.
+  Copy third-party libraries' jar/aar files to your rootProject directory.
+  Generate `BUCK` file for third-party libraries and each sub modules.
+  After you run `./gradlew okbuck` in your rootProject root directory, you can run `buck install app` (suppose your application module names `app` and you have already installed BUCK).

## How to use OkBuck?
1. Add this lines into buildscript dependencies part of root rootProject build.gradle (the latest version is displayed on the download badge above): 

    ```groovy
    classpath "com.github.piasy:okbuck-gradle-plugin:${latest version}"
    ```

2. Add this line into root rootProject build.gradle: 

    ```groovy
    apply plugin: 'com.github.piasy.okbuck-gradle-plugin'
    ```

3. Add this `okbuck` block into root rootProject build.gradle:
    
    ```gradle
    okbuck {
        target "android-23"
        overwrite true
        resPackages = [
            'dummylibrary': 'com.github.piasy.okbuck.example.dummylibrary',
            'app': 'com.github.piasy.okbuck.example',
        ]
    }
    ```
4. Run `./gradlew okbuck` and then run `buck install app`, enjoy your life with BUCK.

## Explanations
+  `android-23` works equally with `compileSdkVersion 23`.
+  `overwrite` is used to control whether overwrite existing buck files; 
+  OkBuck will extract signing config defined in your build.gradle, and the generated BUCK signing configs is put under `/.okbuck/keystore` folder.
  +  You needn't do anything if you have already set **only one** signing config in your build.gradle, but note that there is a default signing config named `debug` generated by android gradle plugin.
  +  But you need to configure git to ignore your signing secret, add this line to your **root rootProject .gitignore file**:

    ```
    .okbuck/keystore
    ```
  +  If you defined multiple signing configs (including the default `debug` one), or you want to put OkBuck's generated signing config in another folder which **must be under your root rootProject folder**, you can set it like below, `keystoreDir` is used to set the path OkBuck to put your generated signing config (relative to your root rootProject folder, no prefix `/`), and `signConfigName` is to set the name of the signing config you want to use.
    
    ```gradle
    okbuck {
        target "android-23"
        keystoreDir ".okbuck/keystore"
        signConfigName "release"
        overwrite true
        resPackages = [
            'dummylibrary': 'com.github.piasy.okbuck.example.dummylibrary',
            'app': 'com.github.piasy.okbuck.example',
        ]
    }
    ```
  +  But also remember to configure git to ignore your signing secret.
  +  Full example could be found in the app module of this repo, [root rootProject build.gradle](build.gradle), [app module build.gradle](app/build.gradle).

+  `resPackages` is used to set Android library module and Android Application module's package name for generated resources (`R` in common cases), you need substitute dummylibrary/app with your own module name (before the colon, if your module is not in the project root dir, i.e. in your `settings.gradle` is `include ':lib:common'`, you should use `common` rather than `:lib:common`), and set the corresponding package name after the colon, which should be the same package name specified in the corresponding module's `AndroidManifest.xml`.
    
+  There are 4 tasks added into your gradle project after apply OkBuck, `okbuck`, `okbuckDebug`, `okbuckRelease`, and `okbuckClean`.
    +  `okbuckClean` will **delete all** files generated by OkBuck.
    +  `okbuck` is the shortcut for `okbuckRelease`.
    +  `okbuckDebug` and `okbuckRelease` will make your `debugCompile` and `releaseCompile` dependencies visible in buck's build, including annotation processor.

## More jobs need to be done
OkBuck can only generate BUCK configs for you, so if your source code is incompatible with BUCK, you need do more jobs.

Read more on [Known caveats wiki page](https://github.com/Piasy/OkBuck/wiki/Known-caveats). 

Maybe there are more caveats waiting for you, but for the super fast build brought by buck, it's worthwhile.

The rest modules in this repo is a full example usage of OkBuck.

## [Known caveats](https://github.com/Piasy/OkBuck/wiki/Known-caveats)

## Troubleshooting
If you come with bugs of OkBuck, please [open an issue](https://github.com/Piasy/OkBuck/issues/new), and it's really appreciated to post the output of `./gradle okbuck --stacktrace` at the same time.

## Contribution
Contributions are welcome! See the [detail todo list](https://github.com/Piasy/OkBuck/wiki/TODO-list).

If you want to help improve OkBuck, please comment on the corresponding issue, with your expected fix due date, I'll pull you in and assign it to you, and push you.

Note that you need create an empty file named `bintray.properties` in `/buildSrc/`, it contains the bintray config, and you don't need it.

## Acknowledgement
+  Thanks for Facebook open source [buck](https://github.com/facebook/buck) build system.
+  Thanks for the discussion and help from [promeG](https://github.com/promeG/) during my development.

## Who is using OkBuck?
User | Repo
--- | ---
[Piasy](https://github.com/Piasy) | [AndroidTDDBootStrap](https://github.com/Piasy/AndroidTDDBootStrap)

If you are using OkBuck in your open source project, [send me a e-mail](mailto:xz4215@gmail.com) with your repo url, I'll add your repo in this list.

## Change log
+  0.3.1: fix [#31](https://github.com/Piasy/OkBuck/issues/31)
+  0.3.0:
  +  fix [#15](https://github.com/Piasy/OkBuck/issues/15) again.
  +  better dependency process, partly fix [Known caveats: handle dependency conflict](https://github.com/Piasy/OkBuck/wiki/Known-caveats#handle-dependency-conflict)
  +  **refactor round 2**, ready for your contributions!
+  0.2.7:
  +  fix overwrite doesn't work bug
  +  fix bug when Android library module doesn't have res dir
  +  .buckconfig will ignore .svn dirs
  +  fix [#6](https://github.com/Piasy/OkBuck/issues/6) again...
+  0.2.6: fix [#14](https://github.com/Piasy/OkBuck/issues/14)
+  0.2.5: fix [#15](https://github.com/Piasy/OkBuck/issues/15)
+  0.2.4: fix [#12](https://github.com/Piasy/OkBuck/issues/12)
+  0.2.3: fix [#11](https://github.com/Piasy/OkBuck/issues/11)
+  0.2.2: fix [#6](https://github.com/Piasy/OkBuck/issues/6)
+  [v0.2.1](https://github.com/Piasy/OkBuck/releases/tag/v0.2.1)
+  [v0.1.0](https://github.com/Piasy/OkBuck/releases/tag/v0.1.0)
+  [v0.0.1](https://github.com/Piasy/OkBuck/releases/tag/v0.0.1)