# zsh Notes
zsh provides some improvements for interactive use compared to Bash.

Syntax is mostly compatible with Bash, but not exactly. 
Typically, zsh is used interactively, while scripts are written in Bash.

## zsh on Ubuntu/Linux Mint
- zsh straight out of the box is very limited
- Oh-my-zsh is a common recommendation, but it's overwhelming.
- Manjaro has an good zsh configuration
- Here, we mimic Manjaro's configuration, with some tweaks.

## Install
``` bash
apt install zsh zsh-autosuggestions zsh-syntax-highlighting fonts-powerline
mkdir ~/.zshstuff
```
Don't run zsh yet.

Recommended: 
- Install "Meslo Nerd Font patched for Powerlevel10k"
- https://github.com/romkatv/powerlevel10k/tree/master#meslo-nerd-font-patched-for-powerlevel10k
- Set your terminal to use `MesloLGS NF`

## Get manjaro's zsh config
``` bash
cd ~/Downloads
git clone --depth=1 https://github.com/Chrysostomus/manjaro-zsh-config
cp manjaro-zsh-config/manjaro-zsh-config ~/.zshstuff/
```
edit `~/.zshstuff/manjaro-zsh-config` and comment out the entire **Plugins** section.

## Install powerlevel10k
`git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ~/.zshstuff/powerlevel10k`

## create and edit ~/.zshrc
``` bash
source ~/.zshstuff/manjaro-zsh-config

# hit Tab twice displays menu
zstyle ':completion:*' menu select

# same history in all open zsh instances
unsetopt inc_append_history
setopt share_history

## Plugins section: Enable fish style features
source /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh

# powerlevel10k
source ~/.zshstuff/powerlevel10k/powerlevel10k.zsh-theme

# To customize prompt, run p10k configure or edit ~/.p10k.zsh.
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
```
## Configure Powerlevel10k
- run zsh, it will automatically run the powerlevel10k config.
- It asks a bunch of questions. Usually, you want the top choice, but not always.
- This creates a config file `~/.p10k.zsh`
- If you want to change something, don't edit the config file manually, it's too complex
- just re-run the config, `p10k configure`
