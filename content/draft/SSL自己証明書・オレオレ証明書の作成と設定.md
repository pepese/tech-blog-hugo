---
title: "SSL自己証明書・オレオレ証明書の作成と設定"
date: 2020-03-05T10:19:35+09:00
slug: ""
tags:
- tag_name
draft: true
archives:
- 2019/06
---

- 自己証明書・オレオレ証明書の作成
- サーバへ設定

<!--more-->

# 自己証明書・オレオレ証明書の作成

SSLを利用するには **SSLサーバ証明書** を設定する必要がある。  
正規にはサーバ証明書を購入する必要があるが、ここでは非正規の自己証明書を自分で作成する方法を記載する。

|ファイル|拡張子|説明|
|:---|:---|:---|
|秘密鍵|.pem|rsa暗号方式をで暗号化されたファイルを現したもので、鍵の種類を表しているわけではない。.key でもよい。|
|秘密鍵|.key|もれたら危険な秘密鍵。|
|公開鍵|.csr|公開鍵。証明書発行要求のファイル。|
|サーバ証明書・デジタル証明書・SSL証明書|.crt|公開鍵に認証局が署名した証明書のファイル。|

以降、 openssl を利用して作成していく。

## 秘密鍵（.key）の作成

```
$ openssl genrsa 2048 > server.key

Generating RSA private key, 2048 bit long modulus
...................................................................+++
...........................+++
e is 65537 (0x10001)
Enter pass phrase:「パスフレーズを入力」
Verifying - Enter pass phrase:「パスフレーズを入力」
```

- `-aes256` のようなオプションで暗号化方式を指定可能
  - 使用可能な暗号化方式を確認するには、 `openssl enc --help`
- 2048 は鍵の長さ
  - 鍵の長さは2048ビット以上が推奨

## 公開鍵（.csr）の作成

```
$ openssl req -new -key server.key > server.csr

Enter pass phrase for server.key:「秘密鍵のパスフレーズ」
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:「国：JPとか入力」
State or Province Name (full name) []:「州・都道府県：Tokyoとか入力」
Locality Name (eg, city) []:「市区町村：区とか入力」
Organization Name (eg, company) []:「会社名：Example Inc.とか入力」
Organizational Unit Name (eg, section) []:「組織名：Exampleとか入力」
Common Name (eg, fully qualified host name) []:「名前：適当に入力」
Email Address []:「メールアドレス」

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:「パスワード」
```

先に作成した秘密鍵を使って作成。

## サーバ証明書（.crt）の作成

```
$ openssl x509 -in server.csr -days 365000 -req -signkey server.key > server.crt

Signature ok
subject=/C=JP/ST=Tokyo/L=xxxx/O=xxxx/OU=xxxx/CN=xxxx/emailAddress=xxxx
Getting Private key
Enter pass phrase for server.key:「秘密鍵のパスフレーズ」
```

# サーバへ設定

HTTPS 化したいサーバ秘密鍵（.key）とサーバ証明書（.crt）を設定すればよい。  
なお、 `curl` で疎通したい場合、オレオレだと認証局を利用しないので `-k` オプションを付与する。

```
$ curl -k -X POST -H "Content-Type: application/json" -d "{\"xxxx\":\"xxxx\"" https://localhost:8080/xxxx
```