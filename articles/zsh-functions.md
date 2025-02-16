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