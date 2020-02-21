---
title: "Appium入門"
date: 2018-01-06T07:44:15+09:00
slug: "appium-basics"
tags:
- appium
- android
- ios
draft: false
archives:
- 2018/01
---

Appium は Selenium WebDriver の一種で、ネイティブアプリの自動テストを可能にする。  
Node.js 上でサーバーとして動作し、HTTP 経由で WebDriver API を通して操作を受け付けるという仕組み。  
Appium の背後には iOS 用, Android 用, Win 用などのドライバがある。

- [API Reference](http://appium.io/slate/en/master/)
- [Tutorial Android](http://appium.io/slate/en/tutorial/android.html)
- [Tutorial iOS](http://appium.io/slate/en/tutorial/ios.html)

<!--more-->

# 環境設定

Homebrew 、 Java 1.8 の導入は省略している。

## Appium

```sh
$ brew update
$ brew install node
$ npm install --global appium appium-doctor wd
```

Appium は以下で起動。  
後述の appium-desktop を使用せず、 Appium のテストコードだけ実行する場合はこれ。

```sh
$ appium &
```

## iOS

Xcode を App Store でインストールしてから以下を実行。  
また、 `brew install carthage` を実行する際、権限不足で `/usr/local/Frameworks` ディレクトリ作成に失敗する。  
そのため、 `brew link carthage` に失敗するので、あらかじめディレクトリを作ってあげてからインストールする。

```sh
$ sudo mkdir /usr/local/Frameworks
$ sudo chown -R $(whoami):admin /usr/local/Frameworks
$ brew install carthage
# $ brew link carthage
$ appium-doctor --ios # インストール、設定が正しいかチェック
$ brew install libimobiledevice --HEAD # 実機接続用のモジュール
$ npm install --global --unsafe-perm ios-deploy
$ sudo xcode-select --switch /Applications/Xcode.app # Xcode のバージョンを指定
```

 その他 iOS に関する細々した設定は[ここ](https://github.com/appium/appium-xcuitest-driver/blob/master/docs/real-device-config.md)を参照。

### シミュレータ

特に必要無いが、シミュレータの画面サイズが大きいのでやってもいい。

1. Xcode を起動
2. Xcode -> Open Developper Tool -> Simulator
3. Window -> Scale -> 50%
4. ホームボタンは Shift + Command + H

### 実機

iOS の場合は、「実機＋アプリ」は `.ipa` ファイル、「エミュレータ＋アプリ」は `.app` ファイルが必要となる。  
また、実機用のアプリは development のプロビジョニングでビルドされている必要があり、かつ実機端末の UDID の指定も合わせて必要。

1. 実機に接続する MacOS PC に App Store から **Apple Configurator** をインストール
2. USB で実機を MacOS PC に接続し、端末を選ぶと、再度バーにAppsというメニューが出てくるのでクリック
3. `.ipa` ファイルをドラッグ＆ドロップすることでアプリを端末にインストールできる
4. appium-desktop との接続は後述

iOS 実機へ WebDriver をインストールするコマンドは以下。  
（なお、筆者は実機で試していない、、、）

```sh
$ xcodebuild build test -project /usr/local/lib/node_modules/appium/node_modules/appium-xcuitest-driver/WebDriverAgent/WebDriverAgent.xcodeproj -scheme WebDriverAgentRunner -destination id=xxxx -configuration Debug
```

## Android

Android 開発環境の構築方法については [Androidアプリ入門 Macで環境構築編](https://blog.pepese.com/entry/android-env-on-mac/) を参照。  
以下のコマンドで正しく環境構築されたか確認する。

```sh
$ appium-doctor --android
```

### エミュレータ

Android エミュレータは iOS シミュレータと異なり、エミュレータを手で作る必要がある。  
（また、各種ツール類を導入する必要がある？）

1. Android Studio を起動
    - 「 Start a new Android Studio project 」で適当にプロジェクトを作る
2. SDK Manager を起動
    - 左のメニューの Appearance & Behavior -> System Settings -> Android SDK を選択
    - SDK Tools を選択して以下をインストール・更新
        - Android SDK Tools
        - Android SDK Platform-tools
        - Android SDK Build-tools
    - SDK Platforms を選択して、利用した API Version をインストール
3. AVD Manager を起動
4. エミュレータを作成
    - ARM 系のプラットフォームは挙動が遅いので x86 、 x86_64 を選択する

#### コマンド

かつては `android` コマンドであったが、 `sdkmanager` と `avdmanager` に移行された。  
以下はメモ程度。

```sh
$ sdkmanager --list
$ sdkmanager "system-images;android-25;google_apis;armeabi-v7a"
$ avdmanager create avd -n test -k "system-images;android-25;google_apis;armeabi-v7a"
$ avdmanager list avd
$ emulator -list-avds
$ emulator -avd test
```

- [sdkmanager](https://developer.android.com/studio/command-line/sdkmanager.html)
- [avdmanager](https://developer.android.com/studio/command-line/avdmanager.html)
- [emulator](https://developer.android.com/studio/run/emulator-commandline.html)

### 実機

Android 実機での自動テスト手順は以下。

1. Android 実機を USB で PC に繋ぎ、実機側で USB デバッグを許可
2. `$ adb devices` で device 番号が表示されることを
3. 実機でアプリを起動して、パッケージ名などを確認（アプリインストールの確認）
    - `$ adb shell pm list packages | grep pepese`
        - `package:org.pepese`
    - `$ adb shell dumpsys activity | grep pepese | grep Intent`
        - `Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10200000 cmp=org.pepese/org.pepese.MainActivity }`
4. appium-desktop を起動
    - Capability を設定して接続（詳細後述）

## appium-desktop

appium-desktop を使用することにより以下のことが可能になる。

- Appiumサーバー起動（ `$ appium &` コマンド打たなくてよくなる）
- 実機やエミュレータと接続
- インスペクターを利用してアプリ画面内の要素の確認やその操作
- 操作のテストコード出力

以下のように導入する。

1. [ここ](https://github.com/appium/appium-desktop/releases/) から最新版を取得してインストール。
    - `appium-desktop-x.x.x.dmg`
2. 「 Simple 」で「 Start Server vx.x.x 」を押下
3. 右上の左のボタン「 Start Inspector Session 」を押下
4. 上のタブを「 Automatic Server 」、下のタブを「 Desired Capability 」の状態で、右下の「 JSON Representation 」にエミュレータや実機へ接続するための設定を記載する
    - [公式：設定ドキュメント](https://appium.io/slate/en/master/?ruby#appium-server-capabilities)

### iOS シミュレータと接続

iOS の場合は、「実機＋アプリ」は `.ipa` ファイル、「シミュレータ＋アプリ」は `.app` ファイルが必要となる。  
（ Android とは異なり、 `appPackage` `appActivity` の設定は不要）

```javascript
{
    "platformName": "iOS",
    "platformVersion": "11.1",
    "deviceName": "iPhone Simulator",
    "automationName": "XCUITest",
    "app": "[アプリまでのパス]"
}
```

### iOS 実機と接続

`udid` は MacOS PC へ実機を接続した後、 **Apple Configurator** で確認する。

```javascript
{
    "platformName": "iOS",
    "platformVersion": "11.1",
    "deviceName": "iPhone Simulator",
    "automationName": "XCUITest",
    "app": "[アプリまでのパス]",
    "udid": "",
    "xcodeOrgId": "<Team ID>",
    "xcodeSigningId": "iPhone Developer",
    "updatedWDABundleId": "io.appium.WebDriverAgentRunner"
}
```

### Android エミュレータと接続

Android エミュレータの場合は、あらかじめエミュレータを起動しておく。

```javascript
{
  "appPackage": "org.pepese",
  "appActivity": "org.pepese.MainActivity",
  "platformName": "Android",
  "automationName": "Appium",
  "platformVersion": "8.1.0",
  "deviceName": "Android Emulator",
  "app": "[アプリまでのパス]"
}
```

### Android 実機と接続

実機の場合は `$ adb devices` で device 番号を取得し、 `deviceName` へ設定する。

#### トラブルシューティング

- `adb install` で `INSTALL_FAILED_NO_MATCHING_ABIS` が出た
    - appium-desktop とエミュレータ接続時、アプリケーションがインストールされるのだが、アプリと ABI の組み合わせが悪いときに発生。
        - アプリが「 x86 」用にビルドされてない、とか「 ARM 」用にビルドされてないとか
    - CPU/ABI に「 x86 」がダメなら「 ARM(armeabi-v7a) 」を、「 ARM 」がダメなら「 x86 」を選択する。
- エミュレータが起動すると `Process system isn't responding` と表示される
    - x86 系マシン（例えば Mac ）で ARM のエミュレータを実行している時に発生
        - ただひたすらエミュレート処理が遅いだけ、大人しく x86 系エミュレータにしたほうがいい
    - ひたすら「 wait 」する

# サンプルを実行

サンプルアプリとサンプルテストコードで Appium の自動テストを動かしてみる。

## iOS

コマンドラインの iOS ビルドツールの準備。

```sh
$ xcode-select --install
$ sudo xcode-select --switch /Applications/Xcode.app
$ xcodebuild -version
Xcode 9.2
Build version 9C40b
```

サンプルアプリとテストコードを取得してセットアップ。

```sh
$ git clone https://github.com/appium/sample-code
$ git clone https://github.com/appium/ios-test-app
$ cd ios-test-app
$ xcodebuild -sdk iphonesimulator
$ cd ..
$ cp -R ios-test-app/build sample-code/sample-code/apps/TestApp/build
$ cd sample-code/sample-code/examples/python
```

`xcodebuild -version -sdk` コマンドで `PlatformVersion` を確認し、 `ios_simple.py` ファイルの 20 行目あたりの `PlatformVersion` を書き直す。  
別ターミナルを起動し `appium &` コマンドで Appium を起動してから以下を実行。

```sh
$ py.test ios_simple.py
```

### appium-desktop との接続

シミュレータ上のサンプルアプリと appium-desktop を接続する際、 Capability には以下を設定する。

```javascript
{
  "platformName": "iOS",
  "platformVersion": "11.2",
  "deviceName": "iPhone Simulator",
  "automationName": "XCUITest",
  "app": "/path/to/ios-test-app/build/Release-iphonesimulator/TestApp.app"
}
```

## Android

エミュレータは起動しておく。  
iOS のテストを実行した後のテイで書く。

```sh
$ py.test android_simple.py
```

動かなかった。  
以下の記事を参考のこと。

- [Appium インストールして起動まで/ iOS・Androidのトラブルシューティング](https://qiita.com/natsuki_summer/items/2d8d60114cdb95929dcb)

### appium-desktop との接続

エミュレータ上のサンプルアプリと appium-desktop を接続する際、 Capability には以下を設定する。

```
{
  "appPackage": "com.example",
  "appActivity": "com.example.toggletest.MainActivity",
  "platformName": "Android",
  "automationName": "Appium",
  "platformVersion": "8.1.0",
  "deviceName": "Android Emulator",
  "app": "/path/to/sample-code/sample-code/apps/ApiDemos/bin/ApiDemos-debug.apk"
}
```

Android Studio で自分で実装・ビルドしたアプリをエミュレータにインストールして接続する場合は以下。

```
{
  "appPackage": "org.pepese.sample.audiosample",
  "appActivity": "org.pepese.sample.audiosample.MainActivity",
  "platformName": "Android",
  "automationName": "Appium",
  "platformVersion": "8.1.0",
  "deviceName": "Android Emulator",
  "app": "/Users/[UserName]/AndroidStudioProjects/[プロジェクト名]/app/build/outputs/apk/debug/app-debug.apk"
}
```
