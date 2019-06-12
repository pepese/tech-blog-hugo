---
title: "Androidアプリ入門 Macで環境構築編"
date: 2018-01-06T07:41:30+09:00
slug: "android-env-on-mac"
tags:
- android
- kotlin
draft: false
archives:
- 2018/01
---

Mac で Android アプリを開発する際の環境構築についてまとめる。  
なお、プログラミング言語は [Kotlin](https://kotlinlang.org/) を選択する。

[公式ドキュメント](https://developer.android.com/studio/intro/index.html)

<!--more-->

# Android Studio の設定

Java 1.8 はインストール済みな前提。  
バージョンは 3.0.1 for Mac 。

## インストール

```sh
$ brew update
$ brew cask install android-studio
```

Homebrew については [Homebrew 入門](https://pepese.github.io/blog/homebrew-basics/) を参照。

## 初回起動時の設定

android-studio でインストールした android-sdk 系のツールは `~/Library/Android/sdk` 配下に配備される。  
以下の初回起動時の設定後にダウンロードされる。

1. Android Studio 起動
2. 「 Do not import settings 」 -> 「 OK 」
3. 「 Next 」
4. 「 custom 」 -> 「 Next 」
5. 「 Default 」 -> 「 Next 」
6. 以下を選択して「 Next 」
    - Android SDK
    - Android SDK Platform
    - Performance (Intel HAXM)
    - Android Virtual Device
7. 任意の Emulator Settings して「 Next 」 -> 「 Finish 」

「[ダウンロードしたアプリケーションの実行許可]の下の方に intel なんたらかんたら」って出たら、Mac の「システム環境設定　> セキュリティとプライバシー」を開いて許可する。  
`~/.bash_profile` に以下を加筆した後 `source ~/.bash_profile` を実行。

```
export JAVA_HOME=`/usr/libexec/java_home -v1.8`
export ANDROID_HOME=$HOME/Library/Android/sdk
PATH=$PATH:$JAVA_HOME/bin
PATH=$PATH:$ANDROID_HOME/platform-tools
PATH=$PATH:$ANDROID_HOME/tools/bin

export PATH
```

`$ adb devices` 、 `$ sdkmanager --help` が実行できることを確認しておく。

## アンインストール

```sh
$ brew cask uninstall android-studio
$ rm -Rf ~/Library/Preferences/AndroidStudio*
$ rm -Rf ~/Library/Logs/AndroidStudio*
$ rm -Rf ~/Library/Caches/AndroidStudio*
$ rm ~/Library/Preferences/com.android.Emulator.plist
$ rm -Rf ~/Library/Android*
$ rm -Rf ~/AndroidStudioProjects
$ rm -Rf ~/.gradle
$ rm -Rf ~/.android
```

# Android Studio の使い方

1. Android Studio 起動
2. 「 Start a new Android Project 」
3. アプリ名、ドメイン名、「 Include Kotlin Support 」にチェックを入れて「 Next 」
4. アプリのプラットフォームを選択して「 Next 」
5. コンポーネント（ Basic Activity でよい）を選択して「 Next 」
6. メインとなる Activity の名前などを設定して「 Next 」->「 Finish 」
    - この時点でプロジェクトの初期設定環境
    - エラーメッセージが出た場合は、メッセージに従い SDK Manager からツールをダウンロードする
7. エミュレータの作成
    - AVD Manager から作成できる
8. 右上の再生ボタンでアプリを実行可能

「 Shift 」を2回押すと、 Search Everywhere が起動する。

# プロジェクト構造

Android プロジェクト構造は以下。  
実際のディレクトリ構造と異なることに注意。

- app
    - manifest ： AndroidManifest.xml がある
    - java ： テストコードを含む Java ソースコードのファイルがある
    - res ： XML レイアウト、UI 文字列、ビットマップ イメージなど、コード以外のすべてのリソースがある（ resource の略）
        - drawable ： 画像
        - layout ： Activity のレイアウト
        - menu ： アプリのメニューのレイアウト
        - mipmap ： 拡大縮小アニメーションに対応した画像
        - values ： 文字列や値を XML で管理
- Gradle Script ： ビルドファイルがある
    - build.gradle (Project: xxxx) ： プロジェクトのすべてのモジュールに適用されるビルド設定
    - build.gradle (Module: app) ： `<project>/<module>/` ディレクトリにあるモジュールレベルのビルド設定
    - gradle-wrapper.properties
    - proguard-rules.pro ： ProGuard はビルド時にアプリのソースコードを難読化するツール
    - gradle.properties ： Gradle デーモンの最大ヒープサイズなど、プロジェクト全体にわたる Gradle 設定
    - settings.gradle ： アプリをビルドするときに含めるモジュールを Gradle に通知するファイル
    - local.properties ： SDK インストールへのパスなど、ビルドシステムのローカル環境プロパティを設定

[公式ドキュメント：ビルドシステム](https://developer.android.com/studio/build/index.html)

## マニフェスト

`AndroidManifest.xml` のこと。  
記載方法は以下を参照。

[公式ドキュメント：アプリ マニフェスト](https://developer.android.com/guide/topics/manifest/manifest-intro.html)

# NDK の導入

Android NDK （ Native Development Kit ）は、C や C++ などのネイティブ コード言語を使用して、アプリの一部を実装するためのツールセット。

1. Android Studio 起動
2. SDK Manager 起動
3. 「 SDK Tools 」から「 NDK 」「 CMake 」「 LLDB 」を選択してインストール
4. NDK を利用するプロジェクトでは「 Include C++ support 」をチェックしておく必要がある

NDK に関するビルドの設定は、モジュールレベルの `build.gradle` で行う。  
例えば以下は、cpp のビルドターゲットの .so を armeabi 、 armeabi-v7a に限定する設定。

```groovy
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "sample.pepese.org.audiosample"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags ""
            }
        }
        ndk { // ★★★★★★★このあたり！！！！★★★★★★★
            // Specifies the ABI configurations of your native
            // libraries Gradle should build and package with your APK.
            abiFilters 'armeabi', 'armeabi-v7a'
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'
    implementation 'com.android.support:design:26.1.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'
}
```
