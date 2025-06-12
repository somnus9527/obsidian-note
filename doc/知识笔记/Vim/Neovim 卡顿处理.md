---
tags:
  - 知识笔记/Vim
---
#### 使用工具profile.nvim

[profile.nvim](https://github.com/stevearc/profile.nvim)

配置如下：
```lua
return {
  'stevearc/profile.nvim',
  config = function()
    local should_profile = os.getenv 'NVIM_PROFILE'
    if should_profile then
      require('profile').instrument_autocmds()
      if should_profile:lower():match '^start' then
        require('profile').start '*'
      else
        require('profile').instrument '*'
      end
    end

    local function toggle_profile()
      local prof = require 'profile'
      if prof.is_recording() then
        prof.stop()
        vim.ui.input({ prompt = 'Save profile to:', completion = 'file', default = 'profile.json' }, function(filename)
          if filename then
            prof.export(filename)
            vim.notify(string.format('Wrote %s', filename))
          end
        end)
      else
        prof.start '*'
      end
    end
    vim.keymap.set('', '<f1>', toggle_profile)
  end,
  cond = function()
    -- 命令行参数NVIM_PROFILE有值才加载
    local should_profile = os.getenv 'NVIM_PROFILE'
    if should_profile then
      return true
    end
  end
}
```
生产profile.json之后使用分析工具进行分析，查看具体哪个function耗时, [分析工具地址](https://ui.perfetto.dev/)

我这边主要就是treesitter和gitsigns耗时

1. treesitter的问题无法处理，只能针对大文件禁用，这里超过50k就禁用
```lua
local max_filesize = 50 * 1024 -- 50 KB
local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(buf))
if ok and stats and stats.size > max_filesize then
  return true
end

```
2. gitsigns在非git repo下加载造成gitsigns一直拉取git状态造成卡顿，还是通过cond判断是否加载处理
```lua
cond = function()
  if vim.api.nvim_command_output '!git rev-parse --is-inside-work-tree' == true then
    return true
  end
end,

```