# 実装作業指示書

今回の準備でアプリは実装していない。本書は次の実装依頼の入口。

## 正本と着手条件

要求と受入条件はspec.md、構成とAPIはarchitecture.md、工程はroadmap.md、役割指示はprompts、メモリの構文はpackages/contracts/contracts.schema.jsonを参照する。過去のピッチ文は背景資料で、現在の仕様を上書きしない。

着手時に対象repo、適用AGENTS、未コミット変更、ブランチ、Dockerへの接続、予定ポート、既存ファイルを確認する。mainの有無を確認してからcodex/作業ブランチを作る。今回の準備ブランチはcodex/implementation-prep。

## 作業の順序と担当範囲

| ID | 変更範囲の候補 | 作業と引渡し | 受入条件 |
|---|---|---|---|
| T-01 | ルートmanifest、Dockerfile、compose.yaml、packages/contracts、apps/server | Docker起動、Scene型と操作検証、SQLite保存、revision・冪等処理。先に型・API契約を固定 | AC-002,008 |
| T-02 | apps/web | 見本を1案作ってから、4種類の要素、選択・移動・属性変更・Undo/Redoを実装 | AC-001,008 |
| T-03 | apps/serverのmemory、messages、migration | 原文保存、範囲、候補、有効化、訂正・失効、所有境界 | AC-003,005,006 |
| T-04 | packages/agents、packages/providers、serverのjobs | mockシナリオ、2役レビュー、判断、ジョブと再開。保存・変更はAPI/サービス経由 | AC-004,007,008 |
| T-05 | apps/webの会話・履歴・記憶表示 | 一言→各工程→結果と原文根拠を同じjobIdで追える画面 | AC-003,004,006 |
| T-06 | packages/providers、接続手順 | 実際に利用可能な公式手段を確認して接続を1種類追加 | AC-008の実接続部分 |

T-01→T-02でM0、T-03〜05でM1、T-06でM2。同時進行は必須にしない。複数担当を使う場合は許可条件を先に確認し、共通contractsの所有者を1名に固定する。BackendとFrontendが同じファイルを同時に変更しない。

## 最初に渡す依頼

[tasks/01-local-editor.md](tasks/01-local-editor.md) を、そのまま実装担当へ渡せる。最初の依頼でM1/M2をまとめて実装しない。まず保存できる直接編集画面を動かし、次の工程へ引き継ぐ。

## 実装するコマンド入口

次は今後用意する入口であり、現在は実行できない。

- docker compose up --build：127.0.0.1:4317で起動。
- pnpm test:core：当該段階の契約・保存・競合に絞る。
- pnpm test:editor：人工ページの編集と保存の主要フロー1本。

pnpmとNodeの版、SQLiteドライバー、依存の固定版は最初の実装で確認し、packageManagerとlockfileへ記録する。推測の版を準備段階でインストールしない。Dockerの内部でbuildと実行を完結できるようにする。

## 確認と受入

通常の確認は原則1回、失敗修正後まで最大2回を目安とする。全suiteとlint/typecheck/buildを毎回束ねない。M0ではSceneの競合・冪等処理を対象テストで確認し、実ブラウザではAC-001の操作を1本通す。本人が明示的に確認を依頼した範囲はその依頼を優先する。

ブラウザの見本作成と操作確認は実装段階の仕事で、今回の準備では行っていない。スマートフォン幅の確認と実機のペン確認を混同しない。実行した確認、未実行の確認、接続・実機の未確認を分けて残す。

## 引渡しと再開

各工程でcommit、起動方法、入力fixture、確認した版、実際の結果、未完了、次の1手をcurrent-state.mdへ更新する。本人データや秘密は記録しない。必要な差分だけをCodex表記付きcommitにし、作業ブランチへpushする。既定ブランチへの直接push、force push、無断の公開・配備はしない。
