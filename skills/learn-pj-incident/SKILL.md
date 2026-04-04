---
name: learn-pj-incident
description: 07-03 障害対応シミュレーション。Issue 起票 → 原因調査 → 修正 → Risk 登録 → 振り返りの全サイクルを体験する。
---

# /learn-pj-incident — 障害対応シミュレーション（07-03）

障害発生から解決・再発防止までのインシデント対応サイクルを体験する演習です。

**所要時間**: 実装パス 約 90 分 / シミュレーションパス 約 45 分
**前提**: `/learn-pj-execute`（07-02）完了済み（または進行中）
**スキル対応**: 統合演習（Spec Eng. + AI Collab 重点）

---

## Step 1: 導入

```
「障害対応シミュレーションです。

実際のプロジェクトでは、デプロイ後に予期しない問題が発生することがあります。
この演習では、障害発生から解決・再発防止までの全サイクルを ProjSight で管理しながら体験します。

使うスキル:
- Spec Eng. — Issue ライフサイクル管理（04-05）
- AI Collab — AI デバッグ（05-03）
- Intent Eng. — リスクの振り返りと登録（04-03）」
```

---

## Step 2: 障害シナリオの提示

### パス判定（AI ファシリテータ向け）

まず、受講者の状況を確認して **実装パス** か **シミュレーションパス** かを決定する:

- **実装パス**: 07-02 で実装済みのコードがある場合。実際にバグを仕込み、修正まで行う。よりリアルな体験になる。
- **シミュレーションパス**: 07-02 が未完了、または実装済みコードがない場合。後述のサンプル素材を使い、MCP 操作と情報構造化の机上演習を行う。

受講者に確認する:

```
「まず確認させてください。07-02 で実装したコードはありますか？
- ある場合 → そのコードに対する障害を体験します（実装パス）
- ない場合 → 提供するサンプル素材を使って演習します（シミュレーションパス）」
```

### シナリオ選択

07-02 の実装内容や受講者のレベルに応じて、以下のシナリオから適切なものを選ぶ:

| シナリオ | 内容                                                             | 難易度 |
| -------- | ---------------------------------------------------------------- | ------ |
| **A**    | 特定の入力でエラーが発生するとユーザーから報告                   | 低〜中 |
| **B**    | CI が突然失敗し始めた。依存ライブラリのバージョン不整合の疑い    | 中     |
| **C**    | パフォーマンスが劣化し、レスポンスが許容範囲を超えるようになった | 中〜高 |

### シミュレーションパス用サンプル素材

シミュレーションパスの場合、以下の素材を受講者に提示する。受講者はこの情報を構造化して Issue に記録し、AI にデバッグを依頼する練習を行う。

<details>
<summary>シナリオ A: 入力エラー</summary>

**ユーザー報告**: 「タスクのタイトルに絵文字を含めると保存時にエラーになります」

**エラーログ**:

```
2026-04-04T10:23:45.123Z ERROR [web-api] ValidationError: Invalid character in title field
    at validateInput (/src/handlers/task.ts:42)
    at createTask (/src/handlers/task.ts:18)
    at Router.handle (/node_modules/express/lib/router.js:234)
Request body: { "title": "🚀 新機能テスト", "deliverableId": "del-001" }
Status: 400 Bad Request
```

**問題のあるコード**:

```typescript
// src/handlers/task.ts
function validateInput(title: string): void {
  const VALID_TITLE = /^[a-zA-Z0-9\s\-_]+$/;
  if (!VALID_TITLE.test(title)) {
    throw new ValidationError('Invalid character in title field');
  }
}
```

</details>

<details>
<summary>シナリオ B: CI 失敗</summary>

**状況**: 昨日まで通っていた CI が今朝から失敗している。コードの変更はしていない。

**CI ログ**:

```
npm warn deprecated uuid@3.4.0: Please upgrade to v7 or higher
npm ERR! code ERESOLVE
npm ERR! ERESOLVE unable to resolve dependency tree
npm ERR! Found: react@19.0.0
npm ERR! node_modules/react
npm ERR!   react@"^19.0.0" from the root project
npm ERR!   peer react@"^18.0.0" from some-ui-lib@2.1.0
npm ERR!   node_modules/some-ui-lib
npm ERR!     some-ui-lib@"^2.0.0" from the root project
npm ERR! Fix the upstream dependency conflict
```

**問題のある package.json（抜粋）**:

```json
{
  "dependencies": {
    "react": "^19.0.0",
    "some-ui-lib": "^2.0.0"
  }
}
```

</details>

<details>
<summary>シナリオ C: パフォーマンス劣化</summary>

**監視アラート**: 「API レスポンスタイム p95 が 5 秒を超過（閾値: 2 秒）」

**パフォーマンスログ**:

```
2026-04-04T09:00:12Z INFO  [api] GET /api/tasks?projectId=proj-001 200 4823ms
2026-04-04T09:00:15Z INFO  [api] GET /api/tasks?projectId=proj-001 200 5102ms
2026-04-04T09:01:22Z INFO  [api] GET /api/tasks?projectId=proj-002 200 312ms
2026-04-04T09:02:45Z INFO  [api] GET /api/tasks?projectId=proj-001 200 6201ms
2026-04-04T09:03:00Z WARN  [db] DynamoDB consumed capacity: 847 RCU (proj-001)
2026-04-04T09:03:00Z WARN  [db] DynamoDB consumed capacity: 12 RCU (proj-002)
```

**問題のあるコード**:

```typescript
// src/handlers/listTasks.ts
async function listTasks(projectId: string) {
  const allEntities = await db.query({
    pk: `PROJECT#${projectId}`,
  });
  // フィルタリングをアプリ側で実行（SK条件なし）
  return allEntities.filter((e) => e._type === 'task');
}
```

</details>

### 問いかけ

```
「以下の障害が発生したと仮定しましょう:

{選択したシナリオの詳細を提示。シミュレーションパスの場合はサンプル素材を含める}

さて、最初に何をしますか？」
```

受講者の対応を見守り、以下のフローに沿って誘導する（ただし最小限の介入に留める）。

> **初学者フォールバック**: 受講者が 10 秒以上回答に詰まった場合、以下の段階的ヒントを出す:
>
> 1. 「ヒント: 障害が起きたとき、まず何を記録に残すべきでしょうか？」
> 2. それでも詰まる場合: 「選択肢で考えてみましょう。a) Issue を起票して問題を記録する b) すぐにコードを修正する c) 上司に報告する — どれが最初のステップとして適切ですか？」
> 3. 回答後: 「そうですね。まず Issue として記録を残すことで、対応の追跡が可能になります。では起票してみましょう。」

---

## Step 3: Issue 起票

```
「まず障害を Issue として記録しましょう。
04-05 で学んだように、upsert_issue で起票してください。
category、severity、description を適切に設定しましょう。」
```

受講者が `upsert_issue` で起票する。以下を含むよう誘導:

- **category**: `bug` / `tech-debt` / `limitation` / `documentation` / `other`
- **severity**: 整数 1〜5 で指定（以下の基準を参考に選択）

| severity | レベル    | 基準                                       | 例                                     |
| -------- | --------- | ------------------------------------------ | -------------------------------------- |
| **5**    | very high | サービス全体が停止、またはデータ損失の恐れ | 全ユーザーがログインできない           |
| **4**    | high      | 主要機能が使えない                         | タスク作成が一切できない               |
| **3**    | medium    | 一部機能に障害があるが回避策がある         | 特定の入力でエラーになるが他の入力は可 |
| **2**    | low       | 軽微な不具合、表示崩れ等                   | UIの文字が一部はみ出す                 |
| **1**    | very low  | ほぼ影響なし                               | ログのフォーマットが不統一             |

- **description**: 問題の症状、影響範囲、再現手順（分かる範囲で）

---

## Step 4: 原因調査

```
「次に原因を調査しましょう。
05-03 で学んだように、AI にデバッグを依頼する際は情報を構造化して渡してください:
- エラーメッセージ（あれば）
- 再現手順
- 期待される動作と実際の動作
- 最近の変更内容」
```

### 実装パスの場合

受講者と AI が協力して実際のコードの原因を特定し、修正を実装する。

### シミュレーションパスの場合

受講者はサンプル素材の情報を構造化して AI に渡し、原因を分析する。分析結果は **Issue の description を更新して「原因分析」セクションを追記する**:

```
upsert_issue(issueId, description: "（既存の description）\n\n## 原因分析\n- 根本原因: ...\n- 影響範囲: ...\n- 修正方針: ...")
```

---

## Step 5: 修正タスクの作成と対応

```
「原因が特定できたら、修正タスクを作成しましょう。」
```

1. `upsert_task` で修正タスクを作成
2. Issue と紐づける: `create_link(sourceId: <taskId>, targetId: <issueId>, type: 'resolves')`
3. `start_work(taskId)` で修正に着手
4. 修正を実装（実装パス）、または修正方針を Issue の description に追記（シミュレーションパス）
5. `complete_work(taskId)` で完了
6. `upsert_issue(status: 'resolved')` で Issue をクローズ

---

## Step 6: 振り返りとリスク登録

```
「修正が完了しました。最後に再発防止を考えましょう。

この障害はどうすれば防げましたか？
リスクとして登録し、今後の対策を明記しましょう。」
```

1. `upsert_risk` で再発防止リスクを登録（mitigation に具体的な対策を記述する）
   - シナリオ A の例: 「入力バリデーションのテストケースに Unicode・絵文字を含める」
   - シナリオ B の例: 「package.json に依存ライブラリの互換バージョンを明記し、CI で依存解決チェックを追加する」
   - シナリオ C の例: 「DynamoDB クエリに SK 条件を追加し、フィルタ式でのフルスキャンを防止する。p95 レスポンスタイムの監視アラートを設定する」
2. Issue と紐づける: `create_link(sourceId: <riskId>, targetId: <issueId>, type: 'caused_by')`

---

## Step 7: 演習の振り返り

```
「障害対応シミュレーションお疲れさまでした！

インシデント対応のフロー:
1. Issue 起票 — 問題を正確に記録する（症状・影響・再現手順）
2. 原因調査 — AI に構造化された情報を渡してデバッグ
3. 修正タスク — Issue と紐づけて追跡可能にする
4. 振り返り — リスク登録で再発防止を仕組み化

ポイント:
- 障害対応中も ProjSight に記録を残すことで、後から振り返れる
- Issue → Task → Risk のリンクが『なぜこの修正が入ったか』の追跡を可能にする
- AI は原因調査の強力な助手だが、情報の構造化は人間の責務

次は /learn-pj-retrospect（07-04）でカリキュラム全体の振り返りを行いましょう。」
```

受講者のタスクを `complete_work(taskId)` で完了にする。
