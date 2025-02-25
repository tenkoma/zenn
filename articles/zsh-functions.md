---
title: "Zsh便利関数とか"
emoji: "😎"
type: "tech"
topics: ["zsh"]
published: false
---

```zsh:.zshrc

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