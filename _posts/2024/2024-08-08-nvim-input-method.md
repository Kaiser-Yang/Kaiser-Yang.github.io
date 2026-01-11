---
layout: post
title: 在 nvim 中使用 rime-ls 配置中文输入法
date: 2024-08-08 21:52:18+0800
last_updated: 2025-10-31 15:54:32+0800
description:
tags:
categories: Vim/Neovim
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 前言

在使用 `nvim` 的过程中，我们往往需要写一些文档性质的文件，而很多文档性质的文件需要中英文混合输入，
而如果我们使用系统的输入法进行操作会面临频繁切换中英文输入模式，
或者需要频繁地使用回车或者上档键将输入法中的英文字符上屏。

如果可以直接在 `nvim` 中使用输入法，那么将会大幅度提升我们的体验。在完成了相关的设置之后，
当我使用这套配置写下这篇文章的时候，总体的体验还是非常不错的。

本文将会介绍如何使用 `blink-cmp + rime-ls`
完成 `nvim` 中实现中英文混合输入，以及过程中遇到的问题和解决思路。

## 安装 `rime-ls`

`rime-ls` 是一个 `LSP`，可以帮助我们在 `nvim` 中配合补全插件使用 `rime` 输入法。

`rime-ls` 在 `GitHub` 上有编译好的二进制文件，直接在
[release](https://github.com/wlh320/rime-ls/releases)
页面下载对应的版本然后放在环境变量中就可以了。

如果是使用 `Arch` 可以直接通过 `yay -Sy --noconfirm rime-ls` 来安装作者提供的 `AUR` 包。

NOTE：还需要安装 `blink-cmp` 和 `lspconfig` 用于配置 `rime-ls`。
由于这几个是非常常用的插件，这里不详细介绍如何安装。

## `rime-ls` 配置

如果你查看了 `rime-ls` 的官方文档，你会获得一份完成了大部分设置的配置。
为了更好的进行配置管理，我创建了一个 `rime_ls.lua` 文件：

```lua
local function contains_unacceptable_character(content)
    if content == nil then return true end
    local ignored_head_number = false
    for i = 1, #content do
        local b = string.byte(content, i)
        if b >= 48 and b <= 57 or b == 32 or b == 46 then
            -- number dot and space
            if ignored_head_number then
                return true
            end
        elseif b <= 127 then
            return true
        else
            ignored_head_number = true
        end
    end
    return false
end

local M = {}

function M.is_rime_item(item)
    if item == nil or item.source_name ~= 'LSP' then return false end
    local client = vim.lsp.get_client_by_id(item.client_id)
    return client ~= nil and client.name == 'rime_ls'
end

function M.rime_item_acceptable(item)
    return
        not contains_unacceptable_character(item.label)
        or
        item.label:match("%d%d%d%d%-%d%d%-%d%d %d%d:%d%d:%d%d%")
end

function M.rime_status_icon()
    return vim.g.rime_enabled and 'ㄓ' or ''
end

function M.get_n_rime_item_index(n, items)
    if items == nil then
        items = require('blink.cmp.completion.list').items
    end
    local result = {}
    if items == nil or #items == 0 then
        return result
    end
    for i, item in ipairs(items) do
        if M.is_rime_item(item) and M.rime_item_acceptable(item) then
            result[#result + 1] = i
            if #result == n then
                break;
            end
        end
    end
    return result
end

--- @class RimeSetupOpts
--- @field filetype string|string[]

--- @param opts RimeSetupOpts
function M.setup(opts)
    opts.filetypes = opts and opts.filetypes or { '*' }
    if type(opts.filetypes) == 'string' then
        opts.filetypes = { opts.filetypes }
    end
    local configs = require('lspconfig.configs')
    vim.g.rime_enabled = false
    configs.rime_ls = {
        default_config = {
            name = "rime_ls",
            cmd = { vim.fn.expand '~/rime-ls/target/release/rime_ls' },
            filetypes = opts.filetype,
            single_file_support = true,
        },
        settings = {},
    }
end

return M
```

这样我便可以像使用一个插件一样使用 `rime-ls` 了。接下来，我在 `lsp_config.lua` 中加载了这个配置，
这里以 `blink-cmp` 为例：

```lua
require('plugins.rime_ls').setup({
    filetype = vim.g.rime_ls_support_filetype,
})
local lspconfig = require('lspconfig')
local lsp_capabilities = vim.lsp.protocol.make_client_capabilities()
lsp_capabilities = require('blink.cmp').get_lsp_capabilities(lsp_capabilities)
lspconfig.rime_ls.setup({
    init_options = {
        enabled = vim.g.rime_enabled,
        shared_data_dir = '/usr/share/rime-data',
        user_data_dir = vim.fn.expand('~/.local/share/rime-ls'),
        log_dir = vim.fn.expand('~/.local/share/rime-ls'),
        always_incomplete = true, -- This is for wubi users
        long_filter_text = true
    },
    capabilities = lsp_capabilities,
    on_attach = rime_on_attach
})
```

你如果需要添加其他配置选项，可以参考 `rime-ls` 的官方文档。

这里讲一下如何配置 `rime-ls`。在上面的代码中，
存在 `user_data_dir = vim.fn.expand('~/.local/share/rime-ls')` 因此配置文件需要放置在这个目录下，
而配置方法与配置 `rime` 输入法是一致的，将你的方案放置在 `~/.local/share/rime-ls` 目录下即可。

## `blink-cmp`

### 数字键自动上屏

`blink-cmp` 的用户要绑定上屏操作是比较简单的：

```lua
-- When there is only one rime item after inputting a number, select it directly
require('blink.cmp.completion.list').show_emitter:on(function(event)
    if not vim.g.rime_enabled then return end
    local col = vim.fn.col('.') - 1
    if event.context.line:sub(col, col):match('%d') == nil then return end
    local rime_item_index = require('plugins.rime_ls').get_n_rime_item_index(2, event.items)
    if #rime_item_index ~= 1 then return end
    vim.schedule(function() require('blink.cmp').accept({ index = rime_item_index[1] }) end)
end)
```

### 一二三候选

```lua
-- NOTE: put those below to blink.cmp keymap setting
-- select first one directly
['<space>'] = {
    function(cmp)
        if not vim.g.rime_enabled then return false end
        local rime_item_index = require('plugins.rime_ls').get_n_rime_item_index(1)
        if #rime_item_index ~= 1 then return false end
        -- If you want to select more than once,
        -- just update this cmp.accept with vim.api.nvim_feedkeys('1', 'n', true)
        -- The rest can be updated similarly
        return cmp.accept({ index = rime_item_index[1] })
    end,
    'fallback' },
-- select second one directly
[';'] = {
    -- FIX: can not work when binding ; as a prefix key
    function(cmp)
        if not vim.g.rime_enabled then return false end
        local rime_item_index = require('plugins.rime_ls').get_n_rime_item_index(2)
        if #rime_item_index ~= 2 then return false end
        return cmp.accept({ index = rime_item_index[2] })
    end, 'fallback' },
-- select third one directly
['\''] = {
    function(cmp)
        if not vim.g.rime_enabled then return false end
        local rime_item_index = require('plugins.rime_ls').get_n_rime_item_index(3)
        if #rime_item_index ~= 3 then return false end
        return cmp.accept({ index = rime_item_index[3] })
    end, 'fallback' }
```

上面的操作不会影响按键的原始功能，例如对于一些 `auto-pairs` 插件,
`<space>` 依然可以在括号中插入两个空格。

### 中文标点

中文标点的使用一直是一个痛点，而如果使用 `rime-ls` 进行标点的频繁切换的话又会影响工作的效率，
为了解决这个问题，我想到了一个简单的方法。这个方法的灵感来源于我平常写英文文档的习惯。众所周知，
在书写英文的时候, 如果输入了一个标点符号后，我们是需要输入一个空格与下一个英文隔开的，
因为英文的标点只占用一个宽度，如果不隔开会导致非常的拥挤。
正是因为这个习惯我想到了可以在 `rime-ls` 启动的时候通过标点加空格的形式插入中文标点，
而在输入法关闭的时候，标点加空格则是正常的添加英文标点与空格。
于是有了以下的代码 (如果使用这种方式的话，需要将 `rime` 标点输入设置为英文标点)：

```lua
local mapped_punc = {
    -- Add more punctuations if you like
    [','] = '，',
    ['.'] = '。',
    [':'] = '：',
    ['?'] = '？',
    ['\\'] = '、'
    -- FIX: can not work now
    -- [';'] = '；',
}
-- Toggle rime
-- This will toggle Chinese punctuations too
map_set({ 'n', 'i' }, '<c-space>', function()
    -- We must check the status before the toggle
    if vim.g.rime_enabled then
        for k, _ in pairs(mapped_punc) do
            pcall(map_del, { 'i' }, k .. '<space>')
        end
    else
        for k, v in pairs(mapped_punc) do
            map_set({ 'i' }, k .. '<space>', function()
                if utils.rime_ls_disabled({ line = vim.api.nvim_get_current_line(),
                        cursor = vim.api.nvim_win_get_cursor(0) }) then
                    feedkeys(k .. '<space>', 'n')
                else
                    feedkeys(v, 'n')
                end
            end)
        end
    end
    vim.cmd('RimeToggle')
end)
```

## 自动开关 `rime-ls`

在写 `markdown` 的时候，我是不会在一些符号中使用中文的，所以我写了这个功能。

这一部分我只在 `blink-cmp` 中进行了实现，但是实现方法是一致的：

```lua
-- When input method is enabled, disable the following patterns
vim.g.disable_rime_ls_pattern = {
    -- disable in ``
    '`(.-)`',
    -- disable in ''
    '\'(.-)\'',
    -- disable in ""
    '"(.-)"',
    -- disable in []
    '%[.-%]',
}
--- context.line will get current line
--- context.cursor will get the cursor position
--- Return if rime_ls should be disabled in current context
function M.rime_ls_disabled(context)
    local line = context.line
    local cursor_column = context.cursor[2]
    for _, pattern in ipairs(vim.g.disable_rime_ls_pattern) do
        local start_pos = 1
        while true do
            local match_start, match_end = string.find(line, pattern, start_pos)
            if not match_start then
                break
            end
            if cursor_column >= match_start and cursor_column < match_end then
                return true
            end
            start_pos = match_end + 1
        end
    end
    return false
end
```

这样只需要在相关位置添加一次判断，对于 `blink-cmp` 我们可以在 `transform_items` 中添加：

```lua
--- @param context blink.cmp.Context
--- @param items blink.cmp.CompletionItem[]
transform_items = function(context, items)
    local TYPE_ALIAS = require('blink.cmp.types').CompletionItemKind
    -- demote snippets
    for _, item in ipairs(items) do
        if item.kind == TYPE_ALIAS.Snippet then
            item.score_offset = item.score_offset - 3
        end
    end
    -- filter non-acceptable rime items
    --- @param item blink.cmp.CompletionItem
    return vim.tbl_filter(function(item)
        local rime_ls = require('plugins.rime_ls')
        if not rime_ls.is_rime_item(item) and
            item.kind ~= TYPE_ALIAS.Text then
            return true
        end
        if require('utils').rime_ls_disabled(context) then
            return false
        end
        item.detail = nil
        return rime_ls.rime_item_acceptable(item)
    end, items)
end
```

## `copilot-chat` 与 `rime-ls`

在我完成了所有配置后，我发现在 `copilot-chat` 的窗口中不能在触发 `rime-ls` 的补全，
在我提了 `issue` 后我根据网友的回答得到了以下的代码：

```lua
vim.api.nvim_create_autocmd('FileType', {
    pattern = rime_ls_filetypes,
    callback = function (env)
        local rime_ls_client = vim.lsp.get_clients({ name = 'rime_ls' })
        -- 如果没有启动 `rime-ls` 就手动启动
        if #rime_ls_client == 0 then
            vim.cmd('LspStart rime_ls')
            rime_ls_client = vim.lsp.get_clients({ name = 'rime_ls' })
        end
        -- `attach` 到 `buffer`
        if #rime_ls_client > 0 then
            vim.lsp.buf_attach_client(env.buf, rime_ls_client[1].id)
        end
    end
})
```

我估计这个不能够自动启动的原因是 `copilot-chat` 窗口并不是一个有效的 `buffer`，
而通过上面的方式启动后会创建一个新的 `buffer` 这个 `buffer` 与 `copilot-chat` 中的内容相同。

## 最后

完整的配置：[rime_ls](https://github.com/Kaiser-Yang/dotfiles/blob/main/.config/nvim/lua/rime_ls.lua)
和
[blink-cmp](https://github.com/Kaiser-Yang/dotfiles/blob/main/.config/nvim/lua/plugin/blink_cmp.lua)。

后续会有新的更改：[dotfiles](https://github.com/Kaiser-Yang/dotfiles)
