---
title: "クエリサービスはユースケースごとに独立させたい"
emoji: "🪬"
type: "tech"
topics: ["アーキテクチャ", "DDD", "CQRS", "Go", "クエリサービス"]
published: true
---

## 🤔 クエリサービスの設計でよく悩む

DDDやCQRSの文脈では、書き込み（コマンド）と読み取り（クエリ）を分離することがあります。書き込み側ではリポジトリが集約エンティティの永続化を担い、読み取り側ではクエリサービスが画面表示やレポートなどに必要なデータ取得を担います。

リポジトリは `UserRepository` に `Create`・`Update`・`Delete`・`GetByID` を生やすのが一般的です。同じノリで `UserQueryService` にも複数メソッドを生やすと、プロダクトが育つにつれてじわじわ辛くなりそうだなと感じています。

いろいろ考えた結果、自分は**シングルアクション（1インターフェース1クエリ）の方が筋が良い**と考えるようになりました。リポジトリとクエリサービスでは、責務の性質が違うからです。

## 📦 なぜリポジトリは複数メソッドを持てるのか

リポジトリの責務は**集約エンティティのライフサイクル管理**です。`Create`・`Update`・`Delete`・`GetByID` はすべて同じ集約エンティティに対する操作であり、概念的な凝集度があります。

```go
type UserRepository interface {
    Create(user *User) error
    Update(user *User) error
    Delete(id UserID) error
    GetByID(id UserID) (*User, error)
}
```

`UserRepository` に `CreateUser` と `DeleteUser` が同居するのは、同じ集約エンティティのライフサイクルという一貫した軸で束ねられているからです。

## 🔍 クエリサービスはなぜ違うのか

クエリは「何を・どんな条件で・どんな形で取得するか」がそれぞれ独立したユースケースです。複数メソッド方式で書くと、こうなりがちです。

```go
type UserQueryService interface {
    GetByID(id UserID) (*UserDTO, error)
    GetByEmail(email string) (*UserDTO, error)
    GetActiveUsersWithCareerPlan() ([]*UserSummaryDTO, error)
    GetUsersForBulkNotification() ([]*NotificationTargetDTO, error)
}
```

一見まとまっているようですが、いくつか気になる点があります。

### 返り値の型がバラバラになる

クエリごとに必要なデータの形が異なります。一覧画面にはサマリーが欲しいし、通知バッチにはメールアドレスと名前だけあればいい。無理にDTOを共通化すると、不要なフィールドだらけの「万能DTO」が生まれて設計が歪みます。

### インターフェースが肥大化する

新しい画面や機能が増えるたびにメソッドが追加され、インターフェースが膨れ上がります。ISP（インターフェース分離の原則）に違反し、利用側が使わないメソッドへの依存を強制されます。

### テストのモックが無駄に大きくなる

ユースケース層のテストで `UserQueryService` をモックすると、テスト対象が使わないメソッドまで含まれます。「このテストで本当に必要な依存は何か」が見えにくくなります。

### 複数エンティティを結合するとき、どこに属するか悩む

これが個人的に一番厄介だと思っている問題です。

たとえば「ユーザーとその所属チーム、さらにチームの進行中プロジェクト」を結合して取得するクエリを考えてみます。これは `UserQueryService` に置くべきでしょうか？ `TeamQueryService`？ `ProjectQueryService`？

```go
// どこに置く？
GetUsersWithTeamAndActiveProjects() ([]*UserTeamProjectDTO, error)
```

エンティティ名でクエリサービスをグルーピングしている限り、この「所属先問題」は避けられません。結局どこかに無理やり押し込むか、苦し紛れに `UserTeamProjectQueryService` のような中途半端な名前を作ることになります。

## ✅ シングルアクションだとどうなるか

1インターフェース1クエリにすると、先ほどの問題が解消されます。

```go
type GetUserByIDQueryService interface {
    Execute(id UserID) (*UserDetailDTO, error)
}

type GetActiveUsersQueryService interface {
    Execute(filter ActiveUserFilter) ([]*UserSummaryDTO, error)
}

type GetUsersWithTeamAndActiveProjectsQueryService interface {
    Execute(teamID TeamID) ([]*UserTeamProjectDTO, error)
}
```

**インターフェースが小さい。** テストのモックは最小限で済み、依存関係が明確になります。

**DTOをクエリ専用に最適化できる。** 「この画面専用の形」を遠慮なく作れます。一覧用のサマリー、詳細画面用のフル情報、バッチ用の最小セットなど、用途に合わせた型を自然に定義できます。

**クエリが増えても既存コードに影響しない。** 新しいクエリサービスを追加するだけで、既存のインターフェースや実装には手を加えません。OCP（開放閉鎖の原則）に沿った拡張になります。

**所属先に悩まない。** クエリの名前がそのままインターフェース名になるので、「これはどのエンティティのクエリサービスに置くべきか」という問い自体が消えます。クエリはエンティティに属するのではなく、ユースケースに属するものだと考えると、この設計は自然に感じられます。

### 例外を設けない方がうまくいく

「単純なルックアップはまとめてもいいのでは？」と思うかもしれません。自分も最初はそう考えていました。でも実際にやってみると、「これは単純か？」「まだまとめていいレベルか？」という判断が都度発生して、結局悩む時間が増えます。

例外なく全部シングルアクションにしてしまえば、その判断自体がなくなります。ファイル数は増えますが、迷いが消えるメリットの方がずっと大きいと感じています。

### DTOもクエリごとに分ける

DTOもクエリごとに専用のものを用意した方がいいと考えています。

「IDで取得するときもメールで取得するときも同じ `UserDTO` でいいじゃん」と思いがちですが、それは**今たまたま同じフィールドで済んでいるだけ**かもしれません。片方のユースケースだけにフィールドが追加されたとき、共通DTOだと不要なフィールドが他方に漏れたり、変更の影響範囲が見えにくくなります。

DTOの中身が「たまたま同じ」なのと「同じであるべき」なのは違います。クエリごとにDTOを分けておけば、ユースケースが分岐しても自然に対応できます。

この「偶然の一致」と「本質的な一致」の区別については、以前に別の記事で詳しく書いています。

https://zenn.dev/soycha/articles/accidental-coincidence-in-design

## 🏷️ 命名は「何を取るか」ではなく「何のためか」

ここまではシングルアクション vs 複数メソッドの話でしたが、もう一つ、クエリサービスの設計で気になっていることがあります。命名です。

たとえば「ユーザーとその所属チーム、さらにチームの進行中プロジェクト」を結合して取得するクエリを、取得内容をそのまま名前にすると `GetUsersWithTeamAndActiveProjectsQueryService` のようになります。結合するエンティティが増えるたびに名前が伸びていく一方です。

自分はこういうとき、取得対象ではなくユースケースで命名するようにしています。

取得対象ベースの命名は、名前が長くなるだけでなく、取得する情報が増えたときに命名の変更を迫られます。もし名前を合わせて修正しなければ、命名と実態が乖離していきます。

```go
// 取得対象ベース → 長くなり、取得情報の変更に名前が追従しなくなる
type GetUsersWithTeamAndActiveProjectsQueryService interface { ... }

// ユースケースベース → シンプルで意図が伝わる
type GetTeamDashboardQueryService interface {
    Execute(teamID TeamID) (*TeamDashboardDTO, error)
}
```

取得するデータの中身は同じでも、「何のために取るか」で名前をつけると、結合先が増えても名前は変わりません。このクエリがどの画面・どの機能のために存在するのかも一目で伝わります。

先ほどのシングルアクションの例も、ユースケースベースで命名するとこうなります。

```go
// Before: 取得対象ベース
type GetUserByIDQueryService interface {
    Execute(id UserID) (*UserDetailDTO, error)
}
type GetActiveUsersQueryService interface {
    Execute(filter ActiveUserFilter) ([]*UserSummaryDTO, error)
}
type GetUsersWithTeamAndActiveProjectsQueryService interface {
    Execute(teamID TeamID) ([]*UserTeamProjectDTO, error)
}

// After: ユースケースベース
type GetUserDetailQueryService interface {
    Execute(id UserID) (*UserDetailDTO, error)
}
type GetActiveUserListQueryService interface {
    Execute(filter ActiveUserFilter) ([]*ActiveUserListDTO, error)
}
type GetTeamDashboardQueryService interface {
    Execute(teamID TeamID) ([]*TeamDashboardDTO, error)
}
```

DTOもクエリの用途に合わせた名前になります。`UserSummaryDTO` は汎用的な名前ですが、`ActiveUserListDTO` なら「アクティブユーザー一覧画面のためのDTO」と用途が明確です。

### 動詞プレフィックスも要らない

ここまでの例では `Get` をつけていましたが、QueryServiceの時点で「取得する」ことは自明です。名詞だけで十分意味が通ります。

```go
// Get は冗長
type GetUserDetailQueryService interface { ... }
type GetActiveUserListQueryService interface { ... }

// 名詞 + QueryService で十分
type UserDetailQueryService interface { ... }
type ActiveUserListQueryService interface { ... }
type TeamDashboardQueryService interface { ... }
```

存在確認や件数取得のような、データそのものを返さないクエリも同じパターンで命名できます。

```go
type UserExistenceQueryService interface {
    Execute(id UserID) (bool, error)
}

type ActiveUserCountQueryService interface {
    Execute(filter ActiveUserFilter) (int, error)
}
```

「Existence」も「Count」も名詞なので、パターンは崩れません。

## 💡 まとめ

リポジトリとクエリサービスは、似た場所にいますが責務の性質が異なります。

- **リポジトリ**: 集約エンティティのライフサイクル管理。同じ集約に対する操作なので、複数メソッドが自然
- **クエリサービス**: ユースケースごとに独立したデータ取得。返す型も条件も異なるので、シングルアクションが原則

「リポジトリと同じノリ」でクエリサービスを設計すると、インターフェースの肥大化・DTOの歪み・テストの複雑化・所属先の曖昧さという形でじわじわコストを払うことになります。
