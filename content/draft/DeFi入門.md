---
title: "DeFi入門"
date: 2020-03-19T07:19:53+09:00
tags:
- DeFi
- Ethereum
draft: true
---

分散型金融（ DeFi:Decentralized Finance）についてまとめる。

# DeFi とは？

DeFi とは、ブロックチェーンをあらゆる金融分野（証券、保険、デリバティブ、レンディングなど）に応用し、分散型で中央管理者を必要としない透明性の高い金融プラットフォーム。  
現在は主に Ethereum を中心とした Defi のエコシステムが出来つつある。

# キーワードベースでざっくり理解

- ICO：Ether(ETH)を調達しする見返りとして 自社発行の ERC20 トークンを配布
- ERC20 トークン：Ethereum ブロックチェーン上でトークンを発行する際の標準規格（Ethereum RFC (Request for Comment)）
  - ERC20 に準拠したトークンであれば、無数に存在する種類のトークンを同じ枠組みで価値移転することができる
  - https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20-token-standard.md
  - Ganache（ローカル開発環境）、Metamask（Ethereumウォレット）、[Truffle](https://www.trufflesuite.com/) （Ethereum の開発フレームワーク）、[Solidity](https://solidity.readthedocs.io/en/develop/)（プログラミング言語）を用いて開発する
- dApp/Decentralized Application（分散型アプリケーション）：スマートコントラクトを活用することで機能
- DEX/Decentralized Exchange（分散取引所）：取引所機能を持つdApps
  - 取引所に資産を預けることなく、ERC20トークンであれば簡単にETHとトレードすることができる
- ステーブルコイン
  - 法定通貨担保型：USDTなど。Tether社が発行量と同等の米ドルを確保して（いるとされている）価格を担保している
  - 仮想通貨担保型：DAI。ETHを担保にしている。
  - 無担保型：Basis。アルゴリズムで供給量を調整する。
- [MakerDAO](https://makerdao.com/ja/)：makerDAOはステーブルコインDAIのプロトコル
  - Collateralized Debt Position (CDP)というスマートコントラクトにより担保されたETHによってDAIの価値を維持するための様々な仕組みが、Ethereum上のスマートコントラクトで実現されている
  - [MakerDAO のステーブルコイン DAI とは！？ スマホでの発行方法！](https://makionaire.com/makerdao-dai/)

# [DeFi Projects](https://defiprime.com/)

- [Alternative Savings](https://defiprime.com/alternative-savings)：代替貯金？
  - [PoolTogether](https://www.pooltogether.com/)：手軽な宝くじdApp
- Analytics
- Asset Management Tools
- DAOs
- Decentralized exchanges/DEXES：両替
  - [Uniswap](https://uniswap.exchange/swap)
- Derivatives：デリバティブ
  - [Synthetix](https://www.synthetix.io/)
- Infrastructure & Dev Tooling
- Insurance：保険
- KYC & Identity
- [Lending(& Borrowing)](https://defiprime.com/decentralized-lending)：ローン・融資（貸す借りる）による利子
  - [Maker](https://cdp.makerdao.com/)
  - [Compound](https://compound.finance/)：Ethereum上の暗号通貨の貸借のためのプロトコル
    - [「Compound」でLending Dappを作ってみる](https://blockchain-insight.ch/jp/2019/11/30/1826)
- Margin trading
- Marketplaces
- Payments
- Prediction markets
- Stablecoins
- Staking
- Tokenization of Assets

以下、 DeFi Project の状態を理解するサイトなど。

- [DeFi Pulse](https://defipulse.com/)
  - DeFi protocols のランキング
  - 1 時間に 1 回 Ethereum からスマートコントラクトを通じて Ether と ECR-20 トークンの残高を取得して TVL（total value locked）を計算している。
  - TLV はプロトコル上に存在する Ether や ECR-20 トークンを USD などに換算した際の合計値
  - TLV を比較することで DeFi protocols の比較を行っている
- [DAppTotal](https://dapptotal.com/defi)

# DeFi で収入を得る方法

## 融資による利子

1. Ether を Wallet に送る
2. DEXES で Ether を DAI に両替する
3. Compound などで融資する（参考：[Earn Interest Income](https://defipulse.com/income)）
4. 利子（interest）を得る

## Pool Together

- 参考
  - https://zenism.jp/defi-blog/pooltogether-how-to/2019/06/25/
  - https://zenism.jp/projects/lending/pooltogether/2019/06/25/

# 参考

- Web
  - [defiprime.com](https://defiprime.com/)
  - [次世代の分散型金融（Defi）サービスを一通り体験してみた](https://lab.stir.network/2019/02/07/tried-experiencing-the-next-generation-of-defi-service/)
  - [HashHub Engineer Blog](https://medium.com/blockchain-engineer-blog)
  - [【未経験者向け】Dapp開発(実装編)のおすすめの学習方法](https://qiita.com/taishinagasaki/items/63441f7e933cd6aa144c)
  - [【イーサリアム】 SolidityとTruffleでペットショップのDappをつくる！](https://zoom-blc.com/how-to-create-first-dapp)
- Community
  - [d10nlab](https://d10nlab.com/)
  - [DeFi Japan - Medium](https://medium.com/defi-japan)
  - [0x Japan](https://medium.com/0x-japan)
- Report
  - [2020年のDeFi(分散型金融)の10のトレンド予測](https://drive.google.com/open?id=1302RnpzLnVriaKYtRJjm-UDSNP9xjCXZ)
  - [使えるDeFiサービスとクリプト×カード決済サービスのまとめ](https://drive.google.com/open?id=1DtxT2E_Nyc1qW_c-GFE-C_uhT1btXtvy)
- Company
  - [MakerDAO](https://makerdao.com/ja/)
  - [HashHub](https://hashhub.tokyo/)
  - [Kyber Network](https://kyber.network/)