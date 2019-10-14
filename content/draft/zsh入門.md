---
title: "Zsh入門"
date: 2019-10-14T16:07:32+09:00
slug: "zsh-basics"
tags:
- zsh
draft: true
archives:
- 2019/10
---

MacOS Catalina から zsh が標準になったので入門してみる。  
なお、深堀はしてない。

<!--more-->

# 初期設定

Catalina にアップデートすると以下が表示されるようになる。

```bash
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
```

以下を実行する。

```bash
$ chsh -s /bin/zsh
Changing shell for <user>.
Password for <user>:
$ cat ~/.bash_profile > ~/.zshenv
```

なお zsh がログインシェルとして起動された際、 `~/.zshenv` -> `~/.zprofile` -> `~/.zshrc` -> `~/.zlogin` の順で評価される。  
上記実行後、新しくターミナルを開いて `$` が `%` になっていれば変わっている。念のため以下のコマンドで確認できる。

```zsh
% echo $SHELL
/bin/zsh
```

また、ターミナル起動時に `command not found` と言われる場合は、 `~/.zshenv` の 1 行目に以下を加筆する。

```zsh
export PATH="/usr/local/bin:$PATH"
```

なお、 bash と zsh の違いであるが、大きくは以下の 2 つ。

1. `~` の評価
  - ユーザホームのパスに使うアレ、評価のされかたが微妙に違うみたい
2. 配列の初期インデックス
  - 0 からじゃなくてなぜか 1 から始まるみたい

細かくは個別の事象で対応するとしよう。

# [Prezto](https://github.com/sorin-ionescu/prezto)

zsh のフレームワークとして [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) をよく聞くが、ここでは [Prezto](https://github.com/sorin-ionescu/prezto) を設定してみる。  
今更こんなことを書いて恐縮だが、 zsh の初期設定をやってしまった人は以下でいったん bash に戻す。

```zsh
% rm ~/.zshenv ~/.zsh_history
% chsh -s /bin/bash
Changing shell for <user>.
Password for <user>:
# ターミナルを再起動
$ echo $SHELL
/bin/bash
```

以下で Prezto を導入する。

1. zsh のローンチ
  - `$ zsh`
2. Prezto の取得
  - `% git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"`
3. zsh の設定を作成
  - 
  ```
  % setopt EXTENDED_GLOB
  for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
    ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
  done
  ```
4. デフォルトシェルを zsh へ
  - 
  ```
  % chsh -s /bin/zsh
  Changing shell for <user>.
  Password for <user>:
  ```
5. ターミナルの再起動

アップデートは以下。

```zsh
% cd $ZPREZTODIR
% git pull
% git submodule update --init --recursive
```

アンインストールは以下。

```
% rm -rf ~/.zprezto ~/.zlogin ~/.zlogout ~/.zpreztorc ~/.zprofile ~/.zshenv ~/.zshrc
```

ここまで書いてアレだが、筆者は馴染まなかったので Prezto 使ってない、、、。