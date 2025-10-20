---
title: "Rust製AIフレームワーク『Miyabi』で実現する完全自動化開発 - Issue作成からデプロイまで"
emoji: "🌸"
type: "tech"
topics: ["rust", "github", "ai", "automation", "devops"]
published: false
---

## 🎯 はじめに - 「開発の自動化」の理想と現実

開発の自動化、理想は「全部AIがやってくれる」ことですよね。

でも現実は：
- Issue管理は手動
- コードレビューは時間がかかる
- デプロイは毎回緊張する

**この理想と現実のギャップを埋めるために、Miyabiを開発しました。**

本記事では、Rust製の完全自律型AIフレームワーク**Miyabi**のアーキテクチャを深掘りし、実際にどう動くのかを詳しく解説します。

---

## 🏗️ Miyabi のアーキテクチャ

### Cargo Workspace 構成（11個のcrate）

```
crates/
├── miyabi-types/             # コア型定義（Agent, Task, Issue等）
├── miyabi-core/              # 共通ユーティリティ（config, logger, retry, cache）
├── miyabi-cli/               # CLIツール (bin)
├── miyabi-agents/            # Coding Agent実装（7個）
├── miyabi-business-agents/   # Business Agent実装（14個）
├── miyabi-github/            # GitHub API統合（octocrab wrapper）
├── miyabi-worktree/          # Git Worktree管理
├── miyabi-llm/               # LLM抽象化層（GPT-OSS-20B、Groq/vLLM/Ollama）
├── miyabi-potpie/            # Potpie AI統合（Neo4j知識グラフ）
├── miyabi-mcp-server/        # MCP Server（JSON-RPC 2.0）
└── codex-miyabi/             # Codex統合（Phase 1）
```

### Entity-Relation Model（12種類のEntity、27の関係性）

Miyabiは、全てのコンポーネントをEntity-Relationモデルで統合的に管理しています。

| ID | Entity | 説明 | Rust型定義 |
|----|--------|------|-----------|
| E1 | **Issue** | GitHub Issue | `crates/miyabi-types/src/issue.rs` |
| E2 | **Task** | 分解されたタスク | `crates/miyabi-types/src/task.rs` |
| E3 | **Agent** | 自律実行Agent | `crates/miyabi-types/src/agent.rs` |
| E4 | **PR** | Pull Request | `crates/miyabi-github/src/pr.rs` |
| E5 | **Label** | GitHub Label（53個） | `docs/LABEL_SYSTEM_GUIDE.md` |
| E6 | **QualityReport** | 品質レポート | `crates/miyabi-types/src/quality.rs` |
| E7 | **Command** | Claude Codeコマンド | `.claude/commands/*.md` |
| E8 | **Escalation** | エスカレーション | `crates/miyabi-types/src/error.rs` |
| E9 | **Deployment** | デプロイ情報 | `crates/miyabi-agents/src/deployment.rs` |
| E10 | **LDDLog** | LDDログ | `crates/miyabi-types/src/workflow.rs` |
| E11 | **DAG** | タスク依存グラフ | `crates/miyabi-types/src/workflow.rs` |
| E12 | **Worktree** | Git Worktree | `crates/miyabi-worktree/src/lib.rs` |

**関係性の例**:
```
R1: Issue --analyzed-by-→ Agent (IssueAgent)
R2: Issue --decomposed-into-→ Task[] (CoordinatorAgent)
R3: Issue --tagged-with-→ Label[]
R4: Issue --creates-→ PR
R9: Agent --executes-→ Task
R10: Agent --generates-→ PR
R11: Agent --creates-→ QualityReport
```

---

## ⚡ 実践: Worktree並列実行アーキテクチャ

### アーキテクチャ概要

```
┌─────────────────────────────────────────────────────────┐
│ CoordinatorAgent (Main Process)                          │
│ - Issue分析・Task分解                                      │
│ - DAG構築・依存関係解決                                     │
│ - Worktree作成・管理                                       │
└─────────────────────────────────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
        ▼           ▼           ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Worktree #1 │ │ Worktree #2 │ │ Worktree #3 │
│ Issue #270  │ │ Issue #271  │ │ Issue #272  │
│             │ │             │ │             │
│ Claude Code │ │ Claude Code │ │ Claude Code │
│ Execution   │ │ Execution   │ │ Execution   │
└─────────────┘ └─────────────┘ └─────────────┘
        │           │           │
        └───────────┼───────────┘
                    │
                    ▼
            ┌─────────────┐
            │ Merge Back  │
            │ to Main     │
            └─────────────┘
```

### 実行フロー

**1. CoordinatorAgent起動**
```bash
miyabi agent run coordinator --issues=270,271,272 --concurrency=2
```

**2. 各IssueにWorktreeを作成**
```
.worktrees/
├── issue-270/  # Issue #270専用Worktree
├── issue-271/  # Issue #271専用Worktree
└── issue-272/  # Issue #272専用Worktree
```

**3. Worktree内でClaude Code実行**
各WorktreeでClaude Codeセッションが起動し、Agent固有の処理を実行します。

**4. 結果をマージ**
各Worktreeでの作業をmainブランチにマージし、統合テストを実行します。

### Worktree内での実行

```bash
cd .worktrees/issue-270
# Claude Codeが以下を実行：
# 1. 要件分析
# 2. コード生成（Rust + Tests）
#    - Rust structs/enums/traits実装
#    - #[cfg(test)] mod tests { ... }
#    - Rustdocコメント（///）
# 3. ドキュメント生成
# 4. Git commit
```

---

## 🤖 Agent の役割分担

### Agent System（BaseAgent trait + 21 Agents）

```rust
use miyabi_agents::BaseAgent;
use miyabi_types::{Task, AgentResult, MiyabiError};
use async_trait::async_trait;

#[async_trait]
pub trait BaseAgent {
    async fn execute(&self, task: Task) -> Result<AgentResult, MiyabiError>;
    async fn escalate(&self, reason: String, target: EscalationTarget) -> Result<(), MiyabiError>;
    async fn log_to_ldd(&self, log: LDDLog) -> Result<(), MiyabiError>;
}
```

### 🔴 リーダー: しきるん（Coordinator）

**役割**: タスク統括、DAG分解、並列実行制御

```rust
use miyabi_agents::CoordinatorAgent;

let coordinator = CoordinatorAgent::new(config);
let task_decomposition = coordinator.decompose_issue(&issue).await?;

// DAG構築
let dag = task_decomposition.dag;
// 並列実行可能タスクを自動検出
let parallel_tasks = dag.get_parallel_tasks();
```

**出力例**:
```
📋 Task Decomposition Complete
├── Task A: 型定義作成 → つくるん（CodeGenAgent）
├── Task B: テスト作成 → つくるん（CodeGenAgent）
└── Task C: ドキュメント作成 → かくちゃん（ContentCreationAgent）

⚡ Parallel Execution Plan:
- Group 1: Task A, B (並列実行可能)
- Group 2: Task C (A, B完了後)

⏱️ Estimated Time: 2.5 hours
```

### 🟢 実行役: つくるん（CodeGen）

**役割**: AI駆動コード生成（Claude Sonnet 4使用）

```rust
use miyabi_agents::CodeGenAgent;

let codegen = CodeGenAgent::new(config);
let result = codegen.execute(&task).await?;

// 自動生成されるもの:
// - Rust structs/enums/traits実装
// - #[cfg(test)] mod tests { ... } 付きテスト
// - /// Rustdocコメント
```

**生成されるファイル例**:
```
crates/miyabi-foo/
├── src/
│   ├── lib.rs           # ライブラリエントリポイント
│   ├── types.rs         # 型定義
│   └── service.rs       # 実装（tests含む）
├── Cargo.toml           # クレート定義
└── README.md            # ドキュメント
```

### 🔵 分析役: めだまん（Review）

**役割**: コード品質判定（100点満点スコアリング）

```rust
use miyabi_agents::ReviewAgent;

let reviewer = ReviewAgent::new(config);
let review = reviewer.execute(&task).await?;

// 品質スコアリング（80点以上でマージ可能）
// - cargo clippy --all-targets --all-features -- -D warnings
// - cargo test --all
// - cargo audit
```

**スコアリング基準**:
```
基準点: 100点
- Clippyエラー: -20点/件
- コンパイルエラー: -30点/件
- Critical脆弱性: -40点/件
- High脆弱性: -20点/件

✅ 80点以上: 合格
⚠️ 60-79点: 要改善
❌ 60点未満: 不合格（エスカレーション）
```

---

## 💻 実装例: Rust コード

### BaseAgent trait の実装例

```rust
use miyabi_agents::BaseAgent;
use miyabi_types::{Task, AgentResult, MiyabiError};
use async_trait::async_trait;

pub struct MyAgent {
    config: AgentConfig,
}

#[async_trait]
impl BaseAgent for MyAgent {
    async fn execute(&self, task: Task) -> Result<AgentResult, MiyabiError> {
        // 1. Task検証
        self.validate_task(&task)?;

        // 2. 実行
        let data = self.perform_task(&task).await?;

        // 3. 結果記録
        let result = AgentResult {
            status: AgentStatus::Success,
            data,
            duration_ms: 2000,
        };

        // 4. LDDログ出力
        self.log_to_ldd(LDDLog {
            session_id: self.session_id.clone(),
            task_id: task.id.clone(),
            result: result.clone(),
        }).await?;

        Ok(result)
    }

    async fn escalate(
        &self,
        reason: String,
        target: EscalationTarget,
    ) -> Result<(), MiyabiError> {
        // エスカレーション処理
        let escalation = EscalationInfo {
            reason,
            target,
            severity: Severity::Sev1Critical,
            context: HashMap::new(),
            timestamp: chrono::Utc::now().to_rfc3339(),
        };

        // GitHub Issue comment
        self.add_escalation_comment(&escalation).await?;

        // Slack/Discord通知
        self.notify_escalation(&escalation).await?;

        Ok(())
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_my_agent() {
        let config = AgentConfig::default();
        let agent = MyAgent { config };

        let task = Task {
            id: "task-1".to_string(),
            title: "Test task".to_string(),
            task_type: TaskType::Feature,
            dependencies: vec![],
        };

        let result = agent.execute(task).await;
        assert!(result.is_ok());
    }
}
```

### async/await + tokio の使い方

```rust
use tokio;
use miyabi_agents::CodeGenAgent;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 複数Agentを並列実行
    let codegen1 = CodeGenAgent::new(config.clone());
    let codegen2 = CodeGenAgent::new(config.clone());
    let codegen3 = CodeGenAgent::new(config.clone());

    let (result1, result2, result3) = tokio::join!(
        codegen1.execute(&task1),
        codegen2.execute(&task2),
        codegen3.execute(&task3),
    );

    println!("Task 1: {:?}", result1?);
    println!("Task 2: {:?}", result2?);
    println!("Task 3: {:?}", result3?);

    Ok(())
}
```

### octocrab による GitHub API 統合

```rust
use octocrab::{Octocrab, models::issues::Issue};

pub async fn fetch_issue(
    owner: &str,
    repo: &str,
    issue_number: u64,
) -> Result<Issue, octocrab::Error> {
    let octocrab = Octocrab::builder()
        .personal_token(std::env::var("GITHUB_TOKEN")?)
        .build()?;

    let issue = octocrab
        .issues(owner, repo)
        .get(issue_number)
        .await?;

    Ok(issue)
}

pub async fn add_labels(
    owner: &str,
    repo: &str,
    issue_number: u64,
    labels: Vec<String>,
) -> Result<(), octocrab::Error> {
    let octocrab = Octocrab::builder()
        .personal_token(std::env::var("GITHUB_TOKEN")?)
        .build()?;

    octocrab
        .issues(owner, repo)
        .add_labels(issue_number, &labels)
        .await?;

    Ok(())
}
```

---

## 🚀 デプロイ自動化 - はこぶん（DeploymentAgent）の動作

### Firebase/Vercelへのデプロイ

```rust
use miyabi_agents::DeploymentAgent;

let deployment_agent = DeploymentAgent::new(config);

let deployment_config = DeploymentConfig {
    environment: Environment::Staging,
    version: "1.2.3".to_string(),
    project_id: "miyabi-staging".to_string(),
    targets: vec!["hosting".to_string(), "functions".to_string()],
    auto_rollback: true,
    health_check_url: "https://staging.example.com/health".to_string(),
};

let deployment_result = deployment_agent.deploy(deployment_config).await?;

// ヘルスチェック
let healthy = deployment_agent.check_health(&deployment_result.deployment_url).await?;
if !healthy {
    deployment_agent.rollback(&deployment_result).await?;
}
```

**デプロイフロー**:
```
1. ビルド（cargo build --release）
2. テスト実行（cargo test --all）
3. Firebase/Vercelへデプロイ
4. ヘルスチェック
5. デプロイ成功/失敗の通知
6. 失敗時は自動ロールバック
```

---

## 📊 今後のロードマップ

### Phase 1: Coding Agents（完了）✅
- CoordinatorAgent
- CodeGenAgent
- ReviewAgent
- IssueAgent
- PRAgent
- DeploymentAgent

### Phase 2: Business Agents（進行中）🚧
- **AIEntrepreneurAgent（完了✅）**: 8フェーズビジネスプラン自動生成
- MarketResearchAgent（計画中）
- ContentCreationAgent（計画中）
- SNSStrategyAgent（計画中）
- YouTubeAgent（計画中）
- SalesAgent（計画中）
- CRMAgent（計画中）
- AnalyticsAgent（計画中）
- 等、合計14個のBusiness Agents

### Phase 3: LLM統合（進行中）🚧
- GPT-OSS-20B対応
- Groq API統合
- vLLM self-hosted統合
- Ollama (Mac mini) 統合

### Phase 4: Potpie AI統合（進行中）🚧
- Neo4j知識グラフ
- RAG engine
- コード解析・インデキシング

---

## 🌟 まとめ

### Miyabiの技術的優位性

1. **Rust 2021 Edition**: 高速・安全・並列実行
2. **Entity-Relationモデル**: 12種類のEntity、27の関係性
3. **Worktree並列実行**: 真の並列処理（コンフリクト最小化）
4. **BaseAgent trait**: 統一的なAgent実装パターン
5. **GitHub OS Integration**: GitHubを「OS」として活用
6. **53ラベル体系**: 状態管理の心臓部
7. **Claude Sonnet 4**: 高品質なコード生成

### 次のステップ

```bash
# 今すぐ試す
cargo install miyabi-cli
miyabi init my-project --interactive
miyabi status --watch

# GitHubリポジトリ
https://github.com/ShunsukeHayashi/Miyabi

# Star をお願いします！⭐
```

### コントリビューション歓迎

- ⭐ Star をつけてください
- 🐛 バグ報告: [GitHub Issues](https://github.com/ShunsukeHayashi/Miyabi/issues)
- 💡 機能提案: [GitHub Discussions](https://github.com/ShunsukeHayashi/Miyabi/discussions)
- 🤝 PR歓迎: [Contributing Guide](https://github.com/ShunsukeHayashi/Miyabi/blob/main/CONTRIBUTING.md)

**Miyabi** - Beauty in Autonomous Development 🌸

🤖 Powered by Claude AI • 🔒 Apache 2.0 License • 💖 Made with Love
