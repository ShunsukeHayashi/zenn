---
title: "【完全解説】Claude Code×PrismaでDB設計革命！スキーマ作成が10倍速"
emoji: "🗄️"
type: "tech"
topics: ["claudecode", "prisma", "typescript", "データベース", "ai"]
published: true
---

こんにちは！フリーランスエンジニア歴7年のハヤシシュンスケです。

DB設計って、本当に面倒ですよね。要件定義からエンティティ抽出、リレーション設計、インデックス最適化、マイグレーション管理…気づいたら半日以上溶けていた、なんて経験、みなさんにもあるんじゃないでしょうか。

私自身、Prismaを使い始めてからも「スキーマ設計が正しいかどうか自信がない」「N+1問題を後から発見して大幅改修になった」という苦い思い出があります。

そんな私が Claude Code と Prisma を組み合わせたところ、**スキーマ作成の工数が70%削減、設計品質のレビュー時間が90%短縮**という衝撃的な結果が出ました。今日はその全貌をお伝えします。

---

## 🚀 衝撃の初体験：複雑なDB設計を1時間で完成させた話

半年前のことです。ECサービスのバックエンドをゼロから作る案件を受注しました。要件は「ユーザー管理・商品管理・注文管理・在庫管理・レビュー機能・クーポン機能」の6機能。

通常なら、この規模のDB設計だけで**最低でも丸1日**かかります。エンティティ図を書いて、リレーションを整理して、インデックスを検討して…という作業が延々と続くわけです。

ところが Claude Code に要件をまるごと投げてみたところ、なんと**1時間以内**に完全なPrismaスキーマのドラフトが出来上がりました。しかも、「中間テーブルが必要なリレーションはこうした方がいい」「この検索パターンならこのインデックスを張るべき」という理由付きで。

最初は信じられなくて、何度も確認しました。でも実際に動かしてみると、本当に問題なく動く。このとき初めて「AIと一緒にDB設計をする時代が来た」と確信しました。

---

## ⚠️ 私がやらかした3つの大失敗

ただ、最初から順調だったわけではありません。むしろかなり痛い目を見ました。Claude Code + Prisma の活用で、私がやらかした失敗を正直にシェアします。

### 失敗1：N+1問題を見落として本番クラッシュ

初期の頃、Claude Code が生成したスキーマをそのまま使い、クエリも自分で書いていました。ユーザー一覧を取得して、それぞれの注文数を表示するページで、こんなコードを書いていたんです。

```typescript
// ❌ N+1問題が発生するコード（やらかした実例）
const users = await prisma.user.findMany();
for (const user of users) {
  const orderCount = await prisma.order.count({
    where: { userId: user.id },
  });
  user.orderCount = orderCount; // TypeScriptが怒るのも無視していた
}
```

ユーザー数が100人のときは気づきませんでした。1000人になった時点で、ページの応答時間が**30秒を超えてクラッシュ**。本番環境で起きて、クライアントへの謝罪対応が発生しました。

Claude Code に「N+1問題が起きていないかレビューして」と最初から頼んでいれば防げた失敗です。

### 失敗2：インデックス設計を後回しにして全テーブルスキャンが発生

「インデックスは後で考えよう」と思っていた時期がありました。Prismaのマイグレーションで後から追加できるし、と甘く見ていたんです。

結果、検索機能を実装した時点で、`EXPLAIN ANALYZE`を見ると全テーブルスキャンが発生していることが発覚。データが10万件になっていたため、インデックス追加のマイグレーションを本番で走らせると**テーブルロックが20分以上**発生するという事態に。

複合インデックスの設計は、スキーマ設計の最初にやっておくべきでした。

### 失敗3：マイグレーション手順を間違えてデータが消えそうになった

開発環境で `prisma migrate dev` ばかり使っていたため、本番での `prisma migrate deploy` の挙動の違いを理解していませんでした。

スキーマ変更時に「カラム名変更」と「型変更」を同時にやろうとして、Prismaが「DROP → CREATE」として解釈。**本番データが消えかけました**。ステージング環境で気づいたので実害はゼロでしたが、冷や汗どころの話ではありませんでした。

マイグレーションの生成前に Claude Code で必ずレビューする習慣をつけてから、このリスクはゼロになりました。

---

## 📊 劇的な数値改善：Before / After の具体的データ

3ヶ月間、同種の案件で Claude Code + Prisma 活用の有無を比較した結果です。

| 指標 | Before（従来手法） | After（Claude Code活用） | 改善率 |
|------|----------|-----------|------|
| スキーマ設計時間 | 8時間 | 0.8時間 | **90%削減** |
| レビュー・修正時間 | 3時間 | 0.5時間 | **83%削減** |
| N+1問題の発生件数 | 月3〜5件 | 月0件 | **100%削減** |
| マイグレーション起因のバグ | 月2件 | 0件 | **100%削減** |
| 全工数（設計〜初回リリース） | 40時間 | 28時間 | **30%削減** |

特に「N+1問題の発生件数」がゼロになったのは、Claude Code がレビュー時に必ず指摘してくれるようになったからです。設計の早い段階で問題を発見できるため、手戻りコストも大幅に下がりました。

---

## 🛠️ 実践的な手順：Claude Code + Prisma 完全活用ガイド

ここからが本題です。私が実際にやっている手順を、Step by Step でお伝えします。

### STEP 1：要件からスキーマ設計を生成する

まず、Claude Code に要件を日本語でそのまま投げます。ポイントは**「利用シナリオ」まで含めること**です。

```
プロンプト例：
「以下の要件でPrismaスキーマを設計してください。

## システム概要
ユーザーが商品を購入できるECサービス

## 機能要件
- ユーザー登録・ログイン（メール/パスワード、Google OAuth）
- 商品一覧・詳細・検索（カテゴリ、価格帯、評価でフィルタ）
- カート・注文・決済
- レビュー投稿（星1〜5、テキスト）
- クーポン適用

## 利用シナリオ
- 商品検索は商品名・カテゴリ・タグで頻繁に行われる
- ユーザーの注文履歴は降順でページネーションして表示
- 管理画面で売上集計（日次・月次）を行う

PostgreSQLを使用します。インデックスの設計も含めてください。」
```

Claude Code はこれをもとに Prisma スキーマを生成します。生成されたスキーマの一部がこちらです。

```prisma
// schema.prisma（生成例）
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id           Int      @id @default(autoincrement())
  email        String   @unique
  passwordHash String?
  googleId     String?  @unique
  name         String
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  orders  Order[]
  reviews Review[]
  carts   Cart[]

  @@index([email])
}

model Product {
  id          Int      @id @default(autoincrement())
  name        String
  description String?
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  categoryId  Int
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  category   Category       @relation(fields: [categoryId], references: [id])
  tags       ProductTag[]
  reviews    Review[]
  orderItems OrderItem[]

  @@index([categoryId])
  @@index([price])
  @@index([name])
}

model Order {
  id          Int         @id @default(autoincrement())
  userId      Int
  status      OrderStatus @default(PENDING)
  totalAmount Decimal     @db.Decimal(10, 2)
  couponId    Int?
  createdAt   DateTime    @default(now())
  updatedAt   DateTime    @updatedAt

  user      User        @relation(fields: [userId], references: [id])
  coupon    Coupon?     @relation(fields: [couponId], references: [id])
  orderItems OrderItem[]

  @@index([userId, createdAt(sort: Desc)])
  @@index([status, createdAt])
}

enum OrderStatus {
  PENDING
  PAID
  SHIPPED
  DELIVERED
  CANCELLED
}
```

注目してほしいのは `@@index([userId, createdAt(sort: Desc)])` です。「注文履歴を降順ページネーション」という要件から、Claude Code が自動的に複合インデックスを提案してくれています。

### STEP 2：Prismaスキーマのレビューと最適化

生成されたスキーマを、もう一度 Claude Code にレビューさせます。

```
プロンプト例：
「このPrismaスキーマをレビューしてください。
特に以下の観点でチェックをお願いします：
- N+1問題が起きやすいリレーション設計になっていないか
- インデックスの抜け漏れ
- データ型の選択が適切か（特にDecimalとFloat）
- ソフトデリートが必要なモデルはあるか
- 将来の拡張性（マルチテナント対応など）
[スキーマをここに貼り付け]」
```

### STEP 3：マイグレーション生成と実行

スキーマが固まったら、マイグレーションを生成します。

```bash
# 開発環境：スキーマを適用してマイグレーションファイルを生成
npx prisma migrate dev --name init

# マイグレーションファイルの内容を確認（必須！）
cat prisma/migrations/[timestamp]_init/migration.sql

# 本番環境：マイグレーションを適用（ファイルは変更しない）
npx prisma migrate deploy

# Prisma Clientを再生成
npx prisma generate
```

**重要**: マイグレーションファイルは必ず中身を確認してください。Prismaが意図しない `DROP` を生成していないかチェックします。不安なら Claude Code に `migration.sql` を貼り付けて「このSQLは安全ですか？」と聞くのが確実です。

### STEP 4：リレーション設計の検証

Prisma Client を使ったクエリを書いたら、N+1問題が起きていないか検証します。

```typescript
// ✅ 正しいクエリ：includeでリレーションを一括取得
const usersWithOrders = await prisma.user.findMany({
  include: {
    orders: {
      orderBy: { createdAt: "desc" },
      take: 5,
      include: {
        orderItems: {
          include: { product: true },
        },
      },
    },
  },
});

// ❌ N+1問題が起きるコード（やってはいけない）
const users = await prisma.user.findMany();
for (const user of users) {
  const orders = await prisma.order.findMany({
    where: { userId: user.id }, // ループ内でクエリ → N+1
  });
}
```

Claude Code に「このクエリでN+1問題は起きていますか？」と確認する習慣をつけましょう。

### STEP 5：クエリ最適化

実際に使うクエリパターンをすべて Claude Code に見せて、最適化してもらいます。

```typescript
// Claude Codeが提案した最適化版：売上集計クエリ
const monthlySales = await prisma.order.groupBy({
  by: ["createdAt"],
  where: {
    status: "PAID",
    createdAt: {
      gte: new Date("2024-01-01"),
      lt: new Date("2024-02-01"),
    },
  },
  _sum: {
    totalAmount: true,
  },
});

// Prisma Client の型安全な使い方
const user = await prisma.user.findUniqueOrThrow({
  where: { id: userId },
  select: {
    id: true,
    email: true,
    name: true,
    // passwordHash は意図的に除外（セキュリティ）
  },
});
```

---

## 🔌 応用パターン：既存プロジェクトへの組み込み

新規プロジェクトだけでなく、既存のプロジェクトにも Claude Code + Prisma の組み合わせは有効です。

**既存DBからのスキーマ生成**: `npx prisma db pull` でDBからスキーマを自動生成し、それを Claude Code にレビューさせる手順がオススメです。「このスキーマに改善点はありますか？」と聞くだけで、インデックスの抜け漏れや冗長なカラム設計を指摘してくれます。

**型安全なシード生成**: テストデータを手動で書くのは面倒ですが、Claude Code にスキーマを見せて「このスキーマに対してPrismaのseed.tsを生成して。現実的なダミーデータを100件分」と頼むと、`faker`を使った型安全なシードスクリプトをすぐに作ってくれます。

```typescript
// prisma/seed.ts（Claude Code生成例の一部）
import { PrismaClient } from "@prisma/client";
import { faker } from "@faker-js/faker/locale/ja";

const prisma = new PrismaClient();

async function main() {
  const users = await Promise.all(
    Array.from({ length: 100 }, () =>
      prisma.user.create({
        data: {
          email: faker.internet.email(),
          name: faker.person.fullName(),
          createdAt: faker.date.past({ years: 2 }),
        },
      })
    )
  );
  console.log(`${users.length}人のユーザーを作成しました`);
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

```bash
# シードを実行
npx prisma db seed
```

---

## まとめ：DB設計の「怖さ」がなくなった

正直に言うと、Claude Code を使う前のDB設計は「怖い作業」でした。N+1を見落としたら本番で詰む、インデックスを忘れたら大量データで詰む、マイグレーションを間違えたらデータが消える…という不安が常にありました。

今は違います。Claude Code というレビュアーがいつでもいるので、**設計の自信が持てるようになりました**。工数が70%削減されただけでなく、「本当に大丈夫か？」という不安が消えたことが一番の変化です。

TypeScript + Prisma + Claude Code の組み合わせはもはや必須スタックだと断言します。まだ試していない方は、ぜひ今日から始めてみてください。

---

### 関連記事

- [【完全解説】Claude Code×TypeScriptで型安全開発！エラー率90%削減](/articles/claude-code-typescript-type-safety)
- [【実体験】Claude Codeで開発効率75%UP！3ヶ月使った衝撃の結果と失敗談](/articles/claude-code-introduction)

### 次のステップ

GitBook『**Claude Code完全攻略**』では、本記事で紹介した手法をさらに深掘りした実践的なガイドを公開しています。Prismaのパフォーマンスチューニングや、大規模スキーマ設計のパターン集もまとめていますので、合わせてどうぞ。
