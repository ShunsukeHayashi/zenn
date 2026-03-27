---
title: "【爆速】Figma×AIでデザイン開発連携！デザインシステム自動生成の衝撃"
emoji: "🎨"
type: "tech"
topics: ["figma", "ai", "デザインシステム", "フロントエンド", "開発効率化"]
published: true
---

フリーランスになって7年。デザイナーとの「翻訳コスト」で何度消耗したかわからない。

「このボタンのホバー色、Figmaには載ってないんだけど...」
「コンポーネントの余白、実装と違うって言われたんだけど何px？」
「デザイントークン？それ何ですか？」

こんな会話を繰り返しながら、私は毎週2〜3時間をデザインと実装の「突き合わせ作業」に費やしていた。ところが去年からFigmaのAI機能を本格活用し始めたことで、この状況が劇的に変わった。

この記事では、**私が実際に失敗しながら確立したFigma×AIのデザイン開発連携ワークフロー**を、コード込みで全部公開する。

---

## デザイナーとのコミュニケーション課題、あなたも抱えていませんか？

典型的なフロントエンド開発の現場では、デザイナーとエンジニアの間に常に「解釈の溝」が生まれる。

Figmaのデザインファイルを渡されても、実装に必要な情報が全部そろっているケースはほぼない。フォントサイズは「14」と書いてあるだけでrem換算が必要だし、カラーパレットに命名規則がなければコンポーネントごとにバラバラな変数名が生まれる。レスポンシブ対応のブレークポイントは「なんとなく」Figmaの複数フレームから読み取るしかない。

私がフリーランスとして複数のスタートアップと仕事をしてきた中で、この問題は規模に関係なく発生していた。チームが小さいほど、むしろ「なんとなく」で乗り切ってしまうぶん、あとでツケが回ってくる。

そんな状況を打開したのが、FigmaのAI機能との出会いだった。

---

## FigmaのAI機能、何ができるのか

2024年以降、FigmaはAI機能を急速に強化している。現時点で特に実用的なのは以下の3つだ。

- **Figma Dev Mode + コード生成**: コンポーネントを選択するだけでReact/CSS/iOSコードを自動生成
- **Variables（変数機能）**: デザイントークンをFigma内で一元管理し、JSONエクスポートが可能
- **Auto Layout + AI提案**: レイアウト構造をAIが解析し、最適なFlexboxコードを提案

これだけ聞くと「夢のような話」に聞こえるかもしれない。実際、私も最初はそう思った。

しかし現実はそう甘くなかった。

---

## 失敗談1：デザインと実装の乖離問題

最初にFigma Dev Modeのコード生成を試したとき、私は喜び勇んで生成されたCSSをそのままプロダクションに突っ込んだ。

```css
/* Figma自動生成コード（問題あり） */
.button-primary {
  width: 200px;
  height: 48px;
  background: #3B82F6;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  color: #FFFFFF;
}
```

一見問題なさそうに見える。だが、このコードには致命的な問題がある。

**ピクセル値がハードコードされている**。レスポンシブ対応をまったく考慮していないため、モバイル端末でレイアウトが崩壊した。さらにデザイナーが後日カラーを `#3B93FF` に変更したとき、このハードコード値は自動では更新されない。

修正に2日かかった。しかも同じコードが10コンポーネントに散在していたため、全部手作業で直す羽目になった。

**教訓**: Figmaの生成コードは「スタート地点」であって「完成形」ではない。

---

## 失敗談2：AI生成コードの品質問題

次の失敗は、外部AIツール（当時はGPT-4ベースのもの）にFigmaのスクリーンショットを渡してコードを生成させたときに起きた。

生成されたTypeScriptコンポーネントは一見きれいだったが、実際にプロジェクトに組み込むと型エラーが大量発生した。

```typescript
// AI生成コード（問題あり）
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant: string; // ← string型では型安全性ゼロ
  size: string;    // ← 同上
}

const Button = ({ label, onClick, variant, size }: ButtonProps) => {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
};
```

`variant` と `size` が `string` 型のため、存在しない値を渡してもコンパイル時にエラーにならない。実行時に壊れて初めて気づく、というパターンを3回繰り返した。

TypeScriptで型安全なコンポーネントを書く方法については、以前書いた記事も参考にしてほしい。

https://zenn.dev/miyabi_society/articles/claude-code-typescript-type-safety

---

## 失敗談3：変数命名の統一失敗

3つ目の失敗が一番地味で、かつ一番根深かった。

Figmaの変数機能でデザイントークンを作ったはいいが、命名規則を決めずに複数のデザイナーが好き勝手に命名した結果、こんな状態になった。

```json
{
  "color/primary": "#3B82F6",
  "colors/brand-blue": "#3B82F6",
  "Brand/Primary Blue": "#3B82F6",
  "button-bg": "#3B82F6"
}
```

**同じ色が4つの名前で登録されている**。エクスポートしたJSONをCSSカスタムプロパティに変換するスクリプトを書いたとき、このカオスに直面した。フロントエンドのCSS変数は最終的に手作業で整理するしかなく、1週間の作業が無駄になった。

---

## 改善後の数値

失敗を重ねてワークフローを確立した結果、以下の改善を実現した。

| 指標 | Before | After | 改善率 |
|------|--------|-------|--------|
| デザイン確認・突き合わせ時間 | 週3時間 | 週0.5時間 | **83%削減** |
| コンポーネント実装速度 | 1個あたり2時間 | 1個あたり45分 | **63%短縮** |
| デザインと実装の差異発生頻度 | 月12件 | 月2件 | **83%削減** |
| デザイントークン更新の反映時間 | 半日〜1日 | 15分 | **95%削減** |

特にデザイントークン更新の反映時間の削減は、導入初月で体感できた。

---

## 実践5ステップ：Figma×AIデザイン開発連携

### ステップ1：Figma Variablesでデザイントークンを整備する

まずFigma側の命名規則を統一する。私が使っているルールはこうだ。

```
カテゴリ/スケール/バリアント
例: color/brand/primary
    spacing/base/md
    typography/heading/xl
```

Figmaの変数コレクションで上記の命名規則を徹底し、JSONエクスポートすると以下のような構造になる。

```json
{
  "color": {
    "brand": {
      "primary": { "value": "#3B82F6", "type": "color" },
      "secondary": { "value": "#8B5CF6", "type": "color" }
    },
    "neutral": {
      "100": { "value": "#F3F4F6", "type": "color" },
      "900": { "value": "#111827", "type": "color" }
    }
  },
  "spacing": {
    "base": {
      "sm": { "value": 8, "type": "dimension" },
      "md": { "value": 16, "type": "dimension" },
      "lg": { "value": 24, "type": "dimension" }
    }
  }
}
```

### ステップ2：デザイントークンをTypeScriptに変換する

エクスポートしたJSONを型安全なTypeScriptに変換するスクリプトを一度作っておくと、以降の更新が自動化できる。

```typescript
// scripts/generate-tokens.ts
import designTokens from '../figma-tokens.json';

type ColorScale = {
  [key: string]: { value: string; type: 'color' };
};

type SpacingScale = {
  [key: string]: { value: number; type: 'dimension' };
};

// デザイントークンをCSS変数に変換
function generateCssVariables(tokens: typeof designTokens): string {
  const lines: string[] = [':root {'];

  // カラートークン展開
  Object.entries(tokens.color).forEach(([category, scales]) => {
    Object.entries(scales as ColorScale).forEach(([scale, token]) => {
      lines.push(`  --color-${category}-${scale}: ${token.value};`);
    });
  });

  // スペーシングトークン展開
  Object.entries(tokens.spacing.base as SpacingScale).forEach(([scale, token]) => {
    lines.push(`  --spacing-${scale}: ${token.value}px;`);
  });

  lines.push('}');
  return lines.join('\n');
}

// TypeScript型定義も自動生成
type ColorToken = 'brand-primary' | 'brand-secondary' | 'neutral-100' | 'neutral-900';
type SpacingToken = 'sm' | 'md' | 'lg';

export function color(token: ColorToken): string {
  return `var(--color-${token})`;
}

export function spacing(token: SpacingToken): string {
  return `var(--spacing-${token})`;
}
```

このスクリプトをCIに組み込めば、Figmaを更新してJSONを再エクスポートするだけで、コードベースのデザイントークンが自動更新される。

### ステップ3：Figma Dev Modeで構造を読み取る

Figma Dev Modeで生成されるコードは「完成形」ではなく「構造の読み取り」に使う。具体的には以下の情報だけを取得する。

- コンポーネントの階層構造（どのFlexboxが入れ子になっているか）
- ギャップ・パディングの値（px値をデザイントークンに対応させる）
- フォントウェイト・ラインハイトの値

### ステップ4：AIでコンポーネントを型安全に実装する

Cursor AIなどのエディタと組み合わせて、ステップ1〜3で整備した情報を元にコンポーネントを生成する。

```typescript
// 型安全なButtonコンポーネント（改善後）
type ButtonVariant = 'primary' | 'secondary' | 'ghost';
type ButtonSize = 'sm' | 'md' | 'lg';

interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: ButtonVariant;
  size?: ButtonSize;
  disabled?: boolean;
}

const sizeStyles: Record<ButtonSize, string> = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg',
};

const variantStyles: Record<ButtonVariant, string> = {
  primary: 'bg-[var(--color-brand-primary)] text-white hover:opacity-90',
  secondary: 'bg-[var(--color-brand-secondary)] text-white hover:opacity-90',
  ghost: 'bg-transparent border border-[var(--color-brand-primary)] text-[var(--color-brand-primary)]',
};

export const Button = ({
  label,
  onClick,
  variant = 'primary',
  size = 'md',
  disabled = false,
}: ButtonProps) => {
  return (
    <button
      className={`rounded-lg font-semibold transition-opacity ${sizeStyles[size]} ${variantStyles[variant]}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
};
```

`variant` と `size` がユニオン型になっているため、存在しない値を渡すとコンパイルエラーになる。Cursor AIを使ったコンポーネント生成の詳細については、以前書いたレビュー記事も参照してほしい。

https://zenn.dev/miyabi_society/articles/cursor-ai-editor-review

### ステップ5：GitHub Actionsで変更を自動検知・反映する

Figmaの変数が更新されたとき、自動でCSSファイルを再生成してPRを作るGitHub Actionsを設定する。

```yaml
# .github/workflows/sync-design-tokens.yml
name: Sync Design Tokens
on:
  schedule:
    - cron: '0 9 * * 1'  # 毎週月曜9時
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      - name: Export Figma tokens
        run: npx figma-export-tokens --token ${{ secrets.FIGMA_TOKEN }}
      - name: Generate CSS variables
        run: npx ts-node scripts/generate-tokens.ts
      - name: Create PR if changed
        uses: peter-evans/create-pull-request@v6
        with:
          title: 'chore: sync design tokens from Figma'
          branch: 'chore/sync-design-tokens'
```

---

## ツール比較表

| ツール | 得意なこと | 苦手なこと | 月額費用 |
|--------|-----------|-----------|---------|
| Figma Dev Mode | 構造読み取り・CSS生成 | 型安全性、プロジェクト固有の規約 | $15〜 |
| Cursor AI | コンポーネント実装・リファクタ | デザインファイルの直接参照 | $20 |
| Tokens Studio | デザイントークン管理・同期 | 学習コスト | 無料〜$24 |
| Claude Code | 複雑なロジック・型設計 | UI的な判断 | $20 |

私の現在のメインスタックは **Figma Dev Mode + Tokens Studio + Cursor AI** の組み合わせだ。この3つが揃うと、デザイナーからFigmaが更新されてから実装に反映されるまでのリードタイムが、従来の「数日〜1週間」から「数時間以内」に短縮される。

---

## まとめ：3つの失敗から学んだこと

フリーランス7年で積み重ねた失敗を整理すると、Figma×AI連携の落とし穴は一貫していた。

1. **AIの生成物を完成形として使わない** — 構造読み取りのスタート地点として活用する
2. **型安全性を犠牲にしない** — `string` 型の乱用は後で必ず爆発する
3. **命名規則を先に決める** — トークン名のカオスは後から整理できない

逆に言えば、この3点を押さえておけば、Figma×AIのワークフローは確実にデザイン開発連携の生産性を引き上げてくれる。

デザイナーとの「翻訳コスト」で消耗していた週3時間が、今では週0.5時間になった。その分を新機能の設計に使えるようになったことが、フリーランスとしての最大の変化だ。

まずはステップ1のデザイントークン整備から始めてみてほしい。命名規則さえ統一できれば、残りは自動化できる。

---

*この記事で紹介したコードはすべて実際のプロジェクトで使用しているものを元にしています。TypeScriptの型設計については[こちらの記事](https://zenn.dev/miyabi_society/articles/claude-code-typescript-type-safety)も合わせてどうぞ。*
