# 2. 導入とセットアップ

## 2.1 システム要件

### 対応OS

Claude Codeは、主要なオペレーティングシステムで動作します：

**macOS**
- バージョン: 10.15 (Catalina) 以上
- アーキテクチャ: Intel/Apple Silicon両対応

**Linux**
- Ubuntu 20.04 以上
- Debian 10 以上
- その他の主要ディストリビューション（要Node.js環境）

**Windows**
- Windows 10/11（WSL経由での利用推奨）
- PowerShellまたはWSL2環境

### 必要な環境

**必須要件**
1. **Node.js v18以上**
   ```bash
   # バージョン確認
   node --version
   # v18.0.0以上であることを確認
   ```

2. **npm（Node Package Manager）**
   ```bash
   # バージョン確認
   npm --version
   ```

3. **インターネット接続**
   - APIアクセスのため常時接続必要
   - プロキシ環境の場合は適切な設定が必要

**推奨環境**
- メモリ: 8GB以上（大規模プロジェクトでは16GB推奨）
- ストレージ: 1GB以上の空き容量
- ターミナル: iTerm2（macOS）、Windows Terminal（Windows）

### 前提条件

**1. Anthropic Consoleアカウント**
- [console.anthropic.com](https://console.anthropic.com)でアカウント作成
- 有効な課金設定（クレジットカード登録）
- APIアクセスの有効化

**2. 代替オプション: Claude ProまたはMaxサブスクリプション**
- Webインターフェースとの統合利用が可能
- 追加のAPI設定不要

## 2.2 インストール手順

### npmによるインストール

**1. グローバルインストール**
```bash
# Claude Codeをグローバルにインストール
npm install -g @anthropic-ai/claude-code

# インストール確認
claude --version
```

⚠️ **重要**: `sudo npm install -g`は使用しないでください。権限問題やセキュリティリスクの原因となります。

**2. npmの権限問題を解決する方法**

もし権限エラーが発生した場合：

```bash
# npmのグローバルディレクトリを変更
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# パスを追加（.bashrc/.zshrcに追記）
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# 再度インストール
npm install -g @anthropic-ai/claude-code
```

### 認証設定

**1. Claude Codeの起動**
```bash
# プロジェクトディレクトリに移動
cd ~/your-project

# Claude Codeを起動
claude
```

**2. 認証プロセス**
初回起動時に認証を求められます：

```
Welcome to Claude Code!
Please choose your authentication method:
1. Anthropic Console (Default)
2. Claude Pro/Max subscription

Select [1]:
```

**3. OAuthフロー**
- ブラウザが自動的に開きます
- Anthropicアカウントでログイン
- アクセス許可を承認
- ターミナルに戻ると認証完了

### 初期設定

**1. 設定ファイルの確認**
```bash
# 設定ファイルの場所
cat ~/.claude/config.json
```

**2. プロジェクト固有の設定（CLAUDE.md）**
```bash
# プロジェクトルートにCLAUDE.mdを作成
cat > CLAUDE.md << 'EOF'
# プロジェクト設定

## コーディング規約
- インデント: スペース2つ
- 命名規則: camelCase
- コメント: 日本語OK

## 使用ライブラリ
- React 18
- TypeScript 5
- Next.js 14

## プロジェクト構造
- src/: ソースコード
- tests/: テストファイル
- docs/: ドキュメント
EOF
```

**3. グローバル設定（~/.claude/CLAUDE.md）**
```bash
# 全プロジェクト共通の設定
cat > ~/.claude/CLAUDE.md << 'EOF'
# グローバル設定

## 基本設定
- 言語: 日本語で応答
- タイムゾーン: Asia/Tokyo
- エディタ: VS Code

## 開発環境
- ターミナル: zsh
- パッケージマネージャー: npm
EOF
```

## 2.3 トラブルシューティング

### よくある問題と対処法

**1. Node.jsバージョンエラー**
```bash
# エラー: Node.js version 16.x.x is not supported
# 解決法: Node.jsをアップデート

# macOS (Homebrew使用)
brew update
brew upgrade node

# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# nvm使用の場合
nvm install 18
nvm use 18
```

**2. 認証エラー**
```bash
# エラー: Authentication failed
# 解決法: 認証情報をリセット

# 認証情報をクリア
rm -rf ~/.claude/auth.json

# 再認証
claude auth
```

**3. プロキシ環境での接続問題**
```bash
# プロキシ設定
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# npmのプロキシ設定
npm config set proxy http://proxy.example.com:8080
npm config set https-proxy http://proxy.example.com:8080
```

**4. WSL特有の問題（Windows）**
```bash
# WSL2のインストール
wsl --install

# Ubuntuのインストール
wsl --install -d Ubuntu-22.04

# WSL内でNode.jsをセットアップ
sudo apt update
sudo apt install nodejs npm
```

### ヘルスチェック

**1. 診断コマンド**
```bash
# システム診断を実行
claude /doctor

# 出力例：
✓ Node.js version: v18.17.0
✓ npm version: 9.8.1
✓ Authentication: Valid
✓ Internet connection: OK
✓ API endpoint: Reachable
✓ Local storage: 856 MB available
```

**2. ログの確認**
```bash
# デバッグログの有効化
export CLAUDE_DEBUG=true

# ログファイルの確認
tail -f ~/.claude/logs/debug.log
```

**3. 一般的な解決手順**
1. **キャッシュのクリア**
   ```bash
   claude cache clear
   ```

2. **設定のリセット**
   ```bash
   claude config reset
   ```

3. **再インストール**
   ```bash
   npm uninstall -g @anthropic-ai/claude-code
   npm install -g @anthropic-ai/claude-code
   ```

**4. サポートリソース**
- 公式ドキュメント: [docs.anthropic.com/claude-code](https://docs.anthropic.com/en/docs/claude-code/overview)
- GitHub Issues: [github.com/anthropics/claude-code/issues](https://github.com/anthropics/claude-code/issues)
- コミュニティフォーラム: Anthropic Discord

---

セットアップが完了したら、次章では Claude Code の基本的な使い方について解説します。ターミナルでの操作方法から、プロジェクト理解機能、コード生成まで、実践的な使用方法を学んでいきましょう。