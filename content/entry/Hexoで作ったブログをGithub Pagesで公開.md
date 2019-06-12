---
title: "Hexoで作ったブログをGithub Pagesで公開"
date: 2017-05-02T07:58:10+09:00
slug: "hexo-github-pages"
tags:
- hexo
- github
draft: false
archives:
- 2017/05
---

はてなブログが一向に *https* 対応しないし、そのほかのリスクも考えて流行りの **静的サイトジェネレータ** の [HEXO](https://hexo.io/) を使ってブログを作成し、[GitHub Pages](https://pages.github.com/)で公開してみる。  
前提として **npm** と **Git** 環境が必要なため以下参照。

- https://blog.pepese.com/entry/mac-dev-environment/

<!--more-->

# Hexoのインストールから起動まで

```sh
$ npm install -g hexo-cli --no-optional // インストール
$ ndenv rehash
$ hexo -version                         // 確認
hexo: 3.3.8
hexo-cli: 1.0.3
os: Darwin 16.6.0 darwin x64
http_parser: 2.7.0
node: 8.3.0
v8: 6.0.286.52
uv: 1.13.1
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
openssl: 1.0.2l
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b
$ hexo init tech-blog                   // ブログプロジェクト作成
$ cd tech-blog
$ npm install --no-optional
$ hexo server                           // 起動
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

筆者がインストールした時（2017/05/02）は、 **--no-optional** オプション無しだと **Error: Cannot find module ./build/default/DTraceProviderBindings** というエラーがでた。

```sh
$ tree
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts  // これは無いかも
|   └── _posts
└── themes
```

## 設定ファイルの編集

`_config.yml` を以下のように設定する。

```yml
# Site
title: ぺーぺーSEのテックブログ
subtitle: 備忘録用メモサイト
description: ぺーぺーSEがプログラミング（Java、JavaScript、Python、Ruby）、クラウド（AWS）、Web構築などのメモを残すサイト。
author: ぺーぺーSE
language: ja
timezone: Japan

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://pepese.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 〜（省略）〜
```

[設定ファイルの公式ドキュメント](https://hexo.io/docs/configuration.html)

## 記事の作成

```sh
$ hexo new [layout] <title>
```

**layout** には **post** 、 **page** 、 **draft** の3種類あり、それぞれ以下のように別のパスに作成される。

|layout|パス|説明|
|:---|:---|:---|
|post|source/_posts|公開記事として作成される|
|page|source|imageやjavascriptなどのアセット？|
|draft|source/_drafts|非公開記事として作成される|

上記のpost、page、draftは、 `scaffolds/` 配下の **post.md** 、 **page.md** 、 **draft.md** が雛形となって生成される。  
この雛形で記事生成時のMarkdownのヘッダ部分（例えば以下）の初期値を設定することができる。

```yml
---
title: {{ title }}
date: {{ date }}
tags:
id:
---
```

`{{ title }}` へは記事のファイル名が、 `{{ date }}` へは `source/_posts` 配下へ記事が作成された段階の日時が自動で設定される。  
`tags:` は以下のように記述することで記事へタグを付与することができ、タグ単位でリンクが作成される。

```yml
tags:
- Hexo
- Github Pages
```

`id:` は、 `_config.yml` へ以下（ `:id` 部分）のように設定することでURLとして扱うことができる。（デフォルトは記事名）

```yml
permalink: :year/:month/:day/:id/
```

記事の削除は、 `rm source/_post/title.md` などのコマンドで直接削除する。  
ドラフトで作成していた記事は以下のコマンドで公開（つまりpostへ移動）される。

```sh
$ hexo publish [layout] <title>
```

### マークダウンの書き方小ネタ

- `<!-- more -->`
    - マークダウン記事の途中に記載すると、ブログトップにて記事が全て表示されずに「Read More」（所謂、 **続きを読む** ）が表示される
- Octopressから移植された特殊な記法（[Tag Plugins](https://hexo.io/docs/tag-plugins.html)）
    - Block QuoteやCode Blockなど

## 静的ファイルの生成

```sh
$ hexo generate
```

上記のコマンドで `public/` 配下に静的ファイル（HTTP/CSS/JS）が作成される。  
後述のブログのデプロイ時には、 `public/` 配下のファイルが公開されることになる。

## テーマの設定

### テーマのインストール

`themes/` ディレクトリ配下にテーマを配置する。  
Githubで公開されているCasperを使用する場合は以下のように取得・配置する。

```sh
$ git clone https://github.com/cgmartin/hexo-theme-bootstrap-blog.git themes/bootstrap-blog

# USAGE
# git clone [Githubリポジトリ] themes/[テーマ名]
```

ブログの見た目を変更したいときは、 `themes/[テーマ名]/layout/` 配下のファイルを編集する。  
テーマを反映させたいときは、 `_config.yml` の `theme` 項目にテーマ名を設定する。

```yml
# 〜（省略）〜

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: bootstrap-blog

# 〜（省略）〜
```

### テーマの編集

`themes/[テーマ名]/layout` 配下を編集することでタイトル、サイドバーメニュー等の編集が可能。  

```sh
.
└── themeName
    ├── _config.yml
    ├── gulpfile.js
    ├── layout
    │   ├── _partial
    │   │   ├── archive.ejs  # アーカイブのレイアウトを定義
    │   │   ├── article.ejs  # 各記事のレイアウトを定義
    │   │   ├── head.ejs     # headタグの中身
    │   │   ├── header.ejs   # ページのヘッダ
    │   │   └── post         # ここに記事の型を入れていく
    │   │       ├── date.ejs
    │   │       ├── nav.ejs
    │   │       ├── tag.ejs
    │   │       └── title.ejs
    │   ├── archive.ejs
    │   ├── index.ejs
    │   └── layout.ejs       # ページ全体のレイアウト
    ├── node_modules
    ├── package.json
    └── source               # CSS/JSなどのアセット
```

ページヘッダやサイドバーなどに変更を加える場合、その際 `themes/[テーマ名]/layout/` 配下のファイルの変更を保持する必要があるので `.git/` を削除する。

```sh
$ rm -fR themes/bootstrap-blog/.git
```

# ブログの公開

## Github Pagesへの公開

以下のライブラリを導入する。

```sh
$ npm install hexo-deployer-git --save --no-optional
```

さらに **_config.yml** に以下の設定を追加する。

```yml
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://pepese@github.com/pepese/pepese.github.io.git
  branch: master
  message: update
```

なお、対応するGithubリポジトリは事前に作成しておくこと。  
`$ hexo deploy` コマンドでGithubリポジトリに反映される。

```sh
$ hexo clean
$ hexo generate
$ hexo deploy # これがデプロイコマンド
```

「 **https://pepese.github.io** 」へアクセスするとブログがGithub Pagesとして公開されていることがわかる。


## HTTPS＋独自ドメインでブログの公開

ここでは独自ドメインを「 **pepese.net** 」として記載する。（ **実際には公開していない** ）  
ホスト名は「 **techblog** 」とする。  
なお、独自ドメインを使用するとGithubが用意してくれている証明書が使えないため *https* にはならない。  

お名前.comを利用している場合は[こちら](http://qiita.com/tiwu_official/items/d7fb6c493ed5eb9ee4fc)を参考。  
[Cloudflare](https://www.cloudflare.com/)を使用したサイトのHTTPS化を以下を参考に行う。  
（お名前.comの有料オプションを使ってHTTPS化も可能だが、ここでは無料でできるCloudflareを使用する。）

- https://rcmdnk.com/blog/2017/01/03/blog-github-web/
- https://www.kaitoy.xyz/2016/07/01/https-support-by-cloudflare/
- http://qiita.com/noraworld/items/89dd85a434a7b759e00c

手順は以下。

**(1)** [Cloudflare](https://www.cloudflare.com/)のアカウント作成

Sign upのリンクからメアドとパスワードを渡してアカウントを作成。

**(2)** Cloudflareにサイトを登録

「Add Your First Domain」にて「pepese.net」を入力して「Begin Scan」。

**(3)** CloudflareのDNSの設定

以下のようになっていればいい。

|Type|Name|Value|TTL|
|:---|:---|:---|:---|
|A|pepese.net|192.30.252.153|Automatic TTL|
|A|pepese.net|192.30.252.154|Automatic TTL|
|CNAME|techblog|pepese.github.io|Automatic TTL|

**(4)** プランの選択

無料のFree Websiteを選択。  
以上を完了すると「Please visit your registrar's dashboard to change your nameservers to the following.」と出て、レジストラサイト（お名前.com など）のネームサーバを変更するように指示される。  
ここでは以下のように指示された。

|Current Nameservers|Change Nameservers to:|
|:---|:---|
|01.dnsv.jp|dina.ns.cloudflare.com|
|02.dnsv.jp|james.ns.cloudflare.com|
|03.dnsv.jp|Remove this nameserver|
|04.dnsv.jp|Remove this nameserver|

上記の指示通りにレジストラサイトの設定（ネームサーバの変更）を行う。  
この設定を行うことで、レジストラサイトのDNSサーバではなく、CloudflareのDNSサーバが使用されるようになる。

**(5)** CNAMEファイルの作成

以下のようにCNAMEファイルを作成する。

```sh
$ echo 'techblog.pepese.net' > source/CNAME
```

**(6)** Hexoの設定ファイル `_config.yml` の編集

ドメイン変更に伴って設定を変更する。

```yml
# 〜（省略）〜

url: https://techblog.pepese.net # ここを変更
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# 〜（省略）〜
```

**(7)** Github Pagesの更新

```sh
$ rm -r public/
$ hexo generate
$ hexo deploy
```

**(8)** その他のCloudflareの設定

好みに応じて以下の設定をする。

- CloudflareのPage Rulesタブで常にHTTPSアクセスとなるように以下を設定する。
    - If the URL matches: http://techblog.pepese.net*
    - Then the settings are: Always use HTTPS
- SpeedタブのAuto Minify項目でCloudflareでキャッシュする時にJavaScript/CSS/HTMLのソースをminifyする設定が可能。


以上で「 **https://techblog.pepese.net** 」へアクセスすると独自ドメインでブログが公開されていることがわかる。（ **実際には公開していない** ）  


# その他の設定、やりたきことメモ

## sitemapの作成

```sh
$ npm install hexo-generator-sitemap --save --no-optional
```

`_config.yml` に以下の設定を追加する。

```yml
sitemap:
  path: sitemap.xml
```

## robots.txtの作成

```sh
$ npm install hexo-generator-robotstxt --save --no-optional
```

`_config.yml` に以下の設定を追加する。

```yml
robotstxt:
  useragent: "*"
  disallow:
    - /images/
  allow:
  sitemap: https://techblog.pepese.net/sitemap.xml
```

## 404ページの作成

```sh
$ touch source/404.md
```

上記のように `source/` 配下に `404.md` ファイルを作成し、下記のような感じで記述する。

```yml
---
title: Not Found
---

お探しの記事は見つかりませんでした。
```

これで `$ hexo generate` を実行すると `404.html` が公開ディレクトリ直下に作成され、Github Pagesの機能で存在しないページへアクセスされた際に表示される。

## Google Adsenseの設置

テーマディレクトリ配下に手を加えていく。

```sh
$ mkdir theme/[テーマ名]/layout/_custome_ad
$ touch themes/[テーマ名]/layout/_custom_ad/google_adsense.ejs
```

`google_adsense.ejs` ファイルを以下のように編集する。

```html
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- 省略 -->
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
```

広告を表示させたい箇所に以下のコードを貼り付ける。

```html
<!-- ad start -->
<%- partial('_custom_ad/google_adsense') %>
<!-- ad end -->
```

`themes/[テーマ名]/layout/layout.ejs` を直接編集してもよい。

## Google Analyticsの設置

Google AnalyticsのトラッキングID（ **UA-xxxxxxxx-x** ）を取得し、 `themes/[テーマ名]/_config.yml` ファイルに以下の設定を追加する。

```yml
google_analytics: UA-xxxxxxxx-x
```

## Amazonアソシエイトの設置

```sh
$ touch themes/[テーマ名]/layout/_custom_ad/amazon_affiliate.ejs
```

`amazon_affiliate_side.ejs` ファイルを以下のように編集する。

```html
<!-- Amazon -->
<script type="text/javascript"><!--
amazon_ad_tag = "xxxxxxxx"; amazon_ad_width = "160"; amazon_ad_height = "600"; amazon_ad_logo = "hide";//--></script>
<script type="text/javascript" src="http://ir-jp.amazon-adsystem.com/s/ads.js"></script>
```

ブログを *https* で公開する場合は、 `src` 属性のURLスキームを `https` へ変更する。  
広告を表示させたい箇所に以下のコードを貼り付ける。

```html
<!-- ad start -->
<%- partial('_custom_ad/amazon_affiliate') %>
<!-- ad end -->
```

`themes/[テーマ名]/layout/layout.ejs` を直接編集してもよい。

## Twitterの設定

### Twitter Cards

`themes/[テーマ名]/_config.yml` ファイルに以下の設定を追加する。

```yml
twitter_id: '@PeePeeSE'
```

これでheadタグ内に以下のメタが追加される。（所謂 **Twitter Cards**）

```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="ぺーぺーSEのテックブログ">
<meta name="twitter:description" content="ぺーぺーSEがプログラミング（Java、JavaScript、Python、Ruby）、クラウド（AWS）、Web構築などのメモを残すサイト。">
<meta name="twitter:creator" content="@PeePeeSE">
```

### フォロー・シェアボタンの設置

```sh
$ mkdir theme/[テーマ名]/layout/_custome_sns
$ touch themes/[テーマ名]/layout/_custom_sns/twitter_follow.ejs
$ touch themes/[テーマ名]/layout/_custom_sns/twitter_share.ejs
```

`twitter_follow.ejs` ファイルを以下のように編集する。（IDは書き換えてね）

```html
<a href="https://twitter.com/PeePeeSE" class="twitter-follow-button" data-show-count="false">Follow @PeePeeSE</a> <script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>
```

`twitter_share.ejs` ファイルを以下のように編集する。（IDは書き換えてね）

```html
<a href="https://twitter.com/share" class="twitter-share-button" data-via="PeePeeSE">Tweet</a> <script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>
```

以下を好きなところに設置。

```html
<!-- twitter follow button start -->
<%- partial('_custom_sns/twitter_follow') %>
<!-- twitter follow button end -->
```

```html
<!-- twitter share button start -->
<%- partial('_custom_sns/twitter_share') %>
<!-- twitter share button end -->
```

`themes/[テーマ名]/layout/layout.ejs` を直接編集してもよい。

## 画像の配置

```sh
$ mkdir source/images
```

上記に画像ファイルを配置する。  
以下のように任意の場所に張り付ける。

```html
<img src="../../images/xxx" alt="画像の説明">
```

**alt** 属性はちゃんと書く。

## RSS Feedの設置

```sh
$ npm install hexo-generator-feed --save --no-optional
```

以下のボタンを好きなところに配置。

```html
<a href="<%= config.url %>/atom.xml" title="RSS Feed"><img src="images/rss_32.png" alt="RSSを購読する"></a>
```

画像（images/rss_32.png）は自分で準備する。

## OGPの設定

**OGP** （Open Graph Protcol）は、FacebookやTwitterなどのSNSでシェアされた際に、そのページのタイトル・URL・概要・サムネイル画像を表示させる仕組みのこと。  
`themes/[テーマ名]/layout/_partial/head.ejs` に下記のような **open_graph** ヘルパーを使った箇所がある。

```html
<%- open_graph({twitter_id: theme.twitter_id, google_plus: theme.google_plus, fb_admins: theme.fb_admins, fb_app_id: theme.fb_app_id}) %>
```

上記である程度OGPの設定はされているがサムネイル画像は設定されていないため、下記のように **image** というプロパティを追加する。（[公式ドキュメント](https://hexo.io/docs/helpers.html#open-graph)）

```html
<%- open_graph({image: theme.ogp_image, twitter_id: theme.twitter_id, google_plus: theme.google_plus, fb_admins: theme.fb_admins, fb_app_id: theme.fb_app_id}) %>
```

上記では、imageパラメータの値として「 **theme.ogp_image** 」を使用しており、 `themes/[テーマ名]/_config.yml` に以下のように記載することで参照できる。

```yml
ogp_image: /images/xxx.gif
```

## URLのクロールとインデックス登録を検索エンジンにリクエストする

- Google（Yahoo）
    - Googleアカウントを取得してGoogle Search Consoleの[Fetch as Google](https://www.google.com/webmasters/tools/googlebot-fetch)からリクエストする。
- [bing](https://www.bing.com/toolbox/submit-site-url)

## 数式を表示できるようにする

以下を実施することにより **Tex** による数式を表示できるようになる。

```sh
$ brew install pandoc
$ npm install hexo-renderer-pandoc --save --no-optional
$ npm install hexo-renderer-mathjax --save --no-optional
```
- [Pandoc ユーザーズガイド 日本語版](http://sky-y.github.io/site-pandoc-jp/users-guide/)
- [参考](http://qiita.com/sky_y/items/7c29909c5cffa28b23d8)

## Gist-it で Github のソースコード貼り付け

以下のようにスクリプトを貼り付ける。

```html
<script src="https://gist-it.appspot.com/github/[Github Account]/[Repository]/blob/[branch]/[path-to-file]?footer=0"></script>
```

例えば以下。

```html
<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_index.js?footer=0"></script>
```

以下のように表示される。

<script src="https://gist-it.appspot.com/github/pepese/js-sample/blob/master/express-sample/app/controllers/get_index.js?footer=0"></script>
