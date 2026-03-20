# CLAUDE.md

## プロジェクト概要
Zenn CLIを使った技術ブログリポジトリ。ソフトウェア設計・開発プラクティスに関する記事を執筆・公開している。

## ディレクトリ構成
- `articles/` - 記事（Markdown）
- `books/` - 本（未使用）
- `images/` - 記事で使用する画像
- `scraps/` - スクラップ

## 記事のフロントマター形式
```yaml
---
title: "記事タイトル"
emoji: "🪬"
type: "tech"
topics: ["トピック1", "トピック2"]
published: true
---
```

## 記事の執筆規約
- 見出しには絵文字を付ける（例: `## 🤔 見出しテキスト`）
- コード例は主にGoで記述する
- `published: false` で下書き、`true` で公開

## コマンド
- `npx zenn preview` - ローカルプレビュー
- `npx zenn new:article --slug <slug>` - 新規記事作成

## コミットメッセージ
- 日本語で記述する
- 記事の内容変更を簡潔に説明する（例: 「クエリサービス記事の導入文を現在の視点ベースに修正」）
