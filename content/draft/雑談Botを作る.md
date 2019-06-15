---
title: 雑談Botを作る
tags:
id: ai-chat
draft: true
---

# 雑談

入りの論文
- [対話行為と話題推定によるラベル伝搬を利用した雑談生成方法の改良](https://kaigi.org/jsai/webprogram/2016/pdf/1120.pdf)

気軽に雑談できるシステムの実現をめざして
- [気軽に雑談できるシステムの実現をめざして](http://www.ntt.co.jp/journal/1609/files/jn20160916.pdf)

# ネガポジ

- [単語感情極性対応表/NP Table](http://www.lr.pi.titech.ac.jp/~takamura/pndic_ja.html)
    - この辞書には岩波国語辞書（岩波書店）に載っている単語がポジティブであるほど+1に近い値、ネガティブであるほど-1に近い値がふられている
    - 全単語55,125中、ポジティブワード（スコア0以上）が5,242語、ネガティブワードが49,983語
- [日本語評価極性辞書](http://www.cl.ecei.tohoku.ac.jp/index.php?Open%20Resources%2FJapanese%20Sentiment%20Polarity%20Dictionary)

# シソーラス

- [分類語彙表](http://pj.ninjal.ac.jp/corpus_center/goihyo.html)
- [日本語 WordNet](http://compling.hss.ntu.edu.sg/wnja/)

# メモ

文章 -> インテント（カテゴリ）に分類 -> インテント（カテゴリ）に基づいて特徴語（エンティティ）を抽出

上記の結果から対話を実施する。対話のあり方としては以下。基本・観点はどんな返事（検索結果）か？  

- FAQ
- （商品など）の検索
    - 不足する情報は聞き返す
- 雑談
    - 挨拶、プロフィールに対する回答（FAQの一部とも考えられる？）

## 分類

<table border="1">
  <tr>
    <td colspan="5" rowspan="1">文法</td>
    <td colspan="2" rowspan="2">ネガポジ</td>
  </tr>
  <tr>
    <td colspan="1" rowspan="2">肯定文</td>
    <td colspan="1" rowspan="2">否定文</td>
    <td colspan="1" rowspan="2">命令文</td>
    <td colspan="2" rowspan="1">疑問文</td>
  </tr>
  <tr>
    <td>Closed Question</td>
    <td>Open Question</td>
    <td>ネガ</td>
    <td>ポジ</td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>
