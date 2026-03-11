# トラブルシューティング・知見まとめ

## 数独の生成アルゴリズムについて

### py-sudoku ライブラリの使い方
```python
from sudoku import Sudoku

# 難易度は 0.0（簡単）〜 1.0（難しい）で指定
puzzle = Sudoku(3).difficulty(0.5)  # 9×9
board = puzzle.board  # None が空白マス
```

`None` を `0` に変換してフロントエンドに渡すと扱いやすい。

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
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  width: min(95vw, 95vh);  /* 縦横の小さい方に合わせる */
  aspect-ratio: 1 / 1;
}
```

### 3×3ブロックの太い境界線
```css
.cell:nth-child(3n) { border-right: 2px solid #333; }
.cell:nth-child(n+19):nth-child(-n+27) { border-bottom: 2px solid #333; }
/* 行の区切りは行コンテナに border-bottom を使う方がシンプル */
```

---

## iPhoneからMacのローカルサーバーにアクセスできない

**チェックリスト（順番に確認）**
1. Macでサーバーが起動しているか（`.command` を起動済みか）
2. iPhoneとMacが **同じWiFi** に接続されているか
3. IPアドレスが正しいか（起動時のコンソールに表示される）
4. MacのIPアドレス: `ipconfig getifaddr en0` で確認
5. Macのファイアウォールがブロックしていないか（システム設定 → ネットワーク → ファイアウォール）

---

## ピンチズームの無効化

数独ボードはズームされると操作しにくいため無効化する。

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0,
      maximum-scale=1.0, user-scalable=no">
```

ただし、アクセシビリティ上の懸念があるため、将来的にはズーム対応も検討。

---

## api.py のコードを変更した場合はサーバーの再起動が必要

uvicorn は `--reload` フラグなしで起動しているため、コード変更は自動反映されない。

**再起動方法**
1. `.command` のターミナルウィンドウを閉じる
2. 「数独を起動.command」を再度ダブルクリック

---

## household-budget との共存

| アプリ | ポート |
|--------|--------|
| household-budget（FastAPI） | 8000 |
| household-budget（Streamlit） | 8501 |
| sudoku（FastAPI） | 8001 |

両アプリを同時起動するときはポート衝突しない。
