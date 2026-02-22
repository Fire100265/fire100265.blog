---
layout: post
title: "Moving From Oh My Zsh to Vanilla Zsh"
date: 2025-02-22 14:55:00 +0800
categories: general
---

# Introduction

Welcome back! It’s been a while since I last wrote a post. I’ve been busy with work and personal projects, but I’m excited to share some updates on my journey with Zsh. Recently, I started thinking about configuring my Zsh environment myself, and I decided to give it a try. This is extremely important to me, after all, since I basically live in the terminal. As a result, I wanted to stop using Oh My Zsh because I wasn’t even using most of the features it offered. I wanted to start from scratch and build my own configuration from the ground up. That way, I could have complete control over my Zsh environment and make it exactly how I wanted it—using only the plugins, features, and tools that I actually needed.

I also wasn’t comfortable using Powerlevel10k, as it felt overly complicated and unnecessarily bloated, despite being extremely fast. Because of the instant prompt feature, it sometimes caused a visual blinking glitch when opening a terminal, which I didn’t enjoy. Finally, I wanted to unify my dotfiles on both Arch and macOS so they could use the same codebase, without requiring me to maintain two separate sets of Zsh configuration files. This is where my journey began.

# My Initial Setup

At the start, I had the usual `.zshrc` and `.p10k.zsh` that everyone who uses these frameworks has. It went something like this:

```zsh
# Enable Powerlevel10k instant prompt. Should stay close to the top of ~/.zshrc.
# Initialization code that may require console input (password prompts, [y/n]
# confirmations, etc.) must go above this block; everything else may go below.
if [[ -r "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh" ]]; then
  source "${XDG_CACHE_HOME:-$HOME/.cache}/p10k-instant-prompt-${(%):-%n}.zsh"
fi

# Path to your oh-my-zsh installation.
export ZSH="$HOME/oh-my-zsh"

# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
ZSH_THEME="powerlevel10k/powerlevel10k"

# Set list of themes to pick from when loading at random
# Setting this variable when ZSH_THEME=random will cause zsh to load
# a theme from this variable instead of looking in $ZSH/themes/
# If set to an empty array, this variable will have no effect.
# ZSH_THEME_RANDOM_CANDIDATES=( "robbyrussell" "agnoster" )
```
You get the idea. It was a standard `.zshrc` that anyone using OMZ would recognize. My only personalizations were a bunch of aliases and functions at the bottom of the file. I was never really happy with this setup. Years ago, when I first started my shell journey, OMZ was a godsend because it let me tap into the power of Zsh while keeping everything convenient. These days, though, I don’t really need the hand-holding anymore and wanted to go my own way without relying on a full framework.

The complexity of `.p10k.zsh` also put me off. I didn’t enjoy having a theme with such a complex configuration file, nor the various “hacky” tricks used to appear fast, such as the instant prompt.

# My New Strategy
## Starship
[Starship](https://github.com/starship/starship) is a minimal, blazing-fast prompt for any shell, written in Rust. I thought this would be a great starting point. I had heard of it a while back but never really bothered trying it. If I was going to move away from P10K, this seemed like the best first step. Here is the configuration I am currently using. It looks very similar to P10K, and I’m very happy with it:
```toml
command_timeout = 200
scan_timeout = 50
add_newline = false

format = """
$os\
$directory\
$git_branch\
$git_status\
$fill\
$custom\
$time\
$line_break\
$character
"""

[fill]
symbol = "·"
style = "bright-black"

[os]
disabled = false
style = "fg:white"
format = "[ $symbol ]($style)"

[os.symbols]
Arch = ""
Macos = ""
Linux = ""

[directory]
style = "fg:cyan"
read_only_style = "fg:red"
format = "[  $path ]($style)"
truncation_length = 4
truncate_to_repo = false

[git_branch]
style = "fg:yellow"
symbol = " "
format = "[  $symbol$branch ]($style)"

[git_status]
style = "fg:yellow"
format = "[ $all_status$ahead_behind ]($style)"
staged = "+$count "
modified = "!$count "
untracked = "?$count "
stashed = "*$count "
conflicted = "~$count "
deleted = "✘$count "
renamed = "»$count "
ahead = "⇡$count "
behind = "⇣$count "
diverged = "⇣$behind_count⇡$ahead_count "

[custom.vim_shell]
when = '''test -n "$EDITOR"'''
command = '''
sh -c '
case "$EDITOR" in
  *nvim*) printf " nvim" ;;
  *vim*)  printf " vim" ;;
  *hx*)   printf "󰣪 hx" ;;
  *helix*) printf "󰣪 hx" ;;
  *)      printf " %s" "${EDITOR##*/}" ;;
esac
'
'''
format = "[ $output ](fg:green)"

[time]
disabled = false
time_format = "%I:%M:%S %p"
format = "[  $time ](fg:bright-black)"

[character]
success_symbol = "[❯](fg:green)"
error_symbol = "[❯](fg:red)"
vimcmd_symbol = "[❮](fg:green)"
```
I’m very happy with this configuration, and my prompt looks and feels just like P10K while being much newer and more actively maintained.

## Vanilla Zsh
This was a process I genuinely enjoyed. The entire time I was using OMZ, I assumed that all of those powerful features were only possible because of OMZ. It never occurred to me that Zsh itself already had most of these features built in, and that I simply wasn’t taking advantage of them. This made me appreciate Zsh much more as a shell and for how far it has evolved from its predecessor, Bash. Here is my new `.zshrc`, which retains all the OMZ features I actually used while remaining simple and framework-free:

```zsh
bindkey -e
autoload -Uz add-zsh-hook compinit

# Draw prompt immediately
add-zsh-hook -Uz precmd _instant_prompt_draw
_instant_prompt_draw() {
  add-zsh-hook -d precmd _instant_prompt_draw
  eval "$(starship init zsh)"
}

# Environment variables from ~/.zprofile
source ~/.zprofile

# Ensure cache dir exists
mkdir -p ~/.cache/zsh

# Command-not-found handler
if [[ "$OSTYPE" == darwin* ]]; then
  hb="${HOMEBREW_PREFIX:-/opt/homebrew}"
  [[ -f "$hb/Library/Homebrew/command-not-found/handler.sh" ]] && \
    source "$hb/Library/Homebrew/command-not-found/handler.sh"
  unset hb
else
  [[ -f /usr/share/doc/pkgfile/command-not-found.zsh ]] && source /usr/share/doc/pkgfile/command-not-found.zsh
fi

# Auto Completions
zcompdump="$HOME/.cache/zsh/zcompdump"
mkdir -p "${zcompdump:h}"

# Rebuild compdump if older than 24 hours
if [[ -f "$zcompdump" ]]; then
  if [[ "$OSTYPE" == darwin* ]]; then
    zcompdump_mtime="$(stat -f %m -- "$zcompdump")"   # macOS/BSD stat
  else
    zcompdump_mtime="$(stat -c %Y -- "$zcompdump")"   # Linux/GNU stat
  fi

  if (( $(date +%s) - zcompdump_mtime < 86400 )); then
    compinit -d "$zcompdump"
  else
    compinit -C -d "$zcompdump"
  fi
else
  compinit -C -d "$zcompdump"
fi
unset zcompdump_mtime

# Compile zcompdump
if [[ -f "$zcompdump" && (! -f "$zcompdump.zwc" || "$zcompdump" -nt "$zcompdump.zwc") ]]; then
  zcompile -R -- "$zcompdump"
fi

# Completion tuning
zstyle ':completion:*' menu select
zstyle ':completion:*' matcher-list \
  'm:{a-zA-Z}={A-Za-z}' \
  'r:|[._-]=* r:|=*' \
  'l:|=* r:|=*'
zstyle ':completion:*' completer _complete _match _approximate
zstyle ':completion:*' use-cache on
zstyle ':completion:*' cache-path ~/.cache/zsh/zcompcache

# Recursive globbing (like **)
setopt globstarshort

# Don't clobber files with redirection
setopt noclobber

# Command-less cd: typing a directory path will cd into it
setopt autocd
setopt autopushd
setopt pushdignoredups

# If globs don't match, don't error (fixes: "no matches found: ...")
setopt nonomatch

# Autocorrection
setopt correct
setopt correctall

# Allows comments in interactive shell
setopt interactivecomments

# Vim resumes job named vim
setopt auto_resume

# Notify immediately about background job status
setopt notify

if [[ "$OSTYPE" == darwin* ]]; then
  hb="${HOMEBREW_PREFIX:-/opt/homebrew}"
  source "$hb/share/zsh-autosuggestions/zsh-autosuggestions.zsh"
  source "$hb/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh"
  source "$hb/share/zsh/plugins/zsh-you-should-use/you-should-use.plugin.zsh"
  unset hb
else
  source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
  source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
  source /usr/share/zsh/plugins/zsh-you-should-use/you-should-use.plugin.zsh
fi

# Word-wise movement with Ctrl+Arrow keys
bindkey '^[[1;5D' backward-word   # Ctrl+Left
bindkey '^[[1;5C' forward-word    # Ctrl+Right
bindkey '^H' backward-kill-word
bindkey '\e[3;5~' kill-word

# Take helper (mkdir + cd)
take() {
  mkdir -p -- "$1"
  cd -- "$1"
}

# Extract helper
extract() {
  if [[ -f "$1" ]]; then
    case "$1" in
      *.tar.bz2|*.tbz2) tar xjf "$1" ;;
      *.tar.gz|*.tgz)   tar xzf "$1" ;;
      *.tar)            tar xf  "$1" ;;
      *.bz2)            bunzip2 "$1" ;;
      *.gz)             gunzip  "$1" ;;
      *.rar)            unrar x "$1" ;;
      *.zip)            unzip   "$1" ;;
      *.Z)              uncompress "$1" ;;
      *.7z)             7z x "$1" ;;
      *) echo "don't know how to extract '$1'" ;;
    esac
  else
    echo "'$1' is not a valid file"
  fi
}
alias x=extract

# Sudo toggle widget (Esc Esc)
sudo-command-line() {
  [[ -z $BUFFER ]] && zle up-history
  if [[ $BUFFER == sudo\ * ]]; then
    BUFFER=${BUFFER#sudo }
  else
    BUFFER="sudo $BUFFER"
  fi
  zle end-of-line
}
zle -N sudo-command-line
bindkey '^[^[' sudo-command-line

# Git aliases
alias g='git'
alias ga='git add'
alias gaa='git add --all'
alias gb='git branch'
alias gba='git branch -a'
alias gbd='git branch -d'
alias gc='git commit'
alias gcm='git commit -m'
alias gco='git checkout'
alias gcb='git checkout -b'
alias gd='git diff'
alias gl='git pull'
alias gp='git push'
alias gst='git status'
alias gss='git status -sb'
alias glog='git log --oneline --decorate --graph --all'

# Common aliases
alias cat="bat"
alias wget="wget2"
alias clear="printf '\033[2J\033[3J\033[1;1H'"
alias cd="z"
alias stow="stow --dotfiles"

# Quick directory jumps
alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias .....='cd ../../../..'

# ls aliases
if [[ "$OSTYPE" == darwin* ]]; then
  _LS="gls --color=auto"
else
  _LS="ls --color=auto"
fi
alias ls="$_LS"
alias ll="$_LS -lh"
alias la="$_LS -lha"
alias l="$_LS -lha"
unset _LS

# Cache dircolors output
if [[ ! -f ~/.cache/zsh/dircolors.zsh ]]; then
  if [[ "$OSTYPE" == darwin* ]]; then
    gdircolors -b >| ~/.cache/zsh/dircolors.zsh
  else
    dircolors -b >| ~/.cache/zsh/dircolors.zsh
  fi
fi
source ~/.cache/zsh/dircolors.zsh

# Atuin
eval "$(atuin init zsh)"

# Zoxide
eval "$(zoxide init zsh)"

```
I have commented the code above to make it easy to understand, and I’m very happy with this configuration file.

# Conclusion
Finally, I’ve moved away from OMZ and P10K. While they aren’t really “bloat,” I wanted fine-grained control over what was running in my shell. This transition felt inevitable for me, and I’m glad I finally took the time to do it. I hope you enjoyed reading this and found it helpful. Thanks!
