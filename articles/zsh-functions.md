---
title: "Zsh便利関数とか"
emoji: "😎"
type: "tech"
topics: ["zsh"]
published: false
---

```zsh:.zshrc
# 履歴設定
# 履歴ファイルの保存先と最大保存件数を設定
HISTFILE=~/.zsh_history
HISTSIZE=100000
SAVEHIST=100000

# 重複したコマンドを履歴に保存しない
setopt hist_ignore_dups

# 履歴にタイムスタンプを追加
setopt extended_history

# 複数のzshセッション間で履歴を共有
setopt share_history

# スペースで始まるコマンドは履歴に保存しない
setopt hist_ignore_space

## 補完設定
# zsh-completions (homebrew)
fpath=($(brew --prefix)/share/zsh/site-functions $fpath)


# 補完機能有効にする
autoload -U compinit
compinit -u

# 補完候補に色つける
autoload -U colors
colors

# 単語の入力途中でもTab補完を有効化
setopt complete_in_word
# 補完候補をハイライト
zstyle ':completion:*:default' menu select=1
# キャッシュの利用による補完の高速化
zstyle ':completion::complete:*' use-cache true
# 大文字、小文字を区別せず補完する
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'
# 補完リストの表示間隔を狭くする
setopt list_packed

## キーバインド
bindkey -e # emacs 風

# menuselect キーマップを有効化
zmodload zsh/complist

# コマンドの打ち間違いを指摘してくれる
setopt correct
SPROMPT="correct: $RED%R$DEFAULT -> $GREEN%r$DEFAULT ? [Yes/No/Abort/Edit] => "

## tmux
# tmux 自動起動
# tmuxが存在し、現在tmux内でない場合に新規/復帰セッションを作成(SSH接続時を除くっ)
if command -v tmux &> /dev/null && [ -z "$TMUX" ] && [ -z "$SSH_CLIENT" ]; then
    tmux attach || tmux new-session
fi

# zoxide (ちょっと便利な cd)
eval "$(zoxide init zsh)"

# zsh-autosuggestions
source $(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh

# Git関連
autoload -Uz vcs_info
 setopt prompt_subst
 zstyle ':vcs_info:git:*' check-for-changes true
 zstyle ':vcs_info:git:*' stagedstr "%F{magenta}!"
 zstyle ':vcs_info:git:*' unstagedstr "%F{yellow}+"
 zstyle ':vcs_info:*' formats "%F{cyan}%c%u[%b]%f"
 zstyle ':vcs_info:*' actionformats '[%b|%a]'
 precmd () { vcs_info }

# プロンプトカスタマイズ
# PROMPT='%(?..
# %B%F{black}%K{red}Exit Code: %?%k%f%b )
# [%B%F{red}%n@mbp2021%f%b:%F{green}%~%f]%F{cyan}$vcs_info_msg_0_%f
# %F{yellow}%%%f '
if command -v starship >/dev/null 2>&1; then
        eval "$(starship init zsh)"
fi

[[ -e ~/.phpbrew/bashrc ]] && source ~/.phpbrew/bashrc
eval "$(direnv hook zsh)"


# pnpm
export PNPM_HOME="/Users/tenkoma/Library/pnpm"
case ":$PATH:" in
  *":$PNPM_HOME:"*) ;;
  *) export PATH="$PNPM_HOME:$PATH" ;;
esac
# pnpm end


# zsh-syntax-hilighting (end of .zshrc)
source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh


[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
eval "$(mise activate zsh)"
```

```zsh:.zprofile

# https://qiita.com/jiroshin/items/b7be60334191ac36b814
function cd-ghq() {
  local destination_dir=$(echo "$(ghq list --full-path)" | fzf)
  if [ -n "$destination_dir" ]; then
    BUFFER="cd $destination_dir"
    zle accept-line
  fi
  zle clear-screen
}
zle -N cd-ghq
bindkey '^Z' cd-ghq

# プロンプトに表示しているコマンドをクリップボードにコピー
copy-current-command() {
    echo -n "$BUFFER" | pbcopy
}
zle -N copy-current-command
bindkey '^x^y' copy-current-command

# 直前に実行したコマンドをメッセージに書いてコミットする
git-commit-with-last-command() {
    local last_command=$(fc -ln -1)
    git commit -a -e -m "\$ $last_command"
}
alias gac-with-command="git-commit-with-last-command"
```

```
# .tmux.conf

## ステータスライン
# 背景色・文字色
set -g status-style bg=colour235,fg=white
# 左側を空にする
set -g status-left ''
# 右側を空にする
set -g status-right ''
set -g window-status-current-style "bg=colour250,fg=black"

## キーバインド
bind-key -n C-] send-keys "git status -sb" Enter
bind-key -n C-z send-keys "z $(echo \"$(ghq list --full-path)\" | fzf)" Enter

# キープレフィックスをCtrl+Gに変更
unbind C-b
set -g prefix C-g
bind C-g send-prefix

# 設定リロードのキーバインド (新しいプレフィックスで)
bind r source-file ~/.tmux.conf \; display "Tmux config reloaded!"

# マウス
# マウスサポートを有効化
set -g mouse on
# スクロールバッファを有効化
set -g history-limit 30000
# マウスホイールでスクロール
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= '#{pane_in_mode}' 'send-keys -M' 'select-pane -t=; copy-mode -e; send-keys -M'"
bind -n WheelDownPane select-pane -t= \; send-keys -M
```