---
name: learn-gw-branch
description: 06-01 ブランチ戦略（必修）。Trunk-based development と task/<number>-<slug> 命名規則を学び、実際にブランチを作成する。
---

# /learn-gw-branch — ブランチ戦略（06-01、必修）

Trunk-based development の概念と、ブランチ命名規則を学び、実際にブランチを作成する演習です。

**所要時間**: 約 30 分
**前提**: `/learn-ac-evaluate`（05-01）完了済み
**スキル対応**: Git Workflow（チーム開発フロー）

---

## Step 1: 導入

```
「Git Workflow — チーム開発のフローを回す力を学びます。

AI 時代でも Git は開発の基盤です。
むしろ AI がコードを高速に生成するからこそ、変更管理の重要性が増しています。

このセクションでは以下を学びます:
- Trunk-based development の考え方
- ブランチ命名規則の意味
- ProjSight タスクとブランチの紐づけ」
```

---

## Step 2: 学習環境の確認

受講者の `learn-projsight` リポジトリ（00-01 で作成済み）で作業する。

リポジトリが見つからない場合:

```bash
# GitHub リポジトリがある場合
gh repo clone learn-projsight && cd learn-projsight

# ない場合はローカルで作成
mkdir -p learn-projsight && cd learn-projsight && git init
```

---

## Step 3: ブランチ戦略の説明

以下の内容を受講者に説明する:

### Trunk-based Development

```
「Trunk-based development とは:
- main ブランチが常にリリース可能な状態を保つ
- 短命なブランチ（short-lived branches）で作業する
- ブランチの寿命は数時間〜数日。長くても 1 週間以内
- main への直接 push は禁止。必ず PR 経由でマージ」
```

### ブランチ命名規則

| パターン               | 例                  | 説明                              |
| ---------------------- | ------------------- | --------------------------------- |
| `task/<number>-<slug>` | `task/42-add-login` | ProjSight のタスク番号 + 短い説明 |
| `fix/<number>-<slug>`  | `fix/15-null-check` | バグ修正                          |

```
「命名規則があると:
- ブランチを見ただけで『何のタスクか』が分かる
- ProjSight のタスク番号で検索できる
- AI エージェントが自動的に正しいブランチ名を生成できる」
```

---

## Step 4: ブランチ作成の実践

受講者に以下を実践してもらう:

1. **タスクの確認**: `list_tasks` で自分のタスクを確認
2. **ブランチの作成**: 適当なタスクを選び、命名規則に従ってブランチを作成

```bash
git checkout -b task/<タスク番号>-<slug>
```

3. **簡単な変更**: README.md やコメントの追加など、小さな変更を加える
4. **コミット**: 変更をコミットする

```bash
git add .
git commit -m "feat: <変更内容の説明>"
```

### コミットメッセージの規則

| プレフィックス | 用途             |
| -------------- | ---------------- |
| `feat:`        | 新機能           |
| `fix:`         | バグ修正         |
| `refactor:`    | リファクタリング |
| `docs:`        | ドキュメント     |
| `test:`        | テスト           |

---

## Step 5: 振り返り

```
「ブランチ戦略のポイント:

- main は常にリリース可能 — 壊れたコードを main に入れない
- 短命ブランチ — 長く生きるブランチはコンフリクトの温床
- 命名規則 — task/<number>-<slug> でタスクとの紐づけが一目瞭然
- コミットメッセージ — プレフィックスで変更の種類を明示

次の /learn-gw-pr では、PR を作成し、ProjSight に紐づける全フローを体験します。」
```

受講者のタスクを `upsert_task(progressPct: 100)` で完了にする。
