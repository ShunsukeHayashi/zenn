---
title: "【完全解説】Claude Code カスタムコマンド作成術！定型作業を3分→30秒に短縮"
emoji: "🚀"
type: "tech"
topics: ["claudecode", "カスタムコマンド", "自動化", "効率化", "チュートリアル"]
published: true
---

前回の記事で「Claude CodeでCI/CDが劇的に改善した」という話をしたところ、「日常的な定型作業も自動化できないの？」というコメントをたくさんいただきました。

確かに、開発現場では毎日同じような作業の繰り返しが多いですよね。私も以前は、コンポーネント作成、API設定、テストファイル生成など、似たような作業に毎日時間を取られていました。

今日は、実際に私が使っているClaude Codeカスタムコマンドシステムと、その驚くべき効果を具体的に解説します！

> **📝 注記**: 本記事は2024年12月時点でのClaude Code v2.1での手法です。コマンドシステムの仕様変更により、実装方法が変わる可能性があります。

## ✅ 【事前チェック】あなたの環境でカスタムコマンドは使える？

### ✅ 必要な環境
- **Claude Code**: v2.0以降（最新版推奨）
- **Node.js**: v18以上
- **Git**: バージョン管理用
- **VS Code**: エディタ連携用（オプション）

### ✅ 基本的なClaude Codeの理解
- 基本コマンドの実行経験
- CLAUDE.mdファイルの設定
- プロジェクト構造の理解

### ✅ 対象となる定型作業
以下のような作業がある場合、大幅な効率化が期待できます：
- Reactコンポーネントの新規作成
- APIエンドポイントの定型設定
- テストファイルの生成
- データベーススキーマの更新
- ドキュメント生成

**💡 初心者の方へ**: Claude Codeの基本操作に慣れてから、カスタムコマンドに挑戦することをお勧めします。

## 🛠️ 【実践編】爆速セットアップ手順

### Step 1: カスタムコマンド環境の準備（10分）

#### 1.1 コマンドディレクトリの作成
```bash
# プロジェクトルートに移動
cd your-project

# Claude Codeコマンドディレクトリを作成
mkdir -p .claude/commands
```

#### 1.2 基本設定ファイルの作成
```javascript
// .claude/commands/config.js
module.exports = {
  // コマンドの基本設定
  defaultTimeout: 30000,
  maxRetries: 3,
  
  // プロジェクト固有の設定
  projectType: 'react-typescript',
  testFramework: 'jest',
  styleFramework: 'styled-components',
  
  // ディレクトリ構造
  directories: {
    components: 'src/components',
    hooks: 'src/hooks',
    utils: 'src/utils',
    tests: 'src/__tests__',
    api: 'src/api',
    types: 'src/types'
  }
};
```

### Step 2: 基本的なカスタムコマンドの作成（15分）

#### 2.1 Reactコンポーネント生成コマンド
```javascript
// .claude/commands/create-component.js
const path = require('path');
const fs = require('fs').promises;
const config = require('./config');

module.exports = {
  name: 'create-component',
  description: 'Create a new React component with TypeScript',
  
  async execute(args) {
    const [componentName, ...options] = args;
    
    if (!componentName) {
      throw new Error('Component name is required');
    }
    
    const componentDir = path.join(config.directories.components, componentName);
    const testDir = config.directories.tests;
    
    // コンポーネントファイルの生成
    const componentTemplate = generateComponentTemplate(componentName, options);
    const testTemplate = generateTestTemplate(componentName);
    const indexTemplate = generateIndexTemplate(componentName);
    
    // ディレクトリ作成
    await fs.mkdir(componentDir, { recursive: true });
    
    // ファイル作成
    await Promise.all([
      fs.writeFile(path.join(componentDir, `${componentName}.tsx`), componentTemplate),
      fs.writeFile(path.join(componentDir, `${componentName}.stories.tsx`), generateStorybookTemplate(componentName)),
      fs.writeFile(path.join(componentDir, 'index.ts'), indexTemplate),
      fs.writeFile(path.join(testDir, `${componentName}.test.tsx`), testTemplate)
    ]);
    
    return {
      success: true,
      message: `Component ${componentName} created successfully`,
      files: [
        `${componentDir}/${componentName}.tsx`,
        `${componentDir}/${componentName}.stories.tsx`,
        `${componentDir}/index.ts`,
        `${testDir}/${componentName}.test.tsx`
      ]
    };
  }
};

function generateComponentTemplate(name, options) {
  const hasProps = options.includes('--props');
  const hasState = options.includes('--state');
  
  return `import React${hasState ? ', { useState }' : ''} from 'react';
import styled from 'styled-components';

${hasProps ? `interface ${name}Props {
  // TODO: Define props
}` : ''}

const ${name}: React.FC${hasProps ? `<${name}Props>` : ''} = (${hasProps ? 'props' : ''}) => {
  ${hasState ? 'const [state, setState] = useState();' : ''}

  return (
    <Container>
      <h1>${name} Component</h1>
      {/* TODO: Implement component */}
    </Container>
  );
};

const Container = styled.div\`
  /* TODO: Add styles */
\`;

export default ${name};
`;
}

function generateTestTemplate(name) {
  return `import React from 'react';
import { render, screen } from '@testing-library/react';
import ${name} from '../components/${name}';

describe('${name}', () => {
  it('renders without crashing', () => {
    render(<${name} />);
    expect(screen.getByText('${name} Component')).toBeInTheDocument();
  });
  
  // TODO: Add more tests
});
`;
}

function generateStorybookTemplate(name) {
  return `import type { Meta, StoryObj } from '@storybook/react';
import ${name} from './${name}';

const meta: Meta<typeof ${name}> = {
  title: 'Components/${name}',
  component: ${name},
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    // TODO: Add default args
  },
};
`;
}

function generateIndexTemplate(name) {
  return `export { default } from './${name}';
`;
}
```

#### 2.2 API作成コマンド
```javascript
// .claude/commands/create-api.js
const path = require('path');
const fs = require('fs').promises;
const config = require('./config');

module.exports = {
  name: 'create-api',
  description: 'Create API endpoint with types and tests',
  
  async execute(args) {
    const [endpointName, method = 'GET'] = args;
    
    if (!endpointName) {
      throw new Error('Endpoint name is required');
    }
    
    const apiDir = config.directories.api;
    const typesDir = config.directories.types;
    const testDir = config.directories.tests;
    
    // テンプレート生成
    const apiTemplate = generateApiTemplate(endpointName, method);
    const typeTemplate = generateTypeTemplate(endpointName);
    const testTemplate = generateApiTestTemplate(endpointName, method);
    
    // ファイル作成
    await Promise.all([
      fs.writeFile(path.join(apiDir, `${endpointName}.ts`), apiTemplate),
      fs.writeFile(path.join(typesDir, `${endpointName}.ts`), typeTemplate),
      fs.writeFile(path.join(testDir, `${endpointName}.api.test.ts`), testTemplate)
    ]);
    
    return {
      success: true,
      message: `API endpoint ${endpointName} created successfully`,
      files: [
        `${apiDir}/${endpointName}.ts`,
        `${typesDir}/${endpointName}.ts`,
        `${testDir}/${endpointName}.api.test.ts`
      ]
    };
  }
};

function generateApiTemplate(name, method) {
  return `import { ${name}Request, ${name}Response } from '../types/${name}';

const API_BASE = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:3000/api';

export const ${name}Api = {
  async ${method.toLowerCase()}(${method === 'GET' ? 'id?: string' : `data: ${name}Request`}): Promise<${name}Response> {
    const url = \`\${API_BASE}/${name.toLowerCase()}\${${method === 'GET' ? 'id ? `/${id}` : ""' : '""'}}\`;
    
    const response = await fetch(url, {
      method: '${method}',
      headers: {
        'Content-Type': 'application/json',
        // TODO: Add authentication headers if needed
      },
      ${method !== 'GET' ? 'body: JSON.stringify(data),' : ''}
    });
    
    if (!response.ok) {
      throw new Error(\`API Error: \${response.status}\`);
    }
    
    return response.json();
  }
};
`;
}

function generateTypeTemplate(name) {
  return `// ${name} API Types

export interface ${name}Request {
  // TODO: Define request interface
}

export interface ${name}Response {
  // TODO: Define response interface
  id: string;
  createdAt: string;
  updatedAt: string;
}

export interface ${name}Error {
  message: string;
  code: string;
}
`;
}

function generateApiTestTemplate(name, method) {
  return `import { ${name}Api } from '../api/${name}';

// Mock fetch
global.fetch = jest.fn();

describe('${name}Api', () => {
  beforeEach(() => {
    (fetch as jest.MockedFunction<typeof fetch>).mockClear();
  });
  
  it('should ${method.toLowerCase()} ${name} successfully', async () => {
    const mockResponse = {
      // TODO: Add mock response data
    };
    
    (fetch as jest.MockedFunction<typeof fetch>).mockResolvedValue({
      ok: true,
      json: async () => mockResponse,
    } as Response);
    
    ${method === 'GET' 
      ? `const result = await ${name}Api.get();`
      : `const result = await ${name}Api.${method.toLowerCase()}({ /* TODO: Add test data */ });`
    }
    
    expect(result).toEqual(mockResponse);
    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining('/${name.toLowerCase()}'),
      expect.objectContaining({
        method: '${method}',
      })
    );
  });
  
  // TODO: Add error handling tests
});
`;
}
```

### Step 3: コマンドランナーの設定（10分）

#### 3.1 メインランナーファイル
```javascript
// .claude/commands/runner.js
const fs = require('fs').promises;
const path = require('path');

class CommandRunner {
  constructor() {
    this.commands = new Map();
    this.loadCommands();
  }
  
  async loadCommands() {
    const commandsDir = path.join(__dirname);
    const files = await fs.readdir(commandsDir);
    
    for (const file of files) {
      if (file.endsWith('.js') && !['config.js', 'runner.js'].includes(file)) {
        try {
          const command = require(path.join(commandsDir, file));
          this.commands.set(command.name, command);
        } catch (error) {
          console.warn(`Failed to load command ${file}:`, error.message);
        }
      }
    }
  }
  
  async execute(commandName, args) {
    const command = this.commands.get(commandName);
    
    if (!command) {
      throw new Error(`Command '${commandName}' not found. Available commands: ${Array.from(this.commands.keys()).join(', ')}`);
    }
    
    try {
      console.log(`Executing command: ${commandName}`);
      const result = await command.execute(args);
      
      if (result.success) {
        console.log(`✅ ${result.message}`);
        if (result.files) {
          console.log('Created files:', result.files.join('\n'));
        }
      }
      
      return result;
    } catch (error) {
      console.error(`❌ Command failed: ${error.message}`);
      throw error;
    }
  }
  
  listCommands() {
    return Array.from(this.commands.entries()).map(([name, command]) => ({
      name,
      description: command.description || 'No description available'
    }));
  }
}

module.exports = new CommandRunner();
```

#### 3.2 Claude Code統合設定
```markdown
<!-- CLAUDE.mdに追加 -->

## カスタムコマンド

利用可能なコマンド：

### /create-component
Reactコンポーネントを作成します。

使用例：
```bash
/create-component Button --props --state
```

生成されるファイル：
- src/components/Button/Button.tsx
- src/components/Button/Button.stories.tsx
- src/components/Button/index.ts
- src/__tests__/Button.test.tsx

### /create-api
APIエンドポイントを作成します。

使用例：
```bash
/create-api users POST
```

生成されるファイル：
- src/api/users.ts
- src/types/users.ts
- src/__tests__/users.api.test.ts

### コマンド実行方法
```bash
# Claude Codeで以下のように実行
claude run .claude/commands/runner.js create-component ButtonComponent --props
```
```

## 🚨 トラブルシューティング：私がハマった問題と解決法

### 問題1: コマンドが認識されない

**症状**: `Command 'create-component' not found`
**原因**: ファイルパスの問題、またはモジュールエクスポートの設定ミス
**解決法**:
```bash
# ファイル構造の確認
ls -la .claude/commands/

# 権限の確認
chmod +x .claude/commands/*.js

# Node.jsでの直接テスト
node .claude/commands/runner.js create-component TestComponent
```

### 問題2: ファイル生成時の権限エラー

**症状**: `Permission denied` エラー
**原因**: ディレクトリの書き込み権限不足
**解決法**:
```bash
# ディレクトリ権限の確認・修正
chmod -R 755 src/
mkdir -p src/components src/__tests__ src/api src/types
```

### 問題3: テンプレートの文字化け

**症状**: 生成されたファイルが文字化けする
**原因**: 文字エンコーディングの問題
**解決法**:
```javascript
// ファイル書き込み時にエンコーディングを明示
await fs.writeFile(filepath, content, { encoding: 'utf8' });
```

### 問題4: Claude Codeとの連携エラー

**症状**: Claude Codeからコマンドが実行できない
**原因**: パスの問題またはNode.js環境の問題
**解決法**:
```bash
# パスの確認
which node
echo $PATH

# Claude Code設定の確認
claude config show

# 直接実行テスト
cd project-root
node .claude/commands/runner.js list
```

## ✅ 動作確認のチェックリスト

セットアップが完了したら、以下を順番に確認してください：

### 基本機能の確認
```bash
# ✅ コマンドランナーの動作確認
node .claude/commands/runner.js list

# ✅ コンポーネント作成テスト
node .claude/commands/runner.js create-component TestButton --props

# ✅ API作成テスト
node .claude/commands/runner.js create-api testUsers POST

# ✅ 生成されたファイルの確認
ls -la src/components/TestButton/
ls -la src/api/
ls -la src/__tests__/
```

### Claude Code連携の確認
```bash
# ✅ Claude Codeでのコマンド実行
claude "コマンドランナーを使ってUserProfileコンポーネントを作成して"

# ✅ 生成されたコードの品質確認
npm run lint
npm run type-check
npm test
```

## 📊 【実践例】実際の作業時間短縮効果

### Before（手動作業）vs After（カスタムコマンド）

#### Reactコンポーネント作成
```
Before:
- ディレクトリ作成: 30秒
- コンポーネントファイル作成: 2分
- テストファイル作成: 1分30秒
- Storybookファイル作成: 1分
- インデックスファイル作成: 30秒
合計: 5分30秒

After:
- コマンド実行: 10秒
- 生成ファイル確認: 20秒
合計: 30秒

短縮率: 91%
```

#### API エンドポイント作成
```
Before:
- APIファイル作成: 3分
- 型定義作成: 2分
- テストファイル作成: 2分30秒
- ドキュメント更新: 1分
合計: 8分30秒

After:
- コマンド実行: 15秒
- 生成コード確認・調整: 45秒
合計: 1分

短縮率: 88%
```

### 3ヶ月間の累積効果

| 作業項目 | 頻度/月 | Before時間 | After時間 | 月間削減時間 |
|----------|---------|------------|-----------|-------------|
| コンポーネント作成 | 20回 | 110分 | 10分 | **100分削減** |
| API作成 | 15回 | 128分 | 15分 | **113分削減** |
| テストファイル生成 | 25回 | 75分 | 10分 | **65分削減** |
| ドキュメント生成 | 10回 | 50分 | 5分 | **45分削減** |

**月間合計削減時間: 323分（5時間23分）**
**年間削減時間: 64時間**

## 🚀 高度なカスタムコマンド例

### データベースマイグレーション自動化

```javascript
// .claude/commands/create-migration.js
module.exports = {
  name: 'create-migration',
  description: 'Create database migration with TypeScript types',
  
  async execute(args) {
    const [tableName, action = 'create'] = args;
    
    const migrationTemplate = `
import { Knex } from 'knex';

export async function up(knex: Knex): Promise<void> {
  return knex.schema.${action}Table('${tableName}', (table) => {
    table.uuid('id').primary().defaultTo(knex.raw('gen_random_uuid()'));
    table.timestamps(true, true);
    
    // TODO: Add specific columns
  });
}

export async function down(knex: Knex): Promise<void> {
  return knex.schema.dropTableIfExists('${tableName}');
}
`;
    
    const typeTemplate = `
export interface ${tableName.charAt(0).toUpperCase() + tableName.slice(1)} {
  id: string;
  createdAt: Date;
  updatedAt: Date;
  // TODO: Add specific fields
}
`;
    
    // ファイル生成処理...
  }
};
```

### 環境設定自動化

```javascript
// .claude/commands/setup-env.js
module.exports = {
  name: 'setup-env',
  description: 'Setup environment configuration for different stages',
  
  async execute(args) {
    const [environment = 'development'] = args;
    
    const envTemplates = {
      development: `
# Development Environment
NODE_ENV=development
API_URL=http://localhost:3001
DATABASE_URL=postgresql://localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379
DEBUG=myapp:*
`,
      staging: `
# Staging Environment
NODE_ENV=staging
API_URL=https://api-staging.myapp.com
DATABASE_URL=\${STAGING_DATABASE_URL}
REDIS_URL=\${STAGING_REDIS_URL}
`,
      production: `
# Production Environment
NODE_ENV=production
API_URL=https://api.myapp.com
DATABASE_URL=\${DATABASE_URL}
REDIS_URL=\${REDIS_URL}
`
    };
    
    // 環境別設定ファイル生成...
  }
};
```

## 💡 実践的な運用テクニック

### チーム共有のベストプラクティス

#### 1. コマンドのバージョン管理
```bash
# .claude/commands/package.json
{
  "name": "project-custom-commands",
  "version": "1.0.0",
  "description": "Custom Claude Code commands for this project",
  "scripts": {
    "test": "jest",
    "lint": "eslint *.js"
  }
}
```

#### 2. ドキュメント自動生成
```javascript
// .claude/commands/generate-docs.js
module.exports = {
  name: 'generate-docs',
  description: 'Generate documentation for all custom commands',
  
  async execute() {
    const runner = require('./runner');
    const commands = runner.listCommands();
    
    const docsContent = `# Custom Commands Documentation

## Available Commands

${commands.map(cmd => `
### ${cmd.name}
${cmd.description}

Usage: \`claude run .claude/commands/runner.js ${cmd.name} [args]\`
`).join('\n')}
`;
    
    await fs.writeFile('docs/custom-commands.md', docsContent);
    return { success: true, message: 'Documentation generated' };
  }
};
```

#### 3. コマンド実行ログ
```javascript
// ログ機能付きランナー
class CommandRunner {
  async execute(commandName, args) {
    const startTime = Date.now();
    
    try {
      const result = await command.execute(args);
      const duration = Date.now() - startTime;
      
      this.logExecution({
        command: commandName,
        args,
        duration,
        success: true,
        timestamp: new Date().toISOString()
      });
      
      return result;
    } catch (error) {
      this.logExecution({
        command: commandName,
        args,
        duration: Date.now() - startTime,
        success: false,
        error: error.message,
        timestamp: new Date().toISOString()
      });
      throw error;
    }
  }
  
  logExecution(data) {
    const logFile = '.claude/commands/execution.log';
    fs.appendFileSync(logFile, JSON.stringify(data) + '\n');
  }
}
```

## 🎓 プロンプトエンジニアリング理論も学習しよう

Claude Codeカスタムコマンドを最大限活用するには、**プロンプトエンジニアリングの基礎知識**が重要です。

**[プロンプトエンジニアリングガイド](https://shusukes-organization.gitbook.io/shunsukepuronputodezain/)**で以下を学習してください：
- 効果的なコード生成プロンプトの設計理論
- テンプレート自動化のためのプロンプトパターン
- エラーハンドリングと品質保証の手法

理論 × 実践でClaude Codeカスタムコマンドスキルを飛躍的に向上させましょう！

## よくある質問

**Q: 「既存のプロジェクトにも適用できますか？」**
A: はい、既存プロジェクトでも使用可能です。ただし、プロジェクトの構造やコーディング規約に合わせてテンプレートをカスタマイズする必要があります。

**Q: 「チーム全体で共有する方法は？」**
A: `.claude/commands`ディレクトリをGitで管理し、チームメンバーが共通のコマンドを使用できるようにすることを推奨します。また、使用方法をREADMEに記載してください。

**Q: 「コマンドが複雑になりすぎた場合は？」**
A: 複雑なコマンドは小さな機能に分割することをお勧めします。また、外部ライブラリやツールとの連携を検討してください。

**Q: 「エラーが発生した場合のデバッグ方法は？」**
A: ログ機能を実装し、実行ログを確認してください。また、コマンドを小さな単位でテストし、問題を特定してください。

## 今すぐできる3ステップ

**Step 1: 基本環境構築（今週末）**
1. `.claude/commands`ディレクトリの作成
2. 基本的なコンポーネント作成コマンドの実装
3. 動作確認とテスト

**Step 2: 実用的なコマンド追加（来週）**
1. API作成コマンドの実装
2. テスト生成コマンドの追加
3. チーム内での共有と改善

**Step 3: 高度な自動化（今月中）**
1. 複合的なコマンドの作成
2. エラーハンドリングの強化
3. ドキュメント自動生成の実装

## まとめ：定型作業から解放される喜び

Claude Codeカスタムコマンドを導入してから、**開発に対する取り組み方が大きく変わりました**。

**Before**: 「また同じような作業...時間がもったいない」
**After**: 「コマンド一発で完了！本当の開発に集中できる」

特に感じる変化：
- **クリエイティブな時間の確保**: 定型作業から解放されて設計に集中
- **品質向上**: 一貫したコード構造とテストの自動生成
- **学習効果**: 最適化されたテンプレートから学ぶ機会

完璧ではありませんが、適切な設計と継続的な改善により、**開発効率の大幅な向上**を実現できています。

もし定型作業で悩んでいる方がいれば、ぜひ一度試してみてください。最初の設定は少し手間ですが、一度動き始めると本当に快適になりますよ！

次回は「Claude Codeメモリ管理で品質爆上げ！CLAUDE.md最適化テクニック」についてお話しする予定です。お楽しみに！

**皆さんのカスタムコマンドのアイデアや改善提案があれば、コメントで教えてください！一緒により良い開発環境を作っていきましょう🚀✨**