---
author: nyo3q1
title: 軽率にGraphQLをはじめる方法
date: 2023/10/15
---

## 自己紹介
- [nyo3q1](https://twitter.com/nyo3q1)
- 新卒入社一年目で尿酸値2ケタ達成
- 知りたいこと
  - jsxテンプレートエンジン的なもの
- 技術スタック
    - Python(Django)
    - TypeScript(React/Preact)



## GraphQLってなに？
- クライアントからサーバーにリクエストを送るWeb APIの一種
- REST APIの代替として使われることが多い
- 取得するデータを柔軟に選択できる
- グラフ理論

---
### グラフ理論はこんな感じのやつ


---

## 何ができるの？
REST APIで出来ること全部できる(はず)

---

### CRUD操作から見るREST APIとの比較
|CRUD|REST API|GraphQL|
|:----|:----|:----|
|Create(新規作成)|POST|Mutation|
|Read(読み込み)|GET|Query|
|Update(上書き)|PUT|Mutation|
|Delete(削除)|DELETE|Mutation|
|データの監視/サブスクリプション|???|subscription|

---

### GraphQLへのクエリをまとめることができる
REST APIではデータ取得を複数回行う場合でも、GraphQLの場合はまとめて取得できる。
<br>
リクエストが1つになるので、通信コストが削減できる。

---

### REST API
A, B, Cという複数の違う種類のデータがある場合は、1つずつ取得する必要がある。

```javascript
const a = await fetch('/api/a');
const b = await fetch('/api/b');
const c = await fetch('/api/c');
```

---

### GraphQL
A, B, Cをまとめて取得できる。
```javascript
const HogePageQuery = graphql`query HogePageQuery {
  a {
    id
    name
  }
  b {
    id
    name
  }
  c {
    id
    name
  }
}`;

const { data } = useQuery(HogePageQuery);
```

---

### クエリごとに取得するデータのフィールドを選択できる
ページによって、データの(全フィールド | 一部のフィールド)が欲しい、を実現できる。
たぶん、バックエンドの実装次第では、SQLのクエリも動的に変更できる。
```graphql
type User{
  id: Int!
  name: String!
  email: String!
}
```

---

#### 全部欲しい場合
```javascript
const APageQuery = graphql`query APageQuery {
  user {
    id
    name
    email
  }
}`;

const { data } = useQuery(APageQuery);
```

#### 一部欲しい場合
```javascript
const BPageQuery = graphql`query BPageQuery {
  user {
    id
  }
}`;

const { data } = useQuery(BPageQuery);
```

## よく分かってないクエリを分からないままなんとかする
2つツールを組み合わせて雰囲気でなんとかする

---

### その1 
[GraphiQL](https://github.com/graphql/graphiql)を使う

#### なにこれ？
- GraphQL IDE
- 指定されたエンドポイント上にあるGraphQLサーバーに対してクエリを投げられる
- エンドポイントに存在するクエリの一覧を表示してくれる
- UIぽちぽちでクエリが組み立てられる

---

#### その2
[GraphQL Code Generator](https://the-guild.dev/graphql/codegen)を使う

#### なにこれ？
- REST APIのOpenAPIのようなもの
- 組み立てたクエリを元に型定義を生成してくれる
- `typescript-react-apollo` プラグインが便利
  - ある程度の規模になると辛くなる時が来るらしい

---

こんな感じのコードが生成される
```typescript
export const FindUserDocument = gql`
    query findUser($userId: ID!) {
  user(id: $userId) {
    ...UserFields
  }
}
  ${UserFieldsFragmentDoc}`;
export function useFindUserQuery(baseOptions: Apollo.QueryHookOptions<FindUserQuery, FindUserQueryVariables>) {
        const options = {...defaultOptions, ...baseOptions}
        return Apollo.useQuery<FindUserQuery, FindUserQueryVariables>(FindUserDocument, options);
      }
```

---

### バックエンドはコードファーストなGraphQLライブラリを選択する
- バックエンドの言語の型でGraphQLの型を意識することなく構築できる
- GraphiQLと組み合わせると、GraphQLのクエリをほぼ意識せずに開発できる。開発者体験が爆増。
- Pythonだったら　[Strawberry](https://github.com/strawberry-graphql/strawberry)

## 少しずつ導入していくには

---

### 管理画面系のページから導入するのが楽
- キャッシュ戦略を気にしなくても良い機能から導入すると楽かも
- GraphQLはデータ取得のクエリであっても、POSTメソッドで実行される(設定次第ではGETにできる)
  - アクセスが多くキャッシュ戦略が重要な画面だと苦労するかも。

---

### 少しずつグラフ構造を整えていく
- いきなり全てのデータの関係性を作るのは大変、必要な部分だけ作る。
- 後付けでフィールドを追加してもエラーにならない、最小構成で進めていくことができる。

---

### 既存のREST APIをGraphQL APIに置き換えるだけでもOK
自分の場合、フレームワークが持っているREST APIの機能 + OpenAPIの組み合わせが辛かったから入れ替え始めた。
<br>
とりあえずお試しで、GraphQLで実装することから始めたので、初めはグラフ構造を入れなかった。

## マジ価値サマリー
- GraphiQL, GraphQL Code Generator, コードファーストなGraphQLライブラリを使えば、GraphQLのクエリを意識せずに開発できる
- 初めはグラフ構造を入れなくてOK


