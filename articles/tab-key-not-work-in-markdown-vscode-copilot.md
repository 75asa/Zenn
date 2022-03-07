---
title: "VSCode で Markdown ファイルの時だけ GitHub Copilot が Tab 補完で動かない"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["VSCode", "Copilot", "Markdown"]
published: false
---

# TL;DR
1. VSCode のキーボードバインドを変更する
1. `sudo vi ~/Library/Application\ Support/Code/User/keybindings.json`
1. 以下を末尾に追加
    ```json
      {
        "key": "tab",
        "command": "markdown.extension.onTabKey",
        "when": "editorTextFocus && !inlineSuggestionVisible && !editorReadonly && !editorTabMovesFocus && !hasOtherSuggestions && !hasSnippetCompletions && !inSnippetMode && !suggestWidgetVisible && editorLangId == 'markdown'"
      },
      {
        "key": "tab",
        "command": "-markdown.extension.onTabKey",
        "when": "editorTextFocus && !editorReadonly && !editorTabMovesFocus && !hasOtherSuggestions && !hasSnippetCompletions && !inSnippetMode && !suggestWidgetVisible && editorLangId == 'markdown'"
      }
    ```
1. VSCode を再起動する
1. VSCode で ⌘ + Shift + P で `Preferences: Open Keyboard Shortcuts (JSON)` を開く
1. 以下のように追加できていることを確認
    ![VSCode-my-keybinding.json](https://i.gyazo.com/546268a053481940b8748093eb35b495.png)
1. Markdown ファイルで GitHub Copilot がタブ補完で入力できることを確認

[![Image from Gyazo](https://i.gyazo.com/ad20af4df46927e574d5e1d4cce5f10f.gif)](https://gyazo.com/ad20af4df46927e574d5e1d4cce5f10f)


# Context


# FYI

https://github.com/yzhang-gh/vscode-markdown/issues/1011

https://github.com/microsoft/vscode/issues/131953

https://zenn.dev/n04h/scraps/e36ff354c2b5bd


