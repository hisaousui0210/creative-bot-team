# Creative Bot Team

個人の意図を引き継ぐAI制作環境。会話と直接操作を組み合わせ、制作の判断と経過を次の編集へ引き継ぐ構想です。

現時点は企画・プロンプト・ディレクトリ構成の準備段階です。動くアプリ、Devin Botsへの設定、実行ログ、性能・利用者評価はまだありません。

## 実装を始めるとき

1. [現在地と再開地点](docs/current-state.md) を確認する。
2. [最初の実装依頼：M0 ローカル編集画面](docs/tasks/01-local-editor.md) を実装担当へ渡す。
3. [作業指示書](docs/implementation.md) の順序で、直接編集画面→mockの一往復→実接続へ進む。

最初の完成単位はAI接続なしで動く保存・直接編集画面です。次に、[人工シナリオ](tests/fixtures/top-lighten/scenario.json) で「トップを軽快に」という依頼の判断と変更を追います。実接続の設定がなくても、この2段階は実装できる計画にしています。

## Botに渡すプロンプト全文

- [Creative Producer](prompts/creative-producer.txt)
- [Extractor](prompts/extractor.txt)
- [参謀A・利用者の意図](prompts/advisor-intent.txt)
- [参謀B・実装と副作用](prompts/advisor-delivery.txt)
- [Builder・実装](prompts/builder.txt)
- [QA・操作確認](prompts/qa.txt)

Devin BotsのGeneral設定にあるInstructionsへ、担当の本文を配置する想定です。子セッションへ渡す場合も同じ本文と、その作業用の入力を渡します。プロンプトは権限を強制する仕組みではなく、接続・ツール側の制御と併用します。

## 資料

- [構想から設計までのピッチ](docs/pitch.txt)
- [要求と受入条件](docs/spec.md)
- [構成と状態管理](docs/architecture.md)
- [段階的な実装計画](docs/roadmap.md)
- [実装作業指示書](docs/implementation.md)
- [現在地と再開地点](docs/current-state.md)
- [JSON契約](packages/contracts/contracts.schema.json)
- [人工データによる例](tests/fixtures/contract-examples.json)

## ディレクトリ

```text
apps/web/                   編集画面・会話・プレビューの実装予定地
apps/server/                API・SQLite・ジョブ管理の実装予定地
packages/contracts/         JSON Schemaと将来の共通型
packages/agents/            役割別の実行処理の実装予定地
packages/providers/         確認済みモデル接続の実装予定地
prompts/                    Botへ渡す指示全文
tests/fixtures/             私信を含まない人工データ
docs/                       企画・設計・工程
```

## 起動と開発

アプリの起動コマンドはまだありません。実装時にDocker Composeによるローカル起動を用意し、日本語の操作手順を追加します。初期のデモは本人のテストプレイが終わるまでローカルで扱います。

Devinに接続する段階ではCreative Producerを1体とし、他の役割は必要な子セッションとして呼び出す計画です。接続前はmockで同じ契約を通します。M0/M1の仕様と受入条件はdocs配下を正本とし、過去のピッチ文は背景資料として扱います。

## 参照した公式資料

- [Devin Bots](https://docs.devin.ai/work-with-devin/bots)
- [Connections and secrets](https://docs.devin.ai/work-with-devin/bots/connections)
- [Access and security](https://docs.devin.ai/work-with-devin/bots/access-and-security)
- [Usage limits](https://docs.devin.ai/work-with-devin/bots/usage-limits)

APIの名前やMCPツールの案は自作するものです。Devinの標準機能として実装済みという意味ではありません。実際のアカウント設定・接続・料金上限は導入時に確認します。
