---
title: "Angular入門"
date: 2017-08-14T08:27:19+09:00
slug: "angular-basics"
tags:
- angular
- typescript
- javascript
- npm
- yarn
- sass
draft: false
archives:
- 2017/08
---

フロントエンドMVCフレームワーク **Angular** （所謂Angular4）を触ってみた。  
ここでは [Angular CLI](https://github.com/angular/angular-cli) を使う。  
なお、 **ndenv** が入ってるテイで書く。導入は下記記事参照。

- [すべての\*\*envを管理するanyenv](https://blog.pepese.com/entry/anyenv/)

<!--more-->

# 環境構築

## Yarnインストール

```sh
$ npm install -g yarn
$ ndenv rehash
$ yarn --version
```

## Angular CLIインストール

Angular CLI のインストール。  
パッケージ管理ツールを `npm` から `yarn` へ変更。

```sh
$ yarn global add @angular/cli
$ ndenv rehash
$ ng set --global packageManager=yarn
$ ng version
_                      _                 ____ _     ___
/ \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
/ △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
/ ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
/_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
           |___/
@angular/cli: 1.3.0
node: 8.3.0
os: darwin x64
```

以降、 `ng` コマンドでいろいろできるようになる。

## プロジェクト作成と起動

```sh
$ ng new angular-sample
installing ng
  create .editorconfig
  create README.md
  create src/app/app.component.css
  create src/app/app.component.html
  create src/app/app.component.spec.ts
  create src/app/app.component.ts
  create src/app/app.module.ts
  create src/assets/.gitkeep
  create src/environments/environment.prod.ts
  create src/environments/environment.ts
  create src/favicon.ico
  create src/index.html
  create src/main.ts
  create src/polyfills.ts
  create src/styles.css
  create src/test.ts
  create src/tsconfig.app.json
  create src/tsconfig.spec.json
  create src/typings.d.ts
  create .angular-cli.json
  create e2e/app.e2e-spec.ts
  create e2e/app.po.ts
  create e2e/tsconfig.e2e.json
  create .gitignore
  create karma.conf.js
  create package.json
  create protractor.conf.js
  create tsconfig.json
  create tslint.json
Installing packages for tooling via yarn.
Installed packages for tooling via yarn.
Successfully initialized git.
Project 'angular-sample' successfully created.
```

`yarn` で作成されていることを確認する。  
また、下記の通りプロジェクトディレクトリに移動して、導入されている Angular のバージョンを確認する。

```sh
$ cd angular-sample
$ ng version
_                      _                 ____ _     ___
/ \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
/ △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
/ ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
/_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
           |___/
@angular/cli: 1.3.0
node: 8.3.0
os: darwin x64
@angular/animations: 4.3.4
@angular/common: 4.3.4
@angular/compiler: 4.3.4
@angular/core: 4.3.4
@angular/forms: 4.3.4
@angular/http: 4.3.4
@angular/platform-browser: 4.3.4
@angular/platform-browser-dynamic: 4.3.4
@angular/router: 4.3.4
@angular/cli: 1.3.0
@angular/compiler-cli: 4.3.4
@angular/language-service: 4.3.4
```

以下で起動。

```sh
$ ng serve
```

## プロジェクト構成

```
.
├── README.md
├── .angular-cli.json                * Angular CLI 設定ファイル
├── .editorconfig                    * エディタの設定ファイル
├── .gitignore
├── e2e                              * エンドツーエンドテストディレクトリ
│   ├── app.e2e-spec.ts
│   ├── app.po.ts
│   └── tsconfig.e2e.json
├── karma.conf.js                    * karma（テストランナー）の設定ファイル
├── package.json                     * npmの設定ファイル
├── protractor.conf.js               * e2eテストの設定ファイル
├── tsconfig.json                    * TypeScriptの設定ファイル
├── tslint.json                      * TypeScriptのLinter設定ファイル
└── src                              * ソースファイルディレクトリ
    ├── app                          * アプリケーションコードディレクトリ
    │   ├── app.component.css
    │   ├── app.component.html
    │   ├── app.component.spec.ts
    │   ├── app.component.ts
    │   └── app.module.ts
    ├── assets
    ├── environments
    │   ├── environment.prod.ts
    │   └── environment.ts
    ├── favicon.ico
    ├── index.html                    * トップ画面、ルートコンポーネントを記載
    ├── main.ts                       * ルートモジュールのロード（メイン）
    ├── polyfills.ts
    ├── styles.css
    ├── test.ts
    ├── tsconfig.app.json             * TypeScriptの設定ファイル
    ├── tsconfig.spec.json            * TypeScriptのテスト設定ファイル
    └── typings.d.ts                  * TypeScriptの型定義ファイル     
```

`.ditorconfig` は[EditorConfig](http://editorconfig.org)というプロジェクトの設定ファイルで、エディタへプラグインなどを入れることにより様々なエディタの設定をプロジェクト単位で可能にする。  
テストは **Jasmine** （テスティングフレームワーク）と **Karma** （テストランナー）の組み合わせで実行される。  
e2e（エンドツーエンド）テストは **protractor** 。

## 主要なコマンド

- `ng new`
    - 新規にAngularプロジェクトを作成する
- `ng build`
    - `src` 配下をビルドして `dist` へ出力する
- `ng serve`
    - Webサーバを起動してアプリケーションを実行する
- `ng generate`
    - blueprints から選択して新たにコードを生成する
- `ng eject`
    - `ng generate` などで作成したコードを取り除き、webpackの設定やスクリプトを変更する
- `ng get`
    - 設定（key-value）から値（value）を取得する
- `ng set`
    - 設定（key-value）に値（value）を追加する
- `ng lint`
    - Linterを実行する
- `ng test`
    - テストを実行する
- `ng e2e`
    - e2eテストを実行する
- `ng xi18n`
    - コードからi18nメッセージを抜く

上記以外や詳細なオプションは `ng help` を参照。

### blueprints の種類

- module
    - `ng generate module sample`
    - モジュール（ `sample/sample.module.ts` ）が作成される。
    - 自動で指定したモジュール名でディレクトリが切られることに注意
- component
    - `ng generate component sample`
    - コンポーネント（以下）を作成し、 `app.module.ts` に自動登録（importおよびメタデータ `declarations` への追加）される。
        - `sample.component.ts` 、 `sample.component.spec.ts` 、 `sample.component.html` 、 `sample.component.scss`
    - なお、既に存在するモジュール名で作成すると、そのモジュール名のディレクトリ内に作成される。
- directive
    - `ng generate directive sample`
    - ディレクティブ（以下）を作成し、 `app.module.ts` に自動登録（importおよびメタデータ `declarations` への追加）される。
        - `sample.directive.ts` 、 `sample.directive.spec.ts`
- pipe
    - `ng generate pipe sample`
    - パイプ（以下）を作成し、 `app.module.ts` に自動登録（importおよびメタデータ `declarations` への追加）される。
        - `sample.pipe.ts` 、 `sample.pipe.spec.ts`
- service
    - `ng generate service sample`
    - サービス（以下）を作成する。
        - `sample.service.ts` 、 `sample.service.spec.ts`
- class
    - `ng generate class sample`
    - 普通のクラス（ `sample.ts` ）が作成される。
- interface
    - `ng generate interface sample`
    - 普通のインターフェース（ `sample.ts` ）が作成される。
- enum
    - `ng generate enum sample`
    - 普通の列挙型（ `sample.enum.ts` ）が作成される。

ディレクトリを切りたい場合は `/` を加えて、 `ng generate service sample/sample-service` のようにする。


## Angular CLI のバージョンアップ方法

グローバルでは以下。

```sh
$ yarn global remove @angular/cli
$ yarn cache clean
$ yarn global add @angular/cli@latest
$ ndenv rehash
```

ローカルプロジェクトでは以下。

```sh
$ rm -rf node_modules dist # use rmdir on Windows
$ yarn add @angular/cli@latest --dev
$ yarn install
```

# Angularの基本

Angularのアーキテクチャは以下の要素から構成される。

- モジュール（@NgModule）
- コンポーネント（@Components）
- テンプレート（Templates）
- メタデータ（Metadata）
- データバインディング（Data Binding）
- ディレクテイブ（@Directive）
- パイプ（@Pipe）
- サービス（Services）・DI（Dependency injection / @Injectable）
- フォームとバリデータ
- Angularライブラリ

上記の各々について後述する。

## モジュール（@NgModule）

Angularのアプリケーションはモジュール単位で機能を管理する。  
モジュールは後述の **コンポーネント** や **テンプレート** を包含する。

Angularアプリケーションは少なくとも１つ **ルートモジュール** を持ち、`AppModule` と命名する。  
ルートモジュールは、 `src/main.ts` からロードされる。  
通常のアプリケーションの場合、ルートモジュールは１つだが、巨大なアプリケーションの場合、複数のルートモジュールを持つこともある。

モジュールは以下のように作成する。

```typescript
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

@NgModule({
  imports:      [ BrowserModule ],
  providers:    [ Logger ],
  declarations: [ AppComponent ],
  exports:      [ AppComponent ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```

`AppModule` クラスに `@NgModule` **デコレータ** （Javaでいうアノテーション）を付与した形になっている。  
デコレータにAngularに則した設定を行い、クラスには何も書かない（たぶん）。  
`@NgModule` の代表的なプロパティは以下の通り。ちなみにデコレータに設定するプロパティを **メタデータ** という。

- `imports`
    - このモジュールに他のモジュールを取り込み、このモジュール内で定義されているコンポーネントやテンプレートが他のモジュールのクラスを使用できるようになる。
    - 他のモジュールの `providers` ・ `exports` に定義されたものを使用できるようになる。
- `providers`
    - このモジュールおよび関係するコンポーネント・サービスへインジェクトするためのサービスクラスを定義・インスタンス化する。
- `declarations`
    - このモジュールに所属させるビュークラスを指定する。Angularのビュークラスには、 **コンポーネント** 、 **ディレクティブ** 、 **パイプ** がある。
- `exports`
    - このモジュール内のクラスを他のモジュールのコンポーネント・テンプレートで使用可能にする。
    - `imports` の逆。
- `bootstrap`
    - **ルートコンポーネント** を定義する。ルートコンポーネントはアプリケーションのメインビュー。
    - **ルートモジュールにだけ** に `bootstarp` プロパティを設定する。

以下が **ルートモジュール** を指定する方法（ `src/main.ts` ）。

```typescritp
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app/app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

また、モジュール内では以下のようにライブラリから他のモジュールをロードすることもできる。

```typescript
import { Component } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
```

ロードしたライブラリモジュールは、 `@NgModule` の `imports` プロパティに指定することにより使用できるようになる。

```typescript
imports:      [ BrowserModule ],
```

## コンポーネント（@Component）

コンポーネントはビューの役割を担う。  
コンポーネントクラス内で定義したフィールド変数やメソッドは、テンプレートから直接使用できる。  
ロジックは基本的にコンポーネント内に記載し、サービスクラスをDIしたりする。

```typescript
@Component({
  moduleId: module.id,
  selector:    'hero-list',
  templateUrl: './hero-list.component.html',
  providers:  [ HeroService ]
})
export class HeroListComponent implements OnInit {
/* . . . */
}
```

`@Component` の代表的なメタデータプロパティは以下。

- moduleId
    - このモジュールが定義されるファイルのES/CommonJSモジュールID
- selector
    - このコンポーネントのHTMLタグ名を定義する。
- template
    - テンプレートを直接記載する。
- templateUrl
    - テンプレートファイルのパスを指定する。
- styles
    - スタイルシート（CSSやSass）を直接記載する。このコンポーネント外には影響が無い。
- styleUrls
    - スタイルシートファイルのパスを指定する。このコンポーネント外には影響が無い。
- providers
    - このコンポーネントがDI経由で使用するサービスクラスを指定する。
    - コンポーネントクラスの `constructor` の引数でも指定する必要がある。

その他の `@Component` のメタデータは[ここ](https://angular.io/docs/ts/latest/api/core/index/Component-decorator.html)で確認できる。

## テンプレート（Templates）

テンプレートはコンポーネントのDOMとして使われるHTML。  
コンポーネントクラスの `@Component` デコレータのメタデータに直接記載（ `template` ）することも、別ファイルとして作成して読み込む（ `templateUrl` ）ことも可能。  
テンプレート内では、HTMLの属性として **ディレクティブ** （ `*ngFor` など）、コンポーネントとのデータ・機能のやりとりに **データバインディング** を使用することができる。

- コンポーネントフィールドへのアクセス
    - `*ngFor` ： コレクションフィールドの値にアクセス
    - `*ngFor="let hero of heroes"` ： コレクションheroesの各要素をheroへ代入
- コンポーネントのメソッド呼び出し
    - `(click)="onClickMe()"` ： クリック時に `onClickMe()` メソッドを呼び出す
    - `(keyup)="onKey($event)"` ： キーアップ時にイベントを引数に `onKey()` メソッドを呼び出す

また、テンプレートにはスタイルシート（CSS、Sassなど）を指定することができ、`@Component` デコレータのメタデータ（ `styles` 、 `styleUrls` ）で指定することができる。

## メタデータ（Metadata）

メタデータは **デコレータ** （ `@` から始まるJavaでいうアノテーション）に設定するプロパティ。  
Angularにクラスがどのように挙動するか知らせる役割。  
`@Injectable` 、 `@Input` 、 `@Output` など様々なデコレータにメタデータを設定できる。  
もう十分前述しているのでこの程度で。


## データバインディング（Data Binding）

テンプレートとコンポーネント間のデータ・機能のやりとりを行う機能。  
以下のように４種類ある。

- 単方向バインド（interpolation）
    - `<li>{{hero.name}}</li>`
    - コンポーネントの `hero.name` プロパティの値を `<li>` 表示する
- プロパティバインド（property binding）
    - `<hero-detail [hero]="selectedHero"></hero-detail>`
    - 親コンポーネントの `selectedHero` の値を、子コンポーネントの `hero` へ渡している
- イベントバインド（event binding）
    - `<li (click)="selectHero(hero)"></li>`
    - ユーザのクリックにより `selectHero()` メソッドが呼び出される。
- 双方向バインド（Two-way data binding）
    - `<input [(ngModel)]="hero.name">`
    - `ngModel` を用いて **参照と更新** を両方同時に実現する。

`ngModel` のようにテンプレートのHTMLタグの属性として記述できるAngularの機能を **ディレクティブ** という。

### イベント

```html
<button (click)="onClickButton()">ボタン1</button>
<button (click)="onClickButtonWithEvent($event)">ボタン2</button>
```

`(click)` のように、イベント名をカッコで囲んだ属性に、イベントを処理するメソッドを記述する。  
引数なしの記述と、イベントオブジェクト `$event` を指定する記述ができる。  
コンポーネントクラス内では以下のようにメソッド定義する。

```typescript
onClickButtonWithEvent(event:any) {
   // targetプロパティでイベントを発生させたオブジェクトを取得
   var button = event.target;
   // ボタンのラベルをtextContentプロパティから取得して表示
   alert("ボタン「" + button.textContent + "」が押下されました");
 }
```

## ディレクテイブ（@Directive）

ディレクティブは前述の通り、テンプレートのHTMLタグの属性として指定できるAngularの機能。  
モジュール・コンポーネント間のデータの授受や、ループ・条件分岐といった動的なDOMの機能を提供する。

```html
<li *ngFor="let hero of heroes"></li>
<hero-detail *ngIf="selectedHero"></hero-detail>
```

`@Directive` デコレータでディレクティブを自作できる。  
ここでは割愛。[参考](https://angular.io/docs/ts/latest/guide/attribute-directives.html)

## パイプ（@Pipe）

Pipeはテンプレート内で文字列操作の機能を提供する。  
例えば、Dateオブジェクトをあるフォーマットに整形して表示できる。

```typescript
// コンポーネントクラスのフィールド
let birthday = new Date(1985,3,1); // これで1985年4月1日 なことに注意
```

上記をテンプレードで以下のように指定することにより **yy/MM/dd** 形式で表示できる。

```html
<p>{{ birthday | date:"yy/MM/dd" }}</p> <!-- 1985/04/01 と表示される -->
```

他にも下記のようなパイプがある。

- DatePipe
    - 上記例。日付を整形する。
    - `date_expression | date[:format]`
- UpperCasePipe
    - 文字列を大文字にする。
    - `expression | uppercase`
- JsonPipe
    - 入力値にJSON.stringfyを実行して返す。
    - `expression | json`

また、パイプは以下のような感じでチェーンできる。

```
{{ birthday | date | uppercase}}
```

さらに、`@Pipe` デコレータでパイプを自作できる。  
ここでは割愛。[参考](https://angular.io/docs/ts/latest/guide/pipes.html)


## サービス（Services）・DI（Dependency injection / @Injectable）

サービスは `export` された通常のtypescriptクラス。  
DIを使用してコンポーネントや他サービスクラスにサービスクラスをインジェクトできる。

### 作成例

```sh
$ ng generate service sample/sample
installing service
  create src/app/sample/sample.service.spec.ts
  create src/app/sample/sample.service.ts
  WARNING Service is generated but not provided, it must be provided to be used
```

上記で作成した `src/app/sample/sample.service.ts` をDIするには、使用するモジュール・コンポーネントにimport、メタデータ `providers` およびコンストラクタへの指定を行う。  
以下コンポーネントへの導入例。

```typescript
...
import { SampleService } from './sample/sample.service' // インポート

@Component({
  ...
  providers: [SampleService], // providers に追加
  ...
})
export class AppComponent {
  title: string;

  constructor(
    private sampleService: SampleService // コンストラクタに設定
  ){
    this.title = sampleService.getTitle();
  }
}
```

### DI・インジェクタ

サービスをモジュールやコンポーネントで使用できるようにインジェクトする機能。  
前述の通り、モジュール・コンポーネントにimport、メタデータ `providers` およびコンストラクタへの指定を行う。

`@Injectable` デコレータがついたサービスクラスは多段Inject（DIのDI）することが可能になる。  
一段Injectはデコレータが無しでもOKだがつけておいても問題ない。

## フォームとバリデータ

http://codezine.jp/article/detail/9596?p=4

## Angularライブラリ

Angularのライブラリには様々用意されており、APIリファレンスは[ここ](https://angular.io/docs/ts/latest/api/)。  
リファレンスではクラスの前に記号がついており、それぞれ以下の意味。

|記号|意味|
|:---|:---|
|D|Directive。ディレクティブ。|
|P|Pipe。パイプ。|
|@|Decorator。デコレータ。Javaでいう所謂アノテーション。|
|C|Class。普通のTypeScriptのクラス。|
|I|Interface。普通のTypeScriptのインターフェース。|
|F|Function。関数。|
|E|Enum。普通の列挙型。|
|T|Type Alias。なんかわからん。|
|K|Const。定数クラス。|


# その他話題、メモ

## TypeScriptの型解決について

解決方法には以下がある。

- tsd
- typings
- @types

Angularは型定義が含まれているため、上記３つは不要。ただし、他のライブラリを用いる場合 `@types` を使用する。（typingsに変わったかも。。。）
