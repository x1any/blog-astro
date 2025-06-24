---
title: '终端模拟器 Wezterm'
published: 2025-06-24T07:54:45.975Z
description: ''
updated: ''
tags:
  - Note
draft: false
pin: 0
toc: true
lang: ''
abbrlink: ''
---
## 官网

* [WezTerm - Wez&apos;s Terminal Emulator](https://wezfurlong.org/wezterm/index.html "WezTerm - Wez's Terminal Emulator")
* [Releases · wez/wezterm · GitHub](https://github.com/wez/wezterm/releases/ "Releases · wez/wezterm · GitHub")

## 配置

路径 `~/.config/wezterm/wezterm.lua`

```lua
local wezterm = require 'wezterm';

local default_prog = { 'pwsh.exe', '-NoLogo' }

local launch_menu = {}
if wezterm.target_triple == 'x86_64-pc-windows-msvc' then
    table.insert(launch_menu, {
        label = 'PowerShell',
        args = { 'pwsh.exe', '-NoLogo' },
    })

    table.insert(launch_menu, {
        label = 'Windows PowerShell',
        args = { 'powershell.exe', '-NoLogo' },
    })

    table.insert(launch_menu, {
        label = 'CMD',
        args = { 'cmd.exe', '-NoLogo' },
    })

    table.insert(launch_menu,  {
        label = 'UCRT64 / zsh',
        args = { 'ucrt64.exe', '-defterm', '-here', '-no-start', '-shell', 'zsh' }
    })

    -- Find installed visual studio version(s) and add their compilation
    -- environment command prompts to the menu
    for _, vsvers in
    ipairs(
        wezterm.glob('Microsoft Visual Studio/20*', 'C:/Program Files (x86)')
    )
    do
        local year = vsvers:gsub('Microsoft Visual Studio/', '')
        table.insert(launch_menu, {
            label = 'x64 Native Tools VS ' .. year,
            args = {
                'cmd.exe',
                '/k',
                'C:/Program Files (x86)/'
                .. vsvers
                .. '/BuildTools/VC/Auxiliary/Build/vcvars64.bat',
            },
        })
    end
end

local materia = wezterm.color.get_builtin_schemes()['MaterialDesignColors']
materia.scrollbar_thumb = '#888888'

local window_decorations = "INTEGRATED_BUTTONS|RESIZE"

local font = wezterm.font_with_fallback({ "Maple Mono NF CN" })

local act = wezterm.action;
local key_bindings = {
    -- F11 切换全屏
    { key = 'F11',       mods = 'NONE',       action = act.ToggleFullScreen },
    -- Alt+L 显示启动菜单
    { key = 'l',         mods = 'ALT',        action = act.ShowLauncher },
    -- Ctrl+Shift+N 新窗口
    { key = 'N',         mods = 'SHIFT|CTRL', action = act.SpawnWindow },
    -- Ctrl+Tab 遍历 tab
    { key = "Tab",       mods = "SHIFT|CTRL", action = act.ActivateTabRelative(1) },
    -- Ctrl+Shift+Tab 反向遍历 tab
    { key = "Tab",       mods = "SHIFT|CTRL", action = act.ActivateTabRelative(-1) },
    -- Ctrl+Shift+W 关闭 panel 且不进行确认
    { key = 'W',         mods = 'SHIFT|CTRL', action = act.CloseCurrentPane { confirm = false } },
    -- Ctrl+Shift+PageUp 向上滚动一页
    { key = 'PageUp',    mods = 'SHIFT|CTRL', action = act.ScrollByPage(-1) },
    -- Ctrl+Shift+PageDown 向下滚动一页
    { key = 'PageDown',  mods = 'SHIFT|CTRL', action = act.ScrollByPage(1) },
    -- Ctrl+Shift+UpArrow 向上滚动一行
    { key = 'UpArrow',   mods = 'SHIFT|CTRL', action = act.ScrollByLine(-1) },
    -- Ctrl+Shift+DownArrow 向下滚动一行
    { key = 'DownArrow', mods = 'SHIFT|CTRL', action = act.ScrollByLine(1) },
}

local mouse_bindings = {
    -- Click 不打开链接
    {
        event = { Up = { streak = 1, button = "Left" } },
        mods = "NONE",
        action = act.CompleteSelection("PrimarySelection"),
    },
    -- CTRL-Click 打开链接
    {
        event = { Up = { streak = 1, button = "Left" } },
        mods = "CTRL",
        action = act.OpenLinkAtMouseCursor,
    },
    -- RCLick 粘贴
    {
        event = { Down = { streak = 1, button = "Right" } },
        mods = "NONE",
        action = act.PasteFrom("Clipboard")
    },
    -- CTRL-RCLick 复制
    {
        event = { Down = { streak = 1, button = "Right" } },
        mods = "CTRL",
        action = act.CopyTo("Clipboard")
    },
}

local config
if wezterm.config_builder then
    config = wezterm.config_builder()
else
    config = {}
end

config.exit_behavior = "Close"
config.initial_cols = 120
config.initial_rows = 36
config.font_size = 12
config.font_shaper = "Harfbuzz"
config.harfbuzz_features = { 'calt=1', 'clig=1', 'liga=0' }
config.tab_max_width = 24
config.enable_wayland = false
config.enable_tab_bar = true
config.enable_scroll_bar = true
config.window_background_opacity = 1.0
config.window_decorations = window_decorations
config.colors = materia
config.font = font
config.default_prog = default_prog
config.launch_menu = launch_menu
config.keys = key_bindings
config.mouse_bindings = mouse_bindings

return config
```
