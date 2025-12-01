---
title: "Webシェーダエディタを自作して、初VJに挑戦してみた"
emoji: "🌈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "WebGL", "React"]
published: false
---
[MYJLab Advent Calendar 2025](https://qiita.com/advent-calendar/2025/myjlab)の2日目を担当します、Sigreniと申します。せっかくなので勝手ながら[グラフィックス全般 Advent Calendar 2025](https://qiita.com/advent-calendar/2025/graphics)へもマルチポストします。よろしくお願いします🤲

軽く自己紹介すると、技術的にはWebフロントエンドを2年ちょっと齧っている修士一年生です。研究室とはあまり関係ありませんが、ここ一年程のマイブームがCGです。そんな中、先日はじめてVJをやる機会があり、シェーダを使ったライブコーディングに挑戦しました。その際にWebシェーダエディタを作成したので、それについてお話しできればと思います。

当日の様子はこんな感じで、なんとか無事にやり遂げられました。
https://x.com/nimono_vr/status/1969408342096560455?s=20

（作成したものは以下になります↓ ぜひいじってみてください〜）
https://somahc.github.io/glsl-livecoding-editor-web/
## 前提知識や背景
冒頭でいきなり横文字をたくさん出してしまいましたが、言葉に馴染みのない方もいると思うので簡単に補足します。
### 「VJ」「シェーダ」とは?
まず、VJとはVisual JockeyやVideo Jockeyの略です。クラブなどで音楽を選択して流すDJはご存じの方が多いと思いますが、その映像版になります。VJは音楽に合った映像を流すことで場を視覚的に盛り上げます。
https://x.com/tktk_1/status/1977315624549487007?s=20

シェーダ(shader)はディスプレイ上にオブジェクトが表示される際、その表示のされ方を決めるプログラムになります。現実を模したリアルな質感にしたり、アニメ調にデフォルメした質感を出したりなど、様々なことができます。Unityなどでゲーム制作の経験がある方は聞いたことがあるかもしれません。シェーダ言語として有名なものにGLSL(OpenGL Shading Language)があります。

基本的にシェーダ単体で使うことはないのですが、レイマーチングなどのテクニックを使うことで、GLSLを書くだけで面白い絵を出すことができます（詳細は割愛します）。俗に”シェーダ芸”と呼ばれたりもします。
https://youtu.be/NX5vcd-ZY9s?si=q0PTH2bcHSzIfBUT&t=1160

今回はシェーダをその場で実際に書くことで映像を変化させる、ライブコーディングスタイルでパフォーマンスしてみました。

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

### UIについて
アプリのUIは、Bonzomaticを意識しつつ、あえてレトロな感じを目指しました。フューチャーレトロっていうんでしょうか。ダサかっこいい的な。映画「ALIEN」のような雰囲気です。
https://pin.it/1tsSH0Cug
https://pin.it/5zbbhotvx

UIは文字や背景が若干光っているような見た目をCSSのtext-shadowやbox-shadowで実現し、画面に映っている感を演出しています。
![](https://storage.googleapis.com/zenn-user-upload/f6896740e28e-20251201.png)

```css:ErrorPanel/index.module.css
.errorMessage {
  font-family: "Sometype Mono", monospace;
  color: white;
  padding: 0 5px;
  text-shadow: 0 0 2.5px var(--color-error), 0 0 5px var(--color-error),
    0 0 10px var(--color-error);

  width: fit-content;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.5;
}
...
```

BEATの部分のアニメーションの仕組みはちょうど一年前の今日にアドベントカレンダーで紹介したCSSアニメーションの内容とほぼ同じです！beat情報をjotaiで管理し、インラインで`translateX()`を指定することでBEATをアニメーションさせています。

https://zenn.dev/somahc/articles/c0b81dfb270e2a

https://gyazo.com/6065168aed751f8e598699e96815ac62
```ts:StatsPanel.tsx
...
const beatMeter = () => {
    const elapsedTime = currentElapsedTime * (shaderBPM / 60);
    const bar = Math.floor(elapsedTime);
    const beat = elapsedTime - bar;
    return 100 - beat * 100;
};

...

<div
    className={style.beatMeterBar}
    style={{ transform: `translateX(-${beatMeter()}%)` }}
></div>
...
```

また、エディタ上部にクソデカ時計があるのは、本番の持ち時間が15~20分と決まっていたからです。パフォーマンスを始めてからのどれくらい時間が経ったか知れると便利だったので、アプリに実装してしまいました。（みんなどうやって時間管理しているんだろう、、？）
日時のほか、シェーダ経過時間はjotaiで管理していたため、そのまま使ってあげています。

![アプリ上部の時計表示](https://storage.googleapis.com/zenn-user-upload/657b62aec410-20251201.png)

昔の監視カメラや動画プレイヤーっぽいUIを意識しています。好みが分かれるデザインだと思いますが、私は割と気に入っています。
https://jp.pinterest.com/pin/898749669399686602/

#### エディタのスタイル
CodeMirrorが提供するコードエディタは柔軟にカスタム可能で、好きなホットキー処理の登録やシンタックスハイライトのスタイルなどを自分で定義できます。

https://codemirror.net/docs/guide/#extension

私はreact-codemirrorという、Reactで扱いやすいCodeMirrorのコンポーネントを提供してくれるライブラリを利用しました。このライブラリを使用する場合は、ReactCodeMirrorコンポーネントのextensionsというpropsにカスタム項目を渡します。

https://github.com/uiwjs/react-codemirror

ここにシンタックスハイライトのスタイルや、ホットキーが押された時の処理などを渡してあげることで、自分だけのエディタが作成できます。最終的には以下のような形になりました。
```tsx:Editor.tsx
<ReactCodeMirror
  value={value}
  onChange={onChange}
  extensions={[
    saveKeymap, // Cmd + Sでコードのコンパイルを走らせる
    glsl(), // glslのシンタックスハイライト(npmパッケージとして入手)
    textBackgroundExtension, // コード文字の背景だけ黒半透明にする
    myTheme, // フォントの指定やカーソルのスタイルなど、エディタ全体のスタイル
    syntaxHighlighting(myHighlightStyle), // コメントは緑、数字は赤など、シンタックスハイライトを一部上書き
  ]}
/>
```
これらのextensionsについてはChatGPTに指示を与えて書いてもらい、気になるところをちょいちょい自分で修正することで実装しました。本来CodeMirrorのドキュメントを読んで実装する箇所ではありますが、その手間が省けました。

特に、文字の背景”のみ”黒半透明にすることにこだわりました。コードエディタの背景全体を黒半透明とかだと簡単だと思うんですが、いかんせん背景でシェーダ実行結果の映像が流れているので、それをできるだけ邪魔したくなかったんですよね。
![文字の背景だけ黒半透明にしている様子](https://storage.googleapis.com/zenn-user-upload/b40cbee98263-20251201.png)

ChatGPTにお願いして、「もっとこうして！！」とリテイクを繰り返し、今の形に落ち着きました。実装的には以下のような感じです。ChatGPTに全てやってもらってしまったのですが、正規表現を駆使しつつ背景を表示する箇所を決めている感じでしょうか。
```ts:textBackgroundExtension.ts
// textBackgroundExtension.ts
import { RangeSetBuilder, type Extension } from "@codemirror/state";
import {
  EditorView,
  Decoration,
  ViewPlugin,
  ViewUpdate,
} from "@codemirror/view";

const textBgMark = Decoration.mark({ class: "cm-textBg" });

const TextBackground = ViewPlugin.fromClass(
  class {
    decorations;
    constructor(view: EditorView) {
      this.decorations = this.build(view);
    }
    update(update: ViewUpdate) {
      if (update.docChanged || update.viewportChanged) {
        this.decorations = this.build(update.view);
      }
    }
    private build(view: EditorView) {
      const builder = new RangeSetBuilder<Decoration>();
      for (const { from, to } of view.visibleRanges) {
        let pos = from;
        while (pos <= to) {
          const line = view.state.doc.lineAt(pos);
          const s = line.text;

          // 行に非空白が1つでもあれば、その行頭→行末まで塗る
          if (/\S/.test(s)) {
            builder.add(line.from, line.to, textBgMark);
          }
          pos = line.to + 1;
        }
      }
      return builder.finish();
    }
  },
  { decorations: (v) => v.decorations }
);

export const textBackgroundExtension: Extension = [
  TextBackground,
  EditorView.baseTheme({
    ".cm-textBg": {
      backgroundColor: "rgba(0, 0, 0, 0.5)",
      boxDecorationBreak: "clone",
      WebkitBoxDecorationBreak: "clone",
      borderRadius: "0",
    },
  }),
];
```

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

https://developer.mozilla.org/ja/docs/Web/API/HTMLCanvasElement/webglcontextlost_event

0b5vr氏制作の[wavenerd](https://0b5vr.com/wavenerd/)の実装で使われていたuseElementというhooksをほぼそのまま使ってcanvasを取得し、WebGLコンテキスト取得を行うuseEffectの依存配列に含めることで解決しました。
https://github.com/0b5vr/wavenerd/blob/dev/src/view/utils/useElement.ts

`useLayoutEffect`はここで初めて知りました。使いかたは`useEffect`と同じで、`useLayoutEffect(setup, dependencies?)`のように使います。違いは、DOMの変更が反映されたあと、ブラウザが画面を描画する前に同期的に実行される点みたいです。StrictModeで行われる初期化→直後に強制クリーンアップ→再初期化の処理で、`canvas.getContext("webgl2")`のようなネイティブリソースの確保を伴う処理は壊れやすいみたいです。useElementでcanvasがstateとして確実にセットされた後でuseEffectが実行されることで、半端なタイミングでの初期化と破棄を防ぐことができ、解決した、、、という理解です。正直StrictModeの挙動をしっかり理解していないのでちょっと曖昧ですが、まあ直ったのでスルーしちゃいました。
https://ja.react.dev/reference/react/useLayoutEffect

#### シェーダのホットスワップ
ライブコーディングパフォーマンスを想定していたので、シェーダを書き換えてホットキーを押すことで背景の実行結果が即座に置き換わる仕様にしています。ここで大事なのは、コンパイルの通らないシェーダを書いた際に映像を途切れさせないことです。実装としてはシェーダコードを渡すとコンパイルしてシェーダオブジェクトを作ってくれる`glCreateShader()`を作っておきます。コンパイル失敗時はエラーメッセージをErrorとしてthrowし、呼び出し元でcatchするようにして、シェーダのスワップが実行されないようにします。catch節ではjotaiで管理している`compileErrorMessageAtom`にコンパイルエラー文をセットすることで、アプリUIのエラー部にエラー内容を表示させています。

```ts:glCreateShader.ts
export function glCreateShader(
  gl: WebGLRenderingContext,
  type: "vertex" | "fragment",
  source: string
): WebGLShader | null {
  const shader = gl.createShader(
    type === "vertex" ? gl.VERTEX_SHADER : gl.FRAGMENT_SHADER
  )!;
  gl.shaderSource(shader, source);
  gl.compileShader(shader);

  if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
    const rawLog = gl.getShaderInfoLog(shader) ?? "";

    // 特殊文字を削除
    const cleanedLog = (() => {
      let end = rawLog.length;
      while (end > 0) {
        const codePoint = rawLog.charCodeAt(end - 1);
        const isAsciiControl = codePoint === 0x7f || codePoint <= 0x1f; // DEL or C0 controls
        if (isAsciiControl) {
          end -= 1;
          continue;
        }
        break;
      }
      return rawLog.slice(0, end).trim();
    })();

    console.error(cleanedLog || undefined);
    throw new Error(cleanedLog || undefined);
  }

  return shader;
}
```

```ts:ShaderCanvas.tsx
...
const setCompileErrorMessage = useSetAtom(compileErrorMessageAtom);
...

// シェーダーコンパイルエラー表示、ホットスワップ
  useEffect(() => {
    if (fsSource == null) return;
    const gl = glRef.current;
    const vsObj = glVertexShaderRef.current;
    if (!gl || !vsObj) return;

    try {
      const fsObj = glCreateShader(gl, "fragment", fsSource);
      if (fsObj == null) return;

      const newProg = glCreateProgram(gl, vsObj, fsObj);
      gl.useProgram(newProg);
      const newLocResolution = gl.getUniformLocation(newProg, "resolution");
      const newLocTime = gl.getUniformLocation(newProg, "time");
      const newLocBPM = gl.getUniformLocation(newProg, "bpm");
      // スワップ
      if (progRef.current) gl.deleteProgram(progRef.current);
      progRef.current = newProg;
      locResolutionRef.current = newLocResolution;
      locTimeRef.current = newLocTime;
      locBPMRef.current = newLocBPM;
      setCompileErrorMessage("");
    } catch (e) {
      if (e instanceof Error) {
        setCompileErrorMessage(e.message);
      } else if (typeof e === "string") {
        setCompileErrorMessage(e);
      } else {
        setCompileErrorMessage(JSON.stringify(e, null, 2));
      }
    }
  }, [fsSource, setCompileErrorMessage]);
...
```

## おわりに
VJ経験が無く、シェーダも今年から本格的に始めた自分にとって、ライブコーディングVJをするのはなかなか勇気がいりました。
エディタのUIの雰囲気やシェーダで作る絵など正解の無い分野で、人前でパフォーマンスをするという締切だけはあるという状況は本当に心臓に悪かったのですが、結果的にはやってみてすごく良かったと感じています。

VJする機会を与えてくださった方たちやライブラリの作成者、シェーダを始めるきっかけを作ってくれた世界中のShader Wizzardたちに感謝です。ありがとうございました！