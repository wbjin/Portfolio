---
title: "Sync-nvim"
description: "A neovim plugin for syncing files with remote hosts"
publishDate: "December 2023"
tags: ["lua", "personal"]
---

<a href="https://github.com/wbjin/sync-nvim">
  <img
    src="/github-mark-white.svg"
    alt="GitHub"
    class="w-20 h-20"
    style="filter: brightness(0) invert(1);"
  />
</a>


## Sync
Sync is a plugin for syncing with remote hosts while developing locally on
neovim.

## Motivation
I built this plugin because I found myself using remote instances a lot for
research and for experimentation. I often needed to use nodes with access to
certain CPUs and/or GPUs and constantly pushing and pulling from github to sync
code on the remote machine was gettting very painful. Sync allows you to sync
your remote codebase with your local codebase easily using a simple command
`:Sync` or `:SyncInclude` using `rsync` underneath the hood. You specify the
files you want to sync in your project in the `/nvim/config.lua` file or sync
the entire codebase excluding certain directories like `env`.
