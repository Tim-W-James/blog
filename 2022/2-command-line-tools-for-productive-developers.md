This guide is a compilation of various command line tools that will improve your productivity in addition to various quality-of-life features.
_Note_: for any `npm` packages, make sure to install globally with `-g`.

 
## Getting Started
1. Download a [Nerd Font](https://www.nerdfonts.com/font-downloads) and set your favourite terminal to use it. I like `Cascaydia Cove Nerd Font` the best.
2. See [my guide](https://dev.to/timwjames/overhaul-your-terminal-with-zsh-plugins-more-3oag) for setting up a custom prompt and various plugins using oh-my-zsh. This also includes some useful aliases, keybinds and terminals to use for different platforms.

 
## General Productivity
- [`tmux`](https://github.com/tmux/tmux/wiki): Terminal multiplexer that gives you tabs, panes and more natively in the shell. With tmux, you can detach terminal sessions so that they continue running in the background, restore sessions, and even reattach them to a different terminal.
![tmux](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nmrk8iwon7zvht3nw9h0.png)
 - See [this](https://dev.to/andrenbrandao/terminal-setup-with-zsh-tmux-dracula-theme-48lm#tmux-amp-dracula-theme) blog to get started with tmux.
 - See my [`.tmux.conf`](https://github.com/Tim-W-James/.dotfiles/blob/main/tmux/.tmux.conf).
 - Alternatively, you can also use [oh-my-tmux](https://github.com/gpakosz/.tmux) along with a bunch of plugins to extend the capabilities of tmux. See my [`.tmux.conf.local`](https://github.com/Tim-W-James/.dotfiles/blob/main/tmux/.tmux.conf.local) which has my current configuration using this framework. This is what my tmux looks like, with a customized status bar: 
![oh-my-tmux](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yizkmnhrgpgd16jn3kz0.png)
- [`bat`](https://github.com/sharkdp/bat#installation): Better `cat` - supports syntax highlighting for a large number of programming and markup languages.
![bat](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wc1m4zsj98ecniw7rup2.png)
- [`diff-so-fancy`](https://github.com/so-fancy/diff-so-fancy#install): Makes your diffs human readable instead of machine readable. Go [here](##Git) to see usage with Git.
 - Can be used with `diff`, as in this alias:
```bash
diffs() {
  diff -u $1 $2 | diff-so-fancy
}
```
- [`tldr`](https://github.com/tldr-pages/tldr): Better `man` (manual) pages.
- [`thefuck`](https://github.com/nvbn/thefuck#installation): Corrects errors in previous console commands.
- [`how2`](https://github.com/santinic/how2#install): Finds the simplest way to do something in a unix shell using a natural language query.
- [`direnv`](https://github.com/direnv/direnv/blob/master/docs/installation.md): Load and unload environment variables depending on the current directory (for `oh-my-zsh` users, see [this](https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/dotenv) alternative)
- [`glow`](https://github.com/charmbracelet/glow#installation): Terminal based markdown reader.

 
## Search
- [`fzf`](https://github.com/junegunn/fzf#using-homebrew): General-purpose command-line fuzzy finder.
![fzf + bat](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7gzkzvo6wqwut2v82g2s.png)
 - fzf can be used for tab completion, history search and more.
 - If you are using oh-my-zsh, add `fzf` to your plugins for keybinds and more.
 - You can use fzf to search and `bat` for file previews using this alias:
```bash
if [[ -x "$(command -v fzf)" ]] && [[ -x "$(command -v bat)" ]]; then
  alias fp="fzf --preview 'bat --color=always --style=numbers --line-range=:500 {}'"
fi
```
- [`rg`](https://github.com/BurntSushi/ripgrep#installation): Better `grep` - ripgrep is a line-oriented search tool that recursively searches the current directory for a regex pattern.

 
## Directory Navigation & Management
- [`colorls`](https://github.com/athityakumar/colorls): Colorizes the `ls` output with color and icons (requires `gem`).
![colorls](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/orwhamjacjdt7pguxszn.png)
 - Includes many useful flags, such as `--gs` for Git status, or `-t` for a tree view:
![colorls tree](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jcy67tm1nj46aitqvgxw.png)
 - I use an alias to replace `ls` with `colorls`:
```bash
if [ -x "$(command -v colorls)" ]; then
    alias ls="colorls"
    alias la="colorls -al"
fi
```
- [`exa`](https://the.exa.website/): Alternative to `colorls` (use the `--icons` flag to get icons like `colorls`).
- [`tree`](https://www.cyberciti.biz/faq/linux-show-directory-structure-command-line/): Visualize directories in a tree-like format (lightweight alternative to `colorls` with the `-t` flag).
![tree](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1d7kg883xlj4xztl5nc2.png)
- [`broot`](https://github.com/Canop/broot#broot): Like tree, but works better with big directories.
- [`z`](https://github.com/rupa/z): Quickly jump between directories based on history (For `zsh` users, it is easier to install [this](https://github.com/agkozak/zsh-z#for-oh-my-zsh-users) plugin)

 
## Utilities
- [`vtop`](https://www.npmjs.com/package/vtop): A graphical activity monitor for the command line.
![vtop](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4sv4c9jfo0dr4lt598vb.png)
- [`croc`](https://github.com/schollz/croc#install): Simple file transfer via CLI.
- [`secman`](https://github.com/scmn-dev/secman#installation-): CLI password manager.
- [`hyperfine`](https://github.com/sharkdp/hyperfine): A command-line benchmarking tool.
- [`gping`](https://github.com/orf/gping#install-cd): Ping, but with a graph.
- [`procs`](https://github.com/dalance/procs#homebrew): Replacement for ps.
- [`dog`](https://github.com/ogham/dog#installation): Command-line DNS client.
- [`duf`](https://github.com/muesli/duf#installation): A better df alternative.

 
## Git
- [`gh`](https://github.com/cli/cli): GitHub CLI - view pull requests, issues, and other GitHub concepts in the terminal.
- [`gitui`](https://github.com/extrawurst/gitui#installation): Git GUI in your terminal.
![GitGUI](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ndhi3sgeslypk0iqu13m.png)
- [`diff-so-fancy`](https://github.com/so-fancy/diff-so-fancy#install): Makes your diffs human readable instead of machine readable.
![diff-so-fancy](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bt1qudr0t57oyi5hoe7t.png)
 - Add [the following](https://github.com/so-fancy/diff-so-fancy#improved-colors-for-the-highlighted-bits) to your [`.gitconfig`](https://github.com/Tim-W-James/.dotfiles/blob/main/.gitconfig):
```bash
[alias]
	dsf = diff --color
[interactive]
	diffFilter = diff-so-fancy --patch
[color]
	ui = true
[color "diff-highlight"]
	oldNormal = red bold
	oldHighlight = red bold 52
	newNormal = green bold
	newHighlight = green bold 22
[color "diff"]
	meta = 11
	frag = magenta bold
	func = 146 bold
	commit = yellow bold
	old = red bold
	new = green bold
	whitespace = red reverse
```
- [commitizen](https://www.npmjs.com/package/commitizen): Get prompted to fill out any required commit fields at commit time.

 
## Package Mangers

There are a wide range of different package managers for different environments (e.g., `APT` for `Ubuntu`) and languages (e.g., `pip` for `python`), but here are some general package managers that are good to have:

- [Homebrew](https://brew.sh/): MacOS, but also available on Linux.
- [Chocolatey](https://docs.chocolatey.org/en-us/choco/setup#more-install-options): Windows.
- [nix](https://github.com/NixOS/nix#installation): For Unix systems, functional package manager.
- [npm](https://nodejs.org/en/download/package-manager/): Package manager for `node`, but good to have.

 
## Specialized Tools
Depending on what technologies you work with day-to-day, these may or may not be relevant:

- [`jq`](https://stedolan.github.io/jq/download/): JSON processor.
- [`httpie`](https://httpie.io/cli): Command-line HTTP client (better `curl`).
- [`wget`](https://www.gnu.org/software/wget/): Another command-line HTTP client.
- [`ngrok`](https://ngrok.com/download): Secure URL to a localhost server.
- [`k9s`](https://github.com/derailed/k9s#installation): Manage your Kubernetes clusters in style.
  ![k9s](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0jelcf1d0z3stu2bqmt8.png)

 
## Next Steps

- Customize your theme and prompt. See my guide [here](https://dev.to/timwjames/overhaul-your-terminal-with-zsh-plugins-more-3oag).
![Custom prompt](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z0g01xs16dcyhj2ooh3h.png)
- Use oh-my-zsh to install some zsh plugins. I've created a guide for that [here](https://dev.to/timwjames/overhaul-your-terminal-with-zsh-plugins-more-3oag). Some notable plugins include:
 - Autocomplete
 - Autosuggestions
 - Syntax highlighting
 - `.env` auto loading
 - Clipboard CLI utilities
 - `web-search` for using search engines via CLI
- Create some aliases for frequently used commands. See all my aliases in my [`aliases.zsh`](https://github.com/Tim-W-James/.dotfiles/blob/main/oh-my-zsh/aliases.zsh). Note that for git aliases, it is best to define them in your [`.gitconfig`](https://github.com/Tim-W-James/.dotfiles/blob/main/.gitconfig).

Feel free to add a comment suggesting any additional CLI tools and I'll add them to the list!