# Zod

## 概要

- TypeScript 向けのスキーマ宣言とデータ検証のためのライブラリ
- 型安全な方法でデータ構造を定義し、それに基づいてデータを検証できる
- TypeScript の型システムと統合されており、コンパイル時に型エラーを検出しやすい

## 導入方法

```
npm install zod
```

## スキーマ宣言

### 基本的な利用法

```
import { z } from 'zod';

const name = z.string();
name.parse('Naoki'); // ok。 'Naoki'を返す
name.parse(100); // Error エラーを投げる
```

### スキーマから型定義

```
type Name = z.infer<typeof name>

const myName: Name = 'Naoki'; // OK
const yourName: Name = 100; // タイプエラー発生
```

### ユースケース

```
import { z } from 'zod';

// 5桁以上11桁未満の string 型
const passWord = z.string().min(5).max(10);

// emailアドレス
const email = z.string().email();

// オブジェクト型
const user = z.object({
 name: z.string(),
 age: z.number(),
})
// -> { name: string, age: number } の object

// 関数型
const myFunc = z.function().args(z.number(), z.string()).returns(z.boolean());
// -> (arg0: number, arg1: string) => boolean の関数

// 型生成も可能
type MyFunc = z.infer<typeof myFunc>;
const isOverMaxLength: MyFunc = (maxLength, userName) => userName.length > maxLength;
isOverMaxLength('hoge', 'fuga'); // タイプエラー発生

```

### スキーマ定義詳細

https://zod.dev/api

※メールや日付なども対応させることができる

## エラーハンドリング

### Zod エラー

```
import { z } from 'zod';

const person = z.object({
  name: z.string(),
  age: z.number(),
});

try {
  person.parse({
    name: 1234,
    age: "31"
  });
} catch (err) {
  if (err instanceof z.ZodError) {
    console.log(err.issues);
  }
}
```

```
[
  // 1つ目のエラー
  {
    code: "invalid_type", // Zodで定義されているエラーコード
    expected: "string", // 期待される型
    received: "number", // 実際に渡された型
    path: ["name"],
    // エラーが発生した値へのパス この場合はプロパティ名
    message: "Expected string, received number"
    // エラーメッセージ
  },
  // 2つ目のエラー
  {
    code: "invalid_type",
    expected: "number",
    received: "string",
    path: ["age"],
    message: "Expected number, received string"
  }
];
```

### flatten で出力を分かりやすく...

```
{
  "formErrors":[],
  "fieldErrors": {
    "name":["Expected string, received number"],
    "age":["Expected number, received string"]
   }
}
︙
︙
{
  "formErrors":["Expected object, received null"],
  "fieldErrors":{}
}
```

### カスタムエラーマップ

```
import { z } from "zod";

const customErrorMap: z.ZodErrorMap = (issue, ctx) => {
  // string と number の invalid_type エラーメッセージのカスタマイズ
  if (issue.code === z.ZodIssueCode.invalid_type) {
    if (issue.expected === "string") {
      return { message: "文字列を入力してください" };
    }
    if (issue.expected === "number") {
      return { message: "数字を入力してください" };
    }
  }
  // 上記以外はデフォルトで定義されたエラーを返す
  return { message: ctx.defaultError };
};
```

グローバルエラーマップに登録

```
z.setErrorMap(customErrorMap);
```

スキーマ宣言に付与

```
z.string({ errorMap: customErrorMap });
```
