---
title: "auth/unauthorized-domainエラーについて"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Firebase, 初心者, React, Vercel]
published: true
---
## 状況
React + Vite + Firebaseで作ったWebアプリをVercelを使ってデプロイした際、ログイン処理をしようとしたら`auth/unauthorized-domain`というエラーが出てしまったので、それについてメモします。
![](https://storage.googleapis.com/zenn-user-upload/750efff8384c-20240313.png)

## 解決策
- Firebaseの承認済みドメインに`vercel.app`を追加する。

## エラーの原因
`auth/unauthorized-domain`というエラーメッセージからも分かる通り、Firebase側で許可されていないドメインからはFirebaseへアクセスできないようです。Firebaseを使ったWebアプリをVercelを使ってデプロイする際は、`vercel.app`をFirebase内Authenticationの設定で「承認済みドメイン」に追加することでアクセスができるようになります。

![](https://storage.googleapis.com/zenn-user-upload/baeb8650b116-20240313.png)

Vercel以外でもドメイン名を同様に追加してあげればこのエラーは解消できると思います。

## 参考
https://github.com/vercel/next.js/discussions/37636