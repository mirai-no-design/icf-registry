# icf-registry

*[English](README.en.md)*

**共通語彙の住所録 ── ICFコード・支援コード・関連語彙・対応関係の小さなレジストリ**

[ICF HUB](https://github.com/mirai-no-design/icf-hub) の第1層（Registry＝共通の意味）です。

このリポジトリは、次のものを登録・参照するためのレジストリです。

- ICFコードへの参照（心身機能 b/s・活動と参加 d・環境因子 e）
- 支援コード・活動/参加/環境因子に関する共通語彙
- 現場語（袋詰め・品出し・接客など、現場で実際に使われる言葉）
- 各組織の独自コードと共通語彙との対応関係（マッピング）

## 設計方針：賢い処理はしない

ここは「住所録」です。持つのは **ID・名称・意味・バージョン・対応関係・出典** 程度に留めます。検索の高度化、推薦、類似度計算、AIによる自動対応づけなどの便利機能は、このリポジトリには入れません（それらはApplications層の自由です）。

**レジストリの正本はこのリポジトリのデータファイルそのもの**です（`registry/` 以下のJSON）。サーバは必須ではありません。読み取り専用APIとして配信する場合の規約は [SPEC.md](SPEC.md) 第6章に定めますが、それはファイルの写しを返すだけのものです。

## 構成

```
registry/
  codesystems.json   # 参照するコード体系の一覧（ICF, 独自体系, 組織コード…）
  concepts.json      # 概念エントリ（ICF参照＋共通語彙）
  terms.json         # 現場語エントリ（概念への対応・信頼度つき）
  mappings.json      # 組織独自コード ⇄ 概念の対応関係
schema/              # 上記のJSON Schema（機械検証用）
examples/            # 最小の登録例
SPEC.md              # レジストリ仕様（正本）
GOVERNANCE.md        # 登録・変更のルール
```

## 最小の例

```json
{
  "id": "icfhub:concept/d740",
  "codesystem": "who-icf",
  "code": "d740",
  "label_ja": "公的な対人関係",
  "layer": "participation",
  "definition_ref": "https://icd.who.int/dev11/l-icf/en",
  "status": "active",
  "since": "0.1.0"
}
```

```json
{
  "id": "icfhub:term/000123",
  "surface_form": "袋詰め",
  "lang": "ja",
  "domain": "work",
  "concepts": [
    { "concept": "icfhub:concept/d440", "confidence": 0.85 },
    { "concept": "icfhub:concept/d210", "confidence": 0.6 }
  ],
  "source": "現場登録（就労支援）",
  "status": "active",
  "since": "0.1.0"
}
```

## WHO ICFの取り扱いについて（重要）

ICF分類本体（コード全文・定義文・分類構造の記述）はWHOの著作物です。本レジストリは **ICFの分類テキストを再配布しません**。concepts はコード番号・短縮ラベル・WHO正本への参照（`definition_ref`）のみを保持します。定義の全文が必要な実装者は、WHOの公式配布物を各自の利用条件に従って参照してください。

## 参加するには

登録・修正の提案はPull Requestで受け付けます。出典の明記が必須です。詳細は [GOVERNANCE.md](GOVERNANCE.md) を参照してください。

## ライセンス

- 仕様書・文書：CC BY 4.0
- レジストリデータ（`registry/`）：CC BY 4.0（ただしICF本体はWHOの利用条件に従う）
- JSON Schema（`schema/`）：MIT（実装への自由な組み込みのため）

引用情報は [CITATION.cff](CITATION.cff) を参照。リリースごとにZenodoでDOIが付与されます。
