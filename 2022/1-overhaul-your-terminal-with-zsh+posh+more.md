This guide will look at various ways you can improve your productivity by extending the functionality of your terminal, including:
- **zsh**: A powerful shell that extends the feature set of bash.
- **zsh plugins**: Auto-suggestions, completion, syntax highlighting and more.
- **prompts + themes**: Customize a clean prompt that displays contextual information, either using Powerlevel10k or Posh.

 
## Getting Started
1. Install [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh#basic-installation) for any Unix based shell or WSL2, which bundles `zsh` with a set of plugins and themes.
2. Download a [Nerd Font](https://www.nerdfonts.com/font-downloads) and set your favourite terminal to use it. I like `Cascaydia Cove Nerd Font` the best.

 
## Zsh Plugins
To [enable plugins](https://github.com/ohmyzsh/ohmyzsh#enabling-plugins) locate your `.zshrc` in your `$HOME` directory, and add to the list of plugins:
```bash
plugins=(
  git
  ...
)
```

### Plugins Included with oh-my-zsh
- `sudo`: Easily prefix your current or previous commands with `sudo` by pressing <kbd>esc</kbd> twice.
- `dotenv`: Automatically load your project `ENV` variables from `.env` file when you cd into project root directory. By default, it will prompt before loading, but you can turn this off by adding the following to your `.zshrc` (though be wary of the security implications of this):
```bash
export ZSH_DOTENV_PROMPT=false
```
- `copypath`: Copies the path of given directory or file to the system clipboard.
- `copyfile`: Puts the contents of a file in your system clipboard so you can paste it anywhere.
- `copybuffer`: This plugin binds the <kbd>ctrl-o</kbd> keyboard shortcut to a command that copies the text that is currently typed in the command line (`$BUFFER`) to the system clipboard.
- `command-not-found`: provide suggested packages to be installed if a command cannot be found.
- `web-search`: This plugin adds aliases for searching with Google, Wiki, Bing, YouTube and other popular services.
- `history`: Provides a couple of convenient aliases for using the history command to examine your command line history.
- `dirhistory`: Adds keyboard shortcuts for navigating directory history and hierarchy. Uses <kbd>alt-arrowkeys</kbd> by default, so ensure these do not conflict with other shortcuts.
- `colorize`: Syntax-highlight file contents of over 300 supported languages and other text formats.
- `colored-man-pages`: Adds colors to man pages.
- `magic-enter`: This plugin makes your enter key magical, by binding commonly used commands to it. You customize the command it uses, and run specific commands in a Git repo:
```bash
MAGIC_ENTER_GIT_COMMAND='git status -sb .'
MAGIC_ENTER_OTHER_COMMAND='ls -la .'
```
- `jsontools`: Handy command line tools for dealing with json data.

Depending on what other software you use, there are plenty of lightweight plugins that improve autocompletion and provide aliases among other things. I use:
- `git`: Aliases for Git
- `gh`: Completions for GitHub cli 
- `git-auto-fetch`: Periodically run `git fetch`
- `nvm`: Completions for nvm
- `npm`:  Completions + aliases for npm
- `aws`: Completions for aws
- `docker`: Completions for docker
- `docker-compose`: Completions + aliases for docker compose
- `kubectl`" Completions + aliases for kubectl
- `vscode`: Aliases for VS Code or VSCodium editor
- `python`: Aliases for python
- `pip`: Automcompletions for pip
- `tmux`: Aliases for tmux

### Powerful Plugins
These plugins do not come with `oh-my-zsh`, so you must clone the repos and place them in `~/ohmyzsh/custom/plugins/` before adding them to the `plugins` list in the `.zshrc`.
- [`zsh-autosuggestions`](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md#oh-my-zsh):
Suggests commands as you type based on history and completions, which can then be selected with <kbd>→</kbd>.
![zsh-autosuggestions](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rek9zpxp2lkjou2n64f3.png)
 - Autosuggestions can either use your `history`, or tab `completion`. Set this with the `ZSH_AUTOSUGGEST_STRATEGY` variable.
 - Has some compatability issues where the suggestion is not cleared after certain actions. To fix this, add the following to your `.zshrc`:
```bash
ZSH_AUTOSUGGEST_CLEAR_WIDGETS+=(bracketed-paste up-line-or-search down-line-or-search expand-or-complete accept-line push-line-or-edit)
```
- [`zsh-syntax-highlighting`](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md#oh-my-zsh): Syntax highlighting for the shell zsh.
![zsh-syntax-highlighting](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n07i2pez1s2z23awk3id.png)
- [`zsh-autocomplete`](https://github.com/marlonrichert/zsh-autocomplete#manual-installation): Provides completion suggestions below the prompt, in addition to a history menu. This one is powerful, but might take some getting used to. Like with autosuggestions, you can choose whether you want to use normal completions (directories, man, etc.) or history search. I like to have autocomplete for normal completions and autosuggestions for history.
![zsh-autocomplete](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ighgss04o86loeocmjhc.png)
 If you're like me and don't like the history menu, but want all the other features, you can change the keybinds to use a simple history search in your `.zshrc` (make sure this is _after_ the plugins have been loaded):
```bash
bindkey '\e[A' history-beginning-search-backward
bindkey '\eOA' history-beginning-search-backward
bindkey '\e[B' history-beginning-search-forward
bindkey '\eOB' history-beginning-search-forward
zle -A {.,}history-incremental-search-forward
zle -A {.,}history-incremental-search-backward
```
 Similarly, to use the default tab completion, set the following options:
```bash
zstyle ':autocomplete:*' widget-style menu-select
bindkey -M menuselect '\r' accept-line
```
 I also don't like the completions to move the prompt around too much, especially for a multiline prompt. We can change this with:
```bash
zstyle ':autocomplete:*' list-lines 7
```
 The plugin also has some issues with the prompt "" showing on some completions and eating inputs. Add this to your `.zshrc` as a workaround (however this sometimes drops completions entirely, but I still prefer this over loosing inputs):
```bash
zstyle ':completion:*' menu select=long
```
 For a full list of config options, go [here](https://github.com/marlonrichert/zsh-autocomplete/blob/main/.zshrc).
- [`zsh-z`](https://github.com/agkozak/zsh-z#for-oh-my-zsh-users): Tool to jump quickly to directories that you have visited frequently.
- [`zsh-direnv`](https://github.com/ptavares/zsh-direnv): Alternative to `dotenv`, which also provides the use of a `.envrc` with utility functions.

To take things further, I recommend checking out [this](https://github.com/unixorn/awesome-zsh-plugins) curated list of plugins. 

 
## Themes + Custom Prompts
We can improve our prompt to contain additional information, such as execution time, performance metrics, and even things like your current Git branch. Set your theme by adding the following to your `.zshrc` (set to `random` until you find something you like):
```bash
ZSH_THEME="mytheme"
```
I like `agnoster` best:
![agnoster](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ind73zeythcovna9u54a.png)
As nice as our prompt looks, we can take things further.
We have 2 options:

### Option A: Powerlevel10k

[Powerlevel10k](https://github.com/romkatv/powerlevel10k/blob/master/README.md#oh-my-zsh) is a highly customizable theme for zsh. It includes some nice features such as an instant prompt while plugins are loading, and a configuration wizard to get you started.

This is what my prompt looks like:

![Powerlevel10k](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i35t7zz1mj4upb5dsfvc.png)

I've modified the configuration a bit to use custom icons for specific directories and cleaned up the git status a bit. See my [`.p10k.zsh`](https://github.com/Tim-W-James/.dotfiles/blob/main/oh-my-zsh/.p10k.zsh)

### Option B: oh-my-posh

[oh-my-posh](https://ohmyposh.dev/docs/linux):
![clean-detailed](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/576uc6ny0fjdzi0bo4x4.png) provides a wide range of themes to choose from, though is missing some features of Powerlevel10k such as instant prompt.
* Follow installation instructions and find a [theme](https://ohmyposh.dev/docs/themes) you like. Some favourites of mine are `clean-detailed`, `blueish`, or `nu4a`.
* Add the path to your chosen theme to `.zshrc`:
```bash
eval "$(oh-my-posh --init --shell zsh --config ~/.mytheme.omp.json)"
```

 
## Other Customization for .zshrc
For reference, see my complete [`.zshrc`](https://github.com/Tim-W-James/.dotfiles/blob/main/oh-my-zsh/.zshrc)
- Disable auto zsh updates: `zstyle ':omz:update' mode disabled`
- Print `...` while waiting for completions to load (doesn't work with zsh-autocomplete): `COMPLETION_WAITING_DOTS="true"`
- Custom aliases. For oh-my-zsh, it is [recommended](https://stephencharlesweiss.com/oh-my-zsh-and-persistent-aliases) to put aliases in a different file: `~/.oh-my-zsh/custom/aliases.zsh`. Some I like to use for networking things:
```bash
alias get-ports="netstat -tulnp | grep LISTEN"
alias get-router="ip route | grep default"
alias get-ip="hostname -I"
```
 You can also create an [alias function with parameters](https://www.thorsten-hans.com/5-types-of-zsh-aliases). For example, to backup a file:
```bash
bak() {
  cp $1{,.bak}
}
```
 We can check that a command exists before setting the alias. For example, we can check docker is installed:
```bash
if [ -x "$(command -v docker)" ]; then
    alias dw="watch \"docker ps --format \\\"table {{.Names}}\t{{.Status}}\\\" -a\""
fi
```
 See all my aliases in my [`aliases.zsh`](https://github.com/Tim-W-James/.dotfiles/blob/main/oh-my-zsh/aliases.zsh). Note that for git aliases, it is best to define them in your [`.gitconfig`](https://github.com/Tim-W-James/.dotfiles/blob/main/.gitconfig).
- Custom keybinds. For example: `bindkey '^H' backward-kill-word` binds <kbd>ctrl-bksp</kbd> to delete by word. Find keycodes [here](https://www.zsh.org/mla/users/2014/msg00266.html).
Those familiar with vim shortcuts can add `bindkey -v` to their `.zshrc`. Be wary of conflicts with other keybinds/plugins!
- Change the default colors for certain directories by adding the following to my `.zshrc` (go [here](https://www.howtogeek.com/307899/how-to-change-the-colors-of-directories-and-files-in-the-ls-command/) for color codes):
```bash
zstyle ':completion:*:default' list-colors \
  "ow=30;43"
```
 Alternatively, you can export a complete color scheme from file:
```bash
eval "$(dircolors ~/.dircolors)";
```
- Set preferred editor for specific files:
```bash
alias -s {cs,ts,html,json,xml,md}=code
```
- Set preferred editior for `ssh`, for example:
```bash
if [[ -n $SSH_CONNECTION ]]; then
  export EDITOR='vim'
else
  export EDITOR='code'
fi
```
- Organize all your environment variables in another file: `export $(cat ~/.my_env)`
- You can automatically install packages, for example you can install nvm:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```
 
## Next Steps
- Install [tmux](https://github.com/tmux/tmux/wiki/Installing) for tabs, panes and more natively in the shell. See [oh-my-tmux](https://github.com/gpakosz/.tmux) and my [`tmux.conf.local`](https://github.com/Tim-W-James/.dotfiles/blob/main/tmux/.tmux.conf.local).
- Install a better terminal. My suggestions:
 - Ubuntu: [Guake](https://guake.readthedocs.io/en/latest/user/installing.html#system-wide-installation)
 - Windows: [Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701#activetab=pivot:overviewtab)
 - MacOS: [iTerm2](https://iterm2.com/)
- Read [this blog](https://dev.to/timwjames/command-line-tools-for-productive-developers-pph) for some general CLI utilies to boost your productivity further.