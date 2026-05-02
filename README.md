# 2026 Fresh MacBook Setup

## Bootstrap From Terminal

### Homebrew

Install Homebrew first:

```zsh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow any post-install instructions Homebrew prints. On Apple Silicon Macs, that usually includes adding Homebrew to your shell environment:

```zsh
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Verify:

```zsh
brew --version
```

### Brew Formulae

Install terminal tools directly:

```zsh
brew install git gh ripgrep fd jq tree uv fnm railway
```

| Tool | Description | Official Link |
|---|---|---|
| Git | Version control system | [git-scm.com](https://git-scm.com/) |
| GitHub CLI | GitHub command-line tool | [cli.github.com](https://cli.github.com/) |
| ripgrep | Fast text search for codebases | [github.com/BurntSushi/ripgrep](https://github.com/BurntSushi/ripgrep) |
| fd | Fast file and directory search | [github.com/sharkdp/fd](https://github.com/sharkdp/fd) |
| jq | JSON processor for the command line | [jqlang.org](https://jqlang.org/) |
| tree | Directory tree viewer | [oldmanprogrammer.net/source.php?dir=projects/tree](https://oldmanprogrammer.net/source.php?dir=projects/tree) |
| uv | Python package and version manager | [docs.astral.sh/uv](https://docs.astral.sh/uv/) |
| fnm | Fast Node.js version manager | [github.com/Schniz/fnm](https://github.com/Schniz/fnm) |
| Railway CLI | Railway deployment platform command-line tool | [docs.railway.com/guides/cli](https://docs.railway.com/guides/cli) |

### Brew Casks

Install Mac apps and app-like tools directly:

```zsh
brew install --cask google-chrome ghostty cursor rectangle maccy codex codex-app
```

| App | Description | Official Link |
|---|---|---|
| Google Chrome | Web browser | [google.com/chrome](https://www.google.com/chrome/) |
| Ghostty | GPU-accelerated terminal emulator | [ghostty.org](https://ghostty.org/) |
| Cursor | AI code editor | [cursor.com](https://www.cursor.com/) |
| Rectangle | Window manager | [rectangleapp.com](https://rectangleapp.com/) |
| Maccy | Clipboard manager | [maccy.app](https://maccy.app/) |
| Codex CLI | OpenAI coding agent in the terminal | [github.com/openai/codex](https://github.com/openai/codex) |
| Codex App | OpenAI Codex desktop app | [openai.com/codex](https://openai.com/codex/) |

### Browser Extensions

Install from the browser:

| Extension | Description | Official Link |
|---|---|---|
| 1Password Chrome Extension | Password manager browser extension | [Chrome Web Store](https://chromewebstore.google.com/detail/1password-%E2%80%93-password-mana/aeblfdkhhhdcdjpifhhbdiojplfjncoa) |

### Brewfile

After cloning this repo, use the checked-in `Brewfile` instead of running the individual `brew install` commands above:

```zsh
brew bundle --file=./Brewfile
```

After changing installed formulae or casks, refresh the checked-in snapshot:

```zsh
brew bundle dump --file=./Brewfile --force
```

## Shell Setup

### `.zshrc`

```zsh
# =============================================================================
# Prompt
# =============================================================================
PROMPT='%~ %# '

# =============================================================================
# History
# =============================================================================
setopt SHARE_HISTORY       # share history across all terminal sessions
setopt INC_APPEND_HISTORY  # write to history immediately, not on tab close

# Enable substring search
autoload -U up-line-or-beginning-search
autoload -U down-line-or-beginning-search

zle -N up-line-or-beginning-search
zle -N down-line-or-beginning-search

bindkey '^[[A' up-line-or-beginning-search   # Up arrow
bindkey '^[[B' down-line-or-beginning-search # Down arrow

# =============================================================================
# Aliases
# =============================================================================
alias ll='ls -lah'
alias c='cd ~/Code'

# =============================================================================
# Node
# =============================================================================
eval "$(fnm env --shell zsh --use-on-cd)"
```

Reload the shell after updating `.zshrc`:

```zsh
exec zsh
```

## Python Setup

### uv

Configure uv-managed Python:

```zsh
uv --version
uv python install
uv python install --default
uv python update-shell
```

Restart the shell after `uv python update-shell`.

## Node Setup

### fnm

Configure fnm and install the latest LTS Node.js:

```zsh
fnm install --lts
fnm default lts-latest
node --version
npm --version
```

If you did not add fnm to `.zshrc` above, add it now:

```zsh
# =============================================================================
# Node
# =============================================================================
eval "$(fnm env --shell zsh --use-on-cd)"
```

## Code Workspace

### `~/Code`

Create a single home for cloned repositories:

```zsh
mkdir -p ~/Code
cd ~/Code
```

## GitHub SSH Setup

Reference: [GitHub SSH key setup docs](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

If you prefer GitHub CLI for authentication, run this before adding SSH keys manually:

```zsh
gh auth login
gh config set git_protocol ssh --host github.com
gh auth status
```

### 1. Check for Existing SSH Keys

```zsh
ls -la ~/.ssh
```

Look for existing public keys such as:

- `id_ed25519.pub`
- `id_rsa.pub`

### 2. Generate a New SSH Key

Replace the email with the email address on your GitHub account:

```zsh
ssh-keygen -t ed25519 -C "your_email@example.com"
```

When prompted:

- Save to the default path: `~/.ssh/id_ed25519`
- Use a passphrase if you want extra protection

Stop before accepting the default path if `~/.ssh/id_ed25519` already exists, unless you intentionally want to replace that key.

### 3. Configure SSH for GitHub

Create or edit `~/.ssh/config`:

```zsh
touch ~/.ssh/config
```

Add:

```sshconfig
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

### 4. Add the Key to the SSH Agent

```zsh
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

### 5. Add the Public Key to GitHub

Add the public key with GitHub CLI:

```zsh
gh ssh-key add ~/.ssh/id_ed25519.pub --title "MacBook $(date +%Y-%m-%d)"
```

Or copy the public key:

```zsh
pbcopy < ~/.ssh/id_ed25519.pub
```

Then add it in GitHub:

`GitHub -> Settings -> SSH and GPG keys -> New SSH key`

Reference: [Add a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

### 6. Test the Connection

```zsh
ssh -T git@github.com
```

Expected success message:

```text
Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

## Git Setup

### Cursor CLI for Git

Install the Cursor shell command:

```text
Cursor -> Command Palette -> Shell Command: Install 'cursor' command in PATH
```

Verify:

```zsh
command -v cursor
cursor --version
```

### Global Git Config

Replace the name and email with your GitHub identity:

```zsh
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor "cursor --wait"
```

### Git Aliases

```zsh
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --decorate --all"
```

| Alias | Expands To | Use |
|---|---|---|
| `git co` | `git checkout` | Switch branches or restore files |
| `git br` | `git branch` | List, create, or delete branches |
| `git st` | `git status` | Show working tree status |
| `git lg` | `git log --oneline --graph --decorate --all` | Show compact branch history |

## GitHub CLI Setup

If you skipped GitHub CLI authentication during SSH setup, authenticate now:

```zsh
gh auth login
gh config set git_protocol ssh --host github.com
gh auth status
```

## Railway CLI Setup

Authenticate Railway CLI:

```zsh
railway login
railway whoami
```

## System Settings

### Security and Privacy

Set:

- [ ] Turn on FileVault: `System Settings -> Privacy & Security -> FileVault`
- [ ] Turn on Firewall: `System Settings -> Network -> Firewall`
- [ ] Require password immediately after screen saver begins or display is turned off: `System Settings -> Lock Screen`
- [ ] Sign in to 1Password and enable browser integration
- [ ] Optional: enable Touch ID for `sudo`

### Finder Defaults

Show all file extensions and hidden files:

```zsh
defaults write NSGlobalDomain AppleShowAllExtensions -bool true
defaults write com.apple.finder AppleShowAllFiles -bool true
killall Finder
```

### Screenshots Folder

Set screenshots to save to `~/Screenshots`:

```zsh
mkdir -p ~/Screenshots
defaults write com.apple.screencapture location ~/Screenshots
killall SystemUIServer
```

### Rectangle Shortcuts

Set:

- [ ] Left Half -> Option + Command + [
- [ ] Right Half -> Option + Command + ]
- [ ] Maximize -> Option + Command + Up Arrow

### Keyboard: Modifier Keys

Path:

`System Settings -> Keyboard -> Keyboard Shortcuts -> Modifier Keys`

Set:

- [ ] Caps Lock -> Escape

### Keyboard: Lock Screen Shortcut

Path:

`System Settings -> Keyboard -> Keyboard Shortcuts -> App Shortcuts`

Add:

- Application: `All Applications`
- Menu Title: `Lock Screen`
- Keyboard Shortcut: `Command + Option + L`

Set:

- [ ] Lock Screen -> Command + Option + L

### Keyboard: Mission Control Shortcuts

Path:

`System Settings -> Keyboard -> Keyboard Shortcuts -> Mission Control`

Enable and set:

- [ ] Move left a space -> Option + [
- [ ] Move right a space -> Option + ]

## Final Verification

Run a final sanity check. After each command, compare your output with the expected result below it.

```zsh
brew doctor
```

Expected result: Homebrew says `Your system is ready to brew.`

```zsh
git --version
```

Expected result: Git prints a version, such as `git version 2.54.0`.

```zsh
gh auth status
```

Expected result: GitHub CLI shows you are logged in to `github.com`, and the active account token is valid.

```zsh
ssh -T git@github.com
```

Expected result: GitHub greets your username and says you successfully authenticated. The message also says GitHub does not provide shell access; that part is normal.

```zsh
uv python list
```

Expected result: uv lists at least one installed Python under `~/.local/share/uv/python`. It may also show other Python versions as available downloads.

```zsh
node --version
npm --version
```

Expected result: both commands print versions. If `node` works but `npm` does not, reload the shell with `exec zsh` and confirm fnm is initialized in `.zshrc`.

```zsh
codex --version
```

Expected result: Codex prints a CLI version, such as `codex-cli 0.128.0`.

```zsh
railway --version
```

Expected result: Railway prints a CLI version, such as `railway 4.44.0`.
