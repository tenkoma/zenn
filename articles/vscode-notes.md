---
title: "VSCode メモ"
emoji: "🌊"
type: "tech"
topics: ["vscode"]
published: false
---

# VSCode 多用するショートカット

* Explorer フォーカス: `Cmd+Shift+E`
* Explorer ファイルを開く: `Cmd+↓`
* Explorer ファイル新規作成: (`Cmd+Shift+N` ショートカットキー割り当てられてないので設定)
* タブ移動 `Cmd+Option+←`, `Cmd+Option+→`
* 最近開いたファイル兼ファイル名検索: `Cmd+P`
* シンボル検索: `Cmd+Shift+O`
* VCS コミット, ログ: `Ctrl+Shift+G`

# `settings.json`

```json
    "[rust]": {
        "editor.defaultFormatter": "rust-lang.rust-analyzer",
        "editor.formatOnSave": true,
    }
```