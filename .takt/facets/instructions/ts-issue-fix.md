# ts-issue-fix: レビュー指摘の修正

## 役割

あなたはコーダーです。並列レビュアーの指摘を修正します。

## 前提確認

以下のレビュー結果をすべて読み込んでください：

- `.tmp/review-results/code-quality.md`: アーキテクチャ・コード品質の指摘（arch-review サブムーブメント出力）
- `.tmp/review-results/false-negative.md`: QA観点（エッジケース・エラーハンドリング・型安全性）の指摘（qa-review サブムーブメント出力）
- `.tmp/review-results/test-quality.md`: テスト品質・PBT 妥当性の指摘（testing-review サブムーブメント出力）

## 手順

### 1. 指摘事項の整理

全レビュー結果を読み、以下を整理してください：

- 各指摘の重要度（Critical / High / Medium / Low）
- 修正が必要な指摘のリスト
- 修正の優先順位（Critical → High → Medium → Low）

### 2. 修正実施

優先度の高い順に修正を実施してください：

**code-quality.md の指摘（アーキテクチャ・コード品質）**
- 層間依存の違反 → 修正する
- 命名規則の違反 → 修正する
- アーキテクチャパターンの逸脱 → 修正する

**false-negative.md の指摘（QA・型安全性）**
- エッジケースの未処理 → 処理を追加する
- エラーハンドリングの不足 → 実装する
- 型安全性の問題 → TypeScript の型を修正する

**test-quality.md の指摘（テスト品質・PBT）**
- テストカバレッジの不足 → テストを追加する
- PBT のプロパティが不十分 → fast-check のプロパティを改善する
- テストの独立性の問題 → テストを修正する

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
git commit -m "fix: #番号 レビュー指摘の修正"
```

## 出力

`{report:commit-log.md}` にコミット一覧を出力してください（フォーマット: `ts-issue-commit-log`）。

## 遷移ルール

- 修正完了 → `reviewers`
