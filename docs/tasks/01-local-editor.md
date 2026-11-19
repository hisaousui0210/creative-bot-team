# 最初の実装依頼：M0 ローカル編集画面

## 依頼

Creative Bot Teamの既存リポジトリで、モデル接続なしに直接編集と保存ができる最小版を実装してください。docs/spec.mdのR-001〜003、R-008〜009と、docs/architecture.mdのM0を契約にします。新しい別アプリへ置き換えず、既存の配置先を使ってください。

## 入力と所有範囲

- apps/server：Scene保存・検証・履歴・冪等処理。
- apps/web：Scene描画と編集UI。
- packages/contracts：SceneとSceneMutationの型・境界検証。
- ルート：workspace、固定した依存、Dockerfile、compose.yaml、日本語README。
- 初期データ：tests/fixtures/top-lighten/scenario.jsonのinitialScene。

最初に未コミット変更と適用AGENTSを確認し、mainの有無を確認してcodex/作業ブランチを作ってください。複数担当を使う場合も共通contractsの所有者を固定し、同じファイルを同時に編集しないでください。

## 作るもの

1. 指定された4要素を表示できる画面と、見た目の見本1案。
2. 要素の追加・選択・移動・サイズ/属性変更・削除。
3. SQLiteへの保存、再読込、Undo/Redo。
4. expectedRevisionとIdempotency-Keyによる二重変更・競合の防止。
5. Dockerだけでbuildと実行ができる起動手順。予定URLはhttp://127.0.0.1:4317。

画面は作品領域と必要な編集操作を中心にし、未実装の会話やBotの動作を成功したように飾らないでください。余分なスローガンや説明文を追加しないでください。

## この依頼で作らないもの

実際のLLM呼出し、Devin接続、Extractor実装、参謀の自律実行、メモリ昇格、SNS投稿、外部公開。人工シナリオは表示・保存の入力としてだけ使い、本人の実データへ混ぜないでください。

## 完了の見せ方

AC-001、AC-002、AC-008の直接編集部分を確認し、起動URL、操作結果、対象の確認コマンドと結果、未確認項目を残してください。実画面で要素追加→移動→属性変更→Undo→Redo→再読込を1本通し、画面幅と表示倍率による座標のズレも見てください。

アプリ実装と、実接続・スマートフォン実機・ペン実機の確認を区別してください。commitと未完了と次の1手をdocs/current-state.mdへ残してください。初期デモはローカルに留めてください。
