---
name: phase-transition
description: フェーズ切替時のチェックリスト。成果物・DR・リスク・Issue の棚卸しとフォークを実行。
---

# /phase-transition

フェーズ切替時に実行するチェックリスト。admin プロファイルのツールを使用する。

---

## 前提

- ProjSight MCP Server のツールを使用する（admin プロファイル必須）
- 各ステップの判断はユーザーに提示し、**承認を得てから** 実行する
- フォーク前に現フェーズの状態を整理し、フォーク後に新フェーズのスコープを定義する

---

## Step 1: 現フェーズの棚卸し

### 1a. 成果物の確認

1. `list_deliverables(projectId)` で全成果物を取得
2. ステータス別に分類してテーブル表示:

| ステータス    | アクション                                                 |
| ------------- | ---------------------------------------------------------- |
| `approved`    | 完了。対応不要                                             |
| `review`      | 承認するか、次フェーズに持ち越すかユーザーに確認           |
| `in_progress` | 完了見込みを確認。延期する場合は次フェーズのスコープに記録 |
| `draft`       | 着手していない。次フェーズに持ち越すかドロップするか判断   |

### 1b. DR 棚卸し

`/dr-review` を実行する。

### 1c. リスク・Issue の確認

1. `list_risks(projectId)` で未対応リスク（`identified` / `planned`）を一覧表示
   - 次フェーズに carry-over するか、close するかユーザーに確認
2. `list_issues(projectId)` で未解決 Issue（`open`）を一覧表示
   - 同上

---

## Step 2: 前提・制約条件の見直し

1. `list_assumptions(projectId)` で前提条件を一覧表示
   - 崩れた前提、陳腐化した前提がないか確認
   - 該当があればユーザーに報告し、影響を評価
2. `list_constraints(projectId)` で制約条件を一覧表示
   - 追加・緩和された制約がないか確認
   - 該当があればユーザーに報告

---

## Step 3: Phase 遷移実行（DD-#137）

プロジェクト内に Phase エンティティが存在する場合（`list_phases` で確認）は **同一プロジェクト内で Phase を遷移** する。Phase が未導入の場合は従来の `fork_project` を使用する。

### Phase エンティティ方式（推奨）

1. ユーザーの承認を得て `create_phase(projectId, name, status: 'active')` を実行
   - 現在の active Phase は自動的に `completed` に遷移する
2. 返却された Phase の `id` を記録
3. Step 1 で carry-over と判断されたリスク・Issue の `phaseId` を新 Phase に更新:
   - `update_risk(riskId, phaseId: <newPhaseId>)`
   - `update_issue(issueId, phaseId: <newPhaseId>)`

### fork_project 方式（Phase 未導入プロジェクト）

1. ユーザーの承認を得て `fork_project(projectId, newName, newDescription)` を実行
2. 返却された新プロジェクト ID を記録

---

## Step 4: 新フェーズのスコープ定義

1. `set_scope(projectId, inScope, outOfScope, phaseId)` で新フェーズのスコープを定義
   - **phaseId**: Step 3 で作成した Phase の ID（Phase 方式の場合）
   - **inScope**: このフェーズで実装するもの
   - **outOfScope**: 次フェーズ以降に延期するもの（永久除外は書かない）
2. Step 1 で持ち越しとなった成果物を新フェーズに紐づけ
3. 必要に応じて新フェーズの成果物・マイルストーンを作成（`phaseId` を設定）

---

## 完了条件

- [ ] 現フェーズの全成果物のステータスが確認済み
- [ ] DR 棚卸しが完了
- [ ] 未対応リスク・未解決 Issue の carry-over 判断が完了
- [ ] 前提・制約条件の見直しが完了
- [ ] 新フェーズが作成済み（Phase エンティティ or fork_project）
- [ ] 新フェーズのスコープが定義済み

すべて完了したら「フェーズ切替完了。新フェーズ [名前] で作業を開始できます。」と報告する。
