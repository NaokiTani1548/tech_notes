# graphQL

## 概要

- API のためのクエリ言語,またはサーバーサイドランタイム

### RestAPI とは違い...

- オーバーフェッチ,アンダーフェッチを防げる
- API のバージョニング管理が楽,新しいフィールドを追加しても、既存のクエリには影響を与えない
- 単一のエンドポイント
- 強い型付けシステム(API で扱えるデータの構造を「スキーマ」として厳密に定義)

### 注意点として...

- リゾルバの実装によっては、意図せず大量のデータベースクエリが発生する「N+1 問題」が発生する可能性がある
- データローダー (DataLoader) といった仕組みを使って対策することができる

## 書き方概要

- 以下二つを記述
  - GraphQL スキーマ（どういうデータを返すかを記載した仕様書のようなもの）
  - GraphQL クエリ(ほしいデータを記載したもの →API のリクエストで使う)

### Query

- データ取得に使われるクエリ

```
# schema.graphql
# ルートオペレーション型  ※！は必須パラメータ
type Query {
  user: User!
}

# User型
type User {
  id: ID!
  name: String!
  email: String!
}
```

```
# user.graphql
query GetUser {
  user {
    id
    name
  }
}
```

```
# レスポンス
{
  "data": {
    "user": {
      "id": "hoge",
      "name": "foo",
    }
  }
}
```

### Mutation

- データ更新に使われるクエリ

```
# schema.graphql
# ルートオペレーション型
type Mutation {
  createUser(data: UserCreateInput!): User!
}

# User型
type User {
  id: ID!
  name: String!
  email: String!
}

# UserCreateInput型
input UserCreateInput {
  id: ID
  name: String!
  email: String!
}
```

```
# user.graphql
mutation CreateUser {
  createUser(data: {
    name: "hoge",
    email: "foo@hoge.co.jp"
  }) {
    name
  }
}
```

```
# レスポンス
{
  "data": {
    "createUser": {
      "name": "hoge",
    }
  }
}
```

### Fragment

- クエリの一部をフラグメント化して再利用する機能
- 以下二つは同義

```
# user.graphql
query GetUser {
  hoge_user: user {
    id
    name
  }
  foo_user: user {
    id
    name
  }
}
```

```
# user.graphql
query GetUser {
  hoge_user: user {
    ...userFragment
  }
  foo_user: user {
    ...userFragment
  }
}

fragment userFragment on User {
   id
   name
}
```

## その他ユースケース

```
# schema.graphql
type Query { #Queryは予約語→データ取得を行いたい処理を記載
  getStudents: [students] # Query名(引数):型
}
input Student {
  name: String!
  age: Int!
  grade: Int!
}
type Mutation { #Mutationも予約後→データ更新系を行いたい処理を記載
  createStudent(input: Student): students # Mutation名(引数:型):型
}

type students { # ←このように独自に型、objectを作る事も可能
  name: String!
  age: Int!
  grade: Int!
}

```

```
# student.graphql
# <query or mutation> <任意の名前>
query getStudents {
    name　# 取得したい値のみを選択
    age
}
```

## ベストプラクティス

#### スキーマ定義ファイルに description とよばれるドキュメントを記述する

```
"""
Images attached to the check run output displayed in the GitHub pull request UI.
"""
input CheckRunOutputImage {
  """
  The alternative text for the image.
  """
  alt: String!

  """
  A short image description.
  """
  caption: String

  # ...
}
```

### 命名

- 一貫性を持たせる
- 型名は具体的に
- フィールドでその型名を繰り返さない
- 入力型と出力型を分けて定義する

```
type Mutation {
  createBook(input: CreateBookInput!): Book
}
```

- サーバーを作成するとき、パスの最後の部分は /graphql で終わる（推奨）

### フィールド

- String 型よりも具体的な型を使えないか考慮(固定値なら enum で定義など)
- あるユースケースに特化させる(スキーマ定義で使い方がわかるような引数設定を)
- 黙的なデフォルト値をリゾルバー内部で定義しない（パラメーターのデフォルト値を設定して、外から動きがわかるように）

### Nullable Non-null

- Non-null(!) -> Nullable は破壊的変更
- 可能な限り Nullable を用いる（エラー時などに Null が帰ってくる）
- 何らかの原因で値を返せない可能性があるフィールドは、Nullable
- フィールドパラメーターはできるだけ Non-null （API の使い方が明確）
- ※あるオブジェクト型のフィールドであり、そのオブジェクトが存在するときに必ず存在することが分かっているデータあれば、そのフィールドは Non-null で定義することができる（そのフィールドを含むオブジェクト自身は Nullable になり得る）。

```
type Payment {
  creditCard: CreaditCard
  giftCode: String
}

type CreditCard {
  number: CreditCardNumber!
  expiration: CreditCardExpiration!
}
```
