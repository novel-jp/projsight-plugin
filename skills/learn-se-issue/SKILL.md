---
name: learn-se-issue
description: 04-05 Issue 管理。バグ発見シナリオで Issue の起票から解決までの一巡を体験する。
---

# /learn-se-issue — Issue ライフサイクル（04-05）

バグ発見シナリオで Issue の起票から解決までの一巡を体験する演習です。

**所要時間**: 約 45 分
**前提**: `/learn-se-dependency`（04-04）完了済み
**スキル対応**: Specification Engineering（自律実行できる仕様）

---

## Step 1: 導入

```
「Issue と Risk の違いを明確にしましょう:
- Issue = 既に起きた問題（バグ、障害、仕様の不備）
- Risk = まだ起きていない問題（可能性と影響を管理）

Issue のライフサイクル:
  open → in_progress → resolved / closed / wont_fix

Issue 管理で重要なのは『トレーサビリティ』です:
  問題の発見 → タスク化 → 修正 → 解決
この一連の流れを追跡できることで、
『なぜこの修正をしたのか』『この問題は解決済みか』が明確になります。」
```

---

## Step 2: バグ発見シナリオ

仮想的なバグシナリオを提示する。04-01 の成果物に関連する例:

| テーマ             | バグシナリオ                                                                           |
| ------------------ | -------------------------------------------------------------------------------------- |
| **ユーザー認証**   | メールアドレスのバリデーションが不完全で、`user@` のような不正な形式が登録できてしまう |
| **ブログ機能**     | 記事タイトルに HTML タグを入力すると、一覧画面で XSS が発生する                        |
| **ダッシュボード** | データが 0 件の時にグラフコンポーネントがクラッシュする                                |
| **自由テーマ**     | 受講者の成果物に合わせたシナリオを一緒に考える                                         |

```
「あなたの成果物でバグが見つかったと想定してください。
Issue を起票してみましょう。

description には以下を Markdown で記載します:
- 問題の詳細 — 何が起きているか
- 影響範囲 — 誰がどの程度影響を受けるか
- 再現手順 — どうすれば再現できるか」
```

受講者に `upsert_issue(category: 'bug', severity, description)` で起票してもらう。

---

## Step 3: タスク化とリンク

Issue に対応するタスクを作成し、紐づける。

1. `upsert_task` で修正タスクを作成（背景に Issue の内容を参照）
2. `create_link(sourceType: 'task', sourceId: taskId, targetType: 'issue', targetId: issueId, linkType: 'resolves')` で紐づけ
3. `start_work(taskId)` でタスクを開始

```
「create_link(type: 'resolves') で Issue とタスクを紐づけます。
これにより:
- タスク側から『どの Issue を解決するための作業か』が分かる
- Issue 側から『どのタスクで対応しているか』が分かる
- 因果関係のトレーサビリティが確保される」
```

---

## Step 4: 解決と完了

修正を実施し、Issue をクローズする。

1. 簡単な修正を実施（バリデーション追加、null チェック等）
2. `complete_work(taskId)` でタスクを完了
3. `upsert_issue(status: 'resolved')` で Issue をクローズ

```
「Issue の解決フロー:
  1. バグ発見 → upsert_issue で起票
  2. タスク作成 → create_link(resolves) で紐づけ
  3. start_work → 修正実施 → complete_work
  4. upsert_issue(status: 'resolved') でクローズ

この 4 ステップが Issue ライフサイクルの基本です。」
```

---

## Step 5: 振り返り

```
「Issue のトレーサビリティ:
  問題 → タスク → 修正 → 解決
  create_link(linkType: 'resolves') で因果関係を追跡可能

Specification Engineering セクションの総まとめ:
- 04-01: 成果物をタスクに分解する技術
- 04-02: start_work → complete_work のワークフロー
- 04-03: リスクを事前に洗い出し、計画する技術
- 04-04: タスク間の依存関係を明示する技術
- 04-05: Issue のライフサイクルとトレーサビリティ

これら全てが『AI が自律実行できる仕様』を構成します。
タスクの内容、順序、リスク、問題の追跡 — 仕様が完全であるほど、
AI は迷わず、正確に、長時間の作業を遂行できます。

おめでとうございます。Specification Engineering セクションの全演習が完了しました。」
```

受講者のタスクを `upsert_task(progressPct: 100)` で完了にする。
