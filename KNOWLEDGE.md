# トラブルシューティング・知見まとめ

## 数独の生成アルゴリズム（クライアントサイドJS）

### 完全盤面の生成
バックトラック法でランダムな完全盤面を生成する。

```js
function fillBoard(board) {
  for (let r = 0; r < 9; r++) {
    for (let c = 0; c < 9; c++) {
      if (board[r][c] === 0) {
        for (const n of shuffle([1,2,3,4,5,6,7,8,9])) {
          if (isValidPlacement(board, r, c, n)) {
            board[r][c] = n;
            if (fillBoard(board)) return true;
            board[r][c] = 0;
          }
        }
        return false;
      }
    }
  }
  return true;
}
```

### 唯一解の保証（MRVヒューリスティック）
セルを除去するたびに解答数をカウントし、1のときだけ除去を確定する。
MRV（Minimum Remaining Values）により、候補が最少のセルから探索するため高速。

```js
function countSolutions(board) {
  // 最少候補セルを探す → そこから分岐 → count >= 2 で即打ち切り
}
```

### 生成時間について
難しい難易度（エキスパート）ほど唯一解チェックの試行が増えるため遅くなる。
「パズル生成中…」のオーバーレイを表示し、`setTimeout(0)` で描画を先に行ってから生成することでUIのブロックを防ぐ。

---

## スマホのタップ操作とキーボード

### 数字入力にinputを使うと問題が起きる
スマホで `<input type="number">` を使うとソフトキーボードが出てレイアウトが崩れる。

**解決策**
カスタムの数字ボタン（1〜9）をHTMLで作り、キーボードを出さない。`input` 要素は使わない。

---

## CSSでのボードレイアウト

### 9×9グリッドをスマホに収める
```css
.board {
  width: min(96vw, 96svh, 480px);
  aspect-ratio: 1 / 1;
}
```
`svh`（Small Viewport Height）を使うとスマホのアドレスバー表示時にも正しく収まる。

### 3×3ブロックの太い境界線
行コンテナの `nth-child(3n)` に `border-bottom` を当てる方法がシンプル。

```css
.row:nth-child(3n) { border-bottom: 2px solid var(--thick-border); }
.cell:nth-child(3n) { border-right: 2px solid var(--thick-border); }
```

---

## ピンチズームの無効化

数独ボードはズームされると操作しにくいため無効化する。

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0,
      maximum-scale=1.0, user-scalable=no">
```

ただし、アクセシビリティ上の懸念があるため、将来的にはズーム対応も検討。

---

## Service Workerの更新タイミング

**症状**
GitHub Pagesのコードを変更してpushしても、ホーム画面アイコンから起動したアプリにすぐ反映されない。

**原因**
Service Workerはバックグラウンドで更新チェックを行い、次回起動時から新バージョンが有効になる。

**対処方法**
- 1回アプリを開いて閉じ、再度開くと新しいバージョンが適用される
- `sw.js` の `CACHE_NAME` のバージョン番号を上げると強制更新できる（例: `sudoku-v1` → `sudoku-v2`）

---

## manifest.json のアイコンについて

アイコン画像ファイルを別途用意せず、SVGをdata URI形式でmanifest.jsonに直接埋め込んでいる。
ただしブラウザによってはdata URIのアイコンを無視する場合があるため、本格運用時はPNGファイルを用意するのが望ましい。
