# ProjSight ワークフロー

ProjSight MCP Server と連携するプロジェクト管理ワークフロー。
別リポジトリにも導入可能な共通ルール。

## はじめて使う方へ

ProjSight を初めて使う場合、このセクションで基本フローを把握してください。以降のセクションは詳細リファレンスです。

### 基本フロー

```
upsert_task → start_work → 実装 → complete_work
```

1. **タスクを作る** — `upsert_task(projectId, title, deliverableId, description)`
2. **作業を開始する** — `start_work(taskId)`
3. **実装する** — コードを書く、ドキュメントを作る、etc.
4. **作業を完了する** — `complete_work(taskId)`

> `/start-task` スキルを使うと、タスク作成から開始までを対話的にガイドしてくれます。初回は `/start-task` の利用を推奨します。

### 最初のタスクを作成する

`upsert_task` でタスクを作成する際、`description` は以下の構成で書くと AI が正しく作業できます:

```markdown
## 背景

なぜこのタスクが必要か

## 対応内容

- 何をするか（箇条書き）

## 完了条件

- [ ] チェックリスト形式で検証可能な条件
```

タスクはどの成果物（deliverable）に紐づくか指定します。成果物一覧は `list_deliverables(projectId)` で確認できます。

### `start_work` の返却内容の読み方

`start_work(taskId)` を実行すると、以下の情報が自動返却されます:

| フィールド              | 内容                                         | 活用方法                            |
| ----------------------- | -------------------------------------------- | ----------------------------------- |
| `task`                  | タスクの詳細（description 含む）             | 作業内容の確認                      |
| `projectContext.counts` | タスク・リスク・Issue・成果物の件数          | プロジェクト全体の状況把握          |
| `projectContext.alerts` | 注意が必要な項目（期限超過、stale タスク等） | 作業前に対処すべき問題の確認        |
| `relatedDeliverable`    | 紐づく成果物の進捗率とステータス             | 全体の中での位置づけ確認            |
| `activePhase`           | 現在アクティブなフェーズ                     | タスク作成時の `phaseId` 設定に使用 |

この自動返却により、AI エージェントは別途 `get_project_context` を呼ばなくても作業に必要なコンテキストを得られます。

### いつ Issue / Risk / Question を使うか

| エンティティ | 使う場面                                                                 | 例                                                     |
| ------------ | ------------------------------------------------------------------------ | ------------------------------------------------------ |
| **Issue**    | バグや問題を発見した時                                                   | 「API のレスポンスが 500 を返す」                      |
| **Risk**     | 将来起きうる問題を識別した時                                             | 「外部 API の仕様変更でデータ取得が止まる可能性」      |
| **Question** | 判断に必要な情報が不足している時                                         | 「認証方式は OAuth と API Key のどちらを採用するか？」 |
| **DR**       | 技術的な意思決定を記録する時（`/start-task` が必要に応じてガイドします） | 「データベースに DynamoDB を選択した理由」             |

詳細は各セクション（Issue ライフサイクル、リスク管理サイクル、質問管理）を参照してください。

---

## タスク作業時

- **注意**: ローカルの進捗表示機能（Claude Code の TaskCreate 等）は補助的な用途。MCP の `upsert_task` → `start_work` → `complete_work` が正式記録であり、省略してはならない
- 対応タスクがなければ → `list_deliverables` で紐付け先を確認し、`upsert_task(deliverableId)` で登録してから開始
- **phaseId の設定**: `start_work` レスポンスに `activePhase` が含まれる場合、タスク作成時に `phaseId` を設定する（DD-#137）。`activePhase` がなければ省略
- 開始 → `start_work(taskId)` — コンテキスト自動返却（activePhase 含む）、get_project_context 不要
- レビュー依頼 → 作業が完了したら `upsert_task(taskId, status: 'in_review')` を実行する。PR がある場合は `attach_pr` の後に実行する。Slack 通知（`task.in_review`）が発火し、PRリンク付きでレビュアーに通知される
- 完了 → ユーザーが確認・承認した後に `complete_work(taskId)` — deliverable 進捗自動算出
- DR/DD 関連タスクの完了時 → 紐づく DR/DD の status を `upsert_dr(status: 'implemented')` で更新する
- 一時中断（後で再開）→ `upsert_task(notes)` で状態を記録、ステータスは `in_progress` のまま
- 取り消し（やり直し）→ `cancel_work(taskId)` — progressPct/notes をクリアし `not_started` に戻す

## 成果物のステータス管理

成果物（deliverable）のステータスは **エンジニアが明示的に管理** する。自動遷移はない。

- **draft → in_progress**: 紐づくタスクの作業を初めて開始する時に `upsert_deliverable(status: 'in_progress')` を実行
  - `start_work` の返却で `relatedDeliverable.status === 'draft'` なら遷移する
- **in_progress → review**: 全タスク完了後、エンジニアがレビュー依頼する時に `upsert_deliverable(status: 'review')` を実行
- **review → approved**: ユーザーが承認した時に `upsert_deliverable(status: 'approved')` を実行

## /design-review 実行タイミング

`/design-review` は CTO 視点の設計レビューを実行するスキル。以下のタイミングで実行すること:

| タイミング               | 必須/推奨 | 理由                                         |
| ------------------------ | --------- | -------------------------------------------- |
| Plan 承認前              | **必須**  | ユーザーが承認する前に設計の妥当性を検証する |
| DR 作成後                | 推奨      | 意思決定の代替案・リスクの見落としを検出する |
| スコープ変更時           | 推奨      | スコープの整合性・YAGNI 違反を検出する       |
| 大規模リファクタリング前 | 推奨      | 既存アーキテクチャとの整合性を確認する       |

- レビュー結果の提案（DR/Risk/Issue 登録）はユーザーに提示し、承認を得てから登録する
- 軽微な変更（typo 修正、1ファイル変更等）では不要

## PR リンク管理

PR（Pull Request）とタスクの紐づけは AI エージェントの責務。GitHub Webhook は使わない（DR-0091）。

### PR 作成時

1. `gh pr create` で PR を作成し、返却された PR URL を取得
2. `attach_pr(taskId, url)` で PR をタスクに紐づける
3. `upsert_task(taskId, status: 'in_review')` でレビュー待ちにする（タスク作業時フロー参照）
4. ブランチ名は `task/<number>-<slug>` を推奨（例: `task/7-pr-workflow-rules`）

### タスク完了時

- `complete_work` 前に PR がマージ済みであることを確認する
- PR リンクが未登録の場合、`complete_work` の応答で suggestion が表示される

## チェックポイント（進捗記録）

長時間タスクや複数セッションにまたがる作業では、中間地点で ProjSight を更新する:

| タイミング               | アクション                                                                |
| ------------------------ | ------------------------------------------------------------------------- |
| タスクの作業を中断する時 | `upsert_task(progressPct, notes)` で現在の状態と次のステップを記録        |
| 大きな区切り到達時       | `upsert_task(progressPct)` で進捗を反映                                   |
| ブロッカー発生時         | `upsert_issue` or `upsert_question` で記録し、タスクの notes に参照を追記 |

**notes の書き方**: 次のセッションで作業を再開できる情報を含める

- 完了した作業の要約
- 次にやるべきこと
- 未解決の問題・判断待ち事項

## リスク管理サイクル

ステータス遷移: `identified` → `analyzing` → `planned` → `resolved` / `closed`（分岐: `escalated`）

| イベント               | アクション                                                                                      |
| ---------------------- | ----------------------------------------------------------------------------------------------- |
| リスク識別時           | `upsert_risk` — mitigation 定義済みなら `planned`、未定義なら `identified`                      |
| 分析開始               | `upsert_risk(status: 'analyzing')` — 影響範囲や対策を検討中                                     |
| 緩和策決定             | `upsert_risk(status: 'planned')` — mitigation フィールドに緩和策を記述                          |
| 緩和タスク開始         | `/start-task` ワークフロー #3 — ownerId 設定 + タスク作成・リンク・開始                         |
| 緩和タスク完了         | `upsert_risk(status: 'resolved')` — resolvedAt 自動記録                                         |
| リスク消滅・対応不要   | `upsert_risk(status: 'closed')`                                                                 |
| 深刻度上昇・即対応必要 | `upsert_risk(status: 'escalated')` — escalatedAt 自動記録、Webhook `risk.escalated` 通知が発火  |
| リスク顕在化           | `upsert_risk(status: 'resolved')` → `upsert_issue` → `create_link(caused_by)` で Issue と紐づけ |

## Issue ライフサイクル

ステータス遷移: `open` → `in_progress` → `resolved` / `closed` / `wont_fix`

| イベント         | アクション                                                                                      |
| ---------------- | ----------------------------------------------------------------------------------------------- |
| バグ・問題発見時 | `upsert_issue(category, severity, description)` — description は Markdown                       |
| 対応開始         | `/start-task` ワークフロー #2 — ownerId 設定 + status: `in_progress` + タスク作成・リンク・開始 |
| 修正完了後       | `upsert_issue(status: 'resolved')` — resolvedAt 自動記録                                        |
| 対応しない       | `upsert_issue(status: 'wont_fix')`                                                              |

## 質問管理

| イベント                   | アクション                                  |
| -------------------------- | ------------------------------------------- |
| 不明点発生時               | `upsert_question(title, body, assigneeId?)` |
| 議論の蓄積                 | `add_question_comment(body)`                |
| 結論確定                   | `answer_question(answer)`                   |
| 重要な方針決定に至った場合 | DR として別途記録（`upsert_dr`）            |

## PM 成果物の登録ルール

自発的に気づいた場合 → 会話内で提案し、ユーザーが承認したら MCP 直接登録する。

## プラン策定時（MUST）

プランはユーザーが承認するため、承認後の実装ステップで MCP 直接登録してよい。

- DRが必要な場合 → `upsert_dr` を実装ステップに含める（ローカルファイルは作成しない）
- DR品質基準:
  - 代替案を2つ以上（各案に却下理由）記載すること — 代替案のない DR は不合格
  - 「何を実装したか」ではなく「何をなぜ選んだか」を記録すること
- 設計ドキュメントが必要な場合 → `add_design_doc` を使用（DR とは別ツール）
- 技術的・スケジュール・依存関係のリスクを評価し、該当あれば → `upsert_risk` を実装ステップに含める
- リスク登録時、mitigation が定義済みなら `status: "planned"` で作成する（`identified` のまま放置しない）

## DR と設計ドキュメントの使い分け（DR-0109）

| 種類                       | ツール           | 用途                                                  | 参照タイミング           |
| -------------------------- | ---------------- | ----------------------------------------------------- | ------------------------ |
| **DD（設計ドキュメント）** | `add_design_doc` | 現行仕様（今どうなっているか） — **開発の第一参照先** | タスク開始時・実装中     |
| **DR（意思決定記録）**     | `upsert_dr`      | 意思決定の経緯（なぜこうなったか）                    | 経緯を辿る時・レビュー時 |

- DD が「正」。開発時は `list_drs(type='design')` で関連 DD を確認してから着手する
- DR はイミュータブルな変更履歴。DD から関連 DR にリンクする
- DR/DD ともにローカルファイルは作成しない。ProjSight（MCP）が正

## 設計ドキュメントの更新ルール

タスク開始時に既存の設計ドキュメント（DD）への影響を **必ず確認** し、必要に応じて更新する（DR-0109）。

| 変更の規模                              | 方法                                 |
| --------------------------------------- | ------------------------------------ |
| 小規模（typo、数値、小さな追記）        | `get_dr` → `upsert_dr(drId, body)`   |
| 大規模（セクション追加/削除、構造変更） | `add_design_doc(supersedes)`         |
| 設計方針の根本変更                      | 新 DR + `add_design_doc(supersedes)` |

## MCP エンティティのテキストフィールド

- テキストフィールドは **Markdown 形式**で記述する（見出し・箇条書き・コードブロック活用）
- 基準: **サイドペインで読んで作業着手できる程度の情報**を含めること。1行の要約文で済ませない
- タスク: 背景・対応内容・完了条件 / リスク: 状況・影響範囲・根拠 / イシュー: 問題・影響・対応箇所
- 設計ドキュメントや DR の body には Mermaid 図（```mermaid）を積極的に活用すること。Web UI でレンダリングされる

## その他

- サーバー応答の suggestions は文脈を踏まえて実行/スキップを判断する
- MCP が Single Source of Truth。ローカルファイルは作成しない
