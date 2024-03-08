---
title: "useEffectの3つの使い方"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [React, useEffect, 初心者]
published: true
---
:::message
(2024/3/8追記)何でもかんでもuseEffectを使えば良いというわけではなく、むしろ「アプリケーションのロジックとデータフローの大部分は、これらの機能に依存しないようにすべき」と公式の方で説明があるようです。
https://ja.react.dev/learn/escape-hatches
指摘してくださった方、ありがとうございます。
:::

React頻出のuseEffectについて、最近ちょっとだけ知ったので備忘録としてまとめます。対象は私のようなReact初心者です。
## ３つのuseEffectの書き方とその違い

### １．毎レンダリング時に実行
```jsx
useEffect(() => {
    console.log("レンダリングされました。");
})
```
一つ目はレンダリングされるたびに実行されるuseEffect()です。

### ２．初回レンダリング時のみ実行
```jsx
useEffect(() => {
    console.log("初回レンダリングが実行されました。");
}, [])
```
二つ目もレンダリング実行時に実行されるのですが、初回レンダリング時のみという制限がつきます。最後に`[]`があるのが一つ目と違います。

### 3.監視対象の変数の更新時に実行
```jsx
useEffect(() => {
    console.log("valが変更されました。");
}, [val])
```
最後は監視対象の変数が更新された際に実行されるものです。監視対象の変数は末尾の`[]`の中に記載します。例では`val`という変数を監視しています。

## useEffectの簡単な例
3つめのある変数を監視するuseEffectを使って、入力した内容がそのまま下に表示される簡単なコードを書いてみました。
```jsx
import { useEffect, useState } from "react";
import "./styles.css";

export default function App() {
  const [inputTxt, setInputTxt] = useState("");
  const [txt, setTxt] = useState("");

  useEffect(() => {
    setTxt(inputTxt);
  }, [inputTxt]);

  return (
    <div className="App">
      <input
        type="text"
        value={inputTxt}
        onChange={(e) => setInputTxt(e.target.value)}
      />
      <p>{txt}</p>
    </div>
  );
}
```
input欄の下に表示させる変数`txt`とinput欄の中の文字列である`inputTxt`の2つ用意して、input欄で何か文字が打たれたら（=inputTxtに変更があったら）、それをuseEffectで検知して`txt`に`inputTxt`の内容をセットするという流れです。
![](https://storage.googleapis.com/zenn-user-upload/cfd90e695813-20240308.png)

## まとめ
useEffect()は基礎中の基礎だと思うので、何回も使って慣れていきたいですね。何か間違いがあれば指摘していただけると幸いです。