---
title: "所謂MEANスタックを作る"
date: 2017-08-14T08:24:46+09:00
slug: "mean-stack-basics"
tags:
- angular
- node.js
- npm
- express
- javascript
draft: false
archives:
- 2017/08
---

Angular（所謂Angular4）でMEANスタックのアプリを作成してみる。  
と言いつつMongoDBについてはほぼ言及しない。

<!--more-->

# 環境構築

## Angular CLI

Angular CLI のインストール。  
パッケージ管理ツールを `npm` から `yarn` へ変更。

```sh
$ npm install -g @angular/cli
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
node: 7.6.0
os: darwin x64
```


# プロジェクト作成手順

以下のアプリケーションを1つのプロジェクト内に同梱して作成する。

- フロントエンド
    - Angularベースのブラウザアプリケーション
    - サーバサイドのREST APIで通信する
    - **Angular CLI** を使用する
    - ソースコードディレクトリは `src/`
- サーバサイド
    - NodeJS、ExpressベースのWebアプリケーション
    - ソースコードディレクトリは `app/`

**フロントエンド、サーバサイドの順** で作成すること。

## フロントエンド

### Angular CLIでプロジェクト作成

SCSSベースで `mean-sample` というプロジェクト名で作成。

```sh
$ ng new mean-sample --style=scss
installing ng
  create .editorconfig
  create README.md
  create src/app/app.component.scss
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
  create src/styles.scss
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
Directory is already under version control. Skipping initialization of git.
Project 'mean-sample' successfully created.
```

`yarn` で作成されていることを確認する。  
また、下記の通りプロジェクトディレクトリに移動して、導入されている Angular のバージョンを確認する。

```sh
$ cd mean-sample
$ ng version
_                      _                 ____ _     ___
/ \   _ __   __ _ _   _| | __ _ _ __     / ___| |   |_ _|
/ △ \ | '_ \ / _` | | | | |/ _` | '__|   | |   | |    | |
/ ___ \| | | | (_| | |_| | | (_| | |      | |___| |___ | |
/_/   \_\_| |_|\__, |\__,_|_|\__,_|_|       \____|_____|___|
           |___/

@angular/cli: 1.3.0
node: 7.6.0
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

以下で Bootstrap を導入する。  
なお、 Bootstrap4-beta は jquery と popper.js に依存する。

```sh
$ yarn add ngx-bootstrap bootstrap@4.0.0-beta jquery@3.2.1 popper.js@1.11.1 --save
```

### angular-cli.json 修正

bootstrapのcss/js、jquery/popper.jsのjsを追加。

```javascript
// 省略
"styles": [
  "../node_modules/bootstrap/dist/css/bootstrap.min.css", // 追加行
  "styles.scss"
],
"scripts": [
  "../node_modules/jquery/dist/jquery.slim.min.js",       // 追加行
  "../node_modules/popper.js/dist/popper.min.js",         // 追加行
  "../node_modules/bootstrap/dist/js/bootstrap.min.js"    // 追加行
],
// 省略
```



### src/app/app.module.ts 修正

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms'; // 追加行
import { HttpModule } from '@angular/http';   // 追加行
import { BsRootModule } from 'ngx-bootstrap'; // 追加行

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule, // 追加行
    HttpModule,  // 追加行
    BsRootModule // 追加行
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

その他Angularの詳細については以下参照。

- [Angular入門](https://blog.pepese.com/entry/angular-basics/)


## サーバサイド

Angular CLIで作成したプロジェクトにサーバサイド、つまりExpressを追加する。  
サーバサイドアプリケーションのソースディレクトリは `app/` だけで完結するようにする。  
フロントエンドとは異なりトランスパイル不要。  
フロントエンドトランスパイル後の `dist` をExpress側で公開ディレクトリに設定する。

サーバサイドアプリケーションを作成する方法は以下を参照。  
「フロントエンドアプリケーションをプロジェクトに同梱する場合、REST APIアプリを作成する場合」のほうの構成。

- [Express入門](https://blog.pepese.com/entry/express-basics/)

## 起動

```
$ ng build                                      // フロントエンドのビルド
$ NODE_ENV=production forever start app/app.js  // サーバサイドアプリ起動
```
