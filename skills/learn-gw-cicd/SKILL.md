---
name: learn-gw-cicd
description: 06-04 CI 結果の読み方。lint, typecheck, build の CI が失敗した場合の対応手順を体験する。
---

# /learn-gw-cicd — CI 結果の読み方（06-04）

CI（Continuous Integration）の結果を読み、失敗時の対応手順を体験する演習です。

**所要時間**: 約 40 分
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
 (コード規約)   (型の整合性)      esbuild
                                  (成果物生成)

ESLint: コードの書き方ルール（未使用変数、console 禁止等）をチェックするツール
tsc --noEmit: TypeScript コンパイラで型だけを検証する（ファイルは出力しない）

各ステップは前のステップが成功した場合のみ実行されます。
つまり lint で失敗すると、typecheck や build は実行されません。

実際の CI 定義は `.github/workflows/ci.yml` にあります。
興味があれば確認してみましょう。」
```

### ProjSight の CI 構成例

| ステップ  | コマンド            | チェック内容          |
| --------- | ------------------- | --------------------- |
| lint      | `pnpm lint`         | ESLint ルール違反     |
| typecheck | `pnpm typecheck`    | TypeScript 型エラー   |
| build     | `pnpm build`        | ビルドエラー          |
| format    | `pnpm format:check` | Prettier フォーマット |

---

## Step 3: エラーを発生させる（サンドボックス）

実際にエラーを含むコードを作成し、CI と同じチェックで失敗を体験する。

```
「まず、意図的にエラーを含むファイルを作りましょう。
以下のコードを `learn-workspace/ci-practice.ts` に作成してください:」
```

受講者に以下のコードを `learn-workspace/ci-practice.ts` として作成させる:

```typescript
import { useState } from 'react';
import { useEffect } from 'react';

export function calculateTotal(price: number, quantity: number): number {
  const result: number = price * quantity;
  console.log('計算結果:', result);
  const tax: string = result * 0.1;
  return result + tax;
}

export function formatPrice(price: number) {
  return `¥${price.toLocaleString()}`;
}
```

このコードには以下のエラーが仕込まれている（受講者には伝えない）:

- 未使用の import（`useState`, `useEffect`）→ lint エラー
- `console.log` → lint エラー
- 型不一致（`string` に `number` を代入）→ typecheck エラー
- フォーマット崩れ（余分なスペース、セミコロン欠落）→ format エラー

### 3-1: lint エラーを体験する

```
「ファイルを作成したら、lint を実行しましょう:」
```

```bash
pnpm lint
```

エラーが出力されたら、受講者に質問する:

```
「lint エラーが表示されました。

エラーメッセージをよく読んで、以下に回答してください:
1. いくつのエラーが検出されましたか？
2. それぞれのエラーの原因は何ですか？
3. どう修正すればよいですか？

ヒント: エラーメッセージの構造は:
[重要度] [メッセージ] [ルール名]
ルール名で検索すると、ルールの詳細と修正方法が分かります。」
```

**受講者の回答を待つ。** 回答が来たらフィードバックを返す:

- 未使用 import（`@typescript-eslint/no-unused-vars`）→ 使わない import を削除
- console 文（`no-console`）→ console.log を削除またはロガーに置き換え
- 正しく読めていたら褒める。見落としがあれば該当行を指摘する

```
「修正してください。修正が終わったら再度 `pnpm lint` を実行して、エラーが消えたことを確認しましょう。」
```

### 3-2: typecheck エラーを体験する

lint が通ったら、次のステップへ:

```bash
pnpm typecheck
```

エラーが出力されたら、受講者に質問する:

```
「型エラーが表示されました。

エラーメッセージをよく読んで、以下に回答してください:
1. エラーコード（TS????）は何ですか？
2. どのファイルの何行目でエラーが出ていますか？
3. 原因と修正方法は何ですか？

ヒント: エラーメッセージの構造は:
error [エラーコード]: [メッセージ]
  [ファイルパス]:[行番号]:[列番号]

エラーコードで検索すると、詳細な説明が見つかります。」
```

**受講者の回答を待つ。** 回答が来たらフィードバックを返す:

- TS2322: `string` 型に `number` の計算結果を代入 → `const tax: number` に修正
- 正しく読めていたら褒める。エラーコードの意味を補足する

```
「修正してください。修正が終わったら再度 `pnpm typecheck` を実行して、エラーが消えたことを確認しましょう。」
```

### 3-3: format エラーを体験する

typecheck が通ったら、フォーマットチェックを実行:

```bash
pnpm format:check
```

エラーが出力されたら説明する:

```
「Prettier（コードフォーマッター）がスタイルの不一致を検出しました。

フォーマットエラーは手動で直す必要はありません。
以下のコマンドで自動修正できます:」
```

```bash
npx prettier --write learn-workspace/ci-practice.ts
```

```
「フォーマットエラーは他のエラーと違い、自動修正できるのが特徴です。
ただし CI 上では自動修正されないので、push 前にローカルで実行する習慣が大切です。」
```

---

## Step 4: build エラーの読み方

build エラーはサンドボックスファイルでは再現しにくいため、典型的な例で読み方を学ぶ:

```
「ビルドエラーの代表例を見てみましょう:

例1: import/export エラー
  Module not found: Can't resolve './Foo'
    at ./src/components/Bar.tsx:3:1

  読み方: [エラー種別]: [メッセージ]
           at [ファイルパス]:[行番号]:[列番号]

  原因: import 先のファイルパスが間違っている、またはファイルが存在しない
  修正: パスのタイポを確認、ファイル名の大文字小文字を確認

例2: 型エラー（typecheck で検出されるが build 時にも出る）
例3: 環境変数エラー — 必要な環境変数が未設定
例4: 依存関係エラー — パッケージのバージョン不整合

build エラーは原因が多岐にわたるため、エラーメッセージの最初の行を正確に読むことが最も重要です。」
```

### AI の活用

```
「CI 失敗時の AI 活用テンプレート:

『以下の CI エラーが発生しました。修正してください。
エラーメッセージ: [全文を貼り付け]
該当ファイル: [ファイルパス]
変更の意図: [何をしようとしていたか]』

05-03（AI デバッグ）で学んだ構造化が活きる場面です。」
```

---

## Step 5: クリーンアップ

演習用ファイルを削除する:

```bash
rm learn-workspace/ci-practice.ts
```

```
「サンドボックスファイルを削除しました。
実際のプロジェクトでも、デバッグ用の一時ファイルは作業後に必ず片付けましょう。」
```

---

## Step 6: 振り返り

```
「CI の結果を読むポイント:

- エラーメッセージは上から順に読む — 最初のエラーが根本原因のことが多い
- ローカルで再現してから修正 — CI 上で試行錯誤しない
- AI にエラーメッセージを渡す時は全文を — 部分的だと誤診する
- push 前にローカルで CI と同等のチェックを実行する習慣をつける
- フォーマットエラーは自動修正できる — 他のエラーとは性質が違う

Git Workflow セクションの総まとめ:
- 06-01: ブランチ戦略 — main を守り、短命ブランチで作業
- 06-02: PR 作成 — 構造化された description + ProjSight 紐づけ
- 06-03: レビュー対応 — 指摘への丁寧な対応 + マージ + complete_work
- 06-04: CI — エラーメッセージを読み、素早く修正

06 Git Workflow セクションが完了です。
/learn で進捗を確認し、次のセクションに進みましょう。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
