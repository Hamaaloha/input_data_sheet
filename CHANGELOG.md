# sheet_input_app.html 開発ログ

## 2026-05-25

### 初期作成
- `fitness_apps/` フォルダに以下のHTMLアプリを作成
  - `index.html` — メニュー画面
  - `sheet_input_app.html` — 測定評価シート データ入力アプリ（本ファイル）
  - `cmj_app.html` — CMJ評価
  - `grip_app.html` — 握力評価
  - `back_strength_app.html` — 背筋力評価
  - `grip_back_app.html` — 握力＋背筋力 複合評価
- GitHub Pages に公開：https://hamaaloha.github.io/input_data_sheet/

---

### OCR 修正（Tesseract.js）
**問題：** `エラー: undefined` および `TypeError: Cannot read properties of null (reading 'SetImageFile')`

**原因と対応：**
1. `createWorker` の `workerPath`/`langPath`/`corePath` を明示指定（CDN パス未解決エラー対応）
2. `canvas.toDataURL('image/png')` に変換してから渡す（`SetImageFile` エラー対応）
3. `createWorker/terminate` を廃止し `Tesseract.recognize()` に統一
4. 画像の blob URL を `fetch → FileReader → dataURL` に変換してから渡す（blob URL 非対応対応）
5. エラーメッセージを `String(e)` でフォールバック（`e.message === undefined` 対応）

**設定：**
- `tessedit_char_whitelist: '0123456789.-'`（数字・小数点のみ）
- `tessedit_pageseg_mode: '6'`（全体OCR）/ `'4'`（範囲OCR）

---

### 一括ペースト機能（Excel ライク）
- テーブルの任意のセルをクリック → Ctrl+V で貼り付け
- Tab 区切り（Excel コピー）とスペース区切りを自動判定
- 貼り付け開始位置は選択中のセルから右下方向に展開

---

### 画像ズーム修正
- `max-width: 100%` が効いてズームできなかった問題を修正
- ズーム時に `max-width: none` に切り替え
- **ドラッグでパン**：マウス左ボタンでつかんで移動
- **ホイールズーム**：カーソル位置を中心にズームイン/アウト
- **左方向へのドラッグ修正**：`justify-content: center`（Flexbox）を `display: block; margin: auto` に変更（左側オーバーフローのスクロール不可バグ解消）

---

### 姓・名の分離（全6シート）
- `氏名`（1列）→ `姓` + `名`（2列）に分割
- CSV/MD 出力でも別列として出力
- 名前一括入力モーダルで以下のフォーマットに対応：
  - タブ区切り（Excel）：`山田　太郎　4年　男`
  - スペース区切り：`山田太郎 4年 男`
  - 姓名に空白：`山田 太郎  4年  男`（2スペースで列区切り）

---

### 自動小数点変換（入力補助）
フォーカスが外れた時点で自動フォーマット。`data-dec` 属性で小数桁数を指定。

| 種目 | 型 | 入力例 | 変換結果 |
|---|---|---|---|
| 30m走（時間） | `stime` | `423` | `4.23` s |
| MB投げ（距離） | `mdist` | `1234` | `12.34` m |
| 握力 | `dec1` | `123` | `12.3` kg |
| 背筋力 | `dec1` | `1234` | `123.4` kg |

---

### CSV/MD インポート
- 「📂 CSV/MD読込」ボタンで過去に出力したファイルを読み込みセルに復元
- 列ヘッダー名でマッピング（種目変更後でも対応）
- CSV のクォート対応、MD テーブル形式対応

---

### 名前一括入力モーダルの改善
- モーダルを開いたとき、**既に入力済みの姓・名・学年・性別がテキストエリアに表示**される
- 説明文とプレースホルダーを統一（Tab/スペース両対応を明記）

---

### 範囲選択 OCR
- 「🔲 範囲OCR」ボタン → カーソルが十字に変わる
- 読み取りたい列をドラッグして囲む → 自動で OCR 実行
- **前処理：** 選択領域を3倍拡大 + オートコントラスト + グレースケール
- `pageseg_mode: 4`（縦1列モード）で認識精度向上

---

### OCR 対象列の統一
- `num` 型のみだった OCR マッピングを `num`/`stime`/`mdist`/`dec1` すべてに対応
- `NUMERIC_TYPES = new Set(['num','stime','mdist','dec1'])` で一元管理
- No.・姓・名・学年・性別は無視し、数値測定列のみを対象に

---

### クリップボード画像ペースト
- テーブル以外の場所で **Ctrl+V** すると、クリップボードの画像が左ペインに表示
- スマホで写真を長押し→「コピー」→ PC ブラウザに貼り付けでも動作
- 貼り付け後はそのままズーム・範囲OCRが使用可能

---

## 技術スタック

| ライブラリ | 用途 | CDN |
|---|---|---|
| PDF.js 3.11.174 | PDF 表示 | jsdelivr |
| Tesseract.js 4 | OCR | jsdelivr |

## ファイル構成

```
fitness_apps/
├── index.html            # メニュー画面
├── sheet_input_app.html  # 測定評価シート データ入力（本ファイル）
├── cmj_app.html          # CMJ評価アプリ
├── grip_app.html         # 握力評価アプリ
├── back_strength_app.html# 背筋力評価アプリ
├── grip_back_app.html    # 握力＋背筋力 複合評価アプリ
├── README.md             # 使い方説明
└── CHANGELOG.md          # 本ファイル
```
