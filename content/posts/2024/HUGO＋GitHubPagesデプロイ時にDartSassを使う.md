---
title: "HUGO＋GitHubPagesデプロイ時にDartSassを使う"
date: "2024-08-26"
draft: false
categories: [ "技術" ]
tags: [ "HUGO", "GitHubActions" ] 
---

ワークフロー内チェックアウト前に以下の処理を記述することで、実行環境でDartSassのコンパイルが実行できるようになります。

```
- name: Install Dart Sass
  run: sudo snap install dart-sass
```

<br>

最終的には以下のようになります。

<br>

**config.yml**
``` config.yml
name: github pages

on:
  push:
    branches:
      - master

  # Actionタブからこのワークフローを手動で実行できる
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Install Dart Sass
        run: sudo snap install dart-sass

      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true  # 拡張機能にアクセスするか

      - name: Build
        run: hugo --minify

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.DEPLOY_SECRET }}
          external_repository: OrangeCocoa/OrangeCocoa.github.io
          publish_branch: master
```

<br>

#### ローカル環境へのScoopのインストールとDartSassのインストール
Macではhomebrewでインストールできますが、今回はWindowsでインストールします。

<br>

Windows PowerShellを開き、以下の2つのコマンドを実行します。  
**Set-ExecutionPolicy**では同意を求めらえれるのでYを入力。

```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

<br>

続けて、PowerShellで以下のコマンドを実行します。  
こっちはコマンドプロンプトでも実行できますが、実行後どっちにしろ環境変数の適用のために終了させないといけないのでそのまま実行。

```
scoop install main/sass
```

<br>

これで`hugo server`コマンド実行時、**transpiler**に**dartsass**を指定してToCSSを使っていた場合にDartSassによるコンパイルが走ります。