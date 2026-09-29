# zsh Shell Setup

### 1. Install zsh and set it as the default shell
Make sure zsh is available on your OS, then set it as the default. One way to do this
is by adding the following line to the top of `~/.profile`:
```
SHELL=/bin/zsh exec /bin/zsh --login
```

### 2. Install oh-my-zsh
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
This clones oh-my-zsh to `~/.oh-my-zsh` and generates a starter `~/.zshrc`.

### 3. Install the Nerd Font
This repo already vendors the font we use (`fonts/AnonymousPro`, "Anonymice Nerd Font") —
no need to download anything from nerdfonts.com. Install is OS-dependent:
- **macOS**: open the `.ttf` files in `fonts/AnonymousPro/` with Font Book, or copy them
  into `~/Library/Fonts`.
- **Linux**: copy the `.ttf` files into `~/.local/share/fonts` (or `/usr/share/fonts`),
  then run `fc-cache -f`.

Then set the terminal's font to "Anonymice Nerd Font Mono" (see `tool_configs/ghostty/config`
for an example).

### 4. Install powerlevel10k
```
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom/themes}/powerlevel10k
```

### 5. Install this repo's zsh config
Rather than re-running the `p10k configure` wizard and re-adding plugins/aliases by hand,
install the config already captured in this repo:
```
cp tool_configs/zsh/zshrc ~/.zshrc
cp tool_configs/zsh/p10k.zsh ~/.p10k.zsh
```
`tool_configs/zsh/zshrc` sets `ZSH_THEME="powerlevel10k/powerlevel10k"`, `plugins=(git)`, vi
keybindings, and pyenv/nvm shims (these no-op safely if pyenv/nvm aren't installed yet —
see the [python3 template](../templates/python3) and add nvm/pyenv setup separately if
needed). `tool_configs/zsh/p10k.zsh` is the full wizard-generated prompt style.

## References
- https://ohmyz.sh/
- https://github.com/romkatv/powerlevel10k
- https://www.nerdfonts.com/
