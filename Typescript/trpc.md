# tRPC

## 概要

- TypeScript 向けの RPC フレームワーク(Node.js のフルスタック向け)
- TypeScript による型安全（型がサーバーとクライアントで自動で同期）
- 柔軟な API 定義

## 利用感

- シンプルな RPC フレームワーク
- REST API の代替として、より簡潔に定義できる

## gRPC とは何が違う？

- gRPC はマイクロサービス、言語間通信、分散システム向け
  - tRPC は型保証としての役割が大きそう
- gRPC は protobuf スキーマによる型定義が利用される
  - tRPC は Zod を用いた型を利用する
- gRPC は多言語対応
  - tRPC は Typescript（フルスタック）のみ
- gRPC は高速（軽量、低レイテンシ）を目的として作られた

## 導入方法

```
npm install @trpc/server zod

# Expressを使って進めるなら...
npm install --save-dev typescript ts-node-dev @types/node @types/express
```

参照: https://github.com/NaokiTani1548/article_manager

### BE 側

tRPC ルーター定義

```
import { initTRPC } from '@trpc/server';
import { z } from 'zod';

const t = initTRPC.create();

export const appRouter = t.router({
  hello: t.procedure.input(z.string()).query(({ input }) => {
    return `こんにちは、${input}さん！`;
  }),
});

export type AppRouter = typeof appRouter;

```

Express サーバー定義

```
import express from 'express';
import cors from 'cors';
import * as trpcExpress from '@trpc/server/adapters/express';
import { appRouter } from './router';

const app = express();
app.use(cors());
app.use(express.json());

app.use(
  '/trpc',
  trpcExpress.createExpressMiddleware({
    router: appRouter,
    createContext: () => ({}),
  })
);

const port = 3001;
app.listen(port, () => {
  console.log(`🚀 サーバーが起動しました → http://localhost:${port}`);
});
```

http://localhost:3001/trpc/hello?input="<your_name>"
で表示確認

### FE 側

tRPC クライアント定義

```
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '../../../server/src/router'; // ← サーバー側の型を参照

export const trpc = createTRPCReact<AppRouter>();
```

ページ実装

```
'use client';
import { trpc } from '@/utils/trpc';
import ReactMarkdown from 'react-markdown';

const ArticlesPage = () => {
  const { data, isLoading, error } = trpc.getAllArticle.useQuery();

  if (isLoading) return <p>読み込み中...</p>;
  if (error) return <p>エラーが発生しました: {error.message}</p>;

  return (
    <div>
      <h1>📝 記事一覧</h1>
      {data?.length === 0 && <p>まだ記事がありません。</p>}

      {data?.map((article) => (
        <div
          key={article.id}
          className="border p-4 rounded mb-4 shadow-sm bg-white"
        >
          <h2>{article.title}</h2>
          <p>
            <ReactMarkdown >{article.content}</ReactMarkdown>
          </p>
          <p>
            投稿日: {article.createdAt ? new Date(article.createdAt).toLocaleString() : '不明'}
          </p>
        </div>
      ))}
    </div>
  );
};

export default ArticlesPage;

```
