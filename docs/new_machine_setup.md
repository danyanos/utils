# New Machine Setup

Checklist for bringing up a new dev box from this repo. Clone this repo first, then work
through the list below. See [README.md](../README.md#manifest) for the canonical
source → destination path for every config.

- [ ] **Homebrew** (or OS package manager) — TODO: not yet captured in this repo.
- [ ] **Shell**: zsh, oh-my-zsh, powerlevel10k, Nerd Font — see
      [docs/zsh_installation.md](zsh_installation.md).
- [ ] **Terminal**: [ghostty](https://ghostty.org) — copy `tool_configs/ghostty/config` to
      `~/.config/ghostty/config`.
      (iTerm2 — TODO: not yet captured in this repo, if you go that route instead.)
- [ ] **Editor**: vim — copy `tool_configs/vim/vimrc` to `~/.vimrc`, and
      `tool_configs/vim/colors` / `tool_configs/vim/pack` to `~/.vim/colors` / `~/.vim/pack`.
      (VS Code — TODO: not yet captured in this repo, beyond the `.vscode/` settings
      bundled in `templates/python3`.)
- [ ] **Fonts**: install `fonts/AnonymousPro` and `fonts/Terminus` (see
      [docs/zsh_installation.md](zsh_installation.md#3-install-the-nerd-font) for
      OS-specific steps).
- [ ] **Claude Code global config**: copy `agents/AGENTS.md` to `~/.claude/CLAUDE.md`.
- [ ] **Python projects**: use `templates/python3` as the starting point for new
      Python apps (per-project, not machine-wide).
