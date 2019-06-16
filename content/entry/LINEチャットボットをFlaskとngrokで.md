---
title: "LINEチャットボットをFlaskとngrokで"
date: 2017-11-01T08:05:09+09:00
slug: "flask-ngrok-chatbot"
tags:
- flask
- yython
- ngrok
- line
draft: false
archives:
- 2017/11
---

[Flask](http://flask.pocoo.org/) と [ngrok](https://ngrok.com/) でメッセージをオウム返しする LINE のチャットボットを作ってみた。  
Heroku で公開したい場合は以下参照。

- [Heroku入門](https://blog.pepese.com/entry/heroku-basics/)

<!--more-->

# セットアップ

```sh
$ pip install Flask gunicorn line-bot-sdk
$ brew cask install ngrok
```

# ソースコード

app.py
```python
from flask import Flask, request, abort
from linebot import LineBotApi, WebhookHandler
from linebot.exceptions import InvalidSignatureError
from linebot.models import MessageEvent, TextMessage, TextSendMessage

app = Flask(__name__)

line_bot_api = LineBotApi('アクセストークン')
handler = WebhookHandler('Channel Secret')

@app.route("/")
def hello_world():
    return "hello world!"

@app.route("/webhook", methods=['POST'])
def webhook():
    # get X-Line-Signature header value
    signature = request.headers['X-Line-Signature']

    # get request body as text
    body = request.get_data(as_text=True)
    app.logger.info("Request body: " + body)

    # handle webhook body
    try:
        handler.handle(body, signature)
    except InvalidSignatureError:
        abort(400)

    return 'OK'


@handler.add(MessageEvent, message=TextMessage)
def handle_message(event):
    line_bot_api.reply_message(
        event.reply_token,
        TextSendMessage(text=event.message.text))


if __name__ == "__main__":
    app.run()
```

Procfile
```
web: gunicorn -b 0.0.0.0:$PORT app:app --log-file=-
```

requirements.txt
```
Flask==0.12.2
gunicorn==19.7.1
line-bot-sdk==1.5.0
```

runtime.txt
```
python-3.6.2
```

# 起動

## ローカルでAP起動

```sh
$ FLASK_APP=app.py flask run
 * Serving Flask app "app"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

## ngrokで公開

```sh
$ ngrok http 5000
ngrok by @inconshreveable                                       (Ctrl+C to quit)

Session Status                online                                            
Version                       2.2.8                                             
Region                        United States (us)                                
Web Interface                 http://127.0.0.1:4040                             
Forwarding                    http://8eeb5441.ngrok.io -> localhost:5000        
Forwarding                    https://8eeb5441.ngrok.io -> localhost:5000       

Connections                   ttl     opn     rt1     rt5     p50     p90       
                              2       0       0.02    0.01    0.01    0.01      

HTTP Requests                                                                   
-------------                                                                   

GET /favicon.ico               404 NOT FOUND                                    
GET /                          200 OK
```

なお好きなホスト名をつけたい場合は、ngrokの有料サービスを利用する必要がある模様。