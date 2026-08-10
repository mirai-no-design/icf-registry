# ICF Registry 仕様 v0.1（Draft）

*[English](SPEC.en.md)*

| 項目 | 内容 |
|---|---|
| ステータス | Draft |
| 版 | 0.1.0 |
| 発行 | 株式会社みらいのでざいん（起草）／ ICF HUB Project |
| ライセンス | 本文 CC BY 4.0 ／ Schema MIT |

本仕様における「しなければならない（MUST）」「すべきである（SHOULD）」「してもよい（MAY）」はRFC 2119の慣例に従う。

## 1. 目的と非目標

### 1.1 目的

異なるサービス・組織が、人の生活機能・活動・参加・環境と支援に関する情報を交換するときに参照する、**共通語彙の住所録**を提供する。

### 1.2 非目標（このレジストリがしないこと）

- 検索の高度化・ランキング・推薦・類似度計算
- 自然言語処理・AIによる自動対応づけ（対応づけの**結果**を登録することはできる）
- 個人データの保持（レジストリは語彙のみを扱う。人に関するデータは一切持たない）
- ICF分類テキストの再配布（第7章）

## 2. データモデル

レジストリは4種類のエントリからなる。すべてのエントリは共通フィールド `id` / `status` / `since` / `deprecated`（任意） / `source`（任意）を持つ。

### 2.1 CodeSystem（コード体系）

参照するコード体系の宣言。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `id` | string | MUST | 体系ID（例：`who-icf`、`org-example-childcheck`） |
| `name` | string | MUST | 名称 |
| `publisher` | string | MUST | 発行者 |
| `version` | string | SHOULD | 参照している版 |
| `url` | string | SHOULD | 正本のURL |
| `license_note` | string | SHOULD | 利用条件に関する注記 |

### 2.2 Concept（概念）

共通語彙の1エントリ。ICFコードへの参照、またはICFに対応しない共通概念（支援コード・個人コンテキスト層など）。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `id` | string | MUST | `icfhub:concept/{code}` 形式の安定ID |
| `codesystem` | string | MUST | CodeSystemのid（例：`who-icf`、`icfhub-support`） |
| `code` | string | MUST | 体系内コード（例：`d740`） |
| `label_ja` / `label_en` | string | MUST(ja) | 短縮ラベル。ICFの場合も**短縮形**に留める |
| `layer` | string | MUST | `body` / `activity` / `participation` / `environment` / `personal` / `support` |
| `parent` | string | MAY | 上位概念のid（階層） |
| `definition_ref` | string | SHOULD | 定義の正本への参照URL（ICFはWHO正本） |
| `version` | string | MUST | エントリの版 |

### 2.3 Term（現場語）

現場で使われる言葉と概念の対応。1つの現場語は複数概念に対応してよい。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `id` | string | MUST | `icfhub:term/{番号}` |
| `surface_form` | string | MUST | 表記（例：袋詰め） |
| `lang` | string | MUST | 言語（BCP 47。例：`ja`） |
| `domain` | string | SHOULD | `work` / `education` / `community` / `care` / `tourism` 等 |
| `concepts[]` | array | MUST | `{concept, confidence(0–1)}` の配列 |
| `source` | string | MUST | 出典（どの現場・文献から来た対応か） |

### 2.4 Mapping（対応関係）

組織独自コードと概念の対応。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `id` | string | MUST | `icfhub:mapping/{番号}` |
| `source_system` | string | MUST | CodeSystemのid（独自体系側） |
| `source_code` | string | MUST | 独自体系内のコード |
| `target_concept` | string | MUST | 対応する概念のid |
| `relation` | string | MUST | `exact` / `broader` / `narrower` / `related`（SKOSの語彙に準拠） |
| `confidence` | number | SHOULD | 0–1 |
| `evidence` | string | MUST | 出典・根拠（誰がどう確認した対応か） |

## 3. 識別子

- エントリIDは `icfhub:{type}/{local}` 形式とし、**一度発行したIDは削除・再利用しない**（MUST）
- 廃止は `status: "deprecated"` と `deprecated: "<版>"` で表現する（MUST）
- 将来、`https://` 形式の解決可能なURI名前空間に移行する場合も、`icfhub:` IDは別名として維持する

## 4. バージョニング

- レジストリ全体はリリース単位で**セマンティックバージョニング**（`MAJOR.MINOR.PATCH`）
- エントリの追加・出典の追記＝MINOR。ラベル誤字修正＝PATCH。意味が変わる変更＝新IDの発行＋旧IDのdeprecated（MAJORの契機）
- 交換時（icf-exchange-spec参照）は、参照したレジストリ版を `registry_version` として明示する（SHOULD）

## 5. 配布形式

- 正本は本リポジトリの `registry/*.json`（UTF-8）
- 各ファイルはトップレベルで `{"registry_version": "...", "entries": [...]}` を持つ
- スキーマ検証：`schema/*.schema.json`（JSON Schema draft 2020-12）に適合しなければならない（MUST）

## 6. 読み取り専用API規約（任意実装）

レジストリをHTTPで配信する場合は、以下の規約に従う。**これはファイルの写しを返すだけのものであり、検索の高度化を提供してはならない（MUST NOT）。**

```
GET /v0/codesystems            → codesystems.json の内容
GET /v0/concepts               → concepts.json の内容（?layer= による絞り込みのみMAY）
GET /v0/concepts/{id}
GET /v0/terms?surface_form=    → 完全一致・前方一致のみMAY
GET /v0/mappings?source_system=
```

- レスポンスはJSON（`application/json; charset=utf-8`）
- すべてのレスポンスに `registry_version` を含める（MUST）
- 認証は不要（公開データのみのため）。個人データをこのAPIに載せてはならない（MUST NOT）

## 7. WHO ICFの取り扱い

- 本レジストリはICF分類テキスト（定義全文・包含/除外注記等）を**保持・再配布しない**（MUST NOT）
- ConceptのICF参照はコード番号・短縮ラベル・`definition_ref` に限る
- 実装者がICF定義全文を必要とする場合は、WHOの公式配布物を各自の利用条件で参照する

## 8. 個人データの禁止

レジストリに個人を特定しうる情報・個人の評価データを登録してはならない（MUST NOT）。人に関するデータの交換は icf-exchange-spec の領域であり、そこでも本人同意の管理が前提となる。
