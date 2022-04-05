# Adding Interactivity

We will continue to explore the JavaScript and WebAssembly interface by adding
some interactive features to our Game of Life implementation. We will enable
users to toggle whether a cell is alive or dead by clicking on it, and
allow pausing the game, which makes drawing cell patterns a lot easier.

> ライフゲームの実装にいくつかのインタラクティブな機能を追加することで、 JavaScript と WebAssembly のインターフェースを引き続き探求していきます。私たちは、ユーザーによるセルクリックで生死を切り替えられるようにしたり、ゲームを一時停止できるようにしてセルパターンをより簡単に描けるようにする予定です。

## Pausing and Resuming the Game

Let's add a button to toggle whether the game is playing or paused. To
`wasm-game-of-life/www/index.html`, add the button right above the `<canvas>`:

> ゲームの再生／一時停止を切り替えるボタンを追加しましょう。 `wasm-game-of-life/www/index.html` の `<canvas>` のすぐ上にボタンを追加してください。

```html
<button id="play-pause"></button>
```

In the `wasm-game-of-life/www/index.js` JavaScript, we will make the following
changes:

* Keep track of the identifier returned by the latest call to
  `requestAnimationFrame`, so that we can cancel the animation by calling
  `cancelAnimationFrame` with that identifier.

* When the play/pause button is clicked, check for whether we have the
  identifier for a queued animation frame. If we do, then the game is currently
  playing, and we want to cancel the animation frame so that `renderLoop` isn't
  called again, effectively pausing the game. If we do not have an identifier
  for a queued animation frame, then we are currently paused, and we would like
  to call `requestAnimationFrame` to resume the game.

Because the JavaScript is driving the Rust and WebAssembly, this is all we need
to do, and we don't need to change the Rust sources.

We introduce the `animationId` variable to keep track of the identifier returned
by `requestAnimationFrame`. When there is no queued animation frame, we set this
variable to `null`.

> JavaScript `wasm-game-of-life/www/index.js` には以下のように変更を入れます。
>
> - 直近の `requestAnimationFrame` の呼び出しで返された識別子を記録しておき、その識別子で `cancelAnimationFrame` を呼び出すことでアニメーションをキャンセルできるようにします。
> - 再生／一時停止ボタンがクリックされたら、待機中のアニメーションフレーム識別子があるかどうかをチェックします。もしあればゲームは現在再生中であり、アニメーションフレームをキャンセルして `renderLoop` が再び呼び出されないようにすることで、実質的にゲームを一時停止させます。待機中のアニメーションフレーム識別子がない場合は一時停止中であり、 `requestAnimationFrame` を呼び出してゲームを再開します。
>
> JavaScript が Rust と WebAssembly を動かして（drive）いるので、必要なのはこれだけで、 Rust のソースを変更する必要はありません。
>
> `requestAnimationFrame` が返す識別子を追跡するために `animationId` 変数を導入します。待機中のアニメーションフレームがない場合、この変数に `null` を設定します。

```js
let animationId = null;

// This function is the same as before, except the
// result of `requestAnimationFrame` is assigned to
// `animationId`.
const renderLoop = () => {
  drawGrid();
  drawCells();

  universe.tick();

  animationId = requestAnimationFrame(renderLoop);
};
```

At any instant in time, we can tell whether the game is paused or not by
inspecting the value of `animationId`:

> 任意の瞬間において、ゲームが一時停止しているかどうかは `animationId` の値から判断できます。

```js
const isPaused = () => {
  return animationId === null;
};
```

Now, when the play/pause button is clicked, we check whether the game is
currently paused or playing, and resume the `renderLoop` animation or cancel the
next animation frame respectively. Additionally, we update the button's text
icon to reflect the action that the button will take when clicked next.

> これで、再生／一時停止ボタンがクリックされると、現在ゲームが一時停止中か再生中かをチェックし、それぞれ `renderLoop` アニメーションを再開するか、次のアニメーションフレームをキャンセルするようになりました。さらに、ボタンのテキストアイコンを更新し、次にクリックされたときにボタンが取るアクションを反映します。

```js
const playPauseButton = document.getElementById("play-pause");

const play = () => {
  playPauseButton.textContent = "⏸";
  renderLoop();
};

const pause = () => {
  playPauseButton.textContent = "▶";
  cancelAnimationFrame(animationId);
  animationId = null;
};

playPauseButton.addEventListener("click", event => {
  if (isPaused()) {
    play();
  } else {
    pause();
  }
});
```

Finally, we were previously kick-starting the game and its animation by calling
`requestAnimationFrame(renderLoop)` directly, but we want to replace that with a
call to `play` so that the button gets the correct initial text icon.

> 最後に、以前は `requestAnimationFrame(renderLoop)` を直接呼び出してゲームとそのアニメーションを開始していましたが、これを `play` の呼び出しに置き換え、ボタンが正しい初期テキストアイコンを取得できるようにします。

```diff
// This used to be `requestAnimationFrame(renderLoop)`.
play();
```

Refresh [http://localhost:8080/](http://localhost:8080/) and we should now be
able to pause and resume the game by clicking on the button!

> [http://localhost:8080/](http://localhost:8080/) をリフレッシュすれば、ボタンをクリックすることでゲームの一時停止と再開ができるようになったはずです。

## Toggling a Cell's State on `"click"` Events

Now that we can pause the game, it's time to add the ability to mutate the cells
by clicking on them.

To toggle a cell is to flip its state from alive to dead or from dead to
alive. Add a `toggle` method to `Cell` in `wasm-game-of-life/src/lib.rs`:

> ゲームを一時停止できるようになったので、次はセルをクリックして変更する機能を追加します。
>
> セルをトグルするというのは、セルの状態を生から死、死から生に反転させることです。 `wasm-game-of-life/src/lib.rs` にある `Cell` に `toggle` メソッドを追加します。

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

To toggle the state of a cell at given row and column, we translate the row and
column pair into an index into the cells vector and call the toggle method on
the cell at that index:

> ある行と列のセルの状態を切り替えるには、行と列のペアをセルベクトルのインデックスに変換し、そのインデックスのセルに対して `toggle` メソッドを呼び出します。

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

This method is defined within the `impl` block that is annotated with
`#[wasm_bindgen]` so that it can be called by JavaScript.

In `wasm-game-of-life/www/index.js`, we listen to click events on the `<canvas>`
element, translate the click event's page-relative coordinates into
canvas-relative coordinates, and then into a row and column, invoke the
`toggle_cell` method, and finally redraw the scene.

> このメソッドは、 JavaScript から呼び出せるように `#[wasm_bindgen]` のアノテーションをつけた `impl` ブロックの中で定義されています。
>
> `wasm-game-of-life/www/index.js` では、 `<canvas>` 要素のクリックイベントを listen し、クリックイベントのページ相対座標をキャンバス相対座標に変換し、さらに行と列に変換して、 `toggle_cell` メソッドを呼び出し、最後にシーンを再描画しています。

```js
canvas.addEventListener("click", event => {
  const boundingRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / boundingRect.width;
  const scaleY = canvas.height / boundingRect.height;

  const canvasLeft = (event.clientX - boundingRect.left) * scaleX;
  const canvasTop = (event.clientY - boundingRect.top) * scaleY;

  const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1);
  const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1);

  universe.toggle_cell(row, col);

  drawGrid();
  drawCells();
});
```

Rebuild with `wasm-pack build` in `wasm-game-of-life`, then refresh
[http://localhost:8080/](http://localhost:8080/) again and we can now draw our
own patterns by clicking on the cells and toggling their state.

> `wasm-game-of-life` 内で `wasm-pack build` によってリビルドし、再度 [http://localhost:8080/](http://localhost:8080/) をリフレッシュすると、セルクリックで状態を切り替えることにより好きなパターンを描画できるようになります。

## Exercises

* Introduce an [`<input type="range">`][input-range] widget to control how many
  ticks occur per animation frame.

* Add a button that resets the universe to a random initial state when
  clicked. Another button that resets the universe to all dead cells.

* On `Ctrl + Click`, insert a
  [glider](https://en.wikipedia.org/wiki/Glider_(Conway%27s_Life)) centered on
  the target cell. On `Shift + Click`, insert a pulsar.

[input-range]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range

> - `<input type="range">` ウィジェットを導入し、アニメーションのフレームごとに発生する tick の数を制御しましょう。
> - クリックするとユニバースをランダムな初期状態にリセットするボタンを追加しましょう。さらに、ユニバースをすべて死んだセルにリセットするボタンを追加しましょう。
> - `Ctrl + クリック` でターゲットセルを中心とするグライダーを挿入するようにしましょう。 `Shift + クリック` でパルサーを挿入するようにしましょう。
