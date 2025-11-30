---
title: "初VJに挑戦するにあたり、Webシェーダエディタを作ってみた"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "WebGL", "React"]
published: false
---
[MYJLab Advent Calendar 2025](https://qiita.com/advent-calendar/2025/myjlab)の2日目を担当します、Sigreniと申します。せっかくなので勝手ながら[グラフィックス全般 Advent Calendar 2025](https://qiita.com/advent-calendar/2025/graphics)へもマルチポストします。よろしくお願いします🤲

軽く自己紹介すると、技術的にはWebフロントエンドを2年ちょっと齧っている修士一年生です。研究室とはあまり関係ありませんが、ここ一年程のマイブームがCGです。そんな中、先日はじめてVJをやる機会があり、シェーダを使ったライブコーディングに挑戦しました。その際にWebシェーダエディタを作成したので、それについてお話しできればと思います。

（作成したものは以下になります↓ ぜひいじってみてください〜）
https://somahc.github.io/glsl-livecoding-editor-web/
## 前提知識や背景
いきなり横文字をたくさん出してしまいましたが、言葉に馴染みのない方もいると思うので簡単に補足します。
### 「VJ」「シェーダ」とは?
まず、VJとはVisual JockeyやVideo Jockeyの略です。クラブなどで音楽を選択して流すDJはご存じの方が多いと思いますが、その映像版になります。VJは音楽に合った映像を流すことで場を視覚的に盛り上げます。
https://x.com/tktk_1/status/1977315624549487007?s=20

シェーダ(shader)はディスプレイ上にオブジェクトが表示される際、その表示のされ方を決めるプログラムになります。現実を模したリアルな質感にしたり、アニメ調にデフォルメした質感を出したりなど、様々なことができます。Unityなどでゲーム制作の経験がある方は聞いたことがあるかもしれません。シェーダ言語として有名なものにGLSL(OpenGL Shading Language)があります。

基本的にシェーダ単体で使うことはないのですが、レイマーチングなどのテクニックを使うことで、GLSLを書くだけで面白い絵を出すことができます（詳細は割愛します）。俗に”シェーダ芸”と呼ばれたりもします。
https://youtu.be/NX5vcd-ZY9s?si=q0PTH2bcHSzIfBUT&t=1160

今回はシェーダを実際に書くことで映像を変化させる、ライブコーディングスタイルでパフォーマンスしてみました。

### なぜ制作したの？
概要はふんわりわかっていただけたと思うのですが、そもそもシェーダエディタを作る必要があるのか、という話なんですが...。結論から言ってしまうと、作りたかったから作りました。（笑）
もうちょっと真面目に話すと、Three.jsのようなライブラリなしでWebGLを直接扱ったことがなく、勉強したいなと思っていた自分にとって良い機会でした。~~あとエディタから作ればシェーダが微妙でも許してもらえると思いました。~~

実際、[Bonzomatic](https://github.com/Gargaj/Bonzomatic)や[Sh4derJockey](https://github.com/slerpyyy/sh4der-jockey)といったパフォーマンスに使えるフリーのシェーダエディタは色々と存在します。Web上でシェーダをかける環境も色々とあるので、調べてみてください。

## 制作したエディタについて
制作したエディタの概要や技術スタックについて紹介します。作ったのが3ヶ月前なのでかなり忘れてるところもあるのですが、思い出しながら書きます。

### 概要
![制作したエディタの画面](https://storage.googleapis.com/zenn-user-upload/ec9401c625d9-20251127.png)
本稿の冒頭に貼ったリンクへアクセスすることで、今回制作したエディタを見ることができます。アクセスすると上のような画面が現れると思います。

コードを書き換えてCmd+S（Ctrl+S）キーでコンパイルを実行します。コンパイルが成功すると背景のシェーダー実行結果が更新されます。
また、画面下のUIからBPMという値を変更することができ、その値はbpmというuniform変数としてシェーダに渡されます。曲に合わせてBPMの値を調整することで、曲のテンポに合った映像を出すことが可能です。詳細はGitHubリポジトリのREADMEを参照してください。（[リンク](https://github.com/Somahc/glsl-livecoding-editor-web)）

## 利用技術
技術スタックは以下の通りです。

### ビルドツール
- Vite

### ライブラリ・パッケージなど
- React
- TypeScript
- react-codemirror
- jotai

かなりシンプルだと思います。要件としてはそこまで多くなく、GLSLコードを書き、背景でその実行結果が再生されれば最低限OKだったので、できるだけシンプルな構成にすることを目指しました。
技術選定にはそれが反映されていると思います。SSRとかはしなくていいのでNext.jsは使っていないし、コードエディタライブラリではMicrosoft製のMonaco Editorが有名ですが、より軽量なCodeMirrorを採用しました。また、CodeMirrorはカスタマイズが柔軟にできるというメリットもありました。（Monaco Editorは使ったことないのであまりわかりませんが、、）

## 大変だった or こだわったところ
実装はリポジトリをクローンしてもらえればいくらでも見れるのですが、ちょっと大変だったところやこだわりポイントをピックアップして紹介したいと思います。

### WebGL
まあ、予想はしてましたが、WebGLを生で扱うのはなかなか骨が折れました。WebGPUより楽とはいえ、セットアップは大変です。
WebGLで何らかの絵を出す流れは、ざっくり以下のようになります。

1. webgl2コンテキストでcanvasを取得する
2. vertex shaderとfragment shaderをそれぞれコンパイルする
3. vertex shaderとfragment shaderをアタッチ・リンクしてシェーダプログラムを作り、`useProgram()`で使用プログラムを指定する
4. VBO(Vertex Buffer Object, 頂点バッファ)を作成し、頂点座標データをvertex shaderのin変数に紐づける

あとは`requestAnimationFrame()`で毎フレームのループ処理を書いて終わりです。具体的には、シェーダに渡すuniform変数をセットと、三角ポリゴンの描画命令を書いてあげます。今回はuniform変数として画面解像度を表すresolutionと経過時間を表すtimeのほかに、アプリのUIから入力されるbpmを用意しています。
:::details WebGLの学習について
正直ここまでの内容はWebGLかOpenGLの経験が無いと謎の単語の羅列にしか見えないと思います。私も学習中で偉そうなことは言えませんが、wgld.orgというサイトで3D描画の基礎から、WebGLでの実装などについてわかりやすく解説されているので、興味のある方は覗いてみると面白いと思います。
https://wgld.org/sitemap.html
:::

#### フルスクリーントライアングル
canvasの画面いっぱいにfragment shaderの実行結果が広がってくれればよかったわけですが、それを実装するためフルスクリーントライアングルというテクニック？をChatGPTから教えてもらい、実装しました。座標空間は(-1,-1)~(1,1)だそうなんですが、要はそれをすべて覆うポリゴンがあれば良いわけです。そこで(-1,-1), (3,-1), (-1,3)を頂点とするポリゴンを描画するようにvertex shaderに指示しています。
作成したエディタで編集するのはfragment shaderのみであるため、この処理やvertex shaderのコード自体についてはハードコードしてあります。
```ts:ShaderCanvas.tsx
...

const vs = `#version 300 es
      layout(location=0) in vec2 pos;       // ← 固定ロケーション、板ポリなのでZは0で固定するから二次元
      void main(){ gl_Position = vec4(pos, 0.0, 1.0); }`;

...

// フルスクリーントライアングルの頂点位置
const vertexPosition = new Float32Array([-1, -1, 3, -1, -1, 3]);

// VBO(Vertex Buffer Object, 頂点バッファ)を作成し、頂点座標データをvertex shaderのin変数に紐づける
const vbo = glCreateBuffer(gl, vertexPosition);
gl.bindBuffer(gl.ARRAY_BUFFER, vbo);
gl.vertexAttribPointer(0, attStride, gl.FLOAT, false, 0, 0);
gl.enableVertexAttribArray(0);

...
```

#### canvas要素の管理
最初は素直にcanvasのrefをuseRefで管理してwebgl2コンテキストの取得処理とかを書いていたのですが、ReactのStrictModeにおいてcanvasが表示されるべきところで四角い悲しげな顔アイコンが表示されてしまっていました。イベントリスナを登録してエラーを出力させてみると、webglcontextlostが発生していることがわかりました。
0b5vr氏制作のwavenerdの実装で使われていたuseElementというhooksをほぼそのまま使ってcanvasを取得し、WebGLコンテキスト取得を行うuseEffectの依存配列に含めることで解決しました。
https://github.com/0b5vr/wavenerd/blob/dev/src/view/utils/useElement.ts

`useLayoutEffect`はここで初めて知りました。使いかたは`useEffect`と同じで、`useLayoutEffect(setup, dependencies?)`のように使います。違いは、DOMの変更が反映されたあと、ブラウザが画面を描画する前に同期的に実行される点みたいです。StrictModeで行われる初期化→直後に強制クリーンアップ→再初期化の処理で、`canvas.getContext("webgl2")`のようなネイティブリソースの確保を伴う処理は壊れやすいみたいです。useElementでcanvasがstateとして確実にセットされた後でuseEffectが実行されることで、半端なタイミングでの初期化と破棄を防ぐことができ、解決した、、、という理解です。正直StrictModeの挙動をしっかり理解していないのでちょっと曖昧ですが、まあ直ったのでスルーしちゃいました。
https://ja.react.dev/reference/react/useLayoutEffect

#### シェーダのホットスワップ
ライブコーディングパフォーマンスを想定していたので、シェーダを書き換えてホットキーを押すことで背景の実行結果が即座に置き換わる仕様にしています。大事なのはコンパイルの通らないシェーダを書いた際に映像を途切れさせないことです。