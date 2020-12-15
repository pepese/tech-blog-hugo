---
title: "Androidアプリ入門 開発編"
date: 2020-02-26T21:25:59+09:00
slug: ""
tags:
- Android
- Kotlin
draft: true
archives:
- 2019/06
---

Kotlin を利用した Android アプリの開発についてまとめる。

- アクティビティ
- リソース

[Developers Guide](https://developer.android.com/guide?hl=ja)

<!--more-->

# アクティビティ

アクティビティを作成する際の構成要素は以下。

- [マニフェスト](https://developer.android.com/guide/topics/manifest/manifest-intro?hl=ja)（ `app > manifests > AndroidManifest.xml` ）
  - アプリの各コンポーネントを定義
- Activity クラス（ `app > java > com.example.myfirstapp > MainActivity` ）
- Layout（ `app > res > layout > activity_main.xml` ）
  - アクティビティの UI のレイアウトを定義する XML ファイル

## アクティビティのライフサイクル

アクティビティの実装する際は [`android.app.Activity` クラス](https://developer.android.com/reference/android/app/Activity.html) を継承し、以下のライフサイクルが定義されている。

<img src="https://developer.android.com/images/activity_lifecycle.png">

1. `onCreate(Bundle)`
  - Activity インスタンスが作られたときに一度だけ呼び出される
  - アクティビティの初期化を行うメソッドで、オーバーライドして利用する
  - 以下のような画面を作成する処理を行う
    - `setContentView(int)` ： 本アクティビティで利用するレイアウトリソースで定義した UI を呼び出す
    - `findViewById(int)` ： テキストボックスなどのウィジェットを呼び出し、設定を行う
2. `onStart()`
  - 画面が表示されたときに呼び出される
  - `onCreate()` 後だけでなく、戻るボタンなどで画面が再表示された際も呼び出される
3. `onResume()`
  - 画面がフォーカスを持ったときに呼び出される
4. `onPause()`
  - ユーザによる操作を受け付ける処理を記載するメソッドで、オーバーライドして利用する
5. `onStop()`
  - 画面が表示されなくなったときに呼び出される
6. `onDestroy()`
  - Activity インスタンスが破棄されるときに一度だけ呼び出される

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.d("MainActivity", "onCreate state:" + lifecycle.currentState)
    }

    override fun onStart() {
        super.onStart()
        Log.d("MainActivity", "onStart state:" + lifecycle.currentState)
    }

    override fun onResume() {
        super.onResume()
        Log.d("MainActivity", "onResume state:" + lifecycle.currentState)
    }

    override fun onPostResume() { // onPostResume()はonResumeの後に呼ばれます。
        super.onPostResume()
        Log.d("MainActivity", "onPostResume state:" + lifecycle.currentState)
    }

    override fun onPause() {
        super.onPause()
        Log.d("MainActivity", "onPause state:"+lifecycle.currentState)
    }

    override fun onStop() {
        super.onStop()
        Log.d("MainActivity", "onStop state:"+lifecycle.currentState)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d("MainActivity", "onDestroy state:"+lifecycle.currentState)
    }
}
```

上記のアプリを起動すると以下のようなログが出る。

```
onCreate state:INITIALIZED
onStart state:CREATED
onResume state:STARTED
onPostResume state:RESUMED
```

<img src="https://codelabs.developers.google.com/codelabs/kotlin-android-training-lifecycles-logging/img/f6b25a71cec4e401.png">

## [UI](https://developer.android.com/guide/topics/ui?hl=ja)

Android アプリの UI は、レイアウト（ `ViewGroup` オブジェクト）とウィジェット（ `View` オブジェクト）の階層を使用して作成さする。  
レイアウトは、その子ビューが画面にどのように配置されるかを制御するコンテナで、ウィジェットは、ボタンやテキスト ボックスなどの UI コンポーネント。  
ただし、Android では、`ViewGroup` および `View` クラスの XML ボキャブラリが提供されるため、ほとんどの UI は XML ファイルで定義され、さらに、Android Studio の Layout Editor を使ってレイアウトを作成することができるため、XML の記述自体も軽減される。

<img src="https://developer.android.com/images/viewgroup_2x.png?hl=ja">

## [インテント](https://developer.android.com/guide/components/intents-filters?hl=ja)

Intent は、別のアプリコンポーネントからのアクションをリクエストするときに使用できるメッセージングオブジェクト。

```kotlin
    const val EXTRA_MESSAGE = "com.example.myfirstapp.MESSAGE"

    class MainActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
        }

        /** Called when the user taps the Send button */
        fun sendMessage(view: View) {
            val editText = findViewById<EditText>(R.id.editText)
            val message = editText.text.toString()
            val intent = Intent(this, DisplayMessageActivity::class.java).apply {
                putExtra(EXTRA_MESSAGE, message)
            }
            startActivity(intent)
        }
    }
```

上記は MainActivity にて sendMessage メソッドを実装している。  
このメソッドはオーバーライド無しで、レイアウトのボタンなどから呼び出されることを想定しており、 **インテント** を作成して次の画面（アクティビティ）にメッセージを送信して遷移する処理を記載している。

- `Intent` コンストラクタには、`Context` と `Class` の 2 つのパラメータがある
- `Context` パラメータは、Activity クラスが Context のサブクラスであるため、最初に使用される
- アプリコンポーネントの Class パラメータには Intent が送信され、ここでは開始されるアクティビティとなる
- `putExtra()` メソッドは EditText の値をインテントに追加し、これにより Intent では、extras というキーと値のペアでデータ型をやり取りが可能となる

使用するキーは `EXTRA_MESSAGE` というパブリック定数で、次のアクティビティでこのキーを使用してテキスト値を取得する。  
インテントのエクストラのキーを接頭語としてアプリのパッケージ名を付けて定義すると、アプリが他のアプリと連携する場合に、キーを一意に識別できるようになりる。  

- `startActivity()` メソッドは、`Intent` で指定された `DisplayMessageActivity` のインスタンスを開始する

## 新規アクティビティの作成

- Android Studio の [Project] ウィンドウで app フォルダを右クリックし、[New] > [Activity] > [Empty Activity] を選択してアクティビティ名などを入力して作成
- 上記を実行することで、Android Studio が自動的にアクティビティに対応するレイアウトファイル（ `activity_*.xml` ）、マニフェストファイルに必要な <activity> 要素を追加・作成してくれる

前画面（アクティビティ）からのメッセージは以下のようにインテント経由で取得する。

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_display_message)

        // Get the Intent that started this activity and extract the string
        val message = intent.getStringExtra(EXTRA_MESSAGE)

        // Capture the layout's TextView and set the string as its text
        val textView = findViewById<TextView>(R.id.textView).apply {
            text = message
        }
    }
```

# [リソース](https://developer.android.com/guide/topics/resources/providing-resources?hl=ja)

プロジェクト `res/` ディレクトリ内に以下のリソースディレクトリを配置することがサポートされている。

|ディレクトリ|リソースタイプ|
|:---|:---|
|animator/|プロパティ アニメーションを定義する XML ファイル。|
|anim/|トゥイーンアニメーションを定義する XML ファイル（プロパティアニメーションをこのディレクトリに保存することもできますが、2 つのタイプを区別するために、プロパティアニメーションは animator/ ディレクトリに保存することが推奨される）。|
|color/|色の状態リストを定義する XML ファイル。色状態リストのリソースをご覧ください。|
|drawable/|ビットマップ ファイル（.png、.9.png、.jpg、.gif）または次のドローアブル リソース サブタイプにコンパイルされる XML ファイル（ビットマップファイル、9-patch、状態リスト、形状、アニメーションドローアブル、その他のドローアブル）※[詳細](https://developer.android.com/guide/topics/resources/drawable-resource?hl=ja)|
|mipmap/|さまざまなランチャー アイコン密度のドローアブル ファイル。※[詳細](https://developer.android.com/studio/projects?hl=ja#mipmap)|
|layout/|ユーザインターフェースのレイアウトを定義する XML ファイル。※[詳細](https://developer.android.com/guide/topics/resources/layout-resource?hl=ja)|
|menu/|オプションメニュー、コンテキストメニュー、サブメニューといった、アプリメニューを定義する XML ファイル。※[詳細](https://developer.android.com/guide/topics/resources/menu-resource?hl=ja)|
|raw/|未加工の形式で保存する任意のファイル。これらのリソースを未加工の InputStream で開くには、リソース ID（R.raw.filename）を指定して Resources.openRawResource() を呼び出す。ただし、元のファイル名とファイル階層にアクセスする必要がある場合は、一部のリソースを assets/ ディレクトリ（res/raw/ の代わりに）に保存することを検討する。assets/ のファイルにはリソース ID が付与されていないため、読み取りには AssetManager が必要。|
|values/|文字列、整数、色など、単純な値を含む XML ファイル。その他の res/ サブディレクトリの XML リソース ファイルは XML ファイル名に基づいて 1 つのリソースを定義するが、values/ ディレクトリのファイルは複数のリソースを表す。このディレクトリのファイルの場合、<resources> 要素の子がそれぞれ 1 つのリソースを定義する。例えば、<string> 要素は R.string リソースを生成し、<color> 要素は R.color リソースを生成する。各リソースの定義には XML 要素を使用するため、ファイルには任意の名前を設定でき、1 つのファイル内にさまざまなリソースを配置できる。ただし、わかりやすくするために、一意のリソースタイプをそれぞれのファイルに配置することもできる。次は、このディレクトリで作成できるリソースの一部のファイル名規則の例。リソースの配列の場合 arrays.xml（型付き配列）、色の値の場合 colors.xml、寸法の値の場合 dimens.xml、文字列の値の場合 strings.xml、スタイルの場合 styles.xml。※[文字列リソース詳細](https://developer.android.com/guide/topics/resources/string-resource?hl=ja)、[スタイルリソース詳細](https://developer.android.com/guide/topics/resources/style-resource?hl=ja)、[その他リソース詳細](https://developer.android.com/guide/topics/resources/more-resources?hl=ja)|
|xml/|Resources.getXML() を呼び出すことによって、実行時に読み込むことができる任意の XML ファイル。ここには、検索可能な設定など、各種の XML 構成ファイルが保存される。|
|font/|.ttf、.otf、.ttc などの拡張子を持つフォント ファイル、または <font-family> 要素を含む XML ファイル。※[詳細](https://developer.android.com/preview/features/fonts-in-xml?hl=ja)|