VSCode is highly extensible and customizable. Though VSCode is technically a text editor, the extensions in this list will start to turn it into a feature rich IDE.

## Getting Started

1. Install [VSCode](https://code.visualstudio.com/download)
2. Enable settings sync (<kbd>CTRL+,</kbd>). Alternatively, use the [Settings Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) extension to sync your settings via GitHub gists.

## General

- [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot): Your AI pair programmer. I, for one, welcome our robot overlords. Note that the preview requires you to [sign up for a waitlist](https://github.com/features/copilot/signup).
- [GitHub Codespaces](https://marketplace.visualstudio.com/items?itemName=GitHub.codespaces): Cloud-hosted development environments that allows you to access your projects from a browser.
- [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer): Launch a development local Server with live reload feature for static & dynamic pages.
- [Live Share](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare): Real-time collaborative development.
  - [Live Share Whiteboard](https://marketplace.visualstudio.com/items?itemName=lostintangent.vsls-whiteboard): Adds a real-time collaborative whiteboard to Visual Studio Live Share sessions.
- [Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh): Open any folder on a remote machine using SSH.
  - [Remote - SSH: Editing Configuration Files](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh-edit): Edit SSH configuration files.
- [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl): Open any folder in the Windows Subsystem for Linux (WSL).
- [Cacher](https://marketplace.visualstudio.com/items?itemName=Cacher.cacher-vscode): Code snippet manger via GitHub gists (paid extension).
- [vscode-spotify](https://marketplace.visualstudio.com/items?itemName=shyykoserhiy.vscode-spotify): Use Spotify inside vscode (requires a Spotify subscription).
- [Browse Lite](https://marketplace.visualstudio.com/items?itemName=antfu.browse-lite): Embedded browser in VS Code.
- [Choose A License](https://marketplace.visualstudio.com/items?itemName=ultram4rine.vscode-choosealicense): Choose a license for your project in VS Code.
- [EditorConfig](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig): EditorConfig Support for Visual Studio Code.
- [vscode-database](https://marketplace.visualstudio.com/items?itemName=bajdzis.vscode-database): Support mysql, postgres, SSL, socked - SQL database.
- [Snyk](https://marketplace.visualstudio.com/items?itemName=snyk-security.snyk-vulnerability-scanner): Easily find and fix vulnerabilities in both your code and open source dependencies with fast and accurate scans.

## Productivity

- [IntelliCode](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode): AI-assisted IntelliSense.
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense): Autocomplete filenames.
- [Version Lens](https://marketplace.visualstudio.com/items?itemName=pflannery.vscode-versionlens): Shows the latest version for each package using code lens.
- [vscode-faker](https://marketplace.visualstudio.com/items?itemName=deerawan.vscode-faker): Generate fake data for name, address, lorem ipsum, commerce and much more.
- [Bookmarks](https://marketplace.visualstudio.com/items?itemName=alefragnani.Bookmarks): Mark lines and jump to them.
- [Peacock](https://marketplace.visualstudio.com/items?itemName=johnpapa.vscode-peacock): Subtly change the workspace color of your workspace.
- [Output Colorizer](https://marketplace.visualstudio.com/items?itemName=IBM.output-colorizer): Syntax highlighting for log files.
- [Bracket Pair Colorizer 2](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2): ~~A customizable extension for colorizing matching brackets.~~
Note: this is now a built-in feature of VSCode, add the following to your settings.json:

```json
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": true,
```

- [Image preview](https://marketplace.visualstudio.com/items?itemName=kisstkondoros.vscode-gutter-preview): Shows image preview in the gutter and on hover.
- [vscode-pdf](https://marketplace.visualstudio.com/items?itemName=tomoki1207.pdf): Display pdf file in VSCode.

## Code Editing

- [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode): General purpose code formatter.
- [Better Comments](https://marketplace.visualstudio.com/items?itemName=ms-vscode.better-comments): Improve your code commenting by annotating with alert, informational, TODOs, and more. I've also configured it to highligh `fixme` and `note`.
- [Error Lens](https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens): Improve highlighting of errors, warnings and other language diagnostics. Some people might find this too distracting, but I think the trade-off is worthwhile as it helps me quickly identify errors. Also works with lots of other extensions, linters and suggestions. I configured it to set a short delay before showing the error.
- [Regex Previewer](https://marketplace.visualstudio.com/items?itemName=chrmarti.regex): Regex matches previewer.
- [change-case](https://marketplace.visualstudio.com/items?itemName=wmaurer.change-case): Quickly change the case (camelCase, CONSTANT_CASE, snake_case, etc) of the current selection or current word.
- [Multiple cursor case preserve](https://marketplace.visualstudio.com/items?itemName=Cardinal90.multi-cursor-case-preserve): Preserves case when editing with multiple cursors.
- [Rewrap](https://marketplace.visualstudio.com/items?itemName=stkb.rewrap): Hard word wrapping for comments and other text at a given column.
- [Hungry Delete](https://marketplace.visualstudio.com/items?itemName=jasonlhy.hungry-delete): Delete an entire block of whitespace or tab.
- [Tab Out](https://marketplace.visualstudio.com/items?itemName=albert.TabOut): Tab out of quotes, brackets, etc.
- [Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-spellchecker): Spell check your code.
- [DotEnv](https://marketplace.visualstudio.com/items?itemName=mikestead.dotenv): Support for dotenv file syntax.
- [Highlight Bad Chars](https://marketplace.visualstudio.com/items?itemName=wengerk.highlight-bad-chars): Highlight bad characters such as No-break space ( ) and the Greek question mark (;) in your source files.

## File Managment

- [File Utils](https://marketplace.visualstudio.com/items?itemName=sleistner.vscode-fileutils): A convenient way of creating, duplicating, moving, renaming and deleting files and directories.
- [advanced-new-file](https://marketplace.visualstudio.com/items?itemName=patbenatar.advanced-new-file): Create new files with a context menu - specify the file type and location. I've replaced my new file shortcut <kbd>CTRL+N</kbd> to use this extension.
- [Scratchpads](https://marketplace.visualstudio.com/items?itemName=shivanshu-gupta.scratchpads): Create multiple scratchpad files for doodling while you're coding. I've set this to the shortcut <kbd>CTRL+ALT+N</kbd>.

## Git

- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens): Supercharge Git within VS Code. Does a whole lot of Git related things - too many to list here.
- [Git History](https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory): View git log, file history, compare branches or commits
- [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph): View a Git Graph of your repository, and perform Git actions from the graph.
- [GitHub Pull Requests](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github): Allows you to review and manage GitHub pull requests and issues.
- [Checkpoints](https://marketplace.visualstudio.com/items?itemName=micnil.vscode-checkpoints): Checkpoints used in between commits for keeping a local short-term history of work in progress files, like bookmarks in you undo-stack.
- [gitignore](https://marketplace.visualstudio.com/items?itemName=codezombiech.gitignore): Lets you pull .gitignore templates quickly and easily.

## Diagramming

- [Draw.io Integration](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio): Integrates Draw.io into VS Code. Lets you use `.drawio.png` files, which are both directly editable and easily exported. I use the theme `atlas`.
- [PlantUML](https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml): Rich PlantUML support for Visual Studio Code.

## Appearance

There are plenty of themes to choose from and this comes down to personal preference, but this is what I'm currently using:

- [One Dark Pro](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme): Atom‘s iconic One Dark theme for Visual Studio Code.
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme): Material Design Icons for Visual Studio Code
- [Material Product Icons](https://marketplace.visualstudio.com/items?itemName=PKief.material-product-icons): Product Icon Theme with Material Icons for VS Code.

## Specialized Extensions

For pretty much any language out there, you can find extensions for syntax highlighting, linting, formatter, and so on. I'll omit most of those sorts of extensions here, and instead focus on some more interesting or unique extensions.

### JavaScript, etc.

- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint): Linting for JavaScript (I know I said no linters, etc. but this is a must-have).
- [npm](https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script): npm support for VS Code.
- [Add jsdoc comments](https://marketplace.visualstudio.com/items?itemName=stevencl.addDocComments): Adds simple jsdoc comments for the parameters of a selected function signature.
- [Wrap Console Log](https://marketplace.visualstudio.com/items?itemName=midnightsyntax.vscode-wrap-console-log): Wrap to console.log by word or selection.
- [Import Cost](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost): Display import/require package size in the editor.
- [Auto Import](https://marketplace.visualstudio.com/items?itemName=steoates.autoimport): Automatically finds, parses and provides code actions and code completion for all available imports.
- [Glean](https://marketplace.visualstudio.com/items?itemName=wix.glean): The extension provides refactoring tools for your React codebase.
- [TypeScript Error Translator](https://marketplace.visualstudio.com/items?itemName=mattpocock.ts-error-translator): TypeScript errors, translated for humans.

### HTML

- [Auto Close Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag): Automatically closes the tag when you type a space after the tag name.
- [Auto Rename Tag](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag): Renames the tag when you type a space after the tag name.
- [Icon Fonts](https://marketplace.visualstudio.com/items?itemName=idleberg.icon-fonts): Snippets for popular icon fonts.

### CSS

- [Colorize CSS](https://marketplace.visualstudio.com/items?itemName=kamikillerto.vscode-colorize): Help visualize css colors in files.
- [CSS Peak](https://marketplace.visualstudio.com/items?itemName=pranaygp.vscode-css-peek): Quickly view the CSS properties of the current element.

### AWS

- [AWS Toolkit](https://marketplace.visualstudio.com/items?itemName=mads-hartmann.aws-toolkit): A collection of AWS tools for Visual Studio Code.
- [AWS Actions](https://marketplace.visualstudio.com/items?itemName=mads-hartmann.aws-actions): Provides AWS actions in the editor.
- [AWS CLI Configure](https://marketplace.visualstudio.com/items?itemName=mads-hartmann.aws-cli-configure): Quickly access the AWS CLI config & credentials files.

### Other

- [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker): Makes it easy to create, manage, and debug containerized applications.
- [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers): Open any folder or repository inside a Docker container.
- [Kubernetes](https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools): Develop, deploy and debug Kubernetes applications.
- [json](https://marketplace.visualstudio.com/items?itemName=ZainChen.json): Json for Visual Studio Code.
- [XML Tools](https://marketplace.visualstudio.com/items?itemName=DotJoshJohnson.xml): XML Formatting, XQuery, and XPath Tools for Visual Studio Code.
- [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml): YAML Language Support by Red Hat, with built-in Kubernetes syntax support.

## Markdown

For writing blogs like this one. VSCode has pretty good out-of-the-box support for Markdown, but there are a few useful extensions.

- [Markdown All in One](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one): All you need to write Markdown (keyboard shortcuts, table of contents, auto preview and more).
- [markdownlint](https://marketplace.visualstudio.com/items?itemName=DavidAnson.vscode-markdownlint): Markdown linting and style checking for Visual Studio Code. Be sure to fine tune these rules to your liking.

## Next Steps

- Install tooling for your favorite language/tool - syntax highlighting, intellisense linting, formatting, etc.
- Get familiar with the command pallet <kbd>CTRL+SHIFT+P</kbd>. You can search for pretty much anything you want to do, extensions included.
- Learn and configure your keyboard shorcuts. Anything from the command pallet can be configured as a keyboard shortcut, and you can set contextual shortcuts for specific languages, etc.
- VSCode has a built-in snippet manager, but you can download additional snippets for specific languages/tools.
- Configure your editor settings. You can see what I'm current using [here](https://github.com/Tim-W-James/.dotfiles). Some notable configuration:
  - Set a ruler so your lines of code don't get too long:

  ```json
  "editor.rulers": [80],
  "workbench.colorCustomizations": {
    "editorRuler.foreground": "#491f1f"
  },
  ```

  - Make your cursor *smooth*:

  ```json
  "editor.cursorBlinking": "phase",
  "editor.cursorSmoothCaretAnimation": true,
  ```

  - Install a custom font. I use [Caskaydia Cove Nerd Font](https://www.nerdfonts.com/font-downloads). Turn on ligatures `"editor.fontLigatures": true,` for multi-character symbols.
  - Turn on file nesting to group together similar files:

  ```json
  "explorer.fileNesting.enabled": true,
  "explorer.fileNesting.patterns": {
    "*.ts": "${capture}.js",
    "*.js": "${capture}.js.map, ${capture}.min.js, ${capture}.d.ts",
    "*.jsx": "${capture}.js",
    "*.tsx": "${capture}.ts",
    "tsconfig.json": "tsconfig.*.json",
    "package.json": "package-lock.json, yarn.lock",
    "README.md": "LICENSE.txt",
    ".eslintrc.*": ".eslintignore, .prettierrc.*, .eslintrc-auto-import.json"
  },
  "explorer.fileNesting.expand": false
  ```

  - Format and organize imports on save:
  
  ```json
  "editor.formatOnSaveMode": "modifications",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  },
  ```

  - Auto fetch from version control:

  ```json
  "git.autofetch": true,
  ```
  
Feel free to comment with any extensions or tips I have missed and I'll add them to the list!
