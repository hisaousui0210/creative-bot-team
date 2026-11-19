# 構成と状態管理

状態: 実装前の決定。既存アプリはまだなく、apps配下は空の配置先。

## 構成と責任範囲

| 配置 | 責務 | 要求 |
|---|---|---|
| apps/web | React + TypeScript、Scene描画、直接編集、会話・ジョブ・記憶の表示 | R-001,002,007,008 |
| apps/server | Fastify + TypeScript、API、SQLite、transaction、ジョブ実行 | R-002〜009 |
| packages/contracts | JSON Schema、Scene・操作・ジョブの共通型と境界検証 | R-001〜008 |
| packages/agents | Extractor、参謀A/B、Producer、Creatorの実行順序 | R-004〜007 |
| packages/providers | mockと確認済み実接続のAdapter | R-008 |
| prompts | 役割別指示全文の正本 | R-004,005 |

pnpm workspaceを使用する。webはViteでbuildし、serverが静的ファイルとAPIを同じoriginから配信する計画。Dockerは1つのアプリサービスとSQLite用named volumeから始める。外部公開は127.0.0.1:4317のみ、コンテナ内は3000。ホスト側のNode/pnpmは必須にせず、buildと実行をコンテナに閉じ込める。manifest・Dockerfile・compose.yamlは最初の実装で作る。

## Sceneと操作（R-001〜003）

Sceneはversion=1、canvas={width,height}、nodes配列。canvasは初期1200×900 CSS pxとし、画面幅に応じて表示を縮小する。M0では自由配置の平坦な配列で、入れ子・自動レイアウト・レスポンシブ生成は扱わない。座標変換は表示倍率を考慮する。

Nodeはid、kind、x、y、width、height、props。kindはframe/text/button/icon。propsはframe=backgroundColor、text=text/color/fontSize、button=label/backgroundColor/color、icon=name/colorに限定。色は6桁HEX、fontSizeは8〜200、アイコンは同梱のallowlist。位置とサイズは有限数値でcanvas内、幅高は1以上。最大200要素。frameは背景・枠の表示要素で、子要素を持つコンテナではない。

SceneMutationはexpectedRevisionとoperations。操作はadd_node(node)、move_node(nodeId,x,y)、resize_node(nodeId,width,height)、set_props(nodeId,props)、remove_node(nodeId)。1要求は最大100操作。未知属性、未知ID、重複ID、不正値は要求全体を拒否する。JSONの任意パスやHTML/JavaScriptを直接適用しない。

ドラッグ中はブラウザ内で描画し、pointerupで1操作を送る。全操作を検証して一括commitし、revisionを単調増加する。Undo/Redoも新revisionとして保存する。Undo後の新規操作はRedo分岐を破棄する。作品のUndoでメモリや会話は巻き戻さない。

## APIと永続化（R-002,003,006,009）

| API | 入力 | 成功応答 |
|---|---|---|
| POST /api/v1/projects | name | 201 projectId |
| GET /api/v1/projects/:id/scene | なし | 200 {revision,scene} |
| POST /api/v1/projects/:id/scene/mutations | SceneMutation | 200 {revision,scene} |
| POST /api/v1/projects/:id/scene/history | direction=undo/redo,expectedRevision | 200 {revision,scene} |
| POST /api/v1/projects/:id/messages | sessionId,text | 201 messageId |
| POST /api/v1/projects/:id/jobs | sessionId,messageId,expectedSceneRevision | 202 jobId |
| GET /api/v1/projects/:id/jobs/:jobId | なし | 200 状態・結果・イベント |
| GET /api/v1/projects/:id/memory | sessionId,keys | 200 MemorySnapshot |
| POST /api/v1/projects/:id/memory/:memoryId/revoke | expectedRevision,reason | 200 失効結果 |

全更新APIはIdempotency-Key必須。project+route+keyとcanonicalな本文ハッシュを保存する。同じキーと同じ本文には元の応答を返し、別本文には409。エラーは{error:{code,message,retryable},requestId}。参謀レビュー、採否、昇格は初期はサーバー内関数を使い、任意のクライアントがroleを名乗るだけで昇格できるAPIを公開しない。

SQLiteはprojects、scenes、scene_history、idempotencyから作る。M1でmessages、goals、jobs、job_events、memory_candidates、reviews、decisions、memories、memory_headsを追加する。作品・履歴・冪等応答を同じtransactionに保存する。メモリの昇格は候補、原文、参謀、goalRevision、expectedRevisionを検証し、旧版失効と新版作成とdecisionを一括commitする。

各表はproject/ownerまたはproject経由で所有を限定する。mock用projectはdataOrigin=syntheticで実データと隔離する。有効メモリは同一scope+対象ID+keyにつき1件。SQLiteのNULLを含むuniqueに依存せず、正規化したscopeKeyまたはscope別indexを使う。

## 役割の実行と接続（R-004〜008）

M1はmock Adapterが固定シナリオを順次返す。Producerは抽出→提案→参謀A/B→採否→変更→記憶→結果を管理する。参謀の初回入力は同一snapshotで、相手の結論を含めない。再検討は最大2ラウンド。fixtureはこの流れを試す人工入力であり、モデルによる自律判断の実績ではない。

Devin Botや他の実接続はM2で利用可能な手段を確認してAdapterへ実装する。Devin固有の機能が必要な運用計画と、アプリがmockで起動する条件を分ける。共有メモリはアプリ側が正本で、Devin自身の記憶は開発上の要約と参照先に使う。

実接続には呼出し方式、取消・再開の対応、使用量、認証、ローカルへの到達性の確認が必要。クラウドのlocalhostやSQLiteを本人のPCと同一と仮定しない。未接続時は自動で別の有料APIへ切り替えない。

## 失敗・取消・再開（R-003,007）

ジョブはspec.mdの状態を使う。起動時に残ったrunningはWORKER_RESTARTEDでfailedにし、自動で外部処理を再送しない。再開は前のjobId/actionIdを参照し、既にcommitされた変更を再適用しない。初期は同時に1つの作品変更ジョブを処理する。

取消はcancelRequestedを保存し、commit前に確認する。適用済みの利用者操作は消さない。Scene・メモリ・目標の版が変わった場合は409を返して最新条件から再提案する。失敗後に不足する記憶保存だけを再開できるよう、作品変更と記憶保存の完了イベントを別々に記録する。

## 認証と復元

ローカルでも同一originを確認する。サーバー起動時に作るローカルセッションをHttpOnly/SameSite=Strict cookieで扱い、変更にはCSRF対策を置く。認証・権限はモデルの自己申告に依存しない。本人データ、資格情報、ログはGitへ入れない。

DBはnamed volumeに保持し、migrationは番号付きで適用する。破壊的migrationと既存DBの削除を自動で行わない。実データを扱う前に、SQLiteの整合したバックアップから別volumeへ復元する手順を用意する。

M0の確認はSceneと保存・競合、M1は根拠・昇格・再開に絞る。実ブラウザの確認と、mockによる自動確認、実接続の確認を区別して報告する。
