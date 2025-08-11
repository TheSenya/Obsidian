## LazyVim Specific Fix

Create `~/.config/nvim/lua/plugins/kitty-fix.lua`:

```lua
return {
  {
    "neovim/nvim-lspconfig",
    opts = function(_, opts)
      -- Fix for double input in Kitty
      if vim.env.TERM == "xterm-kitty" then
        vim.schedule(function()
          vim.cmd([[silent! !kitty @ set-spacing padding=0]])
        end)
      end
      return opts
    end,
  },
}
```
