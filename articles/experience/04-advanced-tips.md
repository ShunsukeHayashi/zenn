# 【上級編】Claude Codeの知られざる活用法｜同時並列作業とカスタムコマンドで生産性爆上げ

こんにちは！Claude Code使い始めて半年の田中です。

前回までの記事で基本的な使い方は紹介しましたが、今回は「もっと深く使いこなしたい」という方向けの上級テクニックをシェアします！

実は先月、複数の案件を同時並行で進める必要があって、「Claude Codeを2つ同時に動かせないかな？」と思ったのがきっかけでした。調べてみると、意外と知られていない便利機能がたくさんあったんです。

特に「Git worktree」と「カスタムスラッシュコマンド」は、知ってるか知らないかで作業効率が全然変わります！

## 発見のきっかけ：「2つのプロジェクトを同時に進めたい」

先月こんな状況になりました：

**同時進行案件**：
- **案件A**: ECサイトの新機能開発（フロントエンド）
- **案件B**: 既存システムのバグ修正（バックエンド）
- **期限**: どちらも来週まで（汗）

普通なら切り替えながら作業するところですが、「Claude Code使ってるんだから、2つ同時に動かせないかな？」と思ったんです。

でも、単純に2つのターミナルでClaude Codeを起動すると、コンテキストがぐちゃぐちゃに...

**そこで見つけたのが「Git worktree」でした。**

## Git worktree：同じリポジトリの「並行世界」を作る

### Git worktreeって何？

簡単に言うと、**同じリポジトリの複数のブランチを、別々のフォルダで同時に作業できる機能**です。

従来の方法：
```bash
git checkout feature-a  # ブランチ切り替え
# 作業...
git checkout bugfix-b   # また切り替え
# 作業...
```

Git worktreeの方法：
```bash
# 同時に両方のブランチで作業可能
ls
project-feature-a/  # feature-aブランチ
project-bugfix-b/   # bugfix-bブランチ
```

### 実際にやってみた

まず、メインプロジェクトから新しいworktreeを作成：

```bash
# 現在のプロジェクトディレクトリで
cd ~/work/my-project

# 新機能用のworktreeを作成
git worktree add ../my-project-feature-a -b feature-a

# バグ修正用のworktreeを作成  
git worktree add ../my-project-bugfix-b bugfix-123
```

結果、こんな構造になりました：
```
work/
├── my-project/          # 元のプロジェクト（mainブランチ）
├── my-project-feature-a/ # 新機能開発（feature-aブランチ）
└── my-project-bugfix-b/  # バグ修正（bugfix-123ブランチ）
```

### Claude Codeを並列実行

それぞれのディレクトリでClaude Codeを起動：

**ターミナル1（新機能開発）**：
```bash
cd ../my-project-feature-a
claude
> この決済機能のフロントエンド実装をお願いします
```

**ターミナル2（バグ修正）**：
```bash
cd ../my-project-bugfix-b  
claude
> APIのレスポンス遅延問題を調査してください
```

**結果**：
- 各Claude Codeセッションが独立したコンテキストを持つ
- ファイルの変更が互いに影響しない
- 切り替えのストレスなし！

### 実際の作業効率

**Before（ブランチ切り替え）**：
- 切り替え時に集中が途切れる
- Claude Codeのコンテキストがリセットされる
- 作業の流れが断続的

**After（Git worktree + 並列Claude Code）**：
- 同時並行で作業可能
- それぞれのコンテキストが保持される
- 作業の流れが連続的

**体感的な生産性**：約1.5倍向上しました！

## 管理のコツと注意点

### worktreeの管理コマンド

```bash
# 現在のworktreeを確認
git worktree list

# 不要になったworktreeを削除
git worktree remove ../my-project-feature-a

# ブランチマージ後の掃除
git worktree prune
```

### 注意点（実際にハマった）

1. **依存関係のインストール忘れ**
   ```bash
   # 各worktreeで忘れずに
   cd ../my-project-feature-a
   npm install  # 必須！
   ```

2. **環境変数の設定**
   ```bash
   # .envファイルも各worktreeにコピー
   cp .env ../my-project-feature-a/
   ```

3. **データベースの競合**
   - 開発用DBを共有する場合は注意
   - 可能ならworktree毎に別DBを使用

## カスタムスラッシュコマンド：よく使う指示を効率化

もう一つの発見が「カスタムスラッシュコマンド」です。

### きっかけ：同じ指示を何度も入力

毎回こんな長い指示を打ってました：

```bash
claude "このコードのパフォーマンスを分析して、
        具体的な最適化案を3つ提案してください。
        メモリ使用量、実行時間、ネットワーク負荷の
        観点で評価をお願いします。"
```

**「これ、短縮できないかな？」**

### プロジェクト共通コマンドの作成

`.claude/commands/`ディレクトリを作成：

```bash
mkdir -p .claude/commands
```

よく使う指示をMarkdownファイルで保存：

```bash
# パフォーマンス分析用
echo "このコードのパフォーマンスを分析して、具体的な最適化案を3つ提案してください。メモリ使用量、実行時間、ネットワーク負荷の観点で評価をお願いします。" > .claude/commands/optimize.md

# セキュリティレビュー用  
echo "このコードのセキュリティ脆弱性をレビューしてください。OWASP Top 10の観点で、具体的な修正案も含めて報告をお願いします。" > .claude/commands/security.md

# テスト生成用
echo "このコンポーネント/関数の包括的なテストケースを作成してください。正常系、異常系、エッジケースすべて含めてお願いします。" > .claude/commands/test.md
```

### 使い方

Claude Codeセッション内で：

```bash
# 長い指示の代わりに
> /project:optimize

# セキュリティレビューは
> /project:security  

# テスト作成は
> /project:test
```

**めちゃくちゃ楽！**

### 引数付きコマンドの作成

さらに便利な「引数付きコマンド」も作れます：

```bash
echo "問題 #$ARGUMENTS の修正をお願いします。以下の手順で進めてください：
1. チケットの内容を理解
2. 関連コードの特定  
3. 根本原因の分析
4. 修正案の実装
5. テストケースの追加" > .claude/commands/fix.md
```

使用例：
```bash
> /project:fix 123
# → "問題 #123 の修正をお願いします..."に展開
```

### 個人用コマンドも作成

プロジェクトを超えて使いたいコマンドは`~/.claude/commands/`に：

```bash
mkdir -p ~/.claude/commands

echo "このコードをリファクタリングしてください。可読性、保守性、パフォーマンスの観点で改善をお願いします。" > ~/.claude/commands/refactor.md
```

使用時：
```bash
> /user:refactor
```

## Unixコマンドとの連携：意外と強力

Claude Codeは実はUnixコマンドとしても使えます。

### パイプライン活用

```bash
# ログファイルの分析
cat error.log | claude -p 'このエラーログを分析して根本原因を特定してください'

# Gitの差分レビュー
git diff | claude -p 'この変更をレビューしてセキュリティ問題がないか確認してください'

# ビルドエラーの解析
npm run build 2>&1 | claude -p 'このビルドエラーの解決方法を教えてください'
```

### 検証プロセスに組み込み

`package.json`に追加：

```json
{
  "scripts": {
    "lint:claude": "claude -p 'このプロジェクトの変更点でタイポや命名の問題がないかチェックしてください'",
    "review:security": "claude -p 'セキュリティの観点でコードレビューをお願いします'"
  }
}
```

### 出力形式の制御

スクリプトに組み込む場合は出力形式を指定：

```bash
# プレーンテキスト（デフォルト）
claude -p 'この関数を説明して' --output-format text

# JSON形式（メタデータ付き）
claude -p 'バグを分析して' --output-format json

# ストリーミングJSON
claude -p 'ログを解析して' --output-format stream-json
```

## 実際の成果：数字で見る効果

これらのテクニックを1ヶ月使った結果：

### ⏰ 時間短縮効果
- **worktree並列作業**: 30%の時間短縮
- **カスタムコマンド**: 定型作業で50%短縮
- **パイプライン連携**: ログ分析で75%短縮

### 💡 副次効果
- **集中力向上**: 切り替えストレスの軽減
- **標準化**: チーム内でのコマンド共有
- **自動化**: CI/CDへの組み込み

## これから試す人へのアドバイス

### 段階的に導入しよう

1. **Week 1**: Git worktreeの基本を試す
2. **Week 2**: 簡単なカスタムコマンドを作成
3. **Week 3**: パイプライン連携を実験
4. **Week 4**: チームでの共有・標準化

### おすすめのカスタムコマンド（スターターセット）

```bash
# 基本的なレビューコマンド
.claude/commands/review.md
.claude/commands/optimize.md  
.claude/commands/test.md
.claude/commands/docs.md

# プロジェクト固有
.claude/commands/api-design.md
.claude/commands/component-check.md
```

### 注意点

1. **コマンド名は分かりやすく**: optimize ✅ opt ❌
2. **チームで命名規則を統一**: 混乱を避ける
3. **定期的にメンテナンス**: 不要なコマンドは削除

## よくある質問

**Q: Git worktreeは本当に安全？**
A: はい。同じGit履歴を共有するので、データが失われることはありません。ただし、ディスク容量は多く使います。

**Q: カスタムコマンドはどこまで複雑にできる？**
A: Markdownファイルなので、かなり詳細な指示も可能です。ただし、あまり長すぎると逆に使いにくくなります。

**Q: チーム全体で使う場合の注意点は？**  
A: `.claude/commands/`をGitにコミットすることで共有できます。命名規則は事前に決めておきましょう。

## 次回予告

今回は上級テクニックをお話ししました。次回は：

「Claude Codeを使ったチーム開発の事例｜導入から定着までの3ヶ月間」

実際にチーム全体でClaude Codeを導入した体験談をお話しする予定です。

## まとめ

Claude Codeは単体でも強力ですが、今回紹介したテクニックで更に活用の幅が広がります：

- **Git worktree**: 並列作業で生産性向上
- **カスタムコマンド**: 定型作業の効率化
- **Unix連携**: 既存ワークフローへの統合

特に複数案件を抱えている方や、チームでの標準化を考えている方には、ぜひ試してもらいたいテクニックです。

みなさんの「こんな使い方もあるよ！」というアイデアもコメントで教えてください🚀

---

**この記事が役に立ったら**
❤️ スキ・コメント・シェアをお願いします！
💬 実際に試してみた感想もお聞かせください✨