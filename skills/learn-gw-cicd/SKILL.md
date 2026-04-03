---
name: learn-gw-cicd
description: 06-04 CI 結果の読み方。lint, typecheck, build の CI が失敗した場合の対応手順を体験する。
---

# /learn-gw-cicd — CI 結果の読み方（06-04）

CI（Continuous Integration）の結果を読み、失敗時の対応手順を体験する演習です。

**所要時間**: 約 30 分
**前提**: `/learn-gw-review`（06-03）完了推奨
**スキル対応**: Git Workflow（チーム開発フロー）

---

## Step 1: 導入

```
「CI は『自動化された品質チェック』です。

PR を作成すると、CI が自動的に以下をチェックします:
- lint — コードスタイル・構文エラー
- typecheck — 型の整合性
- build — ビルドが通るか

CI が失敗した PR はマージできません。
エラーメッセージを正しく読み、素早く修正する力が必要です。」
```

---

## Step 2: CI パイプラインの構成説明

典型的な CI パイプラインの構成を説明する:

```
「一般的な CI パイプライン:

┌─────────┐    ┌────────────┐    ┌─────────┐
│  lint   │ →  │ typecheck  │ →  │  build  │
└─────────┘    └────────────┘    └─────────┘
     ↓              ↓               ↓
  ESLint        tsc --noEmit      Next.js build
  Prettier                        esbuild

各ステップは前のステップが成功した場合のみ実行されます。
つまり lint で失敗すると、typecheck や build は実行されません。」
```

### ProjSight の CI 構成例

| ステップ  | コマンド            | チェック内容          |
| --------- | ------------------- | --------------------- |
| lint      | `pnpm lint`         | ESLint ルール違反     |
| typecheck | `pnpm typecheck`    | TypeScript 型エラー   |
| build     | `pnpm build`        | ビルドエラー          |
| format    | `pnpm format:check` | Prettier フォーマット |

---

## Step 3: エラーメッセージの読み方演習

各 CI ステップの典型的なエラーメッセージを提示し、読み方を学ぶ:

### lint エラー

```
「以下の lint エラーが出ました。原因と修正方法を考えてください:

error  'useState' is defined but never used  @typescript-eslint/no-unused-vars
error  Unexpected console statement           no-console

ヒント: エラーメッセージの構造は:
[重要度] [メッセージ] [ルール名]
ルール名で検索すると、ルールの詳細と修正方法が分かります。」
```

修正方法: 未使用の import を削除、console 文を削除またはロガーに置き換え。

### typecheck エラー

```
「以下の型エラーが出ました:

error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
  src/utils/calc.ts:15:10

ヒント: エラーメッセージの構造は:
error [エラーコード]: [メッセージ]
  [ファイルパス]:[行番号]:[列番号]

TS2345 は『型の不一致』を示すエラーコードです。」
```

### build エラー

```
「ビルドエラーの多くは以下のカテゴリに分かれます:
1. import/export エラー — モジュールが見つからない
2. 型エラー — typecheck で検出されるが build 時にも出る
3. 環境変数エラー — 必要な環境変数が未設定
4. 依存関係エラー — パッケージのバージョン不整合」
```

---

## Step 4: 修正の実践

受講者に以下の流れを実践してもらう:

1. **ローカルで CI と同等のチェックを実行**:

```bash
pnpm lint        # lint チェック
pnpm typecheck   # 型チェック
pnpm build       # ビルド
```

2. **エラーがあれば修正**: エラーメッセージに従って修正
3. **AI の活用**: エラーメッセージをそのまま AI に渡して修正を依頼する

```
「CI 失敗時の AI 活用テンプレート:

『以下の CI エラーが発生しました。修正してください。
エラーメッセージ: [全文を貼り付け]
該当ファイル: [ファイルパス]
変更の意図: [何をしようとしていたか]』

05-03（AI デバッグ）で学んだ構造化が活きる場面です。」
```

---

## Step 5: 振り返り

```
「CI の結果を読むポイント:

- エラーメッセージは上から順に読む — 最初のエラーが根本原因のことが多い
- ローカルで再現してから修正 — CI 上で試行錯誤しない
- AI にエラーメッセージを渡す時は全文を — 部分的だと誤診する
- push 前にローカルで CI と同等のチェックを実行する習慣をつける

Git Workflow セクションの総まとめ:
- 06-01: ブランチ戦略 — main を守り、短命ブランチで作業
- 06-02: PR 作成 — 構造化された description + ProjSight 紐づけ
- 06-03: レビュー対応 — 指摘への丁寧な対応 + マージ + complete_work
- 06-04: CI — エラーメッセージを読み、素早く修正

これで 01〜06 の全スキルセクションが完了です。
/learn で進捗を確認し、07 模擬プロジェクトに進みましょう。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
