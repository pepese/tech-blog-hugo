---
title: "HexoからHugoに移行する"
date: 2019-05-26T16:09:06+09:00
slug: "hugo-basics"
tags:
- hugo
draft: true
archives:
- 2019
- 2019/05
- 2019/05/26
---

Hexo から Hugo に移行する時のメモ。

- Hugo 環境構築
- 記事の作成
- Tips

<!--more-->

# Hugo 環境構築

インストールは以下。

```bash
$ brew install hugo
$ which hugo
/usr/local/bin/hugo
$ hugo version
Hugo Static Site Generator v0.55.6/extended darwin/amd64 BuildDate: unknown
```

プロジェクト作成と初期設定は以下。

```bash
$ hugo new site tech-blog-hugo
Congratulations! Your new Hugo site is created in /path/to/workspace/tech-blog-hugo.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.

$ cd tech-blog-hugo
$ git init
Initialized empty Git repository in /path/to/workspace/tech-blog-hugo/.git/
$ git remote add origin https://github.com/pepese/tech-blog-hugo.git
$ git config --local user.name pepese
$ touch .gitignore
$ echo /public > .gitignore
```

# 記事の作成

コマンドは `hugo -h` あるいは `hugo [command] -h` で確認できる。

## テーマの設定

[Hugoのテーマ一覧](https://themes.gohugo.io/) から好きなテーマを選んで以下のように設定する。

```bash
$ cd themes
git submodule add https://github.com/Vimux/mainroad themes/mainroad
```

## 設定ファイルの編集

`config.toml` を編集する。  
設定はテーマによってクセがあるので、テーマの github をよく見ること。

```
DefaultContentLanguage = "ja"
languageCode = "ja-JP"

baseURL = "https://pepese.github.io/"
title = "ぺーぺーSEのテックブログ"
theme = "mainroad"
enableRobotsTXT = true
googleAnalytics = "xxxx"

[Params]
  description = "備忘録用メモサイト"
  twitter_cards = true
  readmore = true

[Params.sidebar]
  home = "right"
  list = "left"
  single = true
  widgets = ["social", "recent", "taglist"]

[Params.widgets]
  recent_num = 5 # Set the number of articles in the "Recent articles" widget
  tags_counter = false # Enable counter for each tag in "Tags" widget (disabled by default)

[Params.widgets.social]
  twitter = "xxxx"
  github = "xxxx"
```

## 記事の生成と記載

記事は `hugo new <content配下のディレクトリ>/タイトル.md` コマンドで作成する。

```bash
$ hugo new blog/HexoからHugoに移行する.md
/path/to/workspace/tech-blog-hugo/content/blog/HexoからHugoに移行する.md created
```

記事へのパスは `content/` ディレクトリ配下の構造と同じになる。  
上記だと `[baseURL]/blog/HexoからHugoに移行する/` が記事へのパス。  
ファイル名がパスになるのがイヤな場合は、各記事で `slug` を設定すると変更できる。  
また、まだ作成中の記事は `draft` を `true` にする。

```
---
title: "HexoからHugoに移行する"
date: yyyy-MM-dd...
slug: "hugo-basics"
tags:
- hugo
draft: true
---

Hexo から Hugo に移行する時のメモ。

<!--more-->
```

上記の `slug` 設定だと記事へのパスは `[baseURL]/blog/hugo-basics/` になる。  
また、記事生成時の雛形は `archetypes/default.md` となっており、 `<!--more-->` （ READ More ： 続きを読む の境界）など好みの設定を入れておくとよい。

## ローカル実行

`hugo server` コマンドでローカル実行できる。

```bash
$ hugo server -D
```

`-D` オプションを付与することで、ドラフト版の記事もビルド・確認できる。

# Tips

## Github Pagesへの公開

公開用コンテンツが生成された `public/` を普通に Github Pages に対応したリポジトリに push する。

```bash
$ hugo
$ cd public/
$ git init
$ git remote add origin https://pepese@github.com/pepese/pepese.github.io.git
$ git add --all
$ git commit -m "update"
$ git push origin master
```

`branch` 作るとかは好みで。

## アーカイブページの作成

実施中。。。

- https://backport.net/blog/2017/09/08/hugo_monthly_archives/
- http://staff.feedtailor.jp/2016/08/10/hugo_16/

## sitemap.xml の作成

デフォで作成されてる。

## robots.txt の作成

デフォで作成されてるが、自分で変種したい時は以下。

- `config.toml` の設定で `enableRobotsTXT = true` とする
- `/layouts/robots.txt` に雛形を準備する

```:robots.txt
User-agent: *
Disallow : /img/
Disallow : /css/
Disallow : /js/
Sitemap : {{ $.Site.BaseURL }}sitemap.xml
```

## 404ページの作成

デフォで作成されてる。

## Google Adsenseの設置

ポイントは、 **取得したテーマを直接編集不要** ということ。  
Hugo では 同名のファイルであれば、 テーマのレイアウト（ `themes/[テーマ名]/layouts` ）より独自のレイアウト（ `layouts/` ）が優先される。  
そのため、ヘッダ、フッタ、サイドバーなどアドを差し込みたい部品をそれぞれ独自に作成する。

```bash
$ mkdir layouts/partials
$ touch layouts/partials/header.html
$ touch layouts/partials/footer.html
$ touch layouts/partials/sidebar.html
```

上記のファイルについて、利用しているテーマから同名のファイルの中身をコピペして好きな内容を書き加えればカスタマイズできる。  
以下のようなアドを上記のファイルに適宜差し込む。

```html
<!-- ad start -->
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:90px"
     data-ad-client="xxxx"
     data-ad-slot="xxxx"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>
<!-- ad end -->
```

## Google Analyticsの設置

`config.toml` に以下を設定するだけ。

```
googleAnalytics = "UA-xxxxxxxx-x"
```

## Amazonアソシエイトの設置

やり方は Google Adsense の時と同様で、以下のようなアドを `layouts/` 配下のファイルに適宜差し込む。

```html
<!-- ad start -->
<!-- Amazon -->
<script type="text/javascript"><!--
amazon_ad_tag = "xxxxxxxx"; amazon_ad_width = "160"; amazon_ad_height = "600"; amazon_ad_logo = "hide";//--></script>
<script type="text/javascript" src="http://ir-jp.amazon-adsystem.com/s/ads.js"></script>
<!-- ad end -->
```

## Twitterの設定

### Twitter Cards

`config.toml` ファイルに以下の設定を追加する。

```
[Params]
  description = "ぺーぺーSEがプログラミング（Go、Java、JavaScript、Python、Ruby）、クラウド（AWS）、Web構築などのメモを残すサイト。"
  twitter_cards = true

[Params.widgets.social]
  twitter = "@PeePeeSE"
```

これでheadタグ内に以下のメタが追加される。（所謂 **Twitter Cards**）

```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="ぺーぺーSEのテックブログ">
<meta name="twitter:description" content="ぺーぺーSEがプログラミング（Java、JavaScript、Python、Ruby）、クラウド（AWS）、Web構築などのメモを残すサイト。">
<meta name="twitter:creator" content="@PeePeeSE">
```

### フォロー・シェアボタンの設置

これまた やり方は Google Adsense の時と同様。  
以下のようなフォローボタンを `layouts/` 配下のファイルに適宜差し込む。

```html
<a href="https://twitter.com/PeePeeSE" class="twitter-follow-button" data-show-count="false">Follow @PeePeeSE</a> <script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>
```

以下のようなシェアボタンを `layouts/` 配下のファイルに適宜差し込む。

```html
<a href="https://twitter.com/share" class="twitter-share-button" data-via="PeePeeSE">Tweet</a> <script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>
```

## 画像の配置

`static/` 配下に `img` というディレクトリを作ってそこに画像を保存する。  
記事から参照する場合は、 Markdown で

```
![This is a image](/img/img01.png)
```

というふうに記載すると HTML で

```html
<p><img src="/img/img01.png" alt="This is a image" /></p>
```

のように展開される。  
画像サイズなど指定したい場合は Markdown の中に直接 HTML タグを書いてしまう。

## RSS Feedの設置

RSS のファイルはデフォで作成されているが、`index.xml` という名前でちょっとイヤなので `config.toml` に以下の設定をしてリネームする。

```
[outputFormats.RSS]
baseName = "feed"
```

上記で `/feed.xml` で参照できるようになるので、以下のように好きな場所にボタンを設置する。

```html
<!-- RSS Feed start -->
<a href="/feed.xml" title="RSS Feed"><img src="/img/rss_32.png" alt="RSSを購読する"></a>
<!-- RSS Feed end -->
```

## OGPの設定

[OGP](http://ogp.me/) （Open Graph Protcol）は、FacebookやTwitterなどのSNSでシェアされた際に、そのページのタイトル・URL・概要・サムネイル画像を表示させる仕組みのこと。  
以下で OGP が有効になる。

```
[Params]
  opengraph = true
```

以下で画像を設定できる。

```
[Params]
  images = ["img/yaruwo.gif"]
```

なお、上記の設定方法は現在のテーマが利用している `_internal/` 以下のファイルを利用するもので標準の [Internal Template](https://github.com/gohugoio/hugo/tree/master/tpl/tplimpl/embedded/templates) の仕様に従っている。

## URLのクロールとインデックス登録を検索エンジンにリクエストする

- Google（Yahoo）
    - Googleアカウントを取得してGoogle Search Consoleの[Fetch as Google](https://www.google.com/webmasters/tools/googlebot-fetch)からリクエストする。
- [bing](https://www.bing.com/toolbox/submit-site-url)

## 数式を表示できるようにする

実施中。。。

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