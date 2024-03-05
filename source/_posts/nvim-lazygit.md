---
title: nvim 中 lazygit 的使用
date: 2024-03-05 15:16:09
categories:
    - 开发环境
        - nvim
tags:
    - nvim
    - git
---

### 背景简介

`lazygit` 是一个由 Jesse Duffield 开发的命令行界面工具，旨在简化 Git 操作的复杂性，提供一个更加直观和用户友好的方式来执行常见的 Git 命令。它是由 Go 语言编写的，可以在多种操作系统上运行。`lazygit` 提供了一种简便的方法来查看仓库状态、提交历史、差异、分支和更多信息，所有这些都可以通过键盘来进行轻松访问。

### 主要特点

- 界面直观：通过一个简洁的界面，就可以轻松查看和管理暂存区、分支、日志等;
- 操作简单：大多数的 Git 操作都有便捷的操作按键;
- 快速导航：支持快速在提交历史、分支和更改之间导航;
- 功能丰富：支持合并时冲突解决、分支间差异查看、分支重命名等高级功能;

### 安装配置

```bash
# 验证当前是否安装成功
$ lazygit --version
```

#### macos brew 安装

```bash
$ brew install lazygit
```
> 如果 brew 使用的国内的源，有可能会导致安装时出错


#### ubuntu/debian apt-get 安装

```bash
$ sudo add-apt-repository ppa:lazygit-team/release
$ sudo apt-get update
$ sudo apt-get install lazygit
```

#### golang 源码安装

```bash
$ go install github.com/jesseduffield/lazygit@latest
```

### nvim 配置

#### 1. 安装 `lazygit.nvim` 插件
[lazygit.nvim 地址](https://github.com/kdheepak/lazygit.nvim)

插件管理我使用的 `packer.nvim`，所以我在 `plugins.lua` 中添加:

```lua
use({
    "kdheepak/lazygit.nvim",
    -- optional for floating window border decoration
    requires = {
        "nvim-lua/plenary.nvim",
    },
    config⋅=⋅[[require("qwer.plugin-config.lazygit").setup()]]
})
```

> 这里的 `config` 是我自定义的配置文件，用于管理 `lazygit.nvim` 的个性化配置;

#### 2. 配置 `lazygit.nvim` 插件

```lua
-- plugin-config/lazygit.lua 文件
local M = {}

function M.setup()
	local status_ok, _ = pcall(require, "lazygit")
	if not status_ok then
		return
	end
	vim.g.lazygit_floating_window_winblend = 0 -- 浮动窗口的透明度
	vim.g.lazygit_floating_window_scaling_factor = 0.9 -- 浮动窗口的缩放比例
	vim.g.lazygit_floating_window_border_chars = { "╭", "─", "╮", "│", "╯", "─", "╰", "│" } -- 自定义 lazygit 弹出窗口边框字符
	vim.g.lazygit_floating_window_use_plenary = 0 -- 如果可用，使用 plenary.nvim 来管理浮动窗口
	vim.g.lazygit_use_neovim_remote = 1 -- 如果没有安装 neovim-remote，则回退到 0

	vim.g.lazygit_use_custom_config_file_path = 0 -- 如果此值为 1，则会评估配置文件路径
	vim.g.lazygit_config_file_path = "" -- 自定义配置文件路径
end

return M
```

#### 3. 配置 `lazygit.nvim` 快捷键

```lua
-- which-key.lua 文件
g = {
    name = "+ git",
    g = { "<cmd>LazyGit<CR>", "LazyGit" }, -- 启动 LazyGit
},
```

### 常见操作
