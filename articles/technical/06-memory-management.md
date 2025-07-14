# 6. メモリ管理とコンテキスト最適化

## 6.1 CLAUDE.mdの効果的な活用

### 3つのメモリレベル

Claude Codeのメモリ管理システムは、人間の記憶構造からインスピレーションを得た、階層的で効率的な設計となっています。短期記憶と長期記憶、個人的な記憶と集合的な記憶の概念を、開発環境における実用的なシステムとして具現化したものです。この3層のメモリアーキテクチャを理解し、適切に活用することで、Claude Codeとの協働を格段に向上させることができます。

**プロジェクトメモリ（./CLAUDE.md）**

プロジェクトメモリは、Claude Codeメモリシステムの中核を成すものです。これは、プロジェクトルートディレクトリに配置される`CLAUDE.md`ファイルで、そのプロジェクト固有の知識、ルール、慣習を保存します。Gitリポジトリにコミットされることを前提として設計されており、チーム全体で共有される「集合知」として機能します。

プロジェクトメモリの真の価値は、プロジェクトの「DNA」を保存することにあります。これは単なる設定ファイルではなく、プロジェクトの歴史、価値観、そして未来への方向性を記録した生きたドキュメントです。新しいチームメンバーがプロジェクトに参加した際、このファイルを読むことで、技術的な詳細だけでなく、チームの思考パターンや意思決定の背景も理解することができます。

```markdown
# プロジェクト: Next.js企業向けSaaS

## プロジェクト概要
このプロジェクトは、中小企業向けの顧客管理システムです。
シンプルさと拡張性のバランスを重視し、技術的負債を最小限に
抑えながら迅速な機能開発を実現することを目指しています。

## 技術的価値観

### パフォーマンス第一
- ページ読み込み速度：3秒以内必須
- APIレスポンス：200ms以内目標
- バンドルサイズ：500KB以下を維持

### 保守性重視
- 複雑な最適化より理解しやすいコード
- 一つの機能は一つのファイル/フォルダに
- 命名は冗長になっても明確に

### セキュリティの基準
- すべての外部入力はバリデーション必須
- 個人情報は暗号化して保存
- 権限チェックは各エンドポイントで実装

## アーキテクチャ決定記録（抜粋）

### ADR-001: TypeScript導入
**決定**: 全コードベースでTypeScriptを使用
**理由**: 大規模開発における型安全性とIDE支援
**トレードオフ**: 初期学習コストと開発速度の一時的低下

### ADR-003: 状態管理にZustand採用
**決定**: ReduxではなくZustandを使用
**理由**: ボイラープレートの削減とシンプルさ
**制約**: 複雑な状態ロジックは避け、サーバーステートと分離

### ADR-007: UIライブラリとしてChakra UI採用
**決定**: Material-UIからChakra UIに移行
**理由**: カスタマイゼーションの容易さとパフォーマンス
**移行期間**: 2ヶ月（完了予定：2024年3月）

## コーディング規約

### ファイル構造
```
src/
  components/     # 再利用可能なUIコンポーネント
    common/       # 汎用コンポーネント
    forms/        # フォーム関連
    layout/       # レイアウト関連
  features/       # 機能別のコンポーネント
    customers/    # 顧客管理機能
    dashboard/    # ダッシュボード機能
  hooks/          # カスタムフック
  utils/          # ユーティリティ関数
  types/          # 型定義
```

### 命名規則
- コンポーネント: PascalCase（`CustomerList.tsx`）
- フック: camelCase with "use" prefix（`useCustomers.ts`）
- ユーティリティ: camelCase（`formatDate.ts`）
- 定数: SCREAMING_SNAKE_CASE（`API_ENDPOINTS`）

### import順序
1. React関連
2. 外部ライブラリ
3. 内部ユーティリティ
4. 相対インポート
5. 型定義（import type）

## 業務知識

### 顧客分類
- **リード**: 見込み客（contact_statusが'lead'）
- **プロスペクト**: 商談中（contact_statusが'prospect'）
- **カスタマー**: 契約済み（contract_statusが'active'）
- **チャーン**: 解約済み（contract_statusが'cancelled'）

### 権限レベル
- **Admin**: 全機能アクセス可能
- **Manager**: チーム内のデータのみ
- **User**: 自分が担当する顧客のみ
- **ReadOnly**: 閲覧のみ

## よくある問題と解決法

### Q: 顧客データの更新が遅い
A: `useQuery`のstaleTimeを調整。顧客詳細ページでは30秒、
   リストページでは5分に設定済み。

### Q: フォームバリデーションのエラーメッセージが英語になる
A: `react-hook-form`のerrorMessagesオブジェクトを
   `src/utils/validation.ts`で日本語化。新しいバリデーションは
   必ず日本語メッセージを追加。

### Q: 本番環境でのみ発生するエラー
A: 環境変数`NODE_ENV`の値を確認。開発環境では'development'、
   本番では'production'。条件分岐が必要な場合は明示的に記述。

## 禁止事項
- console.logの本番環境への残留（pre-commitフックで検出）
- anyタイプの使用（unknownまたは具体的な型を使用）
- インラインスタイルの使用（Chakra UIのpropsまたはCSS-in-JS）
- 直接的なlocalStorage操作（useLocalStorageフックを使用）

## 最近の決定事項
- パスワードポリシーを強化（最低12文字、英数記号必須）
- APIレスポンスキャッシュ時間を5分から3分に短縮
- エラー追跡にSentryを導入（設定済み）
```

**プロジェクトローカルメモリ（./CLAUDE.local.md）**

プロジェクトローカルメモリは、個人的で一時的な情報を格納するためのスペースです。このファイルは`.gitignore`に含まれ、個人の作業環境に固有の設定や、まだチームで合意されていない実験的なアイデアを保存します。

このレベルのメモリは、開発者の「作業メモ」として機能します。進行中の調査、試行錯誤の記録、個人的なメモ、あるいは他のチームメンバーには関係のない個人的な設定などが含まれます。

```markdown
# 個人作業メモ

## 現在作業中のタスク
- [ ] 顧客インポート機能の性能改善
  - CSVパーサーの最適化（完了）
  - バッチ処理の実装（進行中）
  - プログレスバーの追加（未着手）

## 調査メモ

### パフォーマンス問題調査
**問題**: 顧客リストページの読み込みが5秒かかる
**仮説**: 
1. N+1クエリ問題 → includeでeager loading追加
2. 不要なデータフェッチ → selectで必要フィールドのみ
3. フロントエンドでの重い処理 → useMemoで最適化

**試行結果**:
- includeの追加: 5秒 → 3秒（改善）
- selectの追加: 3秒 → 2秒（改善）
- useMemoの追加: 2秒 → 1.5秒（改善）

### 新技術の検証
**React Server Components**: Next.js 13で試験導入
- メリット: バンドルサイズ削減、SEO向上
- デメリット: 学習コスト、既存コードとの互換性
- 結論: 新機能での限定試験導入を提案

## 個人的な設定
- VS Code設定: tabSize=2, formatOnSave=true
- 作業時間: 9:00-18:00（集中時間: 10:00-12:00, 14:00-16:00）
- 定期レビュー: 毎週金曜日17:00

## チームメンバーとのやり取り
- 田中さん: UIデザインのフィードバック待ち
- 佐藤さん: APIエンドポイントの仕様確認予定
- 山田PM: 機能優先度の再調整が必要

## アイデアメモ
- 顧客満足度の自動分析機能
- AIによる営業メール自動生成
- 顧客行動パターンの可視化ダッシュボード
```

**ユーザーメモリ（~/.claude/CLAUDE.md）**

ユーザーメモリは、すべてのプロジェクトに共通する個人的な設定と好みを保存します。これは開発者の「性格」や「好み」をClaude Codeに伝える重要な手段です。コーディングスタイル、使用を好むライブラリ、避けたい技術、そして個人的な作業パターンなどが含まれます。

このレベルのメモリにより、Claude Codeは開発者個人の好みを学習し、より個人化されたサポートを提供できるようになります。

```markdown
# 個人設定 - Claude Code

## 基本情報
- 名前: 田中太郎
- 経験年数: 8年
- 専門分野: フロントエンド（React/TypeScript）、Node.js
- 現在の役職: シニアエンジニア

## 技術的好み

### 言語・フレームワーク
**好み**:
- TypeScript over JavaScript（型安全性重視）
- React with hooks（クラスコンポーネントは避ける）
- Express.js（シンプルさを重視）
- PostgreSQL（リレーショナルDBを好む）

**避けたいもの**:
- 過度に複雑な抽象化
- 実績の少ない新しいフレームワーク
- マイクロサービス（小規模チームでは）

### コードスタイル
- インデント: スペース2つ
- 行長制限: 80文字
- セミコロン: 必須
- クォート: シングルクォート優先
- 命名: 説明的で長めでも構わない

### アーキテクチャ哲学
- "Composition over Inheritance"
- "Convention over Configuration"
- "KISS principle"（Keep It Simple, Stupid）
- "YAGNI"（You Aren't Gonna Need It）

## 作業パターン

### 開発フロー
1. 要件を小さなタスクに分解
2. 型定義から開始
3. ユニットテストを先に作成
4. 実装
5. 統合テスト
6. リファクタリング

### コミュニケーション
- 技術的な議論は非同期（GitHub Issues/PR）を好む
- 複雑な問題は図解やコード例で説明
- フィードバックは具体的で建設的に

### 学習スタイル
- 公式ドキュメントを重視
- 実際のプロジェクトで試しながら学習
- オンラインコースよりも書籍を好む

## ツール設定

### エディタ: VS Code
```json
{
  "editor.tabSize": 2,
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.preferences.importModuleSpecifier": "relative"
}
```

### Git設定
- コミットメッセージ: Conventional Commits準拠
- ブランチ命名: feature/機能名、fix/バグ名
- PRは小さく、レビューしやすいサイズに

## Claude Codeとの協働

### 期待すること
- 型安全なコードの生成
- パフォーマンスを考慮した実装
- テスタブルなコード構造
- 適切なエラーハンドリング

### 応答形式の好み
- コード例は必ずTypeScriptで
- 説明は簡潔で要点を押さえて
- トレードオフがある場合は明示
- セキュリティ上の考慮点があれば言及

### よく依頼するタスク
- リファクタリングの提案
- パフォーマンス最適化
- テストケースの作成
- API設計のレビュー
```

### 記載すべき内容

効果的なCLAUDE.mdを作成するために、記載すべき内容を体系的に整理することが重要です。これらの情報は、Claude Codeがプロジェクトの文脈を理解し、適切な提案や実装を行うための基盤となります。

**技術仕様と制約条件**

プロジェクトの技術的な基盤と制約を明確に記載することで、Claude Codeは適切な技術選択と実装方針を提案できます。これには、使用技術、パフォーマンス要件、セキュリティ基準、そして技術的負債の現状などが含まれます。

```markdown
## 技術仕様

### 技術スタック
**フロントエンド**:
- React 18.2.0 (Concurrent Features使用)
- TypeScript 5.0.4 (strict mode有効)
- Next.js 13.4.0 (App Router使用)
- Chakra UI 2.6.0 (テーマカスタマイズ済み)
- React Query 4.28.0 (サーバーステート管理)
- React Hook Form 7.43.0 (フォーム処理)

**バックエンド**:
- Node.js 18.16.0 (LTS)
- Express.js 4.18.0
- TypeScript 5.0.4
- Prisma 4.14.0 (ORM)
- PostgreSQL 15.3
- Redis 7.0.11 (セッション・キャッシュ)

**インフラ**:
- AWS ECS Fargate (コンテナ実行)
- AWS RDS (PostgreSQL)
- AWS ElastiCache (Redis)
- AWS CloudFront (CDN)
- AWS S3 (静的ファイル)

### パフォーマンス要件
- **First Contentful Paint**: 1.5秒以内
- **Largest Contentful Paint**: 2.5秒以内
- **Cumulative Layout Shift**: 0.1以下
- **Time to Interactive**: 3.5秒以内
- **API Response Time**: 95%ile で 200ms以内

### セキュリティ要件
- OWASP Top 10への対応必須
- SOC 2 Type II準拠
- PII（個人情報）は暗号化保存
- APIアクセスは全てJWT認証
- CSP (Content Security Policy) 設定済み
- Rate Limiting: 100req/min/IP

### ブラウザサポート
- Chrome/Edge: 最新2バージョン
- Firefox: 最新2バージョン
- Safari: 最新2バージョン
- IE: サポート対象外
- モバイル: iOS Safari 15+, Chrome Mobile 100+
```

**ビジネスロジックとドメイン知識**

システムが扱うビジネスドメインの特殊性や、業界固有のルールを記載することで、Claude Codeはより適切なデータモデルや処理ロジックを提案できます。

```markdown
## ビジネスドメイン知識

### 不動産業界特有のルール

**物件ステータス**:
- 新規: 市場投入から30日以内
- 注目: 問い合わせ10件以上
- 価格改定: 60日間売れ残り→価格見直し
- 成約: 契約締結済み
- 取り下げ: オーナー都合による販売停止

**手数料計算**:
- 売買: 成約価格の3% + 6万円 + 消費税
- 賃貸: 家賃1ヶ月分 + 消費税
- 管理: 家賃の5%（オーナー負担）

**法的制約**:
- 宅建業法: 重要事項説明必須
- 契約書面: 37条書面の作成義務
- 広告規制: 誇大広告の禁止
- クーリングオフ: 事務所以外での契約は8日間

### データ整合性ルール

**物件データ**:
- 住所は必ず都道府県から始まる正規化形式
- 築年数は登記簿謄本ベース（建築確認ではない）
- 面積は㎡表示（坪併記は自動計算）
- 価格は税込み・税抜きを明確に区別

**顧客データ**:
- 個人: 姓名分離、フリガナ必須
- 法人: 正式社名、代表者名、登記住所
- 連絡先: 電話番号は国際形式正規化
- 希望条件: 予算・エリア・間取りは必須
```

**チーム固有の開発プラクティス**

チームが採用している開発手法や、合意されたベストプラクティスを記載することで、Claude Codeは一貫性のある提案を行えます。

```markdown
## 開発プラクティス

### コードレビュー基準

**必須チェック項目**:
1. **型安全性**: anyタイプの使用禁止
2. **パフォーマンス**: 不要な再レンダリングの回避
3. **アクセシビリティ**: WAI-ARIA属性の適切な使用
4. **セキュリティ**: 入力値バリデーションの実装
5. **テスタビリティ**: 副作用の分離

**コーディング規約**:
- 関数は単一責任原則に従う
- マジックナンバーは定数化
- エラーハンドリングは必ず実装
- ログ出力は構造化JSON形式
- コメントは「なぜ」を説明（「何を」ではない）

### テスト戦略

**ユニットテスト**:
- カバレッジ80%以上必須
- Business Logic重点的にテスト
- モックは最小限（実装詳細に依存しない）

**統合テスト**:
- API エンドポイント全て
- データベーストランザクション
- 外部サービス連携（モック使用）

**E2Eテスト**:
- ユーザージャーニー主要パス
- 決済フロー
- 管理者機能

### デプロイメント

**ブランチ戦略**:
- main: 本番環境（常にデプロイ可能状態）
- develop: 開発統合（機能完了時にマージ）
- feature/*: 機能開発
- hotfix/*: 緊急修正

**リリースサイクル**:
- 定期リリース: 隔週火曜日 14:00
- ホットフィックス: 必要時即座
- メンテナンス: 毎月第3日曜日 2:00-6:00
```

### 避けるべき内容

CLAUDE.mdの効果性を最大化するためには、記載すべきでない内容を理解することも重要です。適切でない情報は、Claude Codeの判断を曇らせ、最適でない提案につながる可能性があります。

**汎用的すぎる指示**

「良いコードを書いてください」や「ベストプラクティスに従ってください」といった抽象的な指示は、実際の行動指針として機能しません。代わりに、具体的で測定可能な基準を提示する必要があります。

```markdown
❌ 避けるべき例:
## 品質方針
- 良いコードを書く
- パフォーマンスを考慮する
- セキュリティに気をつける
- 保守しやすくする

✅ 改善された例:
## 品質基準
- 関数の行数は50行以内（例外: 設定オブジェクトは除く）
- API応答時間は95%ileで200ms以内
- 全ての外部入力はJoiスキーマでバリデーション
- 公開関数には必ずJSDocコメント
```

**過度に詳細な実装指示**

CLAUDE.mdは設計思想や制約を伝える場所であり、具体的な実装手順を記載する場所ではありません。実装の詳細は、Claude Codeが文脈に応じて最適化できる余地を残すべきです。

```markdown
❌ 避けるべき例:
## API実装手順
1. Express routerを作成
2. middleware関数を定義
3. バリデーション処理を追加
4. データベースクエリを実行
5. レスポンスを返却

✅ 改善された例:
## API設計原則
- RESTful設計に準拠
- エラーレスポンスはRFC7807準拠
- バージョニングはヘッダーベース
- 冪等性が保証されていない操作はPOST
- リソース操作は適切なHTTPメソッドを使用
```

**一時的な情報**

頻繁に変更される情報や、短期間しか有効でない情報は、CLAUDE.mdではなく他の手段で管理すべきです。

```markdown
❌ 避けるべき例:
## 現在の作業状況
- ユーザー登録機能: 80%完了
- 決済機能: 来週開始予定
- 田中さんが休暇中（今週まで）

✅ 適切な管理場所:
- 作業状況 → プロジェクト管理ツール（Jira、Asana等）
- 個人的なメモ → CLAUDE.local.md
- チーム状況 → チャットツール（Slack、Teams等）
```

## 6.2 コンテキストウィンドウ管理

### 200Kトークンの活用

Claude 4モデルの200,000トークンというコンテキストウィンドウは、従来のAIシステムでは考えられなかった規模の情報を同時に処理できる革命的な能力です。しかし、この膨大な容量を効果的に活用するためには、戦略的なアプローチと深い理解が必要です。

200Kトークンの規模を理解するために、具体的な例で考えてみましょう。これは約15万語、つまり平均的な小説1冊分の文章量に相当します。開発の文脈では、中規模のプロジェクト全体のソースコード、設計ドキュメント、そして関連する技術仕様を同時にメモリに保持できることを意味します。

この能力を最大限に活用するためには、情報の構造化と優先順位付けが不可欠です。最も重要で頻繁に参照される情報を上位に配置し、詳細な実装情報や一時的な情報を下位に配置する階層構造を意識的に設計する必要があります。

**効果的なコンテキスト構造**

```markdown
# コンテキスト階層設計例

## Level 1: Core Context (常時保持) - ~20K tokens
- プロジェクト概要
- 主要なアーキテクチャ決定
- 核となるビジネスロジック
- 重要な制約条件

## Level 2: Active Context (セッション中保持) - ~80K tokens
- 現在作業中のファイル群
- 関連するテストコード
- 最近の変更履歴
- 進行中の議論

## Level 3: Reference Context (必要時ロード) - ~100K tokens
- 詳細なAPI仕様
- 全体のコードベース
- 過去の議論ログ
- 外部ドキュメント

## Level 4: Archive Context (明示的要求時のみ)
- 過去のバージョンのコード
- 廃止された機能の履歴
- 詳細なログ情報
```

**動的コンテキスト管理**

静的な情報配置だけでなく、作業の流れに応じてコンテキストを動的に調整することも重要です。デバッグ作業中には関連するログとエラーパターンを、新機能開発中には類似機能の実装例と設計パターンを、そしてレビュー作業中には品質基準とベストプラクティスを前面に押し出します。

```bash
# コンテキスト最適化の実例

# デバッグモード: エラー情報を最優先
claude --context-priority=debug "
このエラーの原因を調査してください：
TypeError: Cannot read properties of undefined

関連情報の優先順位:
1. エラーパターンデータベース
2. 最近の変更履歴
3. 類似バグの修正履歴
4. 関連するテストケース
"

# 設計モード: アーキテクチャ情報を最優先
claude --context-priority=architecture "
新しいマイクロサービスを設計してください

関連情報の優先順位:
1. 既存のアーキテクチャパターン
2. サービス間通信の規約
3. データ一貫性の戦略
4. 運用・監視の要件
"

# レビューモード: 品質基準を最優先
claude --context-priority=quality "
このプルリクエストをレビューしてください

関連情報の優先順位:
1. コーディング規約
2. セキュリティチェックリスト
3. パフォーマンス基準
4. テスト要件
"
```

### /compactコマンド

`/compact`コマンドは、コンテキストウィンドウが満杯に近づいた際に、重要な情報を保持しながら全体のサイズを圧縮する高度な機能です。これは単純な削除ではなく、情報の本質を保ちながら冗長性を排除する知的な圧縮プロセスです。

compactコマンドの使用には戦略的な判断が必要です。圧縮により、詳細な文脈情報や微妙なニュアンスが失われる可能性があるため、どのタイミングで実行するか、どの情報を優先して保持するかを慎重に考慮する必要があります。

**compactコマンドの内部動作**

```markdown
# compact処理のフェーズ

## Phase 1: 情報分類
- 核心情報（プロジェクト概要、主要決定事項）
- 作業情報（現在のタスク、進行中の変更）
- 参考情報（過去の議論、詳細な実装）
- 一時情報（デバッグ出力、中間結果）

## Phase 2: 圧縮戦略
- 要約生成: 長い議論を主要ポイントに集約
- 重複除去: 同じ情報の複数表現を統合
- 階層化: 詳細情報を見出しレベルに圧縮
- 参照化: 頻繁でない情報を外部参照に変更

## Phase 3: 整合性検証
- 情報の関連性確認
- 重要度スコアの再評価
- 将来の参照可能性判定
```

**compactコマンドの最適な使用タイミング**

```bash
# 長時間セッションでの定期的な圧縮
claude "現在のコンテキストサイズは？"
# 出力: "172K/200K tokens (86%使用)"

# 圧縮の実行
claude "/compact --preserve=current_task,architecture_decisions --level=standard"

# 新しい大きなタスクの開始前
claude "/compact --aggressive --prepare-for=new_feature_development"

# 特定のセッション情報の保存
claude "/compact --save-session=authentication_review_20240115 --level=minimal"
```

**圧縮レベルの選択**

compactコマンドには複数の圧縮レベルがあり、状況に応じて適切なレベルを選択することが重要です。

```yaml
# 圧縮レベル設定例

minimal_compression:
  target_reduction: 20%
  preserve:
    - current_session_context
    - active_files
    - recent_decisions
  compress:
    - detailed_logs
    - redundant_examples
    - verbose_explanations

standard_compression:
  target_reduction: 50%
  preserve:
    - project_core_info
    - current_work_context
    - key_architectural_decisions
  compress:
    - detailed_implementation_history
    - extensive_code_examples
    - lengthy_discussions
  
aggressive_compression:
  target_reduction: 75%
  preserve:
    - essential_project_info
    - immediate_work_context
  compress:
    - all_historical_information
    - detailed_code_examples
    - background_discussions
  summarize:
    - key_decisions_only
    - critical_constraints_only
```

### 外部ドキュメント戦略

コンテキストウィンドウの制限を効果的に管理するために、外部ドキュメント戦略は不可欠な要素です。すべての情報をメインコンテキストに保持するのではなく、参照時にのみロードする「遅延ロード」アプローチを採用することで、メモリ効率と情報アクセシビリティのバランスを最適化できます。

この戦略では、ドキュメントを重要度と参照頻度に基づいて分類し、それぞれに適した保存と参照メカニズムを設計します。頻繁に参照される重要な情報はインラインで保持し、詳細な技術仕様や過去の履歴は外部ファイルとして管理します。

**ドキュメント階層の設計**

```
docs/
├── core/                    # 常時参照ドキュメント
│   ├── project-overview.md
│   ├── architecture-decisions.md
│   └── coding-standards.md
│
├── reference/               # 必要時参照ドキュメント
│   ├── api-specifications/
│   ├── database-schema/
│   └── deployment-guides/
│
├── historical/              # 履歴・アーカイブ
│   ├── meeting-notes/
│   ├── decision-logs/
│   └── version-history/
│
└── external/               # 外部リソースへの参照
    ├── vendor-docs.md
    ├── compliance-requirements.md
    └── industry-standards.md
```

**インテリジェントな参照システム**

```markdown
# CLAUDE.mdでの外部参照例

## データベース設計
詳細なスキーマ情報は@docs/reference/database-schema/current.md を参照

### 主要テーブル概要
- users: 認証・プロフィール情報
- projects: プロジェクト基本情報  
- tasks: タスク管理
- audit_logs: 操作履歴

詳細な関連情報:
- ER図: @docs/reference/database-schema/er-diagram.png
- インデックス設計: @docs/reference/database-schema/indexes.md
- パフォーマンス要件: @docs/reference/performance/database-requirements.md

## API設計
基本方針はRESTful設計に準拠。詳細仕様は@docs/reference/api-specifications/ 参照

### 認証エンドポイント
```
POST /api/auth/login
POST /api/auth/logout  
POST /api/auth/refresh
```

完全なAPI仕様書: @docs/reference/api-specifications/auth-api.yaml
```

**動的ドキュメントロード**

```bash
# 必要時に外部ドキュメントを動的にロード
claude "データベースのパフォーマンス問題を調査してください
        @docs/reference/database-schema/current.md
        @docs/reference/performance/db-metrics-last-week.json
        を参照して分析してください"

# 複数ドキュメントの統合参照
claude "新しい機能のセキュリティレビューを実施してください
        参照ドキュメント:
        @docs/reference/security/security-guidelines.md
        @docs/reference/compliance/gdpr-checklist.md
        @docs/historical/security-incidents/lessons-learned.md"

# 条件付きドキュメントロード
claude "if (performance_issue_detected) {
          load @docs/reference/performance/troubleshooting-guide.md
        }
        if (security_concern_raised) {
          load @docs/reference/security/incident-response.md
        }"
```

**バージョン管理統合**

外部ドキュメント戦略をより効果的にするために、Gitのバージョン管理システムと統合することで、ドキュメントの変更履歴や関連性を自動的に追跡できます。

```bash
# バージョン管理統合による智的参照
claude "ユーザー認証機能の仕様変更を検討中です
        
        現在の仕様: @docs/reference/auth/current-spec.md
        変更履歴: $(git log --oneline docs/reference/auth/)
        関連する過去の議論: @docs/historical/auth-decisions/
        
        変更の影響範囲を分析し、移行計画を作成してください"

# 自動更新通知システム
claude "ウォッチ対象ドキュメントの変更を検出しました:
        
        変更ファイル: docs/reference/api-specifications/user-api.yaml
        変更内容: $(git diff HEAD~1 docs/reference/api-specifications/user-api.yaml)
        
        この変更がコードベースに与える影響を分析してください"
```

## 6.3 プロジェクト固有の設定

### インポート機能

Claude Codeのインポート機能は、CLAUDE.mdファイルの可読性と保守性を向上させる強力な仕組みです。巨大な設定ファイルを単一のファイルで管理することの問題を解決し、関心事の分離と再利用性を実現します。この機能により、チームは専門領域ごとに分離された設定を維持しながら、必要に応じて統合的な視点を提供できます。

インポート機能の真の価値は、情報の階層化と専門化にあります。フロントエンド開発者はUI関連の設定に、バックエンド開発者はAPI設計に、そしてDevOpsエンジニアはインフラ設定に集中できます。しかし、Claude Codeは全体像を把握し、異なる領域間の関連性や依存関係を理解することができます。

**基本的なインポート構文**

```markdown
# CLAUDE.md - メインプロジェクト設定

# プロジェクト概要
このプロジェクトは企業向けCRMシステムです。

# 技術設定のインポート
@import architecture/system-design.md
@import frontend/react-standards.md  
@import backend/api-guidelines.md
@import database/schema-conventions.md
@import devops/deployment-config.md

# セキュリティ設定のインポート
@import security/auth-requirements.md
@import security/data-protection.md
@import compliance/gdpr-guidelines.md

# 開発プロセスのインポート
@import processes/git-workflow.md
@import processes/code-review-standards.md
@import processes/testing-strategy.md
```

**条件付きインポート**

プロジェクトの段階や環境に応じて、異なる設定をインポートする機能は、複雑なプロジェクトの管理に不可欠です。

```markdown
# 環境別設定のインポート
@import environments/development.md
@import environments/staging.md if STAGE=staging
@import environments/production.md if STAGE=production

# 機能フラグ基準のインポート  
@import features/beta-features.md if BETA_ENABLED=true
@import features/experimental.md if EXPERIMENTAL_MODE=true

# チーム別設定
@import teams/frontend-team.md if ROLE=frontend
@import teams/backend-team.md if ROLE=backend  
@import teams/fullstack-guidelines.md if ROLE=fullstack

# プロジェクトフェーズ別
@import phases/mvp-constraints.md if PROJECT_PHASE=mvp
@import phases/scaling-considerations.md if PROJECT_PHASE=scale
@import phases/maintenance-mode.md if PROJECT_PHASE=maintenance
```

**動的インポート**

プロジェクトの状態や最近の活動に基づいて、関連する設定を動的にインポートする機能により、コンテキストの関連性を最大化できます。

```markdown
# 最近の変更に基づく動的インポート
@import-recent security/ if has_security_changes_last_7_days
@import-recent performance/ if performance_issues_detected
@import-recent integration/ if external_api_changes

# 作業内容に基づくインポート
@import database/optimization.md if working_on(database_performance)
@import frontend/accessibility.md if working_on(ui_components)
@import backend/scaling.md if working_on(api_endpoints)

# エラーパターンに基づくインポート
@import troubleshooting/auth-issues.md if recent_auth_errors > 5
@import troubleshooting/database-problems.md if db_connection_failures
@import troubleshooting/deployment-fixes.md if deploy_failures
```

### モジュール化

大規模プロジェクトにおいて、設定の複雑さを管理するためには、モジュール化アプローチが不可欠です。各モジュールは特定の関心事に焦点を当て、他のモジュールとの明確なインターフェースを定義します。これにより、チームメンバーは自身の専門領域に集中しながら、全体の一貫性を保つことができます。

**アーキテクチャモジュール**

```markdown
# architecture/system-design.md

## システムアーキテクチャ

### 全体構成
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   API Gateway   │    │   Microservices │
│   (React SPA)   │◄───┤   (Kong)        │◄───┤   (Node.js)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CDN           │    │   Load Balancer │    │   Database      │
│   (CloudFront)  │    │   (ALB)         │    │   (PostgreSQL)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 設計原則
1. **単一責任**: 各サービスは一つの機能ドメインを担当
2. **疎結合**: サービス間の依存を最小化
3. **高可用性**: 単一障害点の排除
4. **スケーラビリティ**: 水平スケールアウト対応
5. **監視可能性**: 分散トレーシング・ログ集約

### サービス分割基準
- **ユーザー管理サービス**: 認証・認可・プロフィール
- **顧客管理サービス**: CRM機能・顧客データ
- **プロジェクト管理サービス**: タスク・進捗・リソース
- **通知サービス**: メール・プッシュ・SMS
- **レポートサービス**: 分析・レポート生成

### データ整合性戦略
- **強整合性**: 重要な業務データ（顧客情報、決済）
- **結果整合性**: 分析データ、通知履歴
- **補償トランザクション**: 分散処理での障害回復
```

**セキュリティモジュール**

```markdown
# security/auth-requirements.md

## 認証・認可要件

### 認証方式
**プライマリ**: JWT + リフレッシュトークン
- アクセストークン有効期限: 15分
- リフレッシュトークン有効期限: 7日
- トークン回転: セキュリティ強化

**セカンダリ**: OAuth 2.0 / OpenID Connect
- Google Workspace連携
- Microsoft Azure AD連携
- SAML 2.0対応（エンタープライズ）

### 認可モデル
**RBAC (Role-Based Access Control)**
```yaml
roles:
  admin:
    permissions: ["*"]
    description: "システム全体の管理者"
    
  manager:
    permissions: 
      - "team:read"
      - "team:write" 
      - "project:read"
      - "project:write"
      - "report:read"
    description: "チーム・プロジェクト管理者"
    
  developer:
    permissions:
      - "project:read"
      - "task:read" 
      - "task:write"
    description: "開発者"
    
  viewer:
    permissions:
      - "project:read"
      - "task:read"
    description: "読み取り専用ユーザー"
```

### セキュリティヘッダー
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
```

### パスワードポリシー
- 最小12文字
- 英数字記号の組み合わせ必須
- 辞書攻撃対策（一般的な単語の禁止）
- 過去5回のパスワード再利用禁止
- アカウントロック: 5回失敗で30分間
```

**開発プロセスモジュール**

```markdown
# processes/git-workflow.md

## Git ワークフロー

### ブランチ戦略 (GitHub Flow改良版)

```
main ──┬── feature/user-auth ──┐
       │                       ├─► PR ──► main
       ├── feature/dashboard ───┘
       │
       ├── hotfix/login-bug ────► main
       │
       └── release/v2.1.0 ──────► main
```

### ブランチ命名規則
- `feature/機能名`: 新機能開発
- `fix/バグ名`: バグ修正
- `refactor/対象`: リファクタリング
- `docs/対象`: ドキュメント更新
- `hotfix/緊急修正名`: 本番緊急対応
- `release/vX.Y.Z`: リリース準備

### コミットメッセージ (Conventional Commits)
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Type**:
- `feat`: 新機能
- `fix`: バグ修正  
- `docs`: ドキュメント
- `style`: フォーマット
- `refactor`: リファクタリング
- `test`: テスト
- `chore`: ビルド・設定

**例**:
```
feat(auth): add OAuth2 Google integration

- Add Google OAuth2 provider
- Update login component with Google button
- Add necessary environment variables

Closes #123
```

### プルリクエスト要件
1. **必須チェック項目**
   - [ ] 機能要件を満たしている
   - [ ] テストが追加・更新されている
   - [ ] ドキュメントが更新されている
   - [ ] セキュリティレビューを通過
   - [ ] パフォーマンステストが合格

2. **レビュアー要件**
   - 機能PR: 2名以上のレビュー必須
   - バグ修正: 1名以上のレビュー
   - ホットフィックス: シニア開発者の即座レビュー

3. **マージ要件**
   - 全てのチェックがPASS
   - コンフリクトの解決
   - 最新のmainブランチにリベース
```

### チーム共有

効果的なチーム共有を実現するためには、技術的な仕組みだけでなく、組織的なプロセスと文化的な要素も重要です。設定の共有は単なる情報の伝達ではなく、チームの知識と経験の集約、そして継続的な改善のサイクルを構築することです。

**段階的な共有戦略**

```markdown
# Team Adoption Strategy

## Phase 1: Core Team (Week 1-2)
### 対象
- テックリード (1名)
- シニア開発者 (2名)  
- DevOpsエンジニア (1名)

### 活動
- CLAUDE.mdの初期構造設計
- 主要モジュールの作成
- 基本的なインポート構造の確立
- 初期のベストプラクティス文書化

### 成果物
- 基本CLAUDE.md構造
- architecture/core-decisions.md
- processes/basic-workflow.md
- チーム導入ガイド

## Phase 2: Development Team (Week 3-4)
### 対象
- 全開発者 (8名)
- QAエンジニア (2名)

### 活動  
- 個人別CLAUDE.local.md作成支援
- フィードバック収集と設定改善
- 実際のタスクでの使用開始
- 困った時のサポート体制確立

### 成果物
- 個人設定テンプレート
- FAQ文書
- トラブルシューティングガイド
- 使用状況メトリクス

## Phase 3: Extended Team (Week 5-8)
### 対象
- プロダクトマネージャー
- デザイナー
- ビジネスアナリスト

### 活動
- 非技術者向けの設定作成
- クロスファンクショナルなワークフロー
- 全体的な効果測定
- 長期運用体制の確立

### 成果物
- 役割別設定ガイド
- 効果測定レポート
- 改善提案リスト
- 継続的改善プロセス
```

**知識共有のメカニズム**

```markdown
# Knowledge Sharing Mechanisms

## 定期的な知識共有セッション

### Weekly Tech Talk (毎週金曜日 15:00-16:00)
**フォーマット**: 
- Claude Code活用事例共有 (10分)
- 新しい設定やコマンドの紹介 (10分)
- 困ったことと解決方法 (10分)
- ディスカッション (20分)

**記録方法**:
- セッション内容をdocs/knowledge-sharing/に記録
- 有用なティップスはCLAUDE.mdに統合
- 繰り返し出る問題はFAQに追加

### Monthly Deep Dive (毎月第3金曜日 14:00-17:00)
**テーマ例**:
- 複雑なプロンプトエンジニアリング技法
- 大規模プロジェクトでのメモリ管理
- CI/CD統合の高度な活用
- セキュリティとClaude Code

## 非同期知識共有

### Slack統合
```yaml
# .claude/slack-integration.yaml
channels:
  claude-tips:
    purpose: "日々の発見と小技の共有"
    format: "短文 + スクリーンショット"
    
  claude-help:
    purpose: "質問と回答"
    sla: "24時間以内の回答"
    
  claude-showcase:
    purpose: "優れた活用事例の共有"
    format: "詳細な説明 + 効果の定量化"
```

### Git統合の活用
```bash
# 設定変更の追跡とレビュー
git log --oneline CLAUDE.md
git blame CLAUDE.md | grep "特定の設定"

# 設定変更のプルリクエスト
git checkout -b update/claude-settings
# CLAUDE.mdを更新
git add CLAUDE.md
git commit -m "docs: update database connection settings

- 新しい接続プール設定を追加
- パフォーマンス向上のため
- 参考: docs/performance/db-optimization.md"

# レビューリクエスト
gh pr create --title "Claude設定更新: データベース最適化" \
  --body "データベース接続の最適化設定を追加しました。
  全チームメンバーがこの設定を使用してください。"
```

**継続的改善プロセス**

```markdown
# Continuous Improvement Process

## 月次レビューサイクル

### Week 1: データ収集
- 使用状況メトリクスの収集
- チームメンバーからのフィードバック
- パフォーマンス指標の測定
- 困った点の整理

### Week 2: 分析と優先順位付け
- 効果的な設定パターンの特定
- 問題となっている箇所の分析  
- 改善案の検討と優先順位付け
- 新しいベストプラクティスの発見

### Week 3: 実装と検証
- 改善案の実装
- A/Bテストでの検証
- 小規模チームでの試験運用
- フィードバックの収集

### Week 4: 展開と文書化
- 効果が確認された改善の全体展開
- 設定の更新とドキュメント化
- ベストプラクティスの共有
- 次月の改善テーマ設定

## 効果測定指標

### 生産性指標
- 開発タスクの完了時間
- コードレビューの速度
- バグ修正の時間
- 新機能のリードタイム

### 品質指標  
- コード品質スコア
- テストカバレッジ
- 本番障害の頻度
- セキュリティインシデント

### 満足度指標
- チーム満足度調査
- Claude Code使用頻度
- 推奨度 (NPS)
- 学習コストの評価
```

この包括的なメモリ管理とコンテキスト最適化により、Claude Codeは単なるツールから、チーム全体の知識を共有し、継続的に進化する「集合知システム」へと成長します。適切に管理されたメモリシステムは、個人の生産性向上だけでなく、組織全体の学習能力と適応力を大幅に向上させる基盤となるのです。

---

次章では、Claude CodeとCursor、GitHub Copilotなどの他のAIコーディングツールとの詳細な比較を行います。それぞれのツールの強みと特徴、適用場面、そして2025年の最新動向について解説し、プロジェクトやチームに最適なツール選択の指針を提供します。