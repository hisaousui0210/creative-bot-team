# Creative Bot Team

個人の意図を引き継ぐAI制作環境。会話と直接操作を組み合わせ、制作の判断と経過を次の編集へ引き継ぐ構想です。

現時点は企画・プロンプト・ディレクトリ構成の準備段階です。動くアプリ、Devin Botsへの設定、実行ログ、性能・利用者評価はまだありません。

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

最初のBotはCreative Producerを1体とし、他の役割は必要な子セッションとして呼び出す計画です。最初の検証対象は「トップの印象を軽快にして」という一件の要求から、根拠・レビュー・判断・変更・確認結果が残る一連の処理です。

## 参照した公式資料

- [Devin Bots](https://docs.devin.ai/work-with-devin/bots)
- [Connections and secrets](https://docs.devin.ai/work-with-devin/bots/connections)
- [Access and security](https://docs.devin.ai/work-with-devin/bots/access-and-security)
- [Usage limits](https://docs.devin.ai/work-with-devin/bots/usage-limits)

APIの名前やMCPツールの案は自作するものです。Devinの標準機能として実装済みという意味ではありません。実際のアカウント設定・接続・料金上限は導入時に確認します。
