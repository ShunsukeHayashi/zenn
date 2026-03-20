---
title: "AIエージェントのスキルを自己改善させるOSSを作った"
emoji: "🚌"
type: "tech"
topics: ["ai", "agent", "typescript", "oss", "claude"]
published: true
---

こんにちは、ハヤシシュンスケです。

42のAIエージェントを本番で動かし続けて約1年になります。Claude Code、Codex、OpenClaw、自作スクリプト——多種多様なエージェントを並走させていると、どのフレームワークも解決してくれない「運用の穴」に毎週ぶつかるようになりました。

今回はその穴を埋めるために作った OSS **agent-skill-bus** の話をします。

## エージェントが「スキルを忘れる」問題

本番で42エージェントを動かし続けて痛感したのは、**スキルは静かに劣化する**ということです。

```mermaid
graph LR
    subgraph Before["導入前：見えない劣化"]
        S1["Week 1<br/>スコア 0.95"] --> S2["Week 2<br/>スコア 0.80"]
        S2 --> S3["Week 3<br/>スコア 0.40"]
        S3 --> S4["Week 4<br/>💥 障害発覚"]
    end

    subgraph After["導入後：早期検知"]
        A1["Week 1<br/>スコア 0.95"] --> A2["Week 2<br/>スコア 0.80"]
        A2 --> A3["⚡ 劣化検知<br/>自動修復"]
        A3 --> A4["Week 3<br/>スコア 0.90 ✅"]
    end

    style Before fill:#fef2f2,stroke:#ef4444
    style After fill:#f0fdf4,stroke:#16a34a
```

具体的にはこういうことが起きます。

外部APIの仕様が変わる。モデルのアップデートで出力フォーマットが微妙にずれる。認証トークンが期限切れになる。エージェント自体は「動いている」ように見えるのに、品質スコアが3週間かけて0.95から0.40に下がっていく。

実際にやらかしたのは、請求書分類エージェントが2週間にわたって取引を誤分類し続けたケースです。エラーは出ない。ログも正常。でも出力がズレていた。気づいたのはダウンストリームの集計がおかしくなってからでした。

もう1つの問題が**タスクの競合**です。2つのエージェントが同じファイルを同時に編集しようとする。BullMQやRedisストリームはジョブレベルの排他はできますが、「異なるマシンで動く異種エージェントをまたいだファイルレベルのロック」は面倒です。

そして3つ目が**学習しない**問題。失敗→リトライ→また同じ失敗。失敗をスキル定義に還元するループがない。

## agent-skill-bus の3モジュール

この3つの課題を解決するために作ったのが [agent-skill-bus](https://github.com/ShunsukeHayashi/agent-skill-bus) です。

```mermaid
graph TD
    subgraph External["外部変化"]
        API["API仕様変更"]
        DEP["依存パッケージ更新"]
        MODEL["モデルアップデート"]
    end

    subgraph KW["Knowledge Watcher"]
        DETECT["変化検知・差分分析"]
    end

    subgraph BUS["Prompt Request Bus"]
        DAG["DAGキュー"]
        LOCK["ファイルロック"]
        ROUTE["優先度ルーティング"]
    end

    subgraph SI["Self-Improving Skills"]
        MONITOR["品質監視"]
        DRIFT["劣化検知"]
        REPAIR["自動修復 / エスカレーション"]
    end

    API --> DETECT
    DEP --> DETECT
    MODEL --> DETECT
    DETECT -->|修復タスク投入| DAG

    A["Agent A"] --> DAG
    B["Agent B"] --> DAG
    C["Agent C"] --> DAG

    DAG --> LOCK --> ROUTE
    ROUTE -->|実行 & スコア収集| MONITOR
    MONITOR --> DRIFT --> REPAIR
    REPAIR -->|改善タスク| DAG

    style KW fill:#7c3aed,color:#fff
    style BUS fill:#2563eb,color:#fff
    style SI fill:#16a34a,color:#fff
```

### 1. Prompt Request Bus（DAGタスクキュー）

JSONL ベースのタスクキューです。特徴はファイルレベルのロックと依存関係解決。

```typescript
import { PromptRequestBus } from 'agent-skill-bus';

const bus = new PromptRequestBus('./queue');

// 依存関係付きタスク登録
await bus.enqueue({
  id: 'analyze-report',
  prompt: 'Q1の売上レポートを分析して',
  dependsOn: ['fetch-data', 'clean-data'],  // DAG依存
  priority: 'high',
  lockFiles: ['data/q1-report.csv'],        // ファイルロック
});

// タスク取得（依存が解決済みのものだけ返ってくる）
const task = await bus.dequeue({ agentId: 'analyzer' });
```

`critical` 優先度のタスクはキューをバイパスして即時実行。TTL付きのデッドロック防止機構もあります。

### 2. Self-Improving Skills（スキル品質モニタリング）

7ステップの自己改善ループです。スコアが閾値を下回ると、自動で原因分析→修復提案→適用まで回ります。

```mermaid
graph LR
    O["1. OBSERVE<br/>スコア収集"] --> A["2. ANALYZE<br/>トレンド分析"]
    A --> D["3. DIAGNOSE<br/>原因特定"]
    D --> P["4. PROPOSE<br/>修復案生成"]
    P --> E["5. EVALUATE<br/>リスク判定"]
    E --> AP["6. APPLY<br/>自動適用"]
    AP --> R["7. RECORD<br/>結果記録"]
    R -.->|次サイクル| O

    style O fill:#f59e0b,color:#fff
    style A fill:#f59e0b,color:#fff
    style D fill:#ef4444,color:#fff
    style P fill:#8b5cf6,color:#fff
    style E fill:#8b5cf6,color:#fff
    style AP fill:#16a34a,color:#fff
    style R fill:#16a34a,color:#fff
```

- **OBSERVE〜ANALYZE**: スコアを時系列で集め、週次トレンドを計算
- **DIAGNOSE**: 閾値割れの根本原因を特定（API変更？モデル更新？データドリフト？）
- **PROPOSE〜EVALUATE**: 修復案を生成し、リスクレベルで自動適用か人間エスカレーションか判断
- **APPLY〜RECORD**: 低リスクなら自動適用、結果をJSONLに永続化

```typescript
import { SelfImprovingSkill } from 'agent-skill-bus';

const skill = new SelfImprovingSkill({
  name: 'invoice-classifier',
  scoreThreshold: 0.75,    // これを下回ったら検知
  driftThreshold: 0.15,    // 週次で15%以上の変動で検知
  autoApply: 'low-risk',   // リスクが低い修正は自動適用
});

// 実行後にスコアを記録するだけ
await skill.execute(task);
await skill.recordScore(0.82);

// 劣化検知時にコールバック
skill.on('drift-detected', async ({ currentScore, weeklyDelta }) => {
  console.log(`品質劣化: ${weeklyDelta * 100}%低下`);
  // 自動修復プロポーザルが生成される
});
```

これを本番に入れたら、スキルの失敗率が57%下がりました。

### 3. Knowledge Watcher（外部変更検知）

依存パッケージのバージョン変更、外部APIの仕様変更、コミュニティの新パターンを階層的に監視して、変化があったら自動的に修復タスクをキューに投入します。

```typescript
import { KnowledgeWatcher } from 'agent-skill-bus';

const watcher = new KnowledgeWatcher(bus);

await watcher.watch({
  npm: ['openai', 'anthropic', '@google-cloud/aiplatform'],
  apis: [
    { name: 'stripe-api', url: 'https://stripe.com/docs/api/changelog' },
  ],
  checkInterval: '6h',
});
// 変化検知 → 自動で修復タスクをバスに投入
```

## インストールと基本的な使い方

```bash
npm install agent-skill-bus
# または
npx agent-skill-bus init
```

3モジュールは独立しているのでそれぞれ単体でも使えます。

```typescript
// 最小構成: タスクキューだけ使う
import { PromptRequestBus } from 'agent-skill-bus';

const bus = new PromptRequestBus('./my-queue');
await bus.enqueue({ id: 'task-1', prompt: 'こんにちは' });
const task = await bus.dequeue({ agentId: 'my-agent' });
```

## どのモジュールから始めるか

3モジュールは独立しているので、自分の課題に合わせて1つだけ導入できます。

```mermaid
graph TD
    Q{"あなたの課題は？"}
    Q -->|タスクが競合する<br/>依存関係を整理したい| BUS["Prompt Request Bus<br/>DAGキュー + ファイルロック"]
    Q -->|スキルの品質が<br/>劣化していく| SI["Self-Improving Skills<br/>7ステップ自己改善ループ"]
    Q -->|外部APIやパッケージの<br/>変更に追いつけない| KW["Knowledge Watcher<br/>変更検知 + 自動修復"]
    Q -->|全部| ALL["3モジュール統合"]

    BUS --> START["npm install agent-skill-bus"]
    SI --> START
    KW --> START
    ALL --> START

    style Q fill:#6366f1,color:#fff
    style BUS fill:#2563eb,color:#fff
    style SI fill:#16a34a,color:#fff
    style KW fill:#7c3aed,color:#fff
    style ALL fill:#dc2626,color:#fff
```

## 3日で35スター、83%がXのエンジニア経由

公開から3日で35スターをいただきました。流入元を見ると83%がX（旧Twitter）経由で、エンジニアのリツイートが連鎖した形でした。

「マルチエージェントの運用課題を整理してくれた」「ファイルレベルのロックは盲点だった」というコメントが多く、「あ、同じ課題で詰まってた人が他にもいるんだ」と実感しました。

## 技術的なこだわり：依存ゼロ

設計上の制約として**npmの依存をゼロにする**ことを決めていました。

LangChain、BullMQ、Redis——どれも優れたツールですが、「どんなエージェントでも統合できるバス」を作るには、特定のランタイムやインフラに縛られてはいけない。ファイルシステムとNode.js組み込みモジュールだけで動くことで、シェルスクリプトで動くエージェントでも、Python製エージェントでも、理屈上は接続できます。

フレームワーク非依存というのも意識しました。LangGraph、CrewAI、AutoGen——実行グラフの構築は各フレームワークが得意とするところで、そこは任せる。agent-skill-bus はその「下のレイヤー」、つまり運用インフラ層を担う位置づけです。

## 今後やること

- **awesome-list 提出中**: awesome-langchain、awesome-agents への提出を進めています
- **比較ページ**: LangGraph/CrewAI/AutoGen との比較ドキュメントを整備中
- **TypeScript 型定義の強化**: 現在の型定義は最小限なので、より厳密な型を追加予定

マルチエージェントを本番運用している方、スキルの劣化や競合で困っている方にとって少しでも参考になれば幸いです。

Issue や PR もお待ちしています。

## リンク集

| リソース | URL |
|---------|-----|
| **GitHub** | [ShunsukeHayashi/agent-skill-bus](https://github.com/ShunsukeHayashi/agent-skill-bus) |
| **npm** | [agent-skill-bus](https://www.npmjs.com/package/agent-skill-bus) |
| **Examples** | [examples/](https://github.com/ShunsukeHayashi/agent-skill-bus/tree/master/examples) |
| **TypeScript型定義** | [types/index.d.ts](https://github.com/ShunsukeHayashi/agent-skill-bus/blob/master/types/index.d.ts) |
| **他フレームワークとの比較** | [docs/comparison.md](https://github.com/ShunsukeHayashi/agent-skill-bus/blob/master/docs/comparison.md) |
| **Contributing** | [CONTRIBUTING.md](https://github.com/ShunsukeHayashi/agent-skill-bus/blob/master/CONTRIBUTING.md) |
| **ライセンス** | MIT |

### 関連リソース

- [LangGraph](https://github.com/langchain-ai/langgraph) — LangChainベースのエージェントグラフ構築
- [CrewAI](https://github.com/crewAIInc/crewAI) — ロールベースのマルチエージェントフレームワーク
- [AutoGen](https://github.com/microsoft/autogen) — Microsoftのマルチエージェント会話フレームワーク
- [VoltAgent](https://github.com/VoltAgent/voltagent) — TypeScriptエージェントフレームワーク
- [awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) — エージェントスキルのキュレーションリスト
