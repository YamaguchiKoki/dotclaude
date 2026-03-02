# ts-issue-ai-fix: AI レビュー指摘の修正

## 役割

あなたはコーダーです。`ai_review` が出力した `ai-review.md` の指摘を修正します。

## 前提確認

`{report:ai-review.md}` を読み込み、指摘事項をすべて把握してください。

## 手順

### 1. 指摘事項の確認

`ai-review.md` に記載された全指摘事項を読み、優先度（Critical → High → Medium → Low）の順に修正計画を立ててください。

### 2. 修正実施

各指摘事項に対して修正を実施してください：

- フォールバック・デフォルト値の問題 → Fail Fast に書き直す
- 未使用コード・エクスポートの残存 → 削除する
- 後方互換性の無断追加 → 削除する
- 過剰な防御的コーディング → 必要最小限に絞る

### 3. 修正確認

修正後、ビルドとテストが通ることを確認してください：

```bash
npx tsc --noEmit
npm test
```

### 4. コミット

修正内容をコミットしてください：

```bash
# issue番号は {report:issue-plan.md} から取得する
git commit -m "fix: #番号 AI レビュー指摘の修正"
```

## 出力

`{report:commit-log.md}` にコミット一覧を出力してください（フォーマット: `ts-issue-commit-log`）。

## 遷移ルール

- 修正完了 → `reviewers`
- 修正不能（根本的な設計問題など） → `ABORT`
