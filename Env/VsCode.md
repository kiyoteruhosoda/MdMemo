# VsCode Setup

- [VsCode Setup](#vscode-setup)
  - [Install VsCode](#install-vscode)
  - [Profileを作る](#profileを作る)
    - [Python](#python)
    - [Doc Writer Profile](#doc-writer-profile)
      - [プレビューのカスタマイズ](#プレビューのカスタマイズ)
      - [TOC追加](#toc追加)




前提 [WSL](WSL.md) が構築されていること


## Install VsCode

https://code.visualstudio.com/docs/setup/windows



## Profileを作る

### Python

``` VsCode
>profiles: New Profile
```

Pythonを選択

``` VsCode
>WSL: Connect to WSL
```

拡張機能をWSL向けにインストールする

``` VsCode
>Python: Select Interpreter
```


### Doc Writer Profile

``` VsCode
>profiles: New Profile
```

Doc Writer Profile を選択


#### プレビューのカスタマイズ

以下のファイルを修正

``` VsCode
>Markdown Preview Enhanced: Customize CSS
```


#### TOC追加

``` VsCode
Markdown All in One: Create Table of Contents
```
