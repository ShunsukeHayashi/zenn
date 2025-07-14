---
title: "【完全解説】Claude Codeカスタムコマンド作成術！定型作業を3分→30秒に短縮"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "カスタムコマンド", "自動化", "ai", "dx"]
published: true
---

前回の記事で「Claude CodeでCI/CDを爆速構築した」という話をしたところ、「具体的にはどうやってカスタムコマンドを作るの？」というコメントをたくさんいただきました。

確かに、最初のカスタムコマンド設定で挫折しちゃう人も多いんですよね。私も実は、初日に5回も設定ミスで失敗しました（汗）

今日は、実際に私がハマったポイントと、その解決法も含めて、Claude Codeのカスタムコマンド作成を分かりやすく解説します！

## ✅ 【事前チェック】あなたの環境でカスタムコマンドは使える？

まず、以下の項目を確認してください：

- [ ] Claude Code CLI がインストール済み（`claude-code --version`で確認）
- [ ] プロジェクトディレクトリで`claude-code init`実行済み
- [ ] `.claude/`ディレクトリが作成されている
- [ ] エディタでコマンドパレットが使える（VS Code推奨）

**💡 ポイント**: Claude Codeのバージョンは0.1.0以上が必要です。古いバージョンではカスタムコマンドが使えません。

## 🛠️ 【実践編】爆速カスタムコマンド作成手順

### Step 1: カスタムコマンドの基本構造を理解する

Claude Codeのカスタムコマンドは、`.claude/commands/`ディレクトリに配置します：

```bash
# ディレクトリ構造
.claude/
├── commands/
│   ├── test-all.yaml
│   ├── deploy-staging.yaml
│   └── generate-docs.yaml
└── config.yaml
```

### Step 2: 最初のカスタムコマンドを作成

まずは簡単な「全テスト実行コマンド」から始めましょう：

```yaml
# .claude/commands/test-all.yaml
name: test-all
description: "すべてのテストを並列実行"
prompt: |
  以下のタスクを実行してください：
  1. npm testを実行
  2. npm run test:e2eを実行
  3. npm run lintを実行
  4. すべての結果をまとめて報告
  
  エラーがあれば、修正方法も提案してください。
tags:
  - testing
  - quality
shortcuts:
  - "ta"
  - "test"
```

### Step 3: コマンドを実行する

作成したカスタムコマンドは3つの方法で実行できます：

```bash
# 方法1: CLIから直接実行
claude-code run test-all

# 方法2: ショートカットを使用
claude-code run ta

# 方法3: インタラクティブモード
claude-code
> /test-all
```

## 💡 実践的なカスタムコマンド例

### 例1: コンポーネント自動生成コマンド

```yaml
# .claude/commands/create-component.yaml
name: create-component
description: "Reactコンポーネントを自動生成"
parameters:
  - name: component_name
    description: "コンポーネント名"
    required: true
  - name: type
    description: "コンポーネントタイプ（functional/class）"
    default: "functional"
prompt: |
  {{component_name}}という名前のReactコンポーネントを作成してください。
  
  要件：
  - {{type}}コンポーネントとして作成
  - TypeScriptを使用
  - テストファイルも同時に作成
  - Storybookファイルも作成
  - 適切なディレクトリに配置
  
  既存のコンポーネント構造に従ってください。
```

使用例：
```bash
claude-code run create-component --component_name UserProfile --type functional
```

### 例2: データベーススキーマ更新コマンド

```yaml
# .claude/commands/update-schema.yaml
name: update-schema
description: "DBスキーマを更新してマイグレーション作成"
prompt: |
  以下の手順でデータベーススキーマを更新：
  
  1. prisma/schema.prismaを確認
  2. 必要な変更を加える
  3. prisma formatを実行
  4. prisma migrate dev --name {{migration_name}}を実行
  5. 生成された型定義を確認
  6. 関連するAPIエンドポイントも更新
parameters:
  - name: migration_name
    description: "マイグレーション名"
    required: true
```

### 例3: パフォーマンス分析コマンド

```yaml
# .claude/commands/analyze-performance.yaml
name: analyze-performance
description: "アプリケーションのパフォーマンスを分析"
prompt: |
  パフォーマンス分析を実行：
  
  1. Lighthouseレポートを生成
  2. bundle-analyzerでバンドルサイズを確認
  3. 重いコンポーネントを特定
  4. 最適化の提案を作成
  
  分析結果は/reports/performance/に保存してください。
environment:
  - NODE_ENV=production
```

## 🚨 トラブルシューティング：私がハマった問題と解決法

### 問題1: コマンドが認識されない
**原因**: YAMLファイルの構文エラー
**解決法**: 
```bash
# YAMLの検証
claude-code validate commands/test-all.yaml
```

### 問題2: パラメータが渡されない
**原因**: プロンプト内の変数記法が間違っている
**解決法**: 必ず`{{parameter_name}}`の形式を使用

### 問題3: コマンドの実行が遅い
**原因**: プロンプトが複雑すぎる
**解決法**: タスクを分割して、複数の小さなコマンドに分ける

## 🎯 高度なカスタムコマンド技法

### 1. コマンドチェーン

```yaml
# .claude/commands/full-deploy.yaml
name: full-deploy
description: "フルデプロイメントプロセス"
chain:
  - command: test-all
  - command: build-production
  - command: deploy-staging
    condition: "previous_success"
  - command: run-e2e-staging
  - command: deploy-production
    condition: "manual_approval"
```

### 2. 条件付き実行

```yaml
# .claude/commands/smart-test.yaml
name: smart-test
description: "変更されたファイルに応じてテストを実行"
prompt: |
  git diffで変更されたファイルを確認し：
  - *.tsxファイルが変更されていたら、コンポーネントテストを実行
  - *.tsファイルが変更されていたら、ユニットテストを実行
  - *.cssファイルが変更されていたら、ビジュアルリグレッションテストを実行
```

### 3. 外部ツール連携

```yaml
# .claude/commands/create-issue.yaml
name: create-issue
description: "GitHub Issueを自動作成"
prompt: |
  以下の情報でGitHub Issueを作成：
  
  タイトル: {{title}}
  説明: {{description}}
  
  追加で：
  - 関連するコードを分析してラベルを自動付与
  - 類似のIssueがないか確認
  - プロジェクトボードに自動追加
external_tools:
  - github-cli
parameters:
  - name: title
    required: true
  - name: description
    required: true
```

## ✅ 動作確認のチェックリスト

カスタムコマンドを作成したら、以下を確認：

- [ ] `claude-code list`でコマンドが表示される
- [ ] ショートカットが正しく動作する
- [ ] パラメータが正しく渡される
- [ ] エラーハンドリングが適切
- [ ] 実行時間が許容範囲内

## 🎓 カスタムコマンドを極めるために

Claude Codeのカスタムコマンドを最大限活用するには、**プロンプトエンジニアリングの基礎知識**が重要です。

**[プロンプトエンジニアリングガイド](https://shusukes-organization.gitbook.io/shunsukepuronputodezain/)**で以下を学習してください：
- 効果的なプロンプト構造の設計
- コンテキスト情報の最適な伝え方
- タスク分解とチェーン化の技法

理論 × 実践でClaude Codeスキルを飛躍的に向上させましょう！

## 🚀 次のステップ

カスタムコマンドをマスターしたら、次は：

1. **チーム共有**: `.claude/commands/`をGitで管理
2. **ライブラリ化**: 汎用的なコマンドをnpmパッケージ化
3. **CI/CD統合**: GitHub Actionsからカスタムコマンドを実行

みなさんのクリエイティブなカスタムコマンドもぜひシェアしてください！

---

**関連記事**：
- [【実体験】Claude Code×GitHub ActionsでCI/CD自動化！](https://zenn.dev/shunsukehayashi/articles/claude-code-github-actions-cicd)
- [【衝撃】Claude Codeメモリ管理で品質爆上げ！](https://zenn.dev/shunsukehayashi/articles/claude-code-memory-management)