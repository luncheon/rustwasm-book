# Testing Conway's Game of Life

Now that we have our Rust implementation of the Game of Life rendering in the 
browser with JavaScript, let's talk about testing our Rust-generated 
WebAssembly functions.

We are going to test our `tick` function to make sure that it gives us the 
output that we expect.

Next, we'll want to create some setter and getter 
functions inside our existing `impl Universe` block in the
`wasm_game_of_life/src/lib.rs` file. We are going to create a `set_width`
and a `set_height` function so we can create `Universe`s of different sizes.

> Rust で実装したライフゲームが JavaScript でブラウザーにレンダリングされるようになりました。では、 Rust で生成された WebAssembly 関数のテストについて説明します。
>
> まず、 `tick` 関数をテストして、期待通りの出力が得られるかどうかを確認します。
>
> 次に、 `wasm_game_of_life/src/lib.rs` ファイルにある `impl Universe` ブロックの中に setter と getter を作成したいと思います。 `set_width` と `set_height` 関数を作成し、異なるサイズの `Universe` を作成できるようにします。

```rust
#[wasm_bindgen]
impl Universe { 
    // ...

    /// Set the width of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_width(&mut self, width: u32) {
        self.width = width;
        self.cells = (0..width * self.height).map(|_i| Cell::Dead).collect();
    }

    /// Set the height of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_height(&mut self, height: u32) {
        self.height = height;
        self.cells = (0..self.width * height).map(|_i| Cell::Dead).collect();
    }

}
```

We are going to create another `impl Universe` block inside our
`wasm_game_of_life/src/lib.rs` file without the `#[wasm_bindgen]` attribute.
There are a few functions we need for testing that we don't want to expose to
our JavaScript. Rust-generated WebAssembly functions cannot return
borrowed references. Try compiling the Rust-generated WebAssembly with the
attribute and take a look at the errors you get.

We are going to write the implementation of `get_cells` to get the contents of
the `cells` of a `Universe`. We'll also write a `set_cells` function so we can
set `cells` in a specific row and column of a `Universe` to be `Alive.`

> `wasm_game_of_life/src/lib.rs` ファイル内に、 `#[wasm_bindgen]` 属性なしで別の `impl Universe` ブロックを作成します。テストに必要な関数の中には、 JavaScript に公開したくないものがいくつかあります。 Rust で生成された WebAssembly の関数は、借用した参照を返すことができません。 Rust で生成された WebAssembly を `#[wasm_bindgen]` 属性付きでコンパイルして、エラーを見てみましょう。
>
> `Universe` の `cells` の中身を取得するための `get_cells` の実装を書きます。また、 `Universe` の特定の行と列にある `cells` を `Alive` に設定できるように `set_cells` 関数を記述します。

```rust
impl Universe {
    /// Get the dead and alive values of the entire universe.
    pub fn get_cells(&self) -> &[Cell] {
        &self.cells
    }

    /// Set cells to be alive in a universe by passing the row and column
    /// of each cell as an array.
    pub fn set_cells(&mut self, cells: &[(u32, u32)]) {
        for (row, col) in cells.iter().cloned() {
            let idx = self.get_index(row, col);
            self.cells[idx] = Cell::Alive;
        }
    }

}
```

Now we're going to create our test in the `wasm_game_of_life/tests/web.rs` file.

Before we do that, there is already one working test in the file. You can
confirm that the Rust-generated WebAssembly test is working by running
`wasm-pack test --chrome --headless` in the `wasm-game-of-life` directory.
You can also use the `--firefox`, `--safari`, and `--node` options to
test your code in those browsers.

> では、 `wasm_game_of_life/tests/web.rs` ファイルの中にテストを作成します。
>
> その前に、このファイルにはすでに 1 つ動作するテストがあります。 `wasm-game-of-life` ディレクトリで `wasm-pack test --chrome --headless` を実行すると、 Rust が生成した WebAssembly のテストが動作していることを確認できます。また、 `--firefox`, `--safari`, `--node` オプションを使用すると、それらのブラウザーでコードをテストできます。

> （訳注） ブラウザーでテストする場合はテストコードに `wasm_bindgen_test_configure!(run_in_browser);` を含める必要があるようです。 Node.js でテストする場合はテストコードに `wasm_bindgen_test_configure!(run_in_browser);` を含めてはいけないようです。（コマンドラインオプションだけで切り替えられるようにしてほしいところです。）

In the `wasm_game_of_life/tests/web.rs` file, we need to export our
`wasm_game_of_life` crate and the `Universe` type.

> `wasm_game_of_life/tests/web.rs` ファイルでは、 `wasm_game_of_life` クレートと `Universe` 型をエクスポートする必要があります。

> （訳注） Rust 2018 以降、 `extern crate` は不要です。  
> [https://doc.rust-jp.rs/edition-guide/rust-2018/path-changes.html#%E3%81%95%E3%82%88%E3%81%86%E3%81%AA%E3%82%89extern-crate](https://doc.rust-jp.rs/edition-guide/rust-2018/path-changes.html#%E3%81%95%E3%82%88%E3%81%86%E3%81%AA%E3%82%89extern-crate)

```rust
extern crate wasm_game_of_life;
use wasm_game_of_life::Universe;
```

In the `wasm_game_of_life/tests/web.rs` file we'll want to create some
spaceship builder functions.

We'll want one for our input spaceship that we'll call the `tick` function on
and we'll want the expected spaceship we will get after one tick. We picked the
cells that we want to initialize as `Alive` to create our spaceship in the
`input_spaceship` function. The position of the spaceship in the
`expected_spaceship` function after the tick of the `input_spaceship` was
calculated manually. You can confirm for yourself that the cells of the input
spaceship after one tick is the same as the expected spaceship.

> `wasm_game_of_life/tests/web.rs` ファイルに、いくつかのスペースシップビルダー関数を作成したいと思います。
>
> `tick` 関数を呼び出す前の入力スペースシップと、その 1 tick 後の期待値スペースシップがほしいところです。 `input_spaceship` 関数で、スペースシップを作るために `Alive` として初期化したいセルを選びました。 `input_spaceship` から 1 tick 後を表す `expected_spaceship` 関数の `Alive` セルは手で計算しました。入力されたスペースシップの 1 tick 後のセルが期待されるスペースシップと同じであることを自分で確認できます。

```rust
#[cfg(test)]
pub fn input_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(1,2), (2,3), (3,1), (3,2), (3,3)]);
    universe
}

#[cfg(test)]
pub fn expected_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(2,1), (2,3), (3,2), (3,3), (4,2)]);
    universe
}
```

Now we will write the implementation for our `test_tick` function. First, we
create an instance of our `input_spaceship()` and our `expected_spaceship()`.
Then, we call `tick` on the `input_universe`. Finally, we use the `assert_eq!`
macro to call `get_cells()` to ensure that `input_universe` and
`expected_universe` have the same `Cell` array values. We add the
`#[wasm_bindgen_test]` attribute to our code block so we can test our
Rust-generated WebAssembly code and use `wasm-pack test` to test the
WebAssembly code.

> それでは、 `test_tick` 関数の実装を書きましょう。まず、 `input_spaceship()` と `expected_spaceship()` のインスタンスを作成します。そして、 `input_universe` に対して `tick` を呼び出します。最後に、 `assert_eq!` マクロを使用して `get_cells()` を呼び出し、 `input_universe` と `expected_universe` が同じ `Cell` 配列の値を持っていることを確認します。 Rust で生成した WebAssembly のコードをテストできるように、コードブロックに `#[wasm_bindgen_test]` 属性を追加し、 `wasm-pack test` を使って WebAssembly のコードをテストしています。

```rust
#[wasm_bindgen_test]
pub fn test_tick() {
    // Let's create a smaller Universe with a small spaceship to test!
    let mut input_universe = input_spaceship();

    // This is what our spaceship should look like
    // after one tick in our universe.
    let expected_universe = expected_spaceship();

    // Call `tick` and then see if the cells in the `Universe`s are the same.
    input_universe.tick();
    assert_eq!(&input_universe.get_cells(), &expected_universe.get_cells());
}
```

Run the tests within the `wasm-game-of-life` directory by running
`wasm-pack test --firefox --headless`.

> `wasm-pack test --firefox --headless` を実行して `wasm-game-of-life` ディレクトリ内のテストを実行します。
