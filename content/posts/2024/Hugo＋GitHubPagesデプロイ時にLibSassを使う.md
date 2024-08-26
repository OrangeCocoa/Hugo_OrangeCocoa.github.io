---
title: "HUGO＋GitHubPagesデプロイ時にLibSassを使う"
date: "2024-08-25"
draft: false
categories: [ "技術" ]
tags: [ "HUGO", "GitHubActions" ] 
---

GitHubPages構築にGitHubActionsを使ってデプロイしていますが、scssを使って記述したcssをGitHub上でコンパイルするため行なった対応です。  
LibSassは現在公式から非推奨となっていますが、対応した事例が英語のディスカッションでしかなかったため、DartSassに移行する前にひとまず対応してみました。  

---

てきとうに作ったscssファイルを **resources.ToCSS**で変換する処理を普通に書きます。

<br>

**\<head\>タグ内**
``` header.html
{{ $options := (dict "transpiler" "libsass" "targetPath" "css/style.css") }}
{{ with resources.Get "css/tekitou.scss" | toCSS $options | minify }}
<link rel="stylesheet" type="text/css" href="{{ .RelPermalink }}" integrity="{{ .Data.Integrity }}" crossorigin="anonymous">
```

<br>

ワークフロー内でHUGOをセットアップする処理の中に`extended: true`の記述を追加します。  
これはHUGOの拡張バージョンを使用するように設定するものですが、LibSassトランスパイラは拡張版に含まれるものであると[ドキュメント](https://gohugo.io/functions/resources/tocss/)に明記しているため、  
初期値falseのままではLibSassを利用できません。

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

これでGitHubActions上でLibSassが実行できるはずです。