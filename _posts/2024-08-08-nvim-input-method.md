---
layout: post
title: 在 nvim 中使用 rime-ls 配置中文输入法
date: 2024-08-08 21:52:18+0800
last_updated: 2025-05-17 14:49:12
description:
tags:
  - Vim/Neovim
categories: Potpourri
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

本文将会介绍如何使用 `nvim-cmp + rime-ls` 和 `blink-cmp + rime-ls`
完成 `nvim` 中实现中英文混合输入，以及过程中遇到的问题和解决思路。

## 安装 `rime-ls`

`rime-ls` 是一个 `LSP`，可以帮助我们在 `nvim` 中配合补全插件使用 `rime` 输入法。
我们可以使用以下的命令进行安装：

```bash
sudo apt-get install -y clang librime-dev cargo
cd || exit 1
git clone https://github.com/wlh320/rime-ls.git
cd - || exit 1
cd ~/rime-ls || exit 1
cargo build --release || exit 1
cd - || exit 1
```

其中安装的 `clang, librime-dev, cargo` 是安装 `rime-ls` 所需要的依赖。
上面的代码会将 `rime-ls` 安装在家目录下，如果需要更改可以自行调整。

NOTE：还需要安装 `nvim-cmp` 或 `blink-cmp` 了。
除此之外，还需要安装 `lspconfig` 用于配置 `rime-ls`。
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

### 顶字上屏

**注意**：使用顶字上屏可能会造成卡顿，不建议使用。

五笔用户可能会有这样的需求：

```lua
local max_code = 4
local alphabet = 'abcdefghijklmnopqrstuvwxy'
for i = 1, #alphabet do
    local k = alphabet:sub(i, i)
    map_set({ 'i' }, k, function()
        local cursor_column = vim.api.nvim_win_get_cursor(0)[2]
        local confirmed = false
        if vim.g.rime_enabled and cursor_column >= max_code and vim.bo.buftype ~= 'prompt'
            and
            not utils.rime_ls_disabled({ line = vim.api.nvim_get_current_line(),
                cursor = vim.api.nvim_win_get_cursor(0) }) then
            local content_before_cursor = string.sub(vim.api.nvim_get_current_line(),
                1, cursor_column)
            local code = string.sub(content_before_cursor,
                cursor_column - max_code + 1, cursor_column)
            if match_alphabet(code) then
                -- This is for wubi users using 'z' as reverse look up
                if not string.match(content_before_cursor, 'z[' .. alphabet .. ']*$') then
                    local first_rime_item_index = 
                        require('plugins.rime_ls').get_n_rime_item_index(1)
                    if #first_rime_item_index ~= 1 then
                        -- clear the wrong code
                        for _ = 1, max_code do
                            feedkeys('<bs>', 'n')
                        end
                    else
                        require('blink.cmp').accept({ index = first_rime_item_index[1] })
                        confirmed = true
                    end
                end
            end
        end
        if confirmed then
            vim.schedule(function() feedkeys(k, 'n') end)
        else
            feedkeys(k, 'n')
        end
    end)
end
```

## `nvim-cmp`

**注意**：目前我已不再使用 `nvim-cmp`，这部分的配置不再保证正确性，只是提供一个配置思路。

在 `nvim-cmp` 使用 `rime-ls` 在进行快速输入时体验并不是很好。
这是因为在 `nvim-cmp` 中并不是每次按键都会触发补全，
`nvim-cmp` 使用了 `throttle` 和 `debounce` 来减少补全的频率，这也导致在输入过快时，
补全列表不能及时出现。

这一部分的配置尽可能保证补全的正确性，但在输入过快时可能会有卡顿的情况，
这是由于 `nvim-cmp` 的特性导致的。如果想要更好的体验，可以使用 `blink-cmp`。

为了不影响其他不需要中文输入地方的体验，
这一部分的实现逻辑是在启动 `rime-ls` 时增加 `key-mapping` ，
关闭时删除 `key-mapping` 实现。因此配置的整体结构如下：

```lua
map.set({ 'n', 'i' }, '<c-space>', function()
  -- We must check the status before the toggle
  if vim.g.rime_enabled then
    -- delete key mappings here
  else
    -- add key mappings here
  end
  -- toggle Rime
  vim.cmd('RimeToggle')
end, opts())
```

删除按键部分没有难度，后文将重点讨论添加按键部分。

给出几个下面会用到的 `helper` 函数：

```lua
local cmp = require'cmp'

-- NOTE: there is no 'z' in the alphabet
-- The alphabet your schema used
local alphabet = "abcdefghijklmnopqrstuvwxy"

-- Feed keys with term code
-- keys: the keys to feed
-- mode: n for no-remap, m for re-map
local function feedkeys(keys, mode)
    local termcodes = vim.api.nvim_replace_termcodes(keys, true, true, true)
    vim.api.nvim_feedkeys(termcodes, mode, false)
end

-- return true if the content contains Chinese characters
local function contain_chinese_character(content)
    for i = 1, #content do
        local byte = string.byte(content, i)
        if byte >= 0xE4 and byte <= 0xE9 then
            return true
        end
    end
    return false
end

-- Return true if the cmp-entry is acceptable
-- you can define some other rules here to accept more
-- in this situation we only think the entry
-- whose content contains Chinese or is like time is acceptable
local function rime_entry_acceptable(entry)
    return entry ~= nil and entry.source.name == "nvim_lsp"
        and entry.source.source.client.name == "rime_ls"
        and (entry.word:match("%d%d%d%d%-%d%d%-%d%d %d%d:%d%d:%d%d") or
            contain_chinese_character(entry.word))
end

-- Return the first n entries which make rime_entry_acceptable(entry) return true
-- we call those entries rime-ls entries below
local function get_n_rime_ls_entries(n)
    if not cmp.visible() then
        return {}
    end
    local entries = cmp.get_entries()
    local result = {}
    if entries == nil or #entries == 0 then
        return result
    end
    for _, entry in ipairs(entries) do
        if rime_entry_acceptable(entry) then
            result[#result + 1] = entry
            if #result == n then
                break;
            end
        end
    end
    return result
end

-- Confirm the rime-ls entry
-- We use nvim_set_current_line to get the completion text,
-- because cmp.comfirm is slow and configured with throttle
-- for which make the completion list not pop up when typing fast
local function confirm_rime_ls_entry(entry)
    local line = vim.api.nvim_get_current_line()
    local cursor_column = vim.api.nvim_win_get_cursor(0)[2]
    local start = entry.source_insert_range.start.character
    local new_line =
        string.sub(line, 1, start) ..
        entry.word ..
        string.sub(line, cursor_column + 1)
    vim.api.nvim_set_current_line(new_line)
    vim.api.nvim_win_set_cursor(0, { vim.api.nvim_win_get_cursor(0)[1], start + #entry.word })
end

-- check if the content of txt is all in allowed
local function match_alphabet(txt, allowed)
    return string.match(txt, '^[' .. allowed .. ']+$') ~= nil
end

-- check if the last character will trigger rime-ls
local function last_character_in_alphabet()
    local cursor_column = vim.api.nvim_win_get_cursor(0)[2]
    if cursor_column == 0 then
        return false;
    end
    local trigger_alphabet = alphabet
    -- INFO:
    -- If you bind number with select_or_confirm_rime(x, false)
    -- uncomment this line to select more than once
    -- trigger_alphabet = trigger_alphabet .. "1234567890"

    return match_alphabet(string.sub(vim.api.nvim_get_current_line(),
            cursor_column,
            cursor_column),
        trigger_alphabet)
end

-- index: the index-th rime-ls entry
-- select_with_no_num: set true to upload the index-th rime-ls entry directly
--                     set false to feed index as number at first,
--                     then upload if only one rime-ls entry
-- return true if the number of rime-ls entries is enough
local function select_or_confirm_rime(index, select_with_no_num)
    if not last_character_in_alphabet() then
        return false
    end
    local rime_ls_entries = get_n_rime_ls_entries(index)
    if #rime_ls_entries < index then
        return false
    end
    if select_with_no_num then
        confirm_rime_ls_entry(rime_ls_entries[index])
        return true
    end
    local cursor_column = vim.api.nvim_win_get_cursor(0)[2]
    local line = vim.api.nvim_get_current_line()
    local new_line =
        string.sub(line, 1, cursor_column)
        ..
        tostring(index)
        ..
        string.sub(line, cursor_column + 1)
    vim.api.nvim_set_current_line(new_line)
    vim.api.nvim_win_set_cursor(0, { vim.api.nvim_win_get_cursor(0)[1], cursor_column + 1 })
    -- must trigger complete manually here,
    -- otherwise we can not get the new list after inputting a number
    cmp.complete({ config = { sources = { { name = 'nvim_lsp' } } } })
    local first_rime_ls_entry = get_n_rime_ls_entries(2)
    if #first_rime_ls_entry ~= 1 then
        return true
    end
    confirm_rime_ls_entry(first_rime_ls_entry[1])
    return true;
end

-- When k and v are not equal and v is not nil, feed v with remap mode,
-- otherwise feed k with no-remap mode.
-- This function is used when the original key is mapped to other functionality
local function feed_key_helper(k, v)
    if v == nil or k == v then
        feedkeys(k, 'n')
    else
        feedkeys(v, 'm')
    end
end
```

**注意**：后文使用的按键可能被某些插件绑定了一些其他的功能
（例如 `auto-pairs` 绑定 `<space>` 来在括号中插入两个空格），或者你自己绑定了一些功能，
如果你想同时保留两部分功能，你可以参考 <a href="#upload-word">选词功能</a> 部分的配置。

### <a id="upload-word">选词功能</a>

这里以空格首选，分号次选，单引号三选，数字键依次对应候选 1-9 为例，
先定义按键列表：

```lua
local mapped_key = {
    ['<space>'] = '<f30>',
    [';'] = ';',
    ["'"] = '<f31>',
    ['1'] = '1',
    ['2'] = '2',
    ['3'] = '3',
    ['4'] = '4',
    ['5'] = '5',
    ['6'] = '6',
    ['7'] = '7',
    ['8'] = '8',
    ['9'] = '9',
}
```

上面的空格和单引号已经被绑定了，这里以空格为例，先进行如下操作：

```lua
-- find a key will never be used, here we use <f30>
-- bind <f30> to the functionality that <space> should have
-- bind <space> to the functionality that <space> should have
map.set({ 'i' }, '<f30>', '<c-]><c-r>=AutoPairsSpace()<cr>', opts())
map.set({ 'i' }, '<space>', '<c-]><c-r>=AutoPairsSpace()<cr>', opts())
```

绑定上屏功能：

```lua
map.set({ 'i' }, k, function()
    -- when having selected an entry we do not upload
    -- if you want to upload, comment those lines
    if cmp.visible() and cmp.get_selected_entry() ~= nil then
        feed_key_helper(k, v)
        return
    end
    -- NOTE: if you want to use a key to select more than once change true to false
    if k == '<space>' and not select_or_confirm_rime(1, true) or
        k == ';' and not select_or_confirm_rime(2, true) or
        k == "'" and not select_or_confirm_rime(3, true) or
        k:match('[0-9]') and not select_or_confirm_rime(tonumber(k), true) then
        feed_key_helper(k, v);
    end
end, opts())
```

`select_or_confirm_rime(1, true)` 意味着会直接 `confirm` 第一个 rime-ls 候选词，
这表示你不能再进行后续后续选择。如果你喜欢输入句子，而不是单个词，你可以将 `true` 改为 `false`。

建议非语句流形码用户全部使用 `select_or_confirm_rime(x, true)`；
其他用户空格和二三候选使用 `select_or_confirm_rime(x, true)`，
数字按键使用 `select_or_confirm_rime(x, false)`。

### 中文标点

如果使用 `rime-ls` 进行标点输入，因为 lsp 的特性必须伴随一次选择才能触发上屏，
所以在输入标点时会有一些不同的体验。这里提供一种解决方案，即在输入标点时，先输入标点，
然后再输入空格来实现插入中文标点, 首先设置 rime-ls 为西文标点模式，然后增加如下配置：

```lua
local mapped_punc = {
    [','] = '，',
    ['.'] = '。',
    [':'] = '：',
    [';'] = '；',
    ['?'] = '？',
    ['\\'] = '、'
    -- ...
    -- add more you want here
}
-- Chinese punctuations
for k, v in pairs(mapped_punc) do
    map.set({ 'i' }, k .. '<space>', function()
        -- when typing comma or period with space,
        -- upload the first rime_ls entry and make comma and period in Chinese edition
        -- if you don't want this just comment those lines
        if k == ',' or k == '.' then
            select_or_confirm_rime(1, true)
        end

        feed_key_helper(v, v)
    end, opts())
end
```

### 输入过快补全列表消失

使用以上配置后，该问题已经解决。

### 顶字上屏

**注意**：使用顶字上屏可能会造成卡顿，不建议使用。

```lua
-- the max_code of your schema, wubi and flypy are 4
local max_code = 4

local function auto_upload_on_max_code(k)
    local cursor_column = vim.api.nvim_win_get_cursor(0)[2]
    if cursor_column >= max_code then
        local content_before_cursor = string.sub(vim.api.nvim_get_current_line(), 1, cursor_column)
        local code = string.sub(content_before_cursor, cursor_column - max_code + 1, cursor_column)
        if match_alphabet(code, alphabet) then
            -- This is for wubi users using 'z' as reverse look up
            -- If 'z' is you alphabet key, just comment this condition check
            if not string.match(content_before_cursor, 'z[' .. alphabet .. ']*$') then
                local first_rime_ls_entry = get_n_rime_ls_entries(1)
                if #first_rime_ls_entry ~= 1 then
                    -- clear the wrong code
                    -- uncomment this if you don't want to clear the wrong code
                    vim.api.nvim_win_set_cursor(0, { vim.api.nvim_win_get_cursor(0)[1],
                        cursor_column - max_code })
                    vim.api.nvim_set_current_line(string.sub(content_before_cursor, 1,
                        cursor_column - max_code) ..
                        string.sub(content_before_cursor, cursor_column + 1))
                else
                    confirm_rime_ls_entry(first_rime_ls_entry[1])
                    -- update the new cursor column
                    cursor_column = vim.api.nvim_win_get_cursor(0)[2]
                end
            end
        end
    end
    local line = vim.api.nvim_get_current_line()
    local new_line = string.sub(line, 1, cursor_column) .. k .. string.sub(line, cursor_column + 1)
    vim.api.nvim_set_current_line(new_line)
    vim.api.nvim_win_set_cursor(0, { vim.api.nvim_win_get_cursor(0)[1], cursor_column + 1 })
end

for i = 1, #alphabet do
    local k = alphabet:sub(i, i)
    map.set({ 'i' }, k, function() auto_upload_on_max_code(k) end, opts())
end
```

## 自动开关 `rime-ls`

在写 `markdown` 的时候，我是不会在一些符号中使用中文的，所以我写了这个功能。

这一部分我只在 `blink-cmp` 中进行了实现，但是实现方法是一致的：

```lua
-- When input method is enabled, disable the following patterns
vim.g.disable_rime_ls_pattern = {
    -- disable in ``
    '`(.*)`',
    -- disable in ''
    '\'(.*)\'',
    -- disable in ""
    '"(.*)"',
    -- disable in []
    '%[.*%]',
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

完整的配置：

* [blink-cmp](https://github.com/Kaiser-Yang/dotfiles/commit/9901e409c4ae61aae2cb49d99a613e48459eb74b#diff-90ad71516cfa13814a1992897e40e285bf784d1a4d123aad3eb53aac5c1ac5e5)
* [nvim-cmp](https://github.com/Kaiser-Yang/dotfiles/commit/3f027f0e2ebd7e123c2efae0a1b2d3d843756fa6#diff-e76b8f11166782d3be96100d19ebfccfeaea9b0b54f7e805b4c2bb69d6e4b446)

后续会有新的更改：[dotfiles](https://github.com/Kaiser-Yang/dotfiles)
