---
name: learn-gw-pr
description: 06-02 PR 作成。gh pr create → attach_pr → upsert_task(status 'in_review') の全フローを体験する。
---

# /learn-gw-pr — PR 作成（06-02）

PR（Pull Request）を作成し、ProjSight のタスクに紐づける全フローを体験する演習です。

**所要時間**: 約 45 分
**前提**: `/learn-gw-branch`（06-01）完了済み
**スキル対応**: Git Workflow（チーム開発フロー）

---

## Step 1: 導入

```
「PR はコードレビューの起点であり、品質管理の要です。

ProjSight ワークフローでは、PR の作成からマージまでを一貫して管理します:
1. gh pr create — PR を作成
2. attach_pr — タスクに PR を紐づけ
3. upsert_task(status: 'in_review') — レビュー待ちに変更
4. Slack 通知が自動発火 — レビュアーに通知」
```

---

## Step 2: 学習環境の確認

受講者の作業ディレクトリに `learn-workspace/` があることを確認する。
なければ作成し、git リポジトリを初期化する:

```bash
mkdir -p learn-workspace && cd learn-workspace && git init
```

**GitHub リモートの確認**: PR を作成するには GitHub リモートが必要。
リモートが設定されていない場合は、受講者に GitHub リポジトリの作成を案内する。

```
「PR 演習には GitHub リポジトリが必要です。
学習用リポジトリがまだない場合は、以下のコマンドで作成できます:
gh repo create learn-workspace --private --source=. --push」
```

---

## Step 3: PR の作成

06-01 で作成したブランチ（またはこの演習用の新しいブランチ）で作業する:

1. **変更の追加**: 何かしらの意味のある変更を加える（関数追加、バグ修正等）
2. **コミット**: 変更をコミットする
3. **プッシュ**: リモートにプッシュする

```bash
git push -u origin <ブランチ名>
```

4. **PR 作成**: `gh pr create` で PR を作成する

```
「PR の description は以下の構造で書きましょう:

## Summary
- 変更内容の要約（1-3 箇条書き）

## Test plan
- テスト方法のチェックリスト

良い PR description は、レビュアーが『何を見ればいいか』を即座に理解できるものです。」
```

---

## Step 4: ProjSight への紐づけ

PR を ProjSight のタスクと紐づける:

1. **attach_pr**: `attach_pr(taskId, url)` で PR URL をタスクに登録
2. **ステータス更新**: `upsert_task(taskId, status: 'in_review')` でレビュー待ちに変更

```
「attach_pr + upsert_task(status: 'in_review') のセットが ProjSight ワークフローの重要なステップです。

これにより:
- Slack 通知（task.in_review）が発火し、PR リンク付きでレビュアーに通知される
- Web UI でタスクのステータスが『レビュー待ち』に変わる
- PR とタスクのトレーサビリティが確保される」
```

受講者の学習プロジェクトにタスクがない場合は、`upsert_task` で学習用タスクを作成してから紐づける。

---

## Step 5: 振り返り

```
「PR 作成フローのまとめ:

1. ブランチで作業 → コミット → プッシュ
2. gh pr create — 構造化された description を書く
3. attach_pr — ProjSight タスクに紐づけ
4. upsert_task(status: 'in_review') — レビュー待ちに変更

ポイント:
- PR description はレビュアーへの『コンテキスト提供』— Context Engineering の実践
- attach_pr を忘れると complete_work 時に suggestion が出る
- AI エージェントもこのフローに従う — 人間と AI の協働ルール

次の /learn-gw-review では、レビュー指摘への対応とマージの流れを体験します。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
