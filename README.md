# kintone バックアップビューア
*kintoneはcybozuの登録商標です。

kintoneからバックアップした CSV / JSON ファイルを、インストール不要でブラウザだけで閲覧できるビューアです。実体は単一の HTML ファイル ([index.html](index.html)) で、ビルドやサーバーサイド処理は一切必要ありません。

## できること

- レコード CSV を読み込んでレコード一覧を表示・検索・絞り込み・ソート
- レコードをクリックして詳細表示(全フィールド + サブテーブル + コメント)
- 日時フィールド(UTC)をブラウザのタイムゾーンまたは UTC で表示
- Shift_JIS / UTF-8 の自動判定
- コメント・レイアウト・フィールド・ビューを追加で読み込ませることで表示を強化(下記参照)
- 大きい CSV でも画面がフリーズしないよう、パース処理をチャンク分割して進捗バー付きで実行

## 使い方

### 方法1: ファイルを直接開く

1. [index.html](index.html) をブラウザ(Chrome など)で直接開く
2. 「レコード」欄でレコード CSV を選択(必須)。必要に応じて「コメント」「レイアウト」「フィールド」「ビュー」も選択、またはドラッグ&ドロップ
3. 文字コード(自動判定 / Shift_JIS / UTF-8)と日時の表示タイムゾーンを選んで「読み込む」

### 方法2: Web サーバー経由でクエリパラメータから自動読み込み

`file://` では `fetch` が使えないため、簡易 HTTP サーバー越しに開くと URL のクエリパラメータでファイルパスを指定でき、画面操作なしで自動的に読み込まれます。

```
python3 -m http.server 8000
```

```
http://localhost:8000/index.html?records=records.csv&comments=comments.csv&layout=layout.json&field=field.json&views=views.json&encoding=shift_jis
```

同梱の [sample/](sample/) ディレクトリのデモデータでそのまま試す場合:

```
http://localhost:8000/index.html?records=sample%2F%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%83%86%E3%82%B9%E3%83%88_20260909T151535%2B0900.csv&comments=sample%2F%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%83%86%E3%82%B9%E3%83%88_20260909T151336%2B0900_comments.csv&layout=sample/layout.json&field=sample/field.json&views=sample/views.json&encoding=shift_jis
```

| パラメータ | 内容 | 必須 |
| --- | --- | --- |
| `records`(または `csv`) | レコード CSV のパス | ○ |
| `comments` | コメント CSV のパス | - |
| `layout` | レイアウト JSON (`app/form/layout`) のパス | - |
| `field`(または `fields`) | フィールド JSON (`app/form/fields`) のパス | - |
| `views` | ビュー JSON (`app/views`) のパス | - |
| `encoding` | `auto` / `shift_jis` / `utf-8`(省略時 `auto`) | - |
| `tz` | `local` / `utc`(省略時 `local`) | - |

## 読み込ませるファイルと効果

kintoneアプリの管理画面や REST API から取得できるファイル/JSONです。**レコード CSV のみ必須**で、他は読み込ませた分だけ機能が強化されます。

| ファイル | 必須 | 読み込むと... |
| --- | --- | --- |
| レコード CSV | ○ | 一覧・検索・詳細表示ができるようになる |
| コメント CSV | - | 詳細画面にレコードごとのコメントが表示される |
| `layout.json` | - | フォームの並び順どおりにフィールドを正しく列マッピングし、サブテーブルの内容も表示できるようになる(無いと簡易的なフラット表示になる) |
| `field.json` | - | フィールドの種類(日時・選択肢など)を認識し、絞り込みUIやチェックボックスの整形表示が強化される |
| `views.json` | - | 画面上部でビュー(一覧)を切り替えて、表示カラムを変更できるようになる |

デモ用のサンプルデータは [sample/](sample/) ディレクトリに同梱されています(レコード CSV・コメント CSV・`layout.json`・`field.json`・`views.json`)。

## 制限事項

- **添付ファイル(FILE型フィールド)はCSVに含まれないため表示できません**(kintoneのCSVエクスポート自体の仕様)
- ビューの `filterCond`(絞り込み条件)は解釈しておらず、`fields`(表示カラム)と `sort` のみ反映します
- 全レコードをブラウザのメモリ上に保持する方式のため、数百MBを超えるような巨大なCSVには不向きです(読み込み中のフリーズは回避していますが、メモリ不足のリスクは残ります)。その規模を扱う場合は duckDB-wasm 等でのクエリ方式への切り替えを検討してください
- `layout.json` が無い場合、システム項目やサブテーブルの列がラベル単位のフラット表示になり、サブテーブルとしてのグルーピングはされません

## 動作環境

- モダンブラウザ(Chrome / Edge / Safari 等)。`TextDecoder('shift_jis')` を利用するため、対応していない古いブラウザでは文字コード自動判定が動作しません
- クエリパラメータからの自動読み込みは `http(s)://` 経由でのみ動作します(`file://` では手動でファイルを選択してください)

## ライセンス

このリポジトリのコード([index.html](index.html) 等)は [Apache License 2.0](LICENSE) の下で公開しています。

**例外**: [51-modern-default.css](51-modern-default.css) は対象外です。これは Cybozu 製の kintone プラグイン用 CSS フレームワークで、ファイル冒頭のコメントに記載の通り MIT License(Copyright (c) 2014 Cybozu)で提供されています。
