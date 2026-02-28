# Zenn CLI

* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

---------------------------------------------

# Zenn CLIで記事・本を管理する方法

このページでは Zenn CLI の使い方とコマンドの一覧を紹介します。まだインストールしていない方は下記のリンク先をご覧ください。

📘 **[Zenn CLI を導入する →](https://zenn.dev/zenn/articles/install-zenn-cli)**

---

## CLI で記事（article）を管理する

### ファイルの配置ルール

1 つの記事の内容は、1 つのmarkdownファイル（`◯◯.md`）で管理します。ファイルは`articles`という名前のディレクトリ内に含める必要があります。
```
.
└─ articles
   ├── example-article1.md
   └── example-article2.md
```

具体的な例として[Zenn のドキュメント用リポジトリ](https://github.com/zenn-dev/zenn-docs)を見ると分かりやすいかもしれません。

### 記事の作成

> ⚠️ まだ`articles`ディレクトリが存在しない場合は、事前に`npx zenn init`を実行してください

以下のコマンドによりmarkdownファイルを簡単に作成できます。
```bash
$ npx zenn new:article
```

`articles/ランダムなslug.md`というファイルが作成されます。slug（スラッグ）はその記事のユニークな ID のようなものです。

作成されたファイルの中身は次のようになっています。
```yaml
---
title: "" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: [] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
ここから本文を書く
```

コマンド実行時に記事の Front Matter をオプションで指定することもできます。
```bash
$ npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

> ⚠️ slug は`a-z0-9`、ハイフン`-`、アンダースコア`_`の 12〜50 字の組み合わせにする必要があります

#### 本文に画像を挿入するには

以下のいずれかの方法で本文に画像を挿入できます。

1. [Zennの画像のアップロードページ](https://zenn.dev/dashboard/uploader)から画像をアップロードしてURLを貼り付ける
2. [GitHubリポジトリ内に画像を配置する](https://zenn.dev/zenn/articles/deploy-github-images)
3. Gyazoなどの外部サービスにアップロードした画像のURLを貼り付ける

### プレビューする

本文の執筆は、ブラウザでプレビューしながら確認できます。
```bash
$ npx zenn preview # プレビュー開始
```

> 💡 デフォルトでは`localhost:8000`で立ち上がりますが`npx zenn preview --port 3000`というようにポート番号の指定もできます。

> 💡 `npx zenn preview --no-watch`のようにすることでファイルの監視と自動リロードが無効になります。

### 記事を公開する

記事を zenn.dev 上で公開するには`published`オプションが`true`になっていることを確認したうえで、ファイルをコミットし、Zenn と連携されている GitHub リポジトリにプッシュします。

> 💡 デプロイ履歴は[ダッシュボード](https://zenn.dev/dashboard/deploys)から見ることができます。

なおコミットメッセージに`[ci skip]`もしくは`[skip ci]`が含まれていると Zenn でのデプロイがスキップされます。

### 日時を指定して記事を公開する（公開予約する）

公開時間を指定して記事を公開するには、Front Matterにて `published` を `true` にした上で、 `published_at` を指定します。
```yaml
published: true # trueを指定する
published_at: 2050-06-12 09:03 # 未来の日時を指定する
```

> 💡 `published_at` のフォーマットは、 `YYYY-MM-DD` または `YYYY-MM-DD hh:mm` です。タイムゾーンはJST（日本時間）です。

### 過去の公開日時で記事を公開する

他のブログサービスなどからzenn.devに記事を移行する際に公開日時を維持したい場合、`published_at` に過去の日時を指定することで、zenn.dev上での公開日時を指定することができます。
```yaml
published: true # true/falseどちらでもOKです
published_at: 2010-01-01 08:00 # 過去の日時を指定する
```

> ⚠️ 公開日時の指定は一度しかできず、既に設定された値を変更することはできません。

### 記事の更新

記事の更新を行う場合も、markdownファイルを編集し、GitHub リポジトリへプッシュするだけで OK です。このとき slug が同一のものでないと別の記事として作成されてしまうので注意しましょう。

### 記事の削除

削除は[ダッシュボード](https://zenn.dev/dashboard)から行います。安全のため、`articles`ディレクトリからmarkdownファイルを削除しても zenn.dev 上では削除はされません。

---

## CLI で本（book）を管理する

Zenn の本は複数のチャプターで構成されます。

### 本のディレクトリ構成

GitHub リポジトリで本のデータを管理する場合は、次のようなディレクトリ構成にします。
```
.
├─ articles
└─ books
   └── 本のスラッグ
       ├── config.yaml # 本の設定
       ├── cover.png　# カバー画像（JPEGかPNG）
       └── チャプターのスラッグ.md # 各チャプター
```

具体的には、以下のようになります。
```
books
└── my-awesome-book
    ├── config.yaml
    ├── cover.png
    ├── example1.md
    ├── example2.md
    └── example3.md
```

### 本の各ファイルの役割

#### 📄 config.yaml

本の設定ファイルです。以下のように記入してください。
```yaml
title: "本のタイトル"
summary: "本の紹介文"
topics: ["markdown", "zenn", "react"] # トピック（5つまで）
published: true # falseだと下書き
price: 0 # 有料の場合200〜5000
chapters:
  - example1 # チャプター1
  - example2 # チャプター2
  - example3 # チャプター3
```

- **`title`** : 本のタイトルを入力します
- **`summary`** : 本の紹介文を入力します。これは有料の本であっても公開されます
- **`topics`** : トピック（タグ）を 5 つまで指定します
- **`published`** : 公開する場合は`true`にします
- **`price`** : 本を有料で販売するときは`price: 1000`のように記載（200〜5000 の間で 100 円単位）
- **`chapters`** : チャプターの並び順を配列で指定します（入れ子には未対応）
- **`toc_depth`** : 目次の表示設定（0〜3の範囲で指定、0で非表示）

#### 🖼️ カバー画像

本のカバー画像は`cover.png`もしくは`cover.jpeg`というファイル名で配置します。
推奨の画像サイズは**幅 500px・高さ 700px**です。

#### 📄 各チャプターのファイル（`◯◯.md`）

各チャプターのファイル名は「`a-z0-9`、ハイフン`-`、アンダースコア`_`の 1〜50 字の組み合わせ」+ `.md`とします。
```yaml
---
title: "チャプターのタイトル"
---
ここからチャプター本文
```

有料の本で一部のチャプターを無料公開する場合は`free: true`を指定してください。
```yaml
---
title: "タイトル"
free: true
---
```

#### ファイル名で並び順を管理する方法

ファイル名を`チャプター番号.slug.md`という形にすると、その順番通りにチャプターが表示されます。
```
books
└── my-awesome-book
    ├── config.yaml
    ├── 1.intro.md
    ├── 2.foo.md
    └── 3.bar.md
```

この方法でデプロイを行う場合、`config.yaml`の`chapters`は空にしておく必要があります。

### 本の雛形をコマンドで作成する
```bash
$ npx zenn new:book
# 本のslugを指定する場合は以下のようにします。
# npx zenn new:book --slug ここにスラッグ
```

> ⚠️ 本の slug は`a-z0-9`、ハイフン`-`、アンダースコア`_`の 12〜50 字の組み合わせにする必要があります

### 本と各チャプターをプレビューする
```bash
$ npx zenn preview
```

### 最大チャプター数

本のチャプターは1冊あたり最大100個まで作成できます。

### 本の更新

本の更新を行う場合も、ファイルを変更し、GitHub リポジトリへプッシュするだけで OK です。このとき slug（ディレクトリ名）が同一のものでないと別の本として作成されてしまうので注意しましょう。

### 本の削除

削除は[ダッシュボード](https://zenn.dev/dashboard)から行います。`books`ディレクトリからファイルを削除しても zenn.dev 上では削除はされません。