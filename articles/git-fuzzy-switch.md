---
title: "gitとfzfを使ってブランチを切り替える"
emoji: "📘"
type: "tech"
topics: ["git", "fzf"]
published: false
---

https://zenn.dev/yamo/articles/5c90852c9c64ab

こちらが良かった。いろいろ編集して、新規ブランチもそのまま作成できるようにしてみた。

```zsh:.zshrc
function git-fuzzy-switch() {
    # 現在のブランチ名を取得
    local current_branch=$(git rev-parse --abbrev-ref HEAD)

    # ブランチ一覧(現在のブランチには "* "がついている)
    local branches=$(git branch --all)

    # (create from "current-branch")オプションを追加
    local selected=$(echo -e "${branches}\n  (create from \"${current_branch}\")" | fzf --height 80% --layout=reverse \
        --preview "if [[ {} != *\"create from\"* ]]; then git show-graph --color=always \$(echo {} | sed -E \"s/^[* ]?([^*]+).*/\1/\"); fi" \
        --preview-window=right:60% \
        --ansi)

    # キャンセルされた場合は終了
    [ -z "$selected" ] && return 1

    if [[ "$selected" == "  (create from \"${current_branch}\")" ]]; then
        # 新規ブランチ名の入力
        printf "Input new branch name: "
        read new_branch

        # 入力がない場合は終了
        [ -z "$new_branch" ] && return 1

        # 新規ブランチを作成してチェックアウト
        git switch -c "$new_branch"
    else
        # 選択したブランチから印と追加情報を削除して切り替え
        local branch_name=$(echo "$selected" | sed -E 's/^[* ]//' | sed -E 's/^[  ]//' | sed -E 's|remotes/origin/||' )
        git switch "$branch_name"
    fi
}
```

`.gitconfig` の `show-graph` の内容は同じ。

```ini:.gitconfig
[alias]
        show-graph = log --graph --decorate --abbrev-commit --format=format:'%C(blue)%h%C(reset) - %C(green)(%ar)%C(reset)%C(yellow)%d%C(reset)\n  %C(white)%s%C(reset) %C(dim white)- %an%C(reset)'
```

ユーザー入力を受け付る機能を追加しつつキーバインドで呼び出せる関数にする方法がわからなかったため、ZLE対応をやめた。`gfs` にエイリアスを作った。以下のような出力になる。

![](/images/git-fuzzy-switch/git-fuzzy-switch-01.png)

# GitHub Pull Request情報を見てからチェックアウトする

`git-fuzzy-switch` とにた感じでPull Request一覧からチェックアウトしたい。公式ブログで `fzf` と組み合わせる方法が紹介されていた。

https://github.blog/engineering/engineering-principles/scripting-with-github-cli/

Pull Requestの概要をプレビューする機能を追加して以下のようになった。

```zsh:.zshrc
function github-fuzzy-switch() {
    local pr_number=$(gh pr list | fzf --height 80% --layout=reverse --preview "echo {} | awk '{print \$1}' | xargs gh pr view" --ansi | awk '{print $1}')
    if [ -n "$pr_number" ]; then
        gh pr checkout "$pr_number"
    fi
}
```

![](/images/git-fuzzy-switch/git-fuzzy-switch-02.png)

エイリアスは以下の通り。

```zsh:.zshrc
alias gfs="git-fuzzy-switch"
alias ghfs="github-fuzzy-switch"
```