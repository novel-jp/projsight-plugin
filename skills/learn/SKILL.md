---
name: learn
description: ProjSight 学習カリキュラムのハブ。進捗一覧と次の演習を提案する。
---

# /learn

ProjSight 学習カリキュラムの **ナビゲーションハブ** です。
受講者の進捗を表示し、次に取り組むべき演習を提案してください。

---

## Step 1: 学習プロジェクトの特定

1. `list_projects()` でプロジェクト一覧を取得
2. 学習用プロジェクト（Org が学習専用 Org、またはプロジェクト名に「学習」「training」等を含む）を特定
3. 特定できない場合 → ユーザーに「どのプロジェクトで学習していますか？」と確認

---

## Step 2: 進捗データの収集

1. `list_tasks(projectId)` で全タスクを取得
2. 各演習スキルに対応するタスクの status を確認する

### タスクと演習のマッチングルール

タスクが演習に対応するかは、以下の優先順位で判定する:

1. タスクの **title または description に演習番号**（`00-01`, `01-01` 等）を含む → マッチ
2. タスクの **title にスキル名**（`learn-setup`, `learn-pc-compare` 等）を含む → マッチ
3. タスクの **title にセクション名のキーワード**（`セットアップ`, `比較体験`, `Prompt Craft` 等）を含む → 候補として表示し、受講者に確認

いずれにもマッチしないタスクは演習と無関係（受講者の自由タスク等）として扱う。

### 演習とスキルの対応表:

| セクション          | 演習  | スキル名                | 必修 |
| ------------------- | ----- | ----------------------- | ---- |
| 00 セットアップ     | 00-01 | `/learn-setup`          | Yes  |
| 01 Prompt Craft     | 01-01 | `/learn-pc-compare`     | Yes  |
| 01 Prompt Craft     | 01-02 | `/learn-pc-rewrite`     |      |
| 01 Prompt Craft     | 01-03 | `/learn-pc-write`       |      |
| 01 Prompt Craft     | 01-04 | `/learn-pc-review`      |      |
| 01 Prompt Craft     | 01-05 | `/learn-pc-reporting`   |      |
| 02 Context Eng.     | 02-01 | `/learn-ce-analyze`     | Yes  |
| 02 Context Eng.     | 02-02 | `/learn-ce-design`      |      |
| 02 Context Eng.     | 02-03 | `/learn-ce-dd`          |      |
| 02 Context Eng.     | 02-04 | `/learn-ce-maintain`    |      |
| 02 Context Eng.     | 02-05 | `/learn-ce-session`     |      |
| 03 Intent Eng.      | 03-01 | `/learn-ie-analyze`     | Yes  |
| 03 Intent Eng.      | 03-02 | `/learn-ie-write`       |      |
| 03 Intent Eng.      | 03-03 | `/learn-ie-constraints` |      |
| 03 Intent Eng.      | 03-04 | `/learn-ie-casestudy`   |      |
| 03 Intent Eng.      | 03-05 | `/learn-ie-question`    |      |
| 04 Spec Eng.        | 04-01 | `/learn-se-decompose`   | Yes  |
| 04 Spec Eng.        | 04-02 | `/learn-se-workflow`    |      |
| 04 Spec Eng.        | 04-03 | `/learn-se-risk`        |      |
| 04 Spec Eng.        | 04-04 | `/learn-se-dependency`  |      |
| 04 Spec Eng.        | 04-05 | `/learn-se-issue`       |      |
| 05 AI Collab        | 05-01 | `/learn-ac-evaluate`    |      |
| 05 AI Collab        | 05-02 | `/learn-ac-iterate`     |      |
| 05 AI Collab        | 05-03 | `/learn-ac-debug`       | Yes  |
| 05 AI Collab        | 05-04 | `/learn-ac-code-review` |      |
| 05 AI Collab        | 05-05 | `/learn-ac-security`    |      |
| 06 Git Workflow     | 06-01 | `/learn-gw-branch`      | Yes  |
| 06 Git Workflow     | 06-02 | `/learn-gw-pr`          |      |
| 06 Git Workflow     | 06-03 | `/learn-gw-review`      |      |
| 06 Git Workflow     | 06-04 | `/learn-gw-cicd`        |      |
| 07 模擬プロジェクト | 07-01 | `/learn-pj-kickoff`     | Yes  |
| 07 模擬プロジェクト | 07-02 | `/learn-pj-execute`     | Yes  |
| 07 模擬プロジェクト | 07-03 | `/learn-pj-incident`    |      |
| 07 模擬プロジェクト | 07-04 | `/learn-pj-retrospect`  | Yes  |

---

## Step 3: 進捗の判定

各演習の状態を以下のルールで判定する:

| タスク status               | 表示      |
| --------------------------- | --------- |
| `completed`                 | ✅ 完了   |
| `in_progress` / `in_review` | 🔄 進行中 |
| `not_started`               | ⬜ 未着手 |
| タスクが存在しない          | ⬜ 未着手 |

**「次はここ」の判定ロジック**:

1. セクション順（00 → 01 → ...）に走査
2. 各セクション内で演習番号順に走査
3. 最初に見つかった「未着手」または「タスクなし」の演習を `🔵 次はここ` にする
4. ただし前のセクションの必修演習（xx-01）が未完了の場合、そのセクションは「🔒 前提未完了」と表示

---

## Step 4: 出力フォーマット

以下のフォーマットで出力する:

```markdown
# ProjSight 学習カリキュラム

## あなたの進捗

| セクション          | 演習                        | 状態   |
| ------------------- | --------------------------- | ------ |
| 00 セットアップ     | /learn-setup                | {状態} |
| 01 Prompt Craft     | /learn-pc-compare (必修)    | {状態} |
| 01 Prompt Craft     | /learn-pc-rewrite           | {状態} |
| 01 Prompt Craft     | /learn-pc-write             | {状態} |
| 01 Prompt Craft     | /learn-pc-review            | {状態} |
| 01 Prompt Craft     | /learn-pc-reporting         | {状態} |
| 02 Context Eng.     | /learn-ce-analyze (必修)    | {状態} |
| 02 Context Eng.     | /learn-ce-design            | {状態} |
| 02 Context Eng.     | /learn-ce-dd                | {状態} |
| 02 Context Eng.     | /learn-ce-maintain          | {状態} |
| 02 Context Eng.     | /learn-ce-session           | {状態} |
| 03 Intent Eng.      | /learn-ie-analyze (必修)    | {状態} |
| 03 Intent Eng.      | /learn-ie-write             | {状態} |
| 03 Intent Eng.      | /learn-ie-constraints       | {状態} |
| 03 Intent Eng.      | /learn-ie-casestudy         | {状態} |
| 03 Intent Eng.      | /learn-ie-question          | {状態} |
| 04 Spec Eng.        | /learn-se-decompose (必修)  | {状態} |
| 04 Spec Eng.        | /learn-se-workflow          | {状態} |
| 04 Spec Eng.        | /learn-se-risk              | {状態} |
| 04 Spec Eng.        | /learn-se-dependency        | {状態} |
| 04 Spec Eng.        | /learn-se-issue             | {状態} |
| 05 AI Collab        | /learn-ac-evaluate (必修)   | {状態} |
| 05 AI Collab        | /learn-ac-iterate           | {状態} |
| 05 AI Collab        | /learn-ac-debug             | {状態} |
| 05 AI Collab        | /learn-ac-code-review       | {状態} |
| 05 AI Collab        | /learn-ac-security          | {状態} |
| 06 Git Workflow     | /learn-gw-branch (必修)     | {状態} |
| 06 Git Workflow     | /learn-gw-pr                | {状態} |
| 06 Git Workflow     | /learn-gw-review            | {状態} |
| 06 Git Workflow     | /learn-gw-cicd              | {状態} |
| 07 模擬プロジェクト | /learn-pj-kickoff (必修)    | {状態} |
| 07 模擬プロジェクト | /learn-pj-execute (必修)    | {状態} |
| 07 模擬プロジェクト | /learn-pj-incident          | {状態} |
| 07 模擬プロジェクト | /learn-pj-retrospect (必修) | {状態} |

## 次の演習

> **{演習番号} {スキル名}** — {演習の説明}（{所要時間}）
>
> `{スキル名}` を実行して開始してください。

## カリキュラム概要

- **00 セットアップ** — 環境構築と初回体験（1 演習）
- **01 Prompt Craft** — 指示の構造化（5 演習）★ Wave 1
- **02 Context Eng.** — 情報環境の設計（5 演習）★ Wave 2
- **03 Intent Eng.** — 目的と判断基準（5 演習）★ Wave 2
- **04 Spec Eng.** — 自律実行できる仕様（5 演習）★ Wave 3
- **05 AI Collab** — AI との協働（5 演習）★ Wave 3
- **06 Git Workflow** — チーム開発フロー（4 演習）★ Wave 3
- **07 模擬プロジェクト** — 統合演習（4 演習）★ Wave 4

### 受講パス

| パス           | 対象         | 演習数   | 所要時間              |
| -------------- | ------------ | -------- | --------------------- |
| **必修コース** | 全員         | 10 演習  | 約 10h（1.5 日）      |
| **発展コース** | 希望者       | +22 演習 | 追加 15〜20h          |
| **全コース**   | じっくり習得 | 32 演習  | 約 25〜30h（4〜5 日） |
```

---

## 注意事項

- `$ARGUMENTS` が指定された場合、該当セクションの詳細情報のみを表示する
- 全演習完了時は「おめでとうございます！」メッセージと成長サマリーを表示する
- 全 Wave（1〜4）の演習が利用可能。全演習完了時は特別な祝福メッセージを表示する
