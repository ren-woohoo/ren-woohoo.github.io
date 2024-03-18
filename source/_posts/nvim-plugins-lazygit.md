---
title: nvim 插件之 lazygit
date: 2024-03-05 15:16:09
categories:
    - 环境
        - nvim
tags:
    - nvim
    - git
---

### 背景简介

`lazygit` 是一个由 Jesse Duffield 开发的命令行界面工具，旨在简化 Git 操作的复杂性，提供一个更加直观和用户友好的方式来执行常见的 Git 命令。它是由 Go 语言编写的，可以在多种操作系统上运行。`lazygit` 提供了一种简便的方法来查看仓库状态、提交历史、差异、分支和更多信息，所有这些都可以通过键盘来进行轻松访问。

#### 主要特点
- 界面直观：通过一个简洁的界面，就可以轻松查看和管理暂存区、分支、日志等;
- 操作简单：大多数的 Git 操作都有便捷的操作按键;
- 快速导航：支持快速在提交历史、分支和更改之间导航;
- 功能丰富：支持合并时冲突解决、分支间差异查看、分支重命名等高级功能;

### 安装与配置
#### 1. `lazygit` 安装

```bash
# macos 系统
$ brew install lazygit

# ubuntu/debian 系统
$ sudo add-apt-repository ppa:lazygit-team/release
$ sudo apt-get update
$ sudo apt-get install lazygit

# 安装完成
$ lazygit --version
```

#### 2. 加载 `lazygit.nvim` 插件

[插件地址](https://github.com/kdheepak/lazygit.nvim)

```lua
-- packer 安装，plugins.lua 文件
use({
    "kdheepak/lazygit.nvim",
    -- optional for floating window border decoration
    requires = {
        "nvim-lua/plenary.nvim",
    },
    config⋅=⋅[[require("qwer.plugin-config.lazygit").setup()]]
})
```
*** 这里的 `config` 是我自定义的配置文件，用于管理 `lazygit.nvim` 的个性化配置; ***

#### 3. 配置 `lazygit.nvim` 插件

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

#### 3. `lazygit.nvim` 快捷键配置

```lua
-- which-key.lua 文件
g = {
    name = "+ git",
    g = { "<cmd>LazyGit<CR>", "LazyGit" }, -- 启动 LazyGit
},
```

### 常见操作

#### 导航
- 上下移动：使用箭头键 ↑ 和 ↓ 或 j 和 k 在不同的面板或选项之间移动。
- 切换面板：使用 tab 键在状态、日志、分支等面板之间切换。

#### 仓库状态
- 查看状态：启动 lazygit 后，默认面板显示当前仓库的状态，包括更改、暂存、未跟踪等文件。

#### 提交更改
- 暂存文件：选中文件后按 space 键暂存更改。
- 取消暂存：在暂存面板中，选中文件后按 space 键取消暂存。
- 提交：按 c 键打开提交面板，输入提交信息后，按 Ctrl+s 提交，esc 键取消。

#### 分支操作
- 查看分支：切换到分支面板（通常通过 tab 键）。
- 新建分支：在分支面板中，按 n 键新建分支。
- 检出分支：在分支面板中，使用箭头键选中分支后按 Enter 键检出。
- 删除分支：选中分支后按 d 键删除。

#### 日志和历史
- 查看提交日志：切换到日志面板查看提交历史。
- 查看提交详情：在日志面板中，选中提交后按 Enter 键查看详情。

#### 合并和拉取
- 合并分支：在分支面板中，选中要合并的分支后按 m 键合并到当前分支。
- 拉取最新更改：按 F 键从远程仓库拉取最新更改。

#### 冲突解决
- 解决冲突：在状态面板中，选中有冲突的文件，lazygit 会提供选项来选择解决冲突的方式。

#### 其他操作
- 撤销最近的提交：在日志面板中，选中最近的提交后按 z 键撤销。
- 推送到远程：按 P 键推送本地提交到远程仓库。
