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

## Step 2: 演習タスクの作成と学習環境の確認

### 演習追跡タスクの作成

この演習を ProjSight で追跡するためのタスクを作成し、作業を開始する。

1. `list_deliverables(projectId)` で紐づけ先の成果物を確認する
2. `upsert_task` で演習タスクを作成する（タイトル例: 「06-01 ブランチ戦略の演習」）
3. `start_work(taskId)` で作業を開始する

> このタスクの ID を控えておいてください。演習の最後に `complete_work` で完了にします。

### 学習リポジトリの確認

受講者の `learn-projsight` リポジトリ（00-01 で作成済み）で作業する。
`learn-workspace/` 配下にある `learn-projsight` ディレクトリを AI が自動検出して案内する。

リポジトリが見つからない場合:

```bash
# learn-workspace 配下に作成
mkdir -p learn-workspace/learn-projsight && cd learn-workspace/learn-projsight && git init
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
- main への直接 push は禁止。必ず PR 経由でマージ

ProjSight では DR-0089 でこの戦略を採用しています。
理由は『AI エージェントが並行作業しやすく、コンフリクトを最小化できる』ため。
チーム規模が小さいうちは特に、シンプルな main + 短命ブランチが最も効率的です。」
```

### ブランチ命名規則

| パターン               | 例                  | 説明                              |
| ---------------------- | ------------------- | --------------------------------- |
| `task/<number>-<slug>` | `task/42-add-login` | ProjSight のタスク番号 + 短い説明 |

> **slug とは**: 英語の短い説明をハイフン区切りにしたもの（例: `add-login-page`, `fix-null-check`）。日本語のタスク名は英訳して短縮します。

```
「命名規則があると:
- ブランチを見ただけで『何のタスクか』が分かる
- ProjSight のタスク番号で検索できる
- AI エージェントが自動的に正しいブランチ名を生成できる」
```

---

## Step 4: ブランチ作成の実践

受講者に以下を実践してもらう:

### 1. タスクの確認

`list_tasks` で自分のタスクを確認する。

- タスクが存在する場合: 一番番号の小さい `not_started` のタスクを選ぶ
- タスクが存在しない場合: `upsert_task` で演習用タスクを作成する（タイトル例: 「README に学習メモを追加」）

### 2. ブランチの作成

選んだタスクの番号と slug を使ってブランチを作成する。

```bash
git checkout -b task/<タスク番号>-<slug>
```

### 3. 変更を加える

README.md に `## 学習メモ` セクションを追加してください。
内容は自由ですが、例えば今日学んだことを 1〜2 行書いてみましょう。

### 4. コミット

変更をコミットする。`git add` ではファイル名を明示するのがベストプラクティスです。

```bash
git add README.md
git commit -m "docs: README に学習メモセクションを追加"
```

### コミットメッセージの規則

コミットメッセージにプレフィックスをつけると、変更履歴の自動生成やレビュー時の素早い分類に役立ちます。

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

この演習では PR は扱いません。次の /learn-gw-pr で、ブランチから PR を作成し ProjSight に紐づける全フローを体験します。」
```

Step 2 で作成した演習追跡タスクを `complete_work(taskId)` で完了にする。
