---
layout: post
title: Contributing to blink-cmp-git
date: 2025-05-08 20:21:15+0800
last_updated: 2025-05-08 20:21:15+0800
description: A guide to adding support for new Git hosting services in blink-cmp-git.
tags:
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## Brief Introduction

[blink-cmp-git](https://github.com/Kaiser-Yang/blink-cmp-git) is a source for
[blink.cmp](https://github.com/Saghen/blink.cmp). The source is able to provide completion items
related to a git repository. For example, if a git repository's remote is `GitHub`, the source
can get the issues, pull requests, and contributors from the `GitHub` by using `gh` or `curl`.

Actually, the source is able to provide completion items for other git hosting platforms.
This post explains how to configure additional git hosting platforms,
which is useful when you are using your own git hosting platforms.

By the way, if you find that the source has not configured the git hosting platforms you are using,
please feel free to create a pull request or an issue. As for `v3.0.0`, the source supports `GitHub`
and `GitLab`. For enterprise deployments (GitHub Enterprise/GitLab EE), it is possible to support
by updating some configuration.

## Configuration

The source is able to support other git hosting platforms by adding new configuration files in
`lua/blink-cmp-git/default`.

Let's dive into how `GitLab` is supported.

Firstly, we should create a file under `lua/blink-cmp-git/default`. We use `gitlab.lua` here.

Secondly, we should return an object of
[`blink-cmp-git.GCSOptions`](https://github.com/Kaiser-Yang/blink-cmp-git/blob/master/lua/blink-cmp-git/types.lua#L30)
in this file:

```lua
--- @type blink-cmp-git.GCSOptions
return {
    issue = ...
    pull_request = ...
    mention = ...
}
```

The `issue`, `pull_request`, and `mention` are the objects of
[`blink-cmp-git.GCSCompletionOptions`](https://github.com/Kaiser-Yang/blink-cmp-git/blob/master/lua/blink-cmp-git/types.lua#L15),
which has those fields below:

* `enable`: A boolean or a function. If it is a function, it should return a boolean. When `true`
or the function returns `true`, the source will provide completion items for this type.
* `triggers`: A list of strings or a function. If it is a function, it should return a list of
strings. The source will trigger completion when the input matches these strings.
* `get_token`: A string or a function. If it is a function, it should return a string. The result
will be parsed to `get_command_args` field to support configure PAT of some git hosting platforms.
* `get_command`: A string or a function. If it is a function, it should return a string. This is the
command to get the completion items.
* `get_command_args`: A list of strings or a function. If it is a function, it should return a list
of strings. This is the arguments of the command to get the completion items.
* `insert_text_trailing`: A string or a function. If it is a function, it should return a string.
This value will be inserted when users confirm or select the completion item.
* `separate_output`: A function that receives the output of the command and returns a list of any
type. Each item of the return value will be assembled into a completion item.
* `get_label`: A function that receives the item of the list of `separate_output`
and returns a string. Label is the element related with matching.
* `get_kind_name`: A function that receives the item of the list of `separate_output`
and returns a string.
* `get_insert_text`: A function that receives the item of the list of `separate_output`
and returns a string. This value is what will be inserted when the item is confirmed or selected.
* `get_documentation`: A function that receives the item of the list of `separate_output`
and returns a string or an object of
[`blink-cmp-git.DocumentationCommand`](https://github.com/Kaiser-Yang/blink-cmp-git/blob/master/lua/blink-cmp-git/types.lua#L1).
Documentation usually shows the description of the completion item.
* `configure_score_offset`: A function receives the list of completion items. The function is used
to decide how to order the completion items.
* `on_error`: A function receives the return value and standard error output of the command.
The function will be called when the command fails (non-zero return or non-empty standard error).

### `GitLab` Implementation

Let's dive into how to implement all the features of `GitLab`.

For `issue`, `pull_request`, and `mention`. We use a function as the default enable. This function
will return `true` when the remote's URL contains `gitlab.com`:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
local function default_gitlab_enable()
    if
        not utils.command_found('git')
        or not utils.command_found('glab') and not utils.command_found('curl')
    then
        return false
    end
    return utils.get_repo_remote_url():find('gitlab%.com')
end
return {
    issue = {
        enable = default_gitlab_enable,
    },
    pull_request = {
        enable = default_gitlab_enable,
    },
    mention = {
        enable = default_gitlab_enable,
    },
}
```

The `utils` is from `require('blink-cmp-git.utils')`, which provides some useful functions.

For the triggers, we use `#` for `issue`, `!` for `pull_request`, and `@` for `mention`, which are
the default triggers of `GitLab`. The code is:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        triggers = { '#' },
    },
    pull_request = {
        triggers = { '!' },
    },
    mention = {
        triggers = { '@' },
    },
}
```

For `get_token`, we can use an empty string as the default value:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        get_token = '',
    },
    pull_request = {
        get_token = '',
    },
    mention = {
        get_token = '',
    },
}
```

For `get_command`, we hope to use `glab` when found, otherwise use `curl`. For other git hosting
platforms, if there is a CLI tool, we recommend to try to use it.
If not, `curl` can be as the default:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
local function default_gitlab_get_command()
    return utils.command_found('glab') and 'glab' or 'curl'
end
return {
    issue = {
        get_command = default_gitlab_get_command,
    },
    pull_request = {
        get_command = default_gitlab_get_command,
    },
    mention = {
        get_command = default_gitlab_get_command,
    },
}
```

Because we may get `glab` or `curl` as the command, in `get_command_args` we must set different
arguments for them (we only give the code for `issue` here):

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
local function basic_args_for_gitlab_api(token)
    if utils.truthy(token) then return {
        '-H',
        'PRIVATE-TOKEN: ' .. token,
    } end
    return {}
end
return {
    issue = {
        get_command_args = function(command, token)
            local args = basic_args_for_gitlab_api(token)
            if command == 'curl' then
                table.insert(args, '-s')
                table.insert(args, '-f')
                table.insert(
                    args,
                    'https://gitlab.com/api/v4/projects/'
                        .. utils.get_repo_owner_and_repo(true)
                        .. '/issues'
                )
            else
                table.insert(args, 1, 'api')
                table.insert(args, 'projects/' .. utils.get_repo_owner_and_repo(true) .. '/issues')
            end
            return args
        end,
    },
}
```

Please make sure the last argument of `get_command_args` is the URL of the API. This will make it
easy for users to configure if they using enterprise version. The `utils.get_repo_owner_and_repo`
is a function that getting the owner and repository name of the current git repository.
The parameter `true` means that the return value is encoded as URL (using '%20' instead of space).
Some git hosting platforms require encoding, while some do not.
You need to check the API documentation of the git hosting platform.


For `insert_text_trailing`, we can use an empty string as the default value:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        insert_text_trailing = '',
    },
    pull_request = {
        insert_text_trailing = '',
    },
    mention = {
        insert_text_trailing = '',
    },
}
```

For the output of the command, which is a `JSON` list, we can use the method
`json_array_separator` defined in `blink-cmp-git/lua/blink-cmp-git/default/common.lua`
to parse it into a `lua` table:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        separate_output = require('blink-cmp-git.default.common').json_array_separator,
    },
    pull_request = {
        separate_output = require('blink-cmp-git.default.common').json_array_separator,
    },
    mention = {
        separate_output = require('blink-cmp-git.default.common').json_array_separator,
    },
}
```

We strongly recommend you to create a file in `doc/item` to let users know what the item looks
like. For example, the each issue item after separating:

```lua
-- In file 'doc/item/gitlab/issue.lua'
-- All empty strings in the table will be set to nil
return {
    id = 161591198,
    iid = 7758,
    project_id = 34675721,
    title = 'New command to delete labels',
    description = [[
### Problem to solve

Right now you can only crate or list labels, not delete them.

### Proposal

Add a new command to delete labels.

### Links / references
    ]],
    state = 'closed',
    created_at = '2025-02-01T15:37:10.879Z',
    updated_at = '2025-02-05T08:31:45.636Z',
    closed_at = '2025-02-05T08:31:45.593Z',
    closed_by = {
        id = 3457201,
        username = 'viktomas',
        name = 'Tomas Vik',
        state = 'active',
        locked = false,
        avatar_url = 'https://gitlab.com/uploads/-/system/user/avatar/3457201/avatar.png',
        web_url = 'https://gitlab.com/viktomas',
    },
    labels = {
        'devops::create',
        'group::code review',
        'section::dev',
        'type::feature',
    },
    milestone = {
        id = 4599724,
        iid = 108,
        group_id = 9970,
        title = '17.9',
        description = nil,
        state = 'active',
        created_at = '2024-05-24T19:18:50.640Z',
        updated_at = '2024-05-24T19:18:50.640Z',
        due_date = '2025-02-14',
        start_date = '2025-01-11',
        expired = false,
        web_url = 'https://gitlab.com/groups/gitlab-org/-/milestones/108',
    },
    assignees = {
        {
            id = 2090646,
            username = 'rndmh3ro',
            name = 'Sebastian Gumprich',
            state = 'active',
            locked = false,
            avatar_url = 'https://gitlab.com/uploads/-/system/user/avatar/2090646/avatar.png',
            web_url = 'https://gitlab.com/rndmh3ro',
        },
    },
    author = {
        id = 2090646,
        username = 'rndmh3ro',
        name = 'Sebastian Gumprich',
        state = 'active',
        locked = false,
        avatar_url = 'https://gitlab.com/uploads/-/system/user/avatar/2090646/avatar.png',
        web_url = 'https://gitlab.com/rndmh3ro',
    },
    type = 'ISSUE',
    assignee = {
        id = 2090646,
        username = 'rndmh3ro',
        name = 'Sebastian Gumprich',
        state = 'active',
        locked = false,
        avatar_url = 'https://gitlab.com/uploads/-/system/user/avatar/2090646/avatar.png',
        web_url = 'https://gitlab.com/rndmh3ro',
    },
    user_notes_count = 0,
    merge_requests_count = 1,
    upvotes = 0,
    downvotes = 0,
    due_date = nil,
    confidential = false,
    discussion_locked = nil,
    issue_type = 'issue',
    web_url = 'https://gitlab.com/gitlab-org/cli/-/issues/7758',
    time_stats = {
        time_estimate = 0,
        total_time_spent = 0,
        human_time_estimate = nil,
        human_total_time_spent = nil,
    },
    task_completion_status = {
        count = 0,
        completed_count = 0,
    },
    weight = nil,
    blocking_issues_count = 0,
    has_tasks = true,
    task_status = '0 of 0 checklist items completed',
    _links = {
        self = 'https://gitlab.com/api/v4/projects/34675721/issues/7758',
        notes = 'https://gitlab.com/api/v4/projects/34675721/issues/7758/notes',
        award_emoji = 'https://gitlab.com/api/v4/projects/34675721/issues/7758/award_emoji',
        project = 'https://gitlab.com/api/v4/projects/34675721',
        closed_as_duplicate_of = nil,
    },
    references = {
        short = '#7758',
        relative = '#7758',
        full = 'gitlab-org/cli#7758',
    },
    severity = 'UNKNOWN',
    moved_to_id = nil,
    imported = false,
    imported_from = 'none',
    service_desk_reply_to = nil,
    epic_iid = nil,
    epic = nil,
    iteration = nil,
    health_status = nil,
}
```

Now, we can configure the getters for the completion items. For `issue`, we can use the
`iid` and its title as the label, `iid` as the insert text, `Issue` as the kind name, and
use some useful information to assemble the documentation:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
  issue = {
    get_label = function(item)
      return utils.concat_when_all_true('#', item.iid, ' ', item.title, '')
    end,

    get_kind_name = function(_)
      return 'Issue'
    end,

    get_insert_text = function(item)
      return utils.concat_when_all_true('#', item.iid, '')
    end,

    get_documentation = function(item)
      return utils.concat_when_all_true('#', item.iid, ' ', item.title, '\n')
        .. utils.concat_when_all_true(
          'State: ',
          item.discussion_locked and 'locked' 
            or item.draft and 'draft' 
            or item.state,
          '\n'
        )
        .. utils.concat_when_all_true('Author: ', item.author.username, '')
        .. utils.concat_when_all_true(' (', item.author.name, ')')
        .. '\n'
        .. utils.concat_when_all_true('Created at: ', item.created_at, '\n')
        .. utils.concat_when_all_true('Updated at: ', item.updated_at, '\n')
        .. (
          item.state == 'merged' 
          and utils.concat_when_all_true('Merged  at: ', item.merged_at, '\n')
            .. utils.concat_when_all_true('Merged  by: ', item.merged_by.username, '')
            .. utils.concat_when_all_true(' (', item.merged_by.name, ')')
            .. '\n' 
          or item.state == 'closed' 
          and utils.concat_when_all_true('Closed  at: ', item.closed_at, '\n')
            .. utils.concat_when_all_true('Closed  by: ', item.closed_by.username, '')
            .. utils.concat_when_all_true(' (', item.closed_by.name, ')')
            .. '\n' 
          or ''
        )
        .. utils.concat_when_all_true(item.description)
    end
  }
}
```

Sometimes, we can not assemble the documentation from the output of the command,
or we want more detailed information as the documentation.
In this case,
we can return an object of `blink-cmp-git.DocumentationCommand` in the `get_documentation` function,
for example, the `get_documentation` of `mention`:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    mention = {
        get_documentation = function(item)
            return {
                get_token = '',
                get_command = default_gitlab_get_command,
                get_command_args = function(command, token)
                    local args = basic_args_for_gitlab_api(token)
                    if command == 'curl' then
                        table.insert(args, '-s')
                        table.insert(args, '-f')
                        table.insert(args, 'https://gitlab.com/api/v4/users/' .. tostring(item.id))
                    else
                        table.insert(args, 1, 'api')
                        table.insert(args, 'users/' .. tostring(item.id))
                    end
                    return args
                end,
                resolve_documentation = function(output)
                    local user_info = utils.json_decode(output)
                    utils.remove_empty_string_value(user_info)
                    return utils.concat_when_all_true(user_info.username, '')
                        .. utils.concat_when_all_true(' (', user_info.name, ')')
                        .. '\n'
                        .. utils.concat_when_all_true('Location: ', user_info.location, '\n')
                        .. utils.concat_when_all_true('Email: ', user_info.public_email, '\n')
                        .. utils.concat_when_all_true('Company: ', user_info.work_information, '\n')
                        .. utils.concat_when_all_true('Created at: ', user_info.created_at, '\n')
                end,
                on_error = common.default_on_error,
            }
        end,
    },
}
```

For the `configure_score_offset`, we can sort the completion items as original order:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        configure_score_offset = require('blink-cmp-git.default.common').score_offset_origin,
    },
    pull_request = {
        configure_score_offset = require('blink-cmp-git.default.common').score_offset_origin,
    },
    mention = {
        configure_score_offset = require('blink-cmp-git.default.common').score_offset_origin,
    },
}
```

For the `on_error`, we can use the `default_on_error` function defined in
`lua/blink-cmp-git/default/common.lua`:

```lua
-- In file 'lua/blink-cmp-git/default/gitlab.lua'
-- NOTE: Some code is omitted
return {
    issue = {
        on_error = require('blink-cmp-git.default.common').default_on_error,
    },
    pull_request = {
        on_error = require('blink-cmp-git.default.common').default_on_error,
    },
    mention = {
        on_error = require('blink-cmp-git.default.common').default_on_error,
    },
}
```

At last, we just need to add this table to `lua/blink-cmp-git/default/init.lua`:

```lua
-- In file 'lua/blink-cmp-git/default/init.lua'
-- NOTE: Some code is omitted
return {
    git_centers = {
        gitlab = require('blink-cmp-git.default.gitlab'),
    }
}
```

Maybe you need to update the `lua/blink-cmp-git/health.lua` to add some checks:

```lua
-- In file 'lua/blink-cmp-git/health.lua'
-- NOTE: Some code is omitted
function M.check()
    health.start('blink-cmp-git')
    check_command_executable('git')
    local should_check_curl = false
    should_check_curl = should_check_curl
        or not check_command_executable('glab', '"glab" not found, will use "curl" instead.')
    if should_check_curl then check_command_executable('curl') end
end
```

Now it's time to do some simple tests.

If this update works well,
you can update the `README.md` to add some information about the new git center.
Then you can create a pull request to the `blink-cmp-git` repository.
