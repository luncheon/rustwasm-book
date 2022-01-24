# Hello, World!

This section will show you how to build and run your first Rust and WebAssembly
program: a Web page that alerts "Hello, World!"

Make sure you have followed the [setup instructions](setup.html) before beginning.

> この節では初めての Rust and WebAssembly プログラム——"Hello, World!"とアラートするウェブページ——をどうビルドし実行するかを説明します。
>
> 始める前に必ず[セットアップ](setup.html)の指示に従ってください。

## Clone the Project Template

The project template comes pre-configured with sane defaults, so you can quickly
build, integrate, and package your code for the Web.

Clone the project template with this command:

> プロジェクトテンプレートが真っ当な既定の状態に予め設定されるので、ウェブのためのコードを素早くビルドし、統合し、パッケージ化できるようになります。
>
> このコマンドでプロジェクトテンプレートをクローンします:

```text
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

This should prompt you for the new project's name. We will use
**"wasm-game-of-life"**.

> このコマンドを入力すると新しいプロジェクトの名前を入力するよう促されるはずです。ここでは **"wasm-game-of-life"** とします。

```text
wasm-game-of-life
```

## What's Inside

Enter the new `wasm-game-of-life` project

> wasm-game-of-lifeプロジェクトに入ってください

```
cd wasm-game-of-life
```

and let's take a look at its contents:

> そしてその中身を見ましょう:

```text
wasm-game-of-life/
├── Cargo.toml
├── LICENSE_APACHE
├── LICENSE_MIT
├── README.md
└── src
    ├── lib.rs
    └── utils.rs
```

Let's take a look at a couple of these files in detail.

> これらのファイルから二つを詳細に見ましょう。

### `wasm-game-of-life/Cargo.toml`

The `Cargo.toml` file specifies dependencies and metadata for `cargo`, Rust's
package manager and build tool. This one comes pre-configured with a
`wasm-bindgen` dependency, a few optional dependencies we will dig into later,
and the `crate-type` properly initialized for generating `.wasm` libraries.

> `Cargo.toml` ファイルは依存と `cargo`——Rust のパッケージマネージャでありビルドツールである——のメタデータを指定します。このファイルは `wasm-bindgen` への依存とあとで掘り下げるオプションの依存と `crate-type` が `.wasm` ライブラリを生成するよう正しく初期化された状態に予め設定されています。

### `wasm-game-of-life/src/lib.rs`

The `src/lib.rs` file is the root of the Rust crate that we are compiling to
WebAssembly. It uses `wasm-bindgen` to interface with JavaScript. It imports the
`window.alert` JavaScript function, and exports the `greet` Rust function, which
alerts a greeting message.

> `src/lib.rs` ファイルは WebAssembly にコンパイルする Rust クレートのルートです。このファイルは `wasm-bindgen` を JavaScript とのインターフェイスとして使っています。また、 JavaScript 関数の `window.alert` をインポートし、挨拶メッセージをアラートする Rust 関数 `greet` をエクスポートしています。

```rust
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, wasm-game-of-life!");
}
```

### `wasm-game-of-life/src/utils.rs`

The `src/utils.rs` module provides common utilities to make working with Rust
compiled to WebAssembly easier. We will take a look at some of these utilities
in more detail later in the tutorial, such as when we look at [debugging our wasm
code](debugging.html), but we can ignore this file for now.

> `src/utils.rs` モジュールは Rust を使って作業することを簡単にする一般的なユーティリティを提供しています。あとでこれらのユーティリティの詳細をチュートリアル内で——例えば [wasmコードのデバッグ](debugging.html) について考えるときに——見ることになりますが、今はこのファイルを無視することができます。

## Build the Project

We use `wasm-pack` to orchestrate the following build steps:

* Ensure that we have Rust 1.30 or newer and the `wasm32-unknown-unknown`
  target installed via `rustup`,
* Compile our Rust sources into a WebAssembly `.wasm` binary via `cargo`,
* Use `wasm-bindgen` to generate the JavaScript API for using our Rust-generated
  WebAssembly.

To do all of that, run this command inside the project directory:

> `wasm-pack` を使って次のビルド手順を指揮します:
> - Rust 1.30 以上 と `wasm32-unknown-unknown` ターゲットを `rustup` を通してインストールしていることを確認します。
> - `cargo` を使用して Rust のソースを WebAssembly の `.wasm` バイナリにコンパイルします。
> - `wasm-bindgen` を使って Rust で生成した WebAssembly を使用するための JavaScript API を生成します。
>
> それらの全てを行うために、プロジェクトディレクトリでこのコマンドを実行します:

```
wasm-pack build
```

When the build has completed, we can find its artifacts in the `pkg` directory,
and it should have these contents:

> ビルドが完了したとき、その生成物はpkgディレクトリで見つけることができ、中身は次のようになっているはずです:

```
pkg/
├── package.json
├── README.md
├── wasm_game_of_life_bg.wasm
├── wasm_game_of_life.d.ts
└── wasm_game_of_life.js
```

The `README.md` file is copied from the main project, but the others are
completely new.

> `README.md` ファイルはメインプロジェクトからコピーされていますが、他のファイルはまったく新しいものです。

> （訳注） wasm-pack 0.10.2 で実行してみましたが、上記より多くのファイルが生成されました。
>
> pkg/  
> <span style="background:#ABF2BC">├── .gitignore</span>  
> ├── README.md  
> ├── package.json  
> ├── wasm_game_of_life.d.ts  
> ├── wasm_game_of_life.js  
> <span style="background:#ABF2BC">├── wasm_game_of_life_bg.js</span>  
> ├── wasm_game_of_life_bg.wasm  
> <span style="background:#ABF2BC">└── wasm_game_of_life_bg.wasm.d.ts</span>

### `wasm-game-of-life/pkg/wasm_game_of_life_bg.wasm`

The `.wasm` file is the WebAssembly binary that is generated by the Rust
compiler from our Rust sources. It contains the compiled-to-wasm versions of all
of our Rust functions and data. For example, it has an exported "greet"
function.

> `.wasm` ファイルは Rust のソースから Rust コンパイラによって生成された WebAssembly のバイナリです。 Rust の関数とデータ全ての wasm にコンパイルされたバージョンを含んでいます。例えば、エクスポートされた "greet" 関数を含んでいます。

> ここまで [https://moshg.github.io/rustwasm-book-ja/game-of-life/hello-world.html](https://moshg.github.io/rustwasm-book-ja/game-of-life/hello-world.html)  
> ここから DeepL ベースの私訳

### `wasm-game-of-life/pkg/wasm_game_of_life.js`

The `.js` file is generated by `wasm-bindgen` and contains JavaScript glue for
importing DOM and JavaScript functions into Rust and exposing a nice API to the
WebAssembly functions to JavaScript. For example, there is a JavaScript `greet`
function that wraps the `greet` function exported from the WebAssembly
module. Right now, this glue isn't doing much, but when we start passing more
interesting values back and forth between wasm and JavaScript, it will help
shepherd those values across the boundary.

> `.js` ファイルは `wasm-bindgen` によって生成され、 JavaScript グルーを含みます。グルーは DOM と JavaScript 関数を Rust にインポートし、 WebAssembly 関数の素敵な API を JavaScript に公開します。例えば、 WebAssembly モジュールからエクスポートされた `greet` 関数をラップした JavaScript の `greet` 関数があります。今のところ、このグルーがすることは多くありませんが、 wasm と JavaScript との間でより興味深い値をやり取りするようになると、境界を越えてそれらの値を引き回すのに役立つでしょう。

```js
import * as wasm from './wasm_game_of_life_bg';

// ...

export function greet() {
    return wasm.greet();
}
```

> （訳注） `wasm_game_of_life.js` の中身は次の 2 行だけになっていました。
>
> ```js
> import * as wasm from "./wasm_game_of_life_bg.wasm";
> export * from "./wasm_game_of_life_bg.js";
> ```
>
> （この `import * as wasm from "./wasm_game_of_life_bg.wasm";` は意味がありません。なにがしたいんでしょうか。）
>
> 上で `wasm_game_of_life.js` の内容として説明されているコードは `wasm_game_of_life_bg.js` へ移動されていました。  

### `wasm-game-of-life/pkg/wasm_game_of_life.d.ts`

The `.d.ts` file contains [TypeScript][] type declarations for the JavaScript
glue. If you are using TypeScript, you can have your calls into WebAssembly
functions type checked, and your IDE can provide autocompletions and
suggestions! If you aren't using TypeScript, you can safely ignore this file.

> `.d.ts` ファイルには、 JavaScript グルーのための [TypeScript][] の型宣言が含まれています。 TypeScript を使用している場合、 WebAssembly の関数を呼び出す際に型チェックが行われ、 IDE が自動補完やサジェストを提供することができます。 TypeScript を使用していない場合は、このファイルを無視しても大丈夫です。

```typescript
export function greet(): void;
```

[TypeScript]: http://www.typescriptlang.org/

### `wasm-game-of-life/pkg/package.json`

[The `package.json` file contains metadata about the generated JavaScript and
WebAssembly package.][package.json] This is used by npm and JavaScript bundlers
to determine dependencies across packages, package names, versions, and a bunch
of other stuff. It helps us integrate with JavaScript tooling and allows us to
publish our package to npm.

> [`package.json` ファイルには、生成された JavaScript と WebAssembly のパッケージに関するメタデータが含まれています。][package.json] これは npm と JavaScript バンドラーが、パッケージ間の依存関係、パッケージ名、バージョン、その他多くのことを定めるために使用されます。これは JavaScript ツールとの統合を支援し、 npm にパッケージを公開可能にします。

```json
{
  "name": "wasm-game-of-life",
  "collaborators": [
    "Your Name <your.email@example.com>"
  ],
  "description": null,
  "version": "0.1.0",
  "license": null,
  "repository": null,
  "files": [
    "wasm_game_of_life_bg.wasm",
    "wasm_game_of_life.d.ts"
  ],
  "main": "wasm_game_of_life.js",
  "types": "wasm_game_of_life.d.ts"
}
```

[package.json]: https://docs.npmjs.com/files/package.json

## Putting it into a Web Page

To take our `wasm-game-of-life` package and use it in a Web page, we use [the
`create-wasm-app` JavaScript project template][create-wasm-app].

[create-wasm-app]: https://github.com/rustwasm/create-wasm-app

Run this command within the `wasm-game-of-life` directory:

> `wasm-game-of-life` のパッケージを Web ページで使うために、ここでは [`create-wasm-app` JavaScript プロジェクトテンプレート][create-wasm-app] を使用します。
>
> `wasm-game-of-life` ディレクトリでこのコマンドを実行してください:

```
npm init wasm-app www
```

Here's what our new `wasm-game-of-life/www` subdirectory contains:

> 新しい `wasm-game-of-life/www` サブディレクトリの中身はこんな感じです。 <small>（DeepL 翻訳まま）</small>

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── README.md
└── webpack.config.js
```

Once again, let's take a closer look at some of these files.

> もう一度、これらのファイルのいくつかを詳しく見てみましょう。

### `wasm-game-of-life/www/package.json`

This `package.json` comes pre-configured with `webpack` and `webpack-dev-server`
dependencies, as well as a dependency on `hello-wasm-pack`, which is a version
of the initial `wasm-pack-template` package that has been published to npm.

> この `package.json` には、 `webpack` と `webpack-dev-server`、および `hello-wasm-pack` への依存関係があらかじめ設定されています。 `hello-wasm-pack` は npm に公開されている `wasm-pack-template` パッケージの初期バージョンです。（訳注: `wasm-pack-template` は `wasm-game-of-life` の生成に使用したプロジェクトテンプレートです; `hello-wasm-pack` パッケージの中身は `wasm-game-of-life/pkg/` と同じだと思っていいです。）

### `wasm-game-of-life/www/webpack.config.js`

This file configures webpack and its local development server. It comes
pre-configured, and you shouldn't have to tweak this at all to get webpack and
its local development server working.

> このファイルは webpack とそのローカル開発サーバーを設定します。このファイルはあらかじめ設定されているので、 webpack とそのローカル開発サーバーを動作させるために、これを微調整する必要は全くありません。

### `wasm-game-of-life/www/index.html`

This is the root HTML file for the Web page. It doesn't do much other than
load `bootstrap.js`, which is a very thin wrapper around `index.js`.

> これは Web ページのルート HTML ファイルです。これは index.js の非常に薄いラッパーである bootstrap.js をロードすること以外には、あまり多くのことを行いません。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Hello wasm-pack!</title>
  </head>
  <body>
    <script src="./bootstrap.js"></script>
  </body>
</html>
```

> （訳注） `bootstrap.js` の中身は以下の通りです。
>
> ```js
> // A dependency graph that contains any wasm must all be imported
> // asynchronously. This `bootstrap.js` file does the single async import, so
> // that no one else needs to worry about it again.
> import("./index.js")
>   .catch(e => console.error("Error importing `index.js`:", e));
> ```

### `wasm-game-of-life/www/index.js`

The `index.js` is the main entry point for our Web page's JavaScript. It imports
the `hello-wasm-pack` npm package, which contains the default
`wasm-pack-template`'s compiled WebAssembly and JavaScript glue, then it calls
`hello-wasm-pack`'s `greet` function.

> `index.js` は、私たちの Web ページの JavaScript のメインエントリポイントです。デフォルトの `wasm-pack-template` のコンパイル済み WebAssembly と JavaScript グルーが含まれる `hello-wasm-pack` npm パッケージをインポートし、 `hello-wasm-pack` の `greet` 関数を呼び出しています。

```js
import * as wasm from "hello-wasm-pack";

wasm.greet();
```

### Install the dependencies

First, ensure that the local development server and its dependencies are
installed by running `npm install` within the `wasm-game-of-life/www`
subdirectory:

> まず、 `wasm-game-of-life/www` サブディレクトリ内で `npm install` を実行して、ローカルの開発サーバーとその依存関係がインストールされていることを確認します。

```text
npm install
```

This command only needs to be run once, and will install the `webpack`
JavaScript bundler and its development server.

> このコマンドは一度だけ実行する必要があります。 `webpack` JavaScript バンドラーとその開発サーバーをインストールします。

> Note that `webpack` is not required for working with Rust and WebAssembly, it
> is just the bundler and development server we've chosen for convenience
> here. Parcel and Rollup should also support importing WebAssembly as
> ECMAScript modules. You can also use Rust and WebAssembly [without a
> bundler][] if you prefer!
>
>> `webpack` は Rust と WebAssembly を扱うのに必須ではなく、ここでは便宜上バンドラーと開発サーバーとして選んだだけであることに注意してください。 Parcel と Rollup も WebAssembly を ECMAScript モジュールとしてインポートすることをサポートしているはずです。 [バンドラーなしで][without a bundler] Rust と WebAssembly を使用することもできます。

[without a bundler]: https://rustwasm.github.io/docs/wasm-bindgen/examples/without-a-bundler.html

### Using our Local `wasm-game-of-life` Package in `www`

Rather than use the `hello-wasm-pack` package from npm, we want to use our local
`wasm-game-of-life` package instead. This will allow us to incrementally develop
our Game of Life program.

Open up `wasm-game-of-life/www/package.json` and next to `"devDependencies"`, add the `"dependencies"` field,
including a `"wasm-game-of-life": "file:../pkg"` entry:

> npm の `hello-wasm-pack` パッケージを使うのではなく、ローカルの `wasm-game-of-life` パッケージを使用したいと思います。これにより、 Game of Life プログラムを段階的に開発することができます。
>
> `wasm-game-of-life/www/package.json` を開いて `"devDependencies"` の次に `"dependencies"` フィールドを追加して `"wasm-game-of-life": "file:../pkg"` を追加してください:

```js
{
  // ...
  "dependencies": {                     // Add this three lines block!
    "wasm-game-of-life": "file:../pkg"
  },
  "devDependencies": {
    //...
  }
}
```

Next, modify `wasm-game-of-life/www/index.js` to import `wasm-game-of-life`
instead of the `hello-wasm-pack` package:

> 次に、 `wasm-of-life/www/index.js` を修正して、 `hello-wasm-pack` パッケージの代わりに `wasm-game-of-life` をインポートするようにします。

```js
import * as wasm from "wasm-game-of-life";

wasm.greet();
```

Since we declared a new dependency, we need to install it:

> 新しい依存関係を宣言したので、それをインストールする必要があります:

```text
npm install
```

Our Web page is now ready to be served locally!

> 私たちの Web ページがローカルで提供できるようになりました！

> （訳注） 今後も `hello-wasm-pack` は不要なので `npm r hello-wasm-pack` すればいいと思います。

## Serving Locally

Next, open a new terminal for the development server. Running the server in a
new terminal lets us leave it running in the background, and doesn't block us
from running other commands in the meantime. In the new terminal, run this
command from within the `wasm-game-of-life/www` directory:

> 次に、開発サーバー用の新しいターミナルを開きます。新しいターミナルでサーバーを実行することで、バックグラウンドでサーバーを実行したままにでき、その間に他のコマンドを実行できます。新しいターミナルで、 `wasm-game-of-life/www` ディレクトリの中から、次のコマンドを実行します:

```
npm run start
```

Navigate your Web browser to [http://localhost:8080/](http://localhost:8080/)
and you should be greeted with an alert message:

> Web ブラウザで [http://localhost:8080/](http://localhost:8080/) に移動すると、アラートメッセージが表示されるはずです。

[![Screenshot of the "Hello, wasm-game-of-life!" Web page alert](../images/game-of-life/hello-world.png)](../images/game-of-life/hello-world.png)

Anytime you make changes and want them reflected on
[http://localhost:8080/](http://localhost:8080/), just re-run the `wasm-pack
build` command within the `wasm-game-of-life` directory.

> 変更を加えて [http://localhost:8080/](http://localhost:8080/) に反映させたい場合は、 `wasm-game-of-life` ディレクトリで `wasm-pack build` コマンドを再実行すればよいのです。

> （訳注） `wasm-pack watch` は現状ないみたいです。 Issue にはワークアラウンドも。
>
> - watch command · Issue #457 · rustwasm/wasm-pack  
>   [https://github.com/rustwasm/wasm-pack/issues/457](https://github.com/rustwasm/wasm-pack/issues/457)

## Exercises

* Modify the `greet` function in `wasm-game-of-life/src/lib.rs` to take a `name:
  &str` parameter that customizes the alerted message, and pass your name to the
  `greet` function from inside `wasm-game-of-life/www/index.js`. Rebuild the
  `.wasm` binary with `wasm-pack build`, then refresh
  [http://localhost:8080/](http://localhost:8080/) in your Web browser and you
  should see a customized greeting!

> * `wasm-game-of-life/src/lib.rs` の `greet` 関数に `name: &str` パラメータを取り、警告メッセージをカスタマイズするように修正し、 `wasm-game-of-life/www/index.js` 内から `greet` 関数にあなたの名前を渡します。 `.wasm` バイナリを `wasm-pack build` で再構築し、 Web ブラウザで [http://localhost:8080/](http://localhost:8080/) を更新すると、カスタマイズされた挨拶文が表示されるはずです!

  <details>
    <summary>Answer</summary>

    New version of the `greet` function in `wasm-game-of-life/src/lib.rs`:

    ```rust
    #[wasm_bindgen]
    pub fn greet(name: &str) {
        alert(&format!("Hello, {}!", name));
    }
    ```

    New invocation of `greet` in `wasm-game-of-life/www/index.js`:

    ```js
    wasm.greet("Your Name");
    ```

  </details>
