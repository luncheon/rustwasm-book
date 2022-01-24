# Setup

This section describes how to set up the toolchain for compiling Rust programs
to WebAssembly and integrate them into JavaScript.

> このセクションは Rust プログラムを WebAssembly にコンパイルし、 JavaScript と組み合わせる方法を記述します。

## The Rust Toolchain

You will need the standard Rust toolchain, including `rustup`, `rustc`, and
`cargo`.

[Follow these instructions to install the Rust toolchain.][rust-install]

The Rust and WebAssembly experience is riding the Rust release trains to stable!
That means we don't require any experimental feature flags. However, we do
require Rust 1.30 or newer.

> `rustup`、`rustc`、`cargo` を含む標準の Rust ツールチェインが必要になります。
>
> [Rust ツールチェインをインストールするためにこれらの手順に従ってください。][rust-install]
>
> Rust and WebAssembly の体験は stable のリリース列車に乗っています！それは実験的機能は要求しないということです。しかし、Rust 1.30またはそれ以上の新しいバージョンが要求されます。

## `wasm-pack`

`wasm-pack` is your one-stop shop for building, testing, and publishing
Rust-generated WebAssembly.

[Get `wasm-pack` here!][wasm-pack-install]

> `wasm-pack` は Rust により生成された WebAssembly のビルド、テスト、パブリッシュのためのワンストップショップ (訳注: そこだけで全ての必要な買い物ができるような場所のこと) です。
>
> [ここで `wasm-pack` をゲットしましょう！][wasm-pack-install]

## `cargo-generate`

[`cargo-generate` helps you get up and running quickly with a new Rust project
by leveraging a pre-existing git repository as a template.][cargo-generate]

Install `cargo-generate` with this command:

> [`cargo-generate` は既に存在しているGitリポジトリをテンプレートとして利用して新しい Rust プロジェクトを迅速に立ち上げ、実行に移すことを助けます。][cargo-generate]
>
> `cargo-generate` をこのコマンドでインストールしてください:

```
cargo install cargo-generate
```

## `npm`

`npm` is a package manager for JavaScript. We will use it to install and run a
JavaScript bundler and development server. At the end of the tutorial, we will
publish our compiled `.wasm` to the `npm` registry.

[Follow these instructions to install `npm`.][npm-install]

If you already have `npm` installed, make sure it is up to date with this
command:

> `npm` は JavaScript のパッケージマネージャです。 JavaScript バンドラと開発サーバをインストールし、実行するために使用します。チュートリアルの最後に、コンパイルした `.wasm` を `npm` レジストリにパブリッシュします。
>
> [これらの手順に従って `npm` をインストールしてください。][npm-install]
>
> 既に `npm` をインストールしている場合、このコマンドで最新版であるか確認してください:

```
npm install npm@latest -g
```

[rust-install]: https://www.rust-lang.org/tools/install
[npm-install]: https://www.npmjs.com/get-npm
[wasm-pack]: https://github.com/rustwasm/wasm-pack
[cargo-generate]: https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]: https://rustwasm.github.io/wasm-pack/installer/
