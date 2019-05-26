---
title: "HexoからHugoに移行する"
date: 2019-05-26T16:09:06+09:00
draft: false
---

Hexo から Hugo に移行する時のメモ。

- Hugo 環境構築
- 記事の作成
- Tips

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

コマンド一覧は以下。

```bash
$ hugo help
hugo is the main command, used to build your Hugo site.

Hugo is a Fast and Flexible Static Site Generator
built with love by spf13 and friends in Go.

Complete documentation is available at http://gohugo.io/.

Usage:
  hugo [flags]
  hugo [command]

Available Commands:
  check       Contains some verification checks
  config      Print the site configuration
  convert     Convert your content to different formats
  env         Print Hugo version and environment info
  gen         A collection of several useful generators.
  help        Help about any command
  import      Import your site from others.
  list        Listing out various types of content
  new         Create new content for your site
  server      A high performance webserver
  version     Print the version number of Hugo

Flags:
  -b, --baseURL string         hostname (and path) to the root, e.g. http://spf13.com/
  -D, --buildDrafts            include content marked as draft
  -E, --buildExpired           include expired content
  -F, --buildFuture            include content with publishdate in the future
      --cacheDir string        filesystem path to cache directory. Defaults: $TMPDIR/hugo_cache/
      --cleanDestinationDir    remove files from destination not found in static directories
      --config string          config file (default is path/config.yaml|json|toml)
      --configDir string       config dir (default "config")
  -c, --contentDir string      filesystem path to content directory
      --debug                  debug output
  -d, --destination string     filesystem path to write files to
      --disableKinds strings   disable different kind of pages (home, RSS etc.)
      --enableGitInfo          add Git revision, date and author info to the pages
  -e, --environment string     build environment
      --forceSyncStatic        copy all files when static is changed.
      --gc                     enable to run some cleanup tasks (remove unused cache files) after the build
  -h, --help                   help for hugo
      --i18n-warnings          print missing translations
      --ignoreCache            ignores the cache directory
  -l, --layoutDir string       filesystem path to layout directory
      --log                    enable Logging
      --logFile string         log File path (if set, logging enabled automatically)
      --minify                 minify any supported output format (HTML, XML etc.)
      --noChmod                don't sync permission mode of files
      --noTimes                don't sync modification time of files
      --path-warnings          print warnings on duplicate target paths etc.
      --quiet                  build in quiet mode
      --renderToMemory         render to memory (only useful for benchmark testing)
  -s, --source string          filesystem path to read files relative from
      --templateMetrics        display metrics about template executions
      --templateMetricsHints   calculate some improvement hints when combined with --templateMetrics
  -t, --theme strings          themes to use (located in /themes/THEMENAME/)
      --themesDir string       filesystem path to themes directory
      --trace file             write trace to file (not useful in general)
  -v, --verbose                verbose output
      --verboseLog             verbose logging
  -w, --watch                  watch filesystem for changes and recreate as needed

Use "hugo [command] --help" for more information about a command.
```

## テーマの設定

[Hugoのテーマ一覧](https://themes.gohugo.io/) から好きなテーマを選んで以下のように設定する。

```bash
$ cd themes
$ git clone https://github.com/Vimux/mainroad themes/mainroad

git submodule add https://github.com/Vimux/mainroad themes/mainroad
```

## 設定ファイルの編集

`config.toml` を編集する。  
設定はテーマによってクセがあるので、テーマの github をよく見ること。

```
```

## 記事の生成と記載

以下のコマンドで記事を作成する。

```bash
$ hugo new posts/タイトル.md
/path/to/workspace/tech-blog-hugo/content/posts/タイトル.md created
```

上記の Markdown に記事を書く。

## HTML の生成

# Tips

ああ。