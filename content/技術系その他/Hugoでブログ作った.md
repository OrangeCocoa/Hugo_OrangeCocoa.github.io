---
title: "Hugoでブログ作った"
date: "2024-08-02"
draft: false
tags: ["技術", "HTML"] 
---

ホームページを公開する方法なんて何も知らないし、WordPressもまともに使えず挫折したことがあるけども、ふと[Hugo](https://gohugo.io/)で作ってみようと思い至る。

公開先をGitHub PagesにするかNetlifyにするかで迷いましたが、いろいろとめんどくさくないGitHubPagesに決定。  
登録するサービスが少なく済むしね。

注意点ですが、[GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages#github-pages%E3%81%AE%E5%88%A9%E7%94%A8%E4%B8%8A%E3%81%AE%E5%88%B6%E9%99%90)はアフィリエイトやエロサイトなどの運用は恐らく認められていないので、やらないように。


**目次**
- [Hugoのインストール](#install)
- [Hugoのセットアップ](#setup)
- [GitHub リポジトリの作成と、GitHub Actionsへのフック](#GitHub)
- [デプロイエラー解決](#error)
- [コメント欄の追加](#comment)
- [リンクを別タブで開く](#link)
- [mermaid記法を使う](#mermaid)


## Hugoのインストール {#install}
---
[ここ](https://gohugo.io/installation/)参考。  
Windowsなので  
`winget install Hugo.Hugo.Extended`  
でインストール。

環境変数に登録までされるので、これが一番楽。

## Hugoのセットアップ {#setup}
---

コマンドプロンプトを開き、  

`hugo new site [Hugoプロジェクト名]`  
{.deco_background_black}
を実行すると、Hugoのプロジェクトが新規作成される。

いくつかフォルダが作られるが、よく使うのは**content**, **layouts**, **static**の三つ。  
また**hugo.toml**はプロジェクトの設定ファイルになるので、これもよく編集する。  

`hugo`  
{.deco_background_black}
コマンドを実行すると、**public**フォルダにビルドされる。  
今回はGitHub Actionsにビルドを任せるので使わなくてもOK。




## GitHub リポジトリの作成と、GitHub Actionsへのフック {#GitHub}
---
GitHubにHugoのデプロイ先と、Hugoプロジェクトの二つのリポジトリを作る。  
このとき、Hugoのデプロイ先はどうもGitHubアカウント名と同じ名前を使わないといけないそう。（ほんとか？）  
アカ名は**OrangeCocoa**なので、ブランチ名は**OrangeCocoa.github.io**。  
WebページのURLは**https://OrangeCocoa.github.io**になる想定。

諸兄ならばリポジトリの作り方くらい知っていようが、一応手順を記載する。

1. WebのGitHubで空のHugoデプロイ先と、Hugoプロジェクトのリモートリポジトリを作成。

2. [Fork](https://git-fork.com/)を使ってる場合、File > Init New RepositoryでHugoプロジェクトのフォルダを選択してローカルリポジトリを作成。（Hugoデプロイ先はいらない）

3. Hugoプロジェクトのローカルリポジトリタブを選択している状態で、右上にあるConsoleボタンを押してコマンドプロンプトを開き、下記コマンドを実行。  
その後プッシュすればリモートブランチ作成コミットとなる。

`git remote add origin https://github.com/[アカウント名]/[リポジトリ名]`
{.deco_background_black}


それが済んだらデプロイアクセス用のSSH鍵を登録していく。

コマンドで

`ssh-keygen -t ed25519 -f blog_ssh`  
{.deco_background_black}
など実行して適当に公開鍵と秘密鍵を作る。


{{< figure src="/images/sshkey.png" width=1200 >}}


次に公開鍵（.pubとついてる方）をHugoのデプロイ先リポジトリに登録。  
メモ帳などのテキストエディタで開いてコピペ。

このとき、Allow write accessにチェックを入れるのを忘れずに。  
書き込み権限がなくてデプロイに失敗する。
{.deco_background_red}


{{< figure src="/images/deploykey.png" width=1200 >}}


続いてHugoプロジェクトのリポジトリに秘密鍵を登録。  
ここで書いたTitleが実行ファイルのシークレット指定に必要になってくるのでコピペ。


{{< figure src="/images/secretkey.png" width=1200 >}}


その後、プロジェクト直下から `.github/workflows/gh-pages.yml` ファイルを作成して以下の実行処理を記載。


``` gh-pages.yml
name: github pages

on:
  push:
    branches:
      - master  # Set a branch name to trigger deployment

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.DEPLOY_SECRET }}
          external_repository: OrangeCocoa/OrangeCocoa.github.io
          publish_branch: master
```


上記が終わったらHugoプロジェクトをコミットする。

添付画像のとおりビルドが通れば成功。  
ちなみに自分は何度か失敗した。（[デプロイエラー解決](#error)に続く）

{{< figure src="/images/completejob.png" width=1200 >}}


### デプロイエラー解決 {#error}
---
**「Waiting for a runner to pick up this job...」** 
{.deco_background_red} 
Actions上で上記のように表示されてジョブがスタートしない場合、OS指定がミスってるのでubuntu-latestを指定。

**Error: fatal: No url found for submodule path 'public' in .gitmodules**  
{.deco_background_red}
のエラーでBuildが止まった場合、publicやresourcesフォルダなどのHugoが生成するフォルダがデプロイ先リポジトリに含まれているので、削除しておく。

**Error: Unable to locate config file or config directory. Perhaps you need to create a new site.**  
{.deco_background_red}
が出た場合、hugo-version指定が古くてhugo.tomlでなくconfig.tomlを探しに行ってるので、'latest'指定しておく。

**ERROR: The key you are authenticating with has been marked as read only.**  
{.deco_background_red}
が出た場合、デプロイキーに書き込み権限がないのでデプロイキーを登録しなおすこと。

**Error: Action failed with "not found deploy key or tokens"**  
{.deco_background_red}
が出た場合、シークレットが登録されていない。

**Error loading key "/home/runner/.ssh/github": error in libcrypto**  
{.deco_background_red}
が出た場合、シークレットの  
-----BEGIN OPENSSH PRIVATE KEY-----  
-----END OPENSSH PRIVATE KEY-----  
を削って登録した疑惑。全部コピペして。


## コメント欄の追加 {#comment}
---
[Disqus](https://disqus.com/)と[utterances](https://utteranc.es/)がある。  
utterancesが楽そうなのでこっちにした。  
ログイン必須ですがコメントがGitHub issue に保存される模様。  
（Disqusもログイン必須）


[utterances app](https://github.com/apps/utterances)をインストール。

その後、single.htmlのフッター直前に以下のコードを挿入するだけでコメントが表示される。

``` single.html
<script src="https://utteranc.es/client.js"
        repo="OrangeCocoa/OrangeCocoa.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```


Disqusも導入してみたところ、コンソール画面でコメントの集計や削除ができるといった感じで、
しっかり運営するならDisqusの方がよさそうだけど、GitHubで完結する楽さには抗えなかった。


## リンクを別タブで開く {#link}
---

どうやらレンダリング（MarkdownをHTMLコードに変換することをそう呼ぶらしい）処理をオーバーライドする必要がある模様。  
~（よく使うんだから最初から使えるようにしとけ）~

`layouts/_default/_markup/render-link.html`    
を新規作成し、下記コードを記載。  
インデントしたら意図しない空白が入るようにレンダリングされたので一行にまとめた。  
誰か原因教えて。

``` render-link.html
<a href="{{ .Destination | safeURL }}" {{ with .Title }} title="{{ . }}"{{ end }} {{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{.Text}}</a>
```


## mermaid記法を使う {#mermaid}
---
`layouts/_default/_markup/render-codeblock-mermaid.html`  
を新規作成し、下記コードを記載。

``` render-codeblock-mermaid.html
<pre class="mermaid">
	{{- .Inner | safeHTML }}
</pre>
{{ .Page.Store.Set "hasMermaid" true }}
```


既存ファイルの末尾あたりに下記コードを挿入。  
footerに入れた。

``` footer.html
{{ if .Store.Get "hasMermaid" }}
<script type="module">
	import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
	mermaid.initialize({ startOnLoad: true });
</script>
{{ end }}
```


これだけでMarkdown挿入が可能になる。

``` markdown.md
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail!
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```


``` mermaid
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail!
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```