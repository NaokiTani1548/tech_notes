# Prisma

## 概要

- Prisma とは、Node.js/TypeScript 環境で利用できるオープンソースの ORM

## 導入方法

### パッケージインストール

```
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

※init のデフォルトは postgresql
(--datasource-provider sqlite (mysql) を後ろにつければ設定が楽)

### セットアップ

- .env の DATABASE_URL を設定
- スキーマ定義 (prisma/schema.prisma)

```
例
model Article {
  id        Int      @id @default(autoincrement())
  title     String
  content   String
  createdAt DateTime @default(now())
}
```

参考になる: https://zenn.dev/ikekyo/scraps/f6c87fbfd3bf9d

### マイグレーション実行

```
npx prisma generate
npx prisma migrate dev --name <migrate_name>
```

※ generate で.prisma/client が DB 型に合わせて型付きで自動生成される。
※ migrate は変更のたびに実行

### その他マイグレーションコマンド

```
# schema.prisma ファイルのモデルを変更した際に、migration ファイルを生成せずにテーブルを変更する。
npx prisma db push

# 現在のDBの状態を schema.prisma ファイルに反映させる。
npx prisma db pull

# テーブルを一度削除し再生成
npx prisma migrate reset

# ローカルのテーブルを確認
npx prisma studio
```

## Prisma メソッド

### CRUD

https://www.prisma.io/docs/orm/reference/prisma-schema-reference

```
例
async create(article: Article): Promise<Article> {
const data = await prisma.article.create({
    data: {
    title: article.title,
    content: article.content,
    },
});

return new Article(data.title, data.content, data.id, data.createdAt);
}
```

###

## その他

### リレーションの設定

```
外部リレーション例1
model Post {
  id       Int  @id @default(autoincrement())
  author   User @relation(fields: [authorId], references: [id])
  authorId Int
}
```

```
外部リレーション例2
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  Posts Post[]  @relation("author")　
}

model Post {
  id       Int  @id @default(autoincrement())
  author   User @relation("author", fields: [authorId], references: [id])
  authorId Int
}
```

### 初期データ投入

- seed の利用
  https://qiita.com/takekota/items/42f6a9066de60da93161#prismaseedts-feat-relations
