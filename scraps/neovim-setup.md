---
title: "Neovim初心者が設定を進める記録"
emoji: "📝"
type: "scrap"
topics: ["Neovim", "Vim", "macOS", "エディタ"]
closed: false
---

## Neovimを始めるきっかけ

何も知らない状態からNeovimの設定を進めていく記録です。

---

## Neovimのインストール

macOSではHomebrewでインストールできる。

```bash
brew install neovim
```

インストールされたバージョン:
```
NVIM v0.11.6
Build type: Release
LuaJIT 2.1.1771261233
```

---

## 基本設定ファイルの作成

Neovimの設定ファイルは `~/.config/nvim/init.lua` に置く。現在のNeovimはLuaで設定を書くのが主流。

```bash
mkdir -p ~/.config/nvim
```

`~/.config/nvim/init.lua` に以下を記述:

```lua
-- 基本設定

-- 行番号を表示
vim.opt.number = true
vim.opt.relativenumber = true

-- タブとインデント
vim.opt.tabstop = 2
vim.opt.shiftwidth = 2
vim.opt.expandtab = true
vim.opt.smartindent = true

-- 検索
vim.opt.ignorecase = true
vim.opt.smartcase = true
vim.opt.hlsearch = true
vim.opt.incsearch = true

-- 外観
vim.opt.termguicolors = true
vim.opt.cursorline = true
vim.opt.signcolumn = "yes"

-- クリップボード（macOS）
vim.opt.clipboard = "unnamedplus"

-- スワップファイルを無効化
vim.opt.swapfile = false

-- エンコーディング
vim.opt.encoding = "utf-8"
vim.opt.fileencoding = "utf-8"
```

### 設定の意味

| 設定 | 説明 |
|------|------|
| `number` | 行番号を表示 |
| `relativenumber` | 現在行からの相対行番号を表示（移動が楽になる） |
| `tabstop` / `shiftwidth` | タブ幅を2に設定 |
| `expandtab` | タブをスペースに変換 |
| `smartindent` | 自動インデント |
| `ignorecase` / `smartcase` | 検索時に大文字小文字を無視（大文字を含む場合は区別） |
| `hlsearch` | 検索結果をハイライト |
| `termguicolors` | 24ビットカラーを有効化 |
| `cursorline` | カーソル行をハイライト |
| `signcolumn` | 左側にサインカラム（Git変更やエラー表示用）を常に表示 |
| `clipboard` | システムクリップボードと連携 |

---

## Neovimの起動と基本操作

### 起動方法

```bash
# ターミナルで以下を実行
nvim

# ファイルを指定して開く
nvim ファイル名

# 例: init.luaを開く
nvim ~/.config/nvim/init.lua
```

### Vimのモード

Vimには**モード**という概念がある。これがVim系エディタの最大の特徴。

| モード | 説明 | 入り方 |
|--------|------|--------|
| ノーマル | 移動・コマンド実行 | `Esc` |
| インサート | 文字入力 | `i`（カーソル前）、`a`（カーソル後） |
| ビジュアル | 範囲選択 | `v`（文字）、`V`（行）、`Ctrl+v`（矩形） |
| コマンド | 保存・終了など | `:` |

### 最低限覚えるコマンド

| コマンド | 説明 |
|----------|------|
| `:w` | 保存 |
| `:q` | 終了 |
| `:wq` | 保存して終了 |
| `:q!` | 保存せずに強制終了 |
| `i` | インサートモードに入る |
| `Esc` | ノーマルモードに戻る |
| `h` `j` `k` `l` | 左・下・上・右に移動 |
| `dd` | 1行削除 |
| `yy` | 1行コピー |
| `p` | ペースト |
| `u` | 元に戻す（undo） |
| `Ctrl+r` | やり直し（redo） |

---

## dotfilesで設定を管理する

設定ファイルをGitで管理するために、dotfilesリポジトリを作成。

```
~/dotfiles/
├── .config/
│   └── nvim/
│       └── init.lua
└── install.sh
```

`install.sh` でシンボリックリンクを作成:
```bash
ln -s ~/dotfiles/.config/nvim ~/.config/nvim
```

これで `~/.config/nvim` への変更が自動的にdotfilesリポジトリに反映される。

---

## プラグインマネージャー lazy.nvim の導入

Neovimでプラグインを管理するために、現在最も人気のある「lazy.nvim」を導入。

### リーダーキーの設定

まず、プラグイン設定より前にリーダーキーを設定する必要がある。リーダーキーはカスタムショートカットの起点となるキー。

```lua
-- リーダーキーをスペースに設定（lazy.nvimより前に設定する必要あり）
vim.g.mapleader = " "
vim.g.maplocalleader = " "
```

### lazy.nvimのセットアップ

`init.lua` の最後に以下を追加:

```lua
-- lazy.nvim（プラグインマネージャー）のセットアップ
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git",
    "clone",
    "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)

-- プラグインの設定
require("lazy").setup({
  -- ここにプラグインを追加していく
})
```

Neovimを起動すると、lazy.nvimが自動的にインストールされる。

### lazy.nvimの操作

| コマンド | 説明 |
|----------|------|
| `:Lazy` | lazy.nvimのUIを開く |
| `:Lazy sync` | プラグインを同期（インストール・更新） |
| `:Lazy update` | プラグインを更新 |

---

## カラースキーム tokyonight の導入

見た目を良くするためにカラースキームを導入。人気のある「tokyonight」を選択。

lazy.nvimのプラグイン設定に追加:

```lua
require("lazy").setup({
  -- カラースキーム: tokyonight
  {
    "folke/tokyonight.nvim",
    lazy = false,    -- 起動時に読み込む
    priority = 1000, -- 他のプラグインより先に読み込む
    config = function()
      vim.cmd([[colorscheme tokyonight]])
    end,
  },
})
```

### プラグイン設定の読み方

| 項目 | 説明 |
|------|------|
| `"folke/tokyonight.nvim"` | GitHubの `ユーザー名/リポジトリ名` |
| `lazy = false` | 遅延読み込みしない（起動時に読み込む） |
| `priority = 1000` | 読み込み優先度（大きいほど先に読み込む） |
| `config = function()` | プラグイン読み込み後に実行される設定 |

Neovimを起動するとtokyonightが適用される。

---
