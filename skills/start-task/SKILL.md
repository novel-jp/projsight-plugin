---
name: start-task
description: タスクを作成・紐づけ・開始する。コーディング前に必ずこのコマンドを実行すること。
---

# /start-task ワークフロー

ユーザーの入力 `$ARGUMENTS` をもとに、以下の手順を **順番通り・省略なし** で実行してください。

---

## Step 0: ワークフロー判定

入力内容を分析し、該当するワークフローを1つ選択する:

| #   | 条件                                                        | ワークフロー                                     |
| --- | ----------------------------------------------------------- | ------------------------------------------------ |
| 1   | 機能追加/変更/削除で DR が必要                              | DR作成 → タスク作成 → リンク → 開始              |
| 2   | バグ・問題対応で Issue が関連                               | Issue作成(必要なら) → タスク作成 → リンク → 開始 |
| 3   | リスク緩和で既存 Risk が関連                                | タスク作成 → リンク → 開始                       |
| 4   | 既存タスクID が指定されている                               | そのタスクで開始                                 |
| 5   | DR 改訂（既存 DR の supersede）                             | DR作成(supersedes) → タスク作成 → リンク → 開始  |
| 6   | 設計ドキュメント作成（スキーマ設計・API仕様・画面設計など） | 設計Doc作成 → タスク作成 → リンク → 開始         |

**DR vs 設計ドキュメントの判定基準**:

- **DR（#1/#5）**: 「なぜその選択をしたか」を記録する（意思決定の記録）
- **設計Doc（#6）**: 「何を作るか」の詳細仕様を記録する（設計の仕様書）
- 両方必要な場合: DR → 設計Doc → タスク の順で作成し、それぞれリンクする

判定に迷う場合はユーザーに確認する。

---

## Step 1: 前提エンティティの作成（ワークフロー #1, #2, #3, #5, #6）

### ワークフロー #1 / #5: DR 作成

1. `upsert_dr(projectId, title, context, decision, alternatives, consequences, ...)` を実行
   - #5 の場合は `supersedes` パラメータを指定
2. 返却された DR の `id` と `number` を記録

### ワークフロー #6: 設計ドキュメント作成

1. `add_design_doc(projectId, title, body, status?)` を実行
   - `body` にフル Markdown で設計仕様を記述（必須）
   - `status` は省略で `accepted`（デフォルト）
   - 既存設計の改訂の場合は `supersedes` パラメータを指定
2. 返却された設計ドキュメントの `id` と `number` を記録

### ワークフロー #2: Issue 作成（未登録の場合）

1. 対応する Issue が未登録なら `upsert_issue(projectId, title, category, severity, description)` を実行
   - **description は Markdown 形式**で、`## 問題` / `## 影響` / `## 対応箇所` を含める
2. 既存 Issue がある場合は `list_issues` で特定
3. Issue の `id` と `number` を記録

### ワークフロー #3: Risk 特定

1. `list_risks` で対象リスクを特定
2. Risk の `id` と `number` を記録

### ワークフロー #4: 既存タスク

1. `start_work(taskId)` を実行して **Step 1.5 へ進む**（Step 4 ではない）

---

## Step 1.5: 設計ドキュメント影響確認（全ワークフロー共通）

タスクの内容が既存の設計ドキュメントに影響するかを確認する。

1. `list_drs(projectId, type='design')` で設計ドキュメント一覧を取得
2. タスクの内容と照合し、影響がありそうなドキュメントをユーザーに提示
3. ユーザーが「更新不要」と判断 → **スキップ**
4. 更新が必要な場合、変更の規模に応じて方法を選択:

| 変更の規模                                   | 方法                                 | 手順                                           |
| -------------------------------------------- | ------------------------------------ | ---------------------------------------------- |
| **小規模**（typo修正、数値更新、小さな追記） | `upsert_dr(body)`                    | `get_dr` → body 修正 → `upsert_dr(drId, body)` |
| **大規模**（セクション追加/削除、構造変更）  | `add_design_doc(supersedes)`         | 新版を作成、旧版は自動で superseded            |
| **設計方針の根本変更**                       | 新 DR + `add_design_doc(supersedes)` | 意思決定を DR に記録してから設計Doc を新版作成 |

5. 更新した設計ドキュメントの `id` を記録（Step 3 でタスクとリンクする）

> **ワークフロー #4（既存タスク）の場合**: Step 1.5 完了後、Step 4 へスキップする。
> **ワークフロー #6（設計Doc新規作成）の場合**: 新規作成なので通常スキップ。ただし他の既存設計Docへの影響があれば確認する。

---

## Step 2: タスク作成（ワークフロー #1, #2, #3, #5, #6）

1. `list_deliverables(projectId)` で紐付け先の成果物を確認
2. ユーザーと紐付け先を確認（不明な場合）
3. **phaseId の決定**: `list_phases(projectId)` で active Phase を確認。active Phase があればその `id` を `phaseId` として使用する
4. `upsert_task(projectId, title, deliverableId, description, phaseId?)` を実行
   - **description は Markdown 形式**で、以下の構成で記述すること:
     - `## 背景` — なぜこのタスクが必要か
     - `## 対応内容` — 何をするか（箇条書き）
     - `## 完了条件` — チェックリスト形式
   - 基準: サイドペインで読んで作業着手できる程度の情報量
   - **phaseId**: active Phase があれば設定。なければ省略
5. 返却された Task の `id` を記録

### Step 2.5: タスク記述の品質チェック

作成したタスクの description を以下の観点でセルフチェックし、不足があればユーザーに提案する。
**強制ブロックではなくガイド形式**（ユーザーが「不要」と判断すればスキップ可能）。

| チェック観点         | 確認内容                                                                                                 | 不足時の提案例                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| **背景**             | `## 背景` セクションがあり、「なぜやるか」が書かれているか                                               | 「背景が未記入です。このタスクが必要な理由を追記しますか？」                         |
| **対応内容の具体性** | `## 対応内容` が箇条書きで具体的か。「〜を改善する」「〜を対応する」のような曖昧表現のみになっていないか | 「対応内容が抽象的です。具体的な変更箇所・操作を追記しますか？」                     |
| **完了条件**         | `## 完了条件` がチェックリスト形式で、**検証可能**（Yes/No で判定できる）か                              | 「完了条件が未記入です。どうなったら完了かを追記しますか？」                         |
| **制約**             | やらないこと・スコープ外が明記されているか（任意だが推奨）                                               | 「制約（やらないこと）の明記を推奨します。スコープの膨張を防げます。追記しますか？」 |

- 4 項目すべて OK → 「タスク記述の品質: 良好」と報告し、次のステップへ
- 1 項目以上不足 → 不足項目を一覧で提示し、ユーザーに改善するか確認。改善する場合は `upsert_task(taskId, description)` で更新

---

## Step 3: リンク作成（ワークフロー #1, #2, #3, #5, #6）

前提エンティティとタスクを紐づける:

| ワークフロー | create_link パラメータ                                                                                      |
| ------------ | ----------------------------------------------------------------------------------------------------------- |
| #1 / #5      | `sourceType: "dr", sourceId: <drId>, targetType: "task", targetId: <taskId>, linkType: "implements"`        |
| #2           | `sourceType: "issue", sourceId: <issueId>, targetType: "task", targetId: <taskId>, linkType: "resolves"`    |
| #3           | `sourceType: "risk", sourceId: <riskId>, targetType: "task", targetId: <taskId>, linkType: "mitigates"`     |
| #6           | `sourceType: "dr", sourceId: <designDocId>, targetType: "task", targetId: <taskId>, linkType: "implements"` |

Step 1.5 で設計ドキュメントを更新した場合は、そのドキュメントもタスクにリンクする:

| 条件                        | create_link パラメータ                                                                                      |
| --------------------------- | ----------------------------------------------------------------------------------------------------------- |
| 設計Doc を更新/新版作成した | `sourceType: "dr", sourceId: <designDocId>, targetType: "task", targetId: <taskId>, linkType: "implements"` |

`create_link(projectId, sourceType, sourceId, targetType, targetId, linkType)` を実行。

---

## Step 4: タスク開始

1. `start_work(taskId)` を実行。返却されるプロジェクトコンテキスト（counts, alerts）を確認し、重要なアラートがあればユーザーに報告する。
2. 返却された `relatedDeliverable.status` が `draft` の場合 → `upsert_deliverable(deliverableId, status: 'in_progress')` を実行して成果物を着手状態にする。
3. **ワークフロー #2（Issue 対応）の場合**: Issue の担当者とステータスを更新する
   - まず `get_issue(issueId)` で現在の状態を確認する
   - Issue の `ownerId` が既に設定されていて自分以外の場合 → **ユーザーに「既に {ownerName} が担当しています。上書きしますか？」と確認**。承認されなければスキップ
   - Issue の `status` が `in_progress` の場合 → **ユーザーに「既に対応中です。続行しますか？」と確認**。承認されなければスキップ
   - 上記チェックを通過した場合のみ:
     - `upsert_issue(issueId, ownerId: <auth.userId>)` で担当者を設定
     - Issue の `status` が `open` の場合のみ → `upsert_issue(issueId, status: 'in_progress')` で着手状態にする
4. **ワークフロー #3（Risk 緩和）の場合**: Risk の担当者を更新する
   - まず `get_risk(riskId)` で現在の状態を確認する
   - Risk の `ownerId` が既に設定されていて自分以外の場合 → **ユーザーに「既に {ownerName} が担当しています。上書きしますか？」と確認**。承認されなければスキップ
   - 上記チェックを通過した場合のみ:
     - `upsert_risk(riskId, ownerId: <auth.userId>)` で担当者を設定

---

## 完了条件

以下がすべて満たされていること:

- [ ] 前提エンティティ（DR/設計Doc/Issue/Risk）が MCP に登録済み
- [ ] 設計ドキュメントの影響確認が完了している（更新不要 or 更新済み）
- [ ] タスクが MCP に登録済み
- [ ] create_link でエンティティ間が紐づいている
- [ ] start_work でタスクが in_progress になっている
- [ ] 紐づく成果物が draft なら in_progress に遷移済み
- [ ] ワークフロー #2: Issue の ownerId が設定済み、status が open なら in_progress に遷移済み
- [ ] ワークフロー #3: Risk の ownerId が設定済み

すべて完了したら以下を報告する:

```
セットアップ完了。ブランチを作成してコーディングを開始してください。
ブランチ名: task/<number>-<slug>
```

※ `/start-task` 内で git 操作（ブランチ作成・push 等）は行わない。ポータビリティ維持のため、ブランチ作成はエンジニアまたはプロジェクト固有のルール（CLAUDE.md）に委ねる。
