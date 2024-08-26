---
title: "HUGO＋GitHubPages＋GitHubActionsでWebサイトを公開しよう"
date: "2024-08-02"
draft: false
categories: [ "技術" ]
tags: [ "HUGO", "GitHubActions" ] 
---

Webページ作成はHTMLべた書きでめんどくさくて時間もかかるし大して面白くない、レンダリングエンジン作る方が面白そう、とそんなイメージを持っていましたが、  
HUGOとGitHubActionsを使って公開まで自動化ができるというのを見て興味が出る。  
この記事は実際にデプロイして公開までやってみた備忘録になります。  
現時点でWebの知識もお金も全くないです。

<br>

公開先をGitHub PagesにするかNetlifyにするかで迷いましたが、いろいろとめんどくさくないGitHubPagesに決定。  
ブログっぽいものとして最低限必要な要素を解説していきます。

<br>

注意点ですが、[GitHub Pages](https://docs.github.com/ja/pages/getting-started-with-github-pages/about-github-pages#github-pages%E3%81%AE%E5%88%A9%E7%94%A8%E4%B8%8A%E3%81%AE%E5%88%B6%E9%99%90)はアフィリエイトなどの運用は恐らく認められていません。

<br>

GitHubでソースコード公開してるので、気になったら参考にどうぞ。  
https://github.com/OrangeCocoa/Hugo_OrangeCocoa.github.io

<br>

クイックスタートガイドは[ここ](https://juggernautjp.info/getting-started/quick-start/)

<br>

HUGOでよく使うGoテンプレートの基本構文は[ここ](https://juggernautjp.info/templates/introduction/)

<br>

（リンクを英語ドキュメントから日本語ドキュメントに差し替えましたが、一部翻訳ページがないものもあります）

<br>

**目次**
- [HUGOのインストール](#hugo-install)
- [HUGOのセットアップ](#hugo-setup)
- [シングルページとリストページの作成](#create-page)
- [TOPページの作成](#create-top)
- [GitHub リポジトリの作成と、GitHub Actionsへのフック](#hook-github)
- [デプロイエラー解決](#deploy-error)
- [画像の挿入](#insert-image)
- [コメント欄の追加](#add-comment)
- [Markdownの変換挙動をカスタマイズする](custom-markdown)
- [リンクを別タブで開く](#markdown-link)
- [Markdown内でmermaid記法を使う](#markdown-mermaid)
- [Markdown内でCSS定義したスタイルを使う](#markdown-css)
- [Markdown内でGoAT記法を使う](#markdown-goat)
- [Google検索結果に表示させる](#google-search)


---
### HUGOのインストール {#hugo-install}
[ここからDL](https://juggernautjp.info/installation/)  
Windowsなので  

`winget install Hugo.Hugo.Extended`  
でインストール。

環境変数に登録までされるので、これが一番楽。


---
### HUGOのセットアップ {#hugo-setup}
コマンドプロンプトを開き、  

``` s
# カレントディレクトリにHugoのプロジェクトが新規作成される
hugo new site [Hugoプロジェクト名]

cd [Hugoプロジェクト名]

# HTMLのビルド
hugo

# ローカルサーバー起動
hugo server

```

<br>

`hugo new site` でいくつかフォルダが作られるが、よく使うのは**content**, **layouts**, **static**の三つ。  
また**hugo.toml**はプロジェクトの設定ファイルになるので、これもよく編集します。  

<br>

フォルダ構成はこんな感じ。
<div style="width: 300px">

```goat
project
   |
   +---- archetypes
   +---- assets
   +---- content
   +---- data
   +---- layouts
   +---- static
   +---- themes
   .---- hugo.toml
```

</div>

- [archetypes](https://juggernautjp.info/content-management/archetypes/)・・・Markdown記事のテンプレート保存先
- [assets](https://juggernautjp.info/content-management/image-processing/#global-resources)    ・・・グローバルリソース
- [content](https://juggernautjp.info/content-management/page-bundles/)   ・・・Markdownで書いた記事の保存先、HTMLからアクセスする場合に画像や動画
- [data](https://gohugo.io/content-management/data-sources/)      ・・・データファイルの保存先、CSVやJSONなど
- layouts   ・・・HTMLファイルや分割ファイルの保存先
- [static](https://juggernautjp.info/content-management/static-files/)    ・・・CSSや画像ファイルの保存先、Markdownからアクセスする静的リソースはここに入れる
- [themes](https://juggernautjp.info/hugo-modules/theme-components/)    ・・・Hugoのテーマの保存先
- [hugo.toml](https://juggernautjp.info/getting-started/configuration/) ・・・Webサイトの設定ファイル、バージョンによって名前がconfig.tomlだったりする

<br>

プロジェクトのディレクトリで `hugo` コマンドを実行すると、リリース用のURLを設定したHTML群が**public**フォルダにビルドされます。  
今回はGitHub Actionsにビルドを任せるので使わなくてもOK。

<br>

`hugo server` コマンドを実行すると、URL接続先をローカルサーバ用にしたHTML群が**public**フォルダにビルドされ、1313番ポートが開放されます。  
この状態で**http://localhost:1313/**にアクセスすると、レイアウトなどの確認が可能になります。  
また、ローカルサーバーモード起動中はホットリロードが有効なので、MarkdownやHTMLを編集すると、保存した段階でビルドが走り即座に反映が可能。  
HTMLを編集したときに限り稀にビルドされないため（キャッシュあるからいいや的な何かが働いてる？）、なんか思ってたのと違う場合はCtrl＋Cで一度サーバーを落として再度ビルドをかけてみるといいかも。

<br>

#### hugo.toml
作成された**hugo.toml**ファイルにWebページの情報を記載します。
``` hugo.toml
baseURL = 'https://OrangeCocoa.github.io'
languageCode = 'ja-jp'
title = 'ぶろぐ'
```

<br>

その他の設定項目については[こちら](https://juggernautjp.info/getting-started/configuration/)、マークアップについては[こちら](https://juggernautjp.info/getting-started/configuration-markup/)を参照。

<br>

#### header.html, footer.html, style.css
分割定義するため、ヘッダとフッタ用のHTMLファイルを作成します。

<br>

**layouts/partials**というフォルダを作成し、**header.html**と**footer.html**を作成。

``` header.html
<!DOCTYPE html>
    <html lang="ja">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, user-scalable=no">
        <link rel="stylesheet" href="{{ .Site.BaseURL }}/style.css">
        <title>（仮）</title>
    </head>
    <body>
        <div>
            <header>
                <div>（仮）</a></div>
            </header>
        </div>
```

``` footer.html
        <footer>
            <div class="footer">© 2024 （仮） All rights reserved.</div>
        </footer>
    </body>
</html>
```

<br>

**.Site.BaseURL**と書いている箇所がありますが、  
html内のグローバルスコープは[Page](https://juggernautjp.info/variables/page/)なので、  
`{{  }}` で囲い、ドット演算子を用いることで**Page**内の関数呼び出しや変数の参照が可能。  
この場合は**hugo.toml**で指定した**baseURL**を呼び出しています。

<br>

続いて、**static**フォルダ内に**style.css**を作成。

``` style.css
html {
    scroll-behavior: smooth;
}

body {
    background-color : #f5f5f5;
}

header {
    padding-top : 1%;
    padding-bottom : 1%;
}

div {
    display: block;
    unicode-bidi: isolate;
    padding-top : 5px;
    padding-bottom : 5px;
}

.footer {
    text-align: center;

    /*　範囲選択を禁止する*/
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
```

<br>

CSSに関してはHUGO固有の挙動などはないので、適宜必要になった時に調べて追記していきます。

<br>

範囲選択禁止部分のベンダープレフィックスについては[こちら](https://developer.mozilla.org/ja/docs/Glossary/Vendor_Prefix)  
（ブラウザに使われているレンダリングエンジンによって挙動が異なるので、全部指定してるって認識でOK）


---
### シングルページとリストページの作成 {#create-page}
HUGOにおいて、Webページに必要なHTMLファイルは[テンプレート](https://juggernautjp.info/templates/introduction/)によって自動生成されます。

<br>

#### layouts/_default/single.html
まず[シングルページテンプレート](https://juggernautjp.info/templates/single-page-templates/)を作成します。

``` single.html
{{ partial "header.html" . }}
        <div>
            <div>{{ .Date.Format "2006.01.02" }}</div>
            <div>{{ .Title }}</div>
            <div>{{ .Content }}</div>
        </div>
{{ partial "footer.html" . }}   
```

<br>

#### layouts/_default/list.html
次に[リストテンプレート](https://juggernautjp.info/templates/lists/)を作成します。

``` list.html
{{ partial "header.html" . }}
        <div>
            <div>記事一覧</div>
            {{ range (.Paginator 10).Pages }}
            <a>{{ .Date.Format "2006.01.02" }} </a>
            <a href="{{ .RelPermalink }}">{{ .Title }}</a><br>
            {{ end }}
        </div>
{{ partial "footer.html" . }}
```
<br>

contentフォルダ下に新しくフォルダを作ってページ作成すると、フォルダごとのシングルページを管理できるindexページが作成されます。  

<br>

#### content/[フォルダ名]/[記事名].md
最後にシングルページの内容をMarkdownで記述します。  
single.htmlの **.Content**によってレンダリングされます。

<br>

``` 記事.md
---
title: "タイトル"
date: "2024-08-02"
draft: false
categories: [ "" ]
tags: [ "" ] 
---

### 見出し
本文
```

<br>

---で囲まれている部分は、ページのメタデータで[フロントマター](https://juggernautjp.info/content-management/front-matter/)と呼ばれます。
いろいろ設定できるほか、ユーザー定義もできるため、ページの細かいカテゴリ分けや属性として利用できます。

<br>

文字列や数値、bool値、日付といったものはそのまま任意の変数を追加するだけで利用でき、HTMLから **.Param.[名前]**で取得できます。
くわしくは[こちら](https://juggernautjp.info/function/param/)

<br>

文字列配列を設定する場合は[タクソノミー](https://juggernautjp.info/content-management/taxonomies/)設定を行ないます。
タクソノミーで追加された値は、public/tags/~のように一覧化され、
html内で **.Site.Taxonomies.tags**と呼ぶことで、定義された文字列がすべて取得できます。


---
### TOPページの作成 {#create-top}

#### layouts/index.html
``` index.html
{{ partial "header.html" . }}
<div id="top_right">
    <div>新着記事</div>
    <div>
        <!--全記事をループで回す-->
        {{ range first 10 .Site.RegularPages.ByDate.Reverse }}
        <a>{{ .Date.Format "2006.01.02" }} </a>
        <div>
            <a href="{{ .RelPermalink }}" style="font-size:22px;">{{ .Title }}</a>
        </div>
        <br>
        {{ end }}
    </div>
</div>
{{ partial "footer.html" . }}
```

<br>

`{{ partial "header.html" . }}`  
で分割HTMLの呼び出しを行なっています。  
（phpであれば`<?php include "header.html" ?>`と記述するところ）

<br>

**.Site.RegularPages**を使うことで、シングルページのみをフォルダ関係なく全て取得できるので、それを**range**ループでそれぞれのインスタンスを取得します。  
（**.Site.Pages**を使うとリストページも取得する）

<br>

**.Date.Format**でフロントマター指定した"date"の項目をフォーマット指定で出力します。

<br>

**.RelPermalink**でページのURLを取得しhref指定、  
**.Title**でフロントマター指定した"title"を出力します。


---
### GitHub リポジトリの作成と、GitHub Actionsへのフック {#hook-github}
GitHubにHUGOのデプロイ先と、HUGOプロジェクトの二つのリポジトリを作ります。  
このとき、HUGOのデプロイ先はどうもGitHubアカウント名と同じ名前を使わないと、URLが**[アカウント名].github.io/[ブランチ名]**の構成になるらしい。
アカ名は**OrangeCocoa**なので、ブランチ名は**OrangeCocoa.github.io**。  
これでURLは**https://OrangeCocoa.github.io**になる想定。  
気に入らなければサイト名のGitHubアカウントを新規に取得してしまってもいい。

<br>

エンジニアならリポジトリの作り方くらい知ってると思いますが、一応手順を記載します。

<br>

1. WebのGitHubで空のHUGOデプロイ先と、HUGOプロジェクトのリモートリポジトリを作成。

2. [Fork](https://git-fork.com/)を使ってる場合、File > Init New RepositoryでHUGOプロジェクトのフォルダを選択してローカルリポジトリを作成。（Hugoデプロイ先はいらない）

3. HUGOプロジェクトのローカルリポジトリタブを選択している状態で、右上にあるConsoleボタンを押してコマンドプロンプトを開き、下記コマンドを実行。  
その後プッシュすればリモートブランチ作成コミットとなります。

<br>

`git remote add origin https://github.com/[アカウント名]/[リポジトリ名]`

<br>

それが済んだらデプロイのためのSSH鍵を登録していきます。

<br>

コマンドプロンプトなどで

`ssh-keygen -t ed25519 -f blog_ssh`  
を実行して適当に公開鍵と秘密鍵を作成。

<br>

![sshkey](/images/sshkey.png)

<br>

次に公開鍵（.pubとついてる方）をHUGOのデプロイ先リポジトリに登録。  
メモ帳などのテキストエディタで開いてコピペ。

<br>

このとき、Allow write accessにチェックを入れるのを忘れずに。  
書き込み権限がなくてデプロイに失敗します。
{.deco_background_red}

<br>

![deploykey](/images/deploykey.png)

<br>

続いてHUGOプロジェクトのリポジトリに秘密鍵を登録。  
ここで書いたTitleが実行ファイルのシークレット指定に必要になってくるのでコピペ。

<br>

![secretkey](/images/secretkey.png)

<br>

その後、プロジェクト直下から `.github/workflows/gh-pages.yml` ファイルを作成して以下の実行処理を記載。

<br>

``` gh-pages.yml
name: github pages

on:
  push:
    branches:
      - master  # ブランチ名に対してデプロイを実行

  # Actionタブからこのワークフローを手動で実行できる
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: true  # サブモジュールに登録されたテーマもフェッチするか
          fetch-depth: 0    # 全タグ・全ブランチ・全履歴を取得する

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

<br>

上記が終わったらHUGOプロジェクトをコミット。

<br>

添付画像のとおりビルドが通れば成功です。  
ちなみに自分は何度か失敗しました。（[デプロイエラー解決](#error)に続く）

<br>

{{< figure src="/images/completejob.png" width=1200 >}}

<br>

#### デプロイエラー解決 {#deploy-error}
**「Waiting for a runner to pick up this job...」**
{.deco_background_red} 
Actions上で上記のように表示されてジョブがスタートしない場合、OS指定がミスってるのでubuntu-latestを指定。

<br>

**Error: fatal: No url found for submodule path 'public' in .gitmodules**  
{.deco_background_red}
のエラーでBuildが止まった場合、publicやresourcesフォルダなどのHugoが生成するフォルダがデプロイ先リポジトリに含まれているので、削除しておく。

<br>

**Error: Unable to locate config file or config directory. Perhaps you need to create a new site.**  
{.deco_background_red}
が出た場合、hugo-version指定が古くてhugo.tomlでなくconfig.tomlを探しに行ってるので、'latest'指定しておく。

<br>

**ERROR: The key you are authenticating with has been marked as read only.**  
{.deco_background_red}
が出た場合、デプロイキーに書き込み権限がないのでデプロイキーを登録しなおすこと。

<br>

**Error: Action failed with "not found deploy key or tokens"**  
{.deco_background_red}
が出た場合、シークレットが登録されていない。

<br>

**Error loading key "/home/runner/.ssh/github": error in libcrypto**  
{.deco_background_red}
が出た場合、シークレット登録の際、  
-----BEGIN OPENSSH PRIVATE KEY-----  
-----END OPENSSH PRIVATE KEY-----  
を削って登録した疑惑。全部コピペすること。


---
### 画像の挿入 {#insert-image}
普通にWebに上がってる画像をURL指定で参照してもいいですが、手持ちの画像を表示させたいときは**static**フォルダに画像を格納します。  
今回は**images**フォルダを作り、そこに必要な画像を入れていきます。

<br>

HUGOのMarkdownでは  
`![name](/images/~~~.png)`  
と記述します。

<br>

また、元サイズから変更したい場合は
`\{\{< figure src="/images/~~~.png" width=1200 >\}\}`
のように記述すると、figureタグに変換してくれます。


---
### コメント欄の追加 {#add-comment}
[Disqus](https://disqus.com/)と[utterances](https://utteranc.es/)が使えそうです。  
utterancesが楽そうなのでこっちに。  
ログイン必須ですがコメントがGitHub issue に保存される模様。  
（Disqusもログイン必須）

<br>

[utterances app](https://github.com/apps/utterances)をインストール。

<br>

その後、single.htmlのフッター直前に以下のコードを挿入するだけでコメントが表示されます。

<br>

``` single.html
<script src="https://utteranc.es/client.js"
        repo="OrangeCocoa/OrangeCocoa.github.io"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

<br>

Disqusも導入してみたところ、コンソール画面でコメントの集計や削除ができるといった感じで、
しっかり運営するならDisqusの方がよさそうですが、GitHubで完結する楽さには抗えませんでした。

<br>

**追記**：
issueではなくdiscussionsにコメント蓄積する[giscus](https://giscus.app/ja)が使えそうだったので、こっちにしました。

<br>

リポジトリ設定のGeneralからDiscussions機能を有効化。  
※デプロイ先のリポジトリです。

<br>

![discussions](/images/discussions.png)

<br>

次に[ここ](https://github.com/apps/giscus)から対象のリポジトリへgiscusのインストールを行なう。

<br>

その後、[giscus](https://giscus.app/ja)の設定ページから貼り付けるためのコードを生成して、utterancesと同じようにフッターに記述します。  
カテゴリはAnnouncementsを指定した方がいいらしい。  
設定した項目によるが、だいたいこんな感じ。  

<br>

``` footer.html
<script src="https://giscus.app/client.js"
        data-repo="[リポジトリを記述]"
        data-repo-id="[リポジトリIDを記述]"
        data-category="[カテゴリ名を記述]"
        data-category-id="[カテゴリIDを記述]"
        data-mapping="pathname"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="preferred_color_scheme"
        data-lang="ja"
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>
```

---
### Markdownの変換挙動をカスタマイズする {custom-markdown}
**layouts/_default/_markup**フォルダ内に**render-{kind}.html**のファイルを作成することで、レンダリング（MarkdownをHTMLコードに変換することをそう呼ぶらしい）の挙動を変えることができます。  
詳しくは[ここ](https://juggernautjp.info/templates/render-hooks/)


---
### リンクを別タブで開く {#markdown-link}
別タブでリンクを開くためには、どうやらレンダリング処理をオーバーライドする必要がある模様。  
~（よく使うんだから最初から使えるようにしとけ）~

<br>

`layouts/_default/_markup/render-link.html`    
を新規作成し、下記コードを記載。  
インデントしたら意図しない空白が入るようにレンダリングされたので一行にまとめました。  
誰か原因教えて。

<br>

``` render-link.html
<a href="{{ .Destination | safeURL }}" {{ with .Title }} title="{{ . }}"{{ end }} {{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{.Text}}</a>
```


---
### Markdown内でCSS定義したスタイルを使う {#markdown-css}
**hugo.toml**内に下記を追記。

<br>

``` hugo.toml
[markup.goldmark.parser.attribute]
block = true
```

<br>

さらに**style.css**内で適用したいスタイルを定義。
``` style.css
.deco_background_red
{
    color: #fff; /* 白文字 */
    background-color: #ee6666; /* 赤背景 */
    padding: 0.1em 0.2em 0.1em 0.2em;
}
```

<br>

これでスタイル指定したい箇所の下に`{.deco_background_red} `と書けば、CSSに定義されたスタイルを適用することができます。

<br>

**Goldmark**については[こちら](https://juggernautjp.info/getting-started/configuration-markup/#goldmark)、そして[リポジトリ](https://github.com/yuin/goldmark/)  
GoldmarkはMarkdownをレンダリングする際に使われるパーサーで、以前は[blackfriday]()が使われていました。  
blackfridayを使った説明記事を見かけたら回れ右推奨。


---
### Markdown内でmermaid記法を使う {#markdown-mermaid}
[ここ](https://juggernautjp.info/content-management/diagrams/#mermaid-diagrams)参照。
`layouts/_default/_markup/render-codeblock-mermaid.html`  
を新規作成し、下記コードを記載。

<br>

``` render-codeblock-mermaid.html
<pre class="mermaid">
	{{- .Inner | safeHTML }}
</pre>
{{ .Page.Store.Set "hasMermaid" true }}
```

<br>

既存ファイルの末尾あたりに下記コードを挿入。  
footer.htmlに入れました。

<br>

``` footer.html
{{ if .Store.Get "hasMermaid" }}
<script type="module">
	import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.esm.min.mjs';
	mermaid.initialize({ startOnLoad: true });
</script>
{{ end }}
```

<br>

これだけでMarkdown挿入が可能に。

<br>

```` markdown.md
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
````

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


---
### Markdown内でGoAT記法を使う {#markdown-goat}
デフォルトでサポートされているので、特に何も設定しなくても使えます。
詳しくは[ここ](https://juggernautjp.info/content-management/diagrams/)

<br>

そのまま書くと現在のdivサイズに合わせてドデカく表示されるので、
記事内でサイズコントロールしたい場合は[Markdown内でCSS定義したスタイルを使う](#markdown-css)と合わせてスタイル指定してあげると良さそう。

<br>

```` test.md
<div style="width: 800px">

```goat
+-------------------+                           ^                      .---.
|    A Box          |__.--.__    __.-->         |      .-.             |   |
|                   |        '--'               v     | * |<---        |   |
+-------------------+                                  '-'             |   |
                       Round                                       *---(-. |
  .-----------------.  .-------.    .----------.         .-------.     | | |
 |   Mixed Rounded  | |         |  / Diagonals  \        |   |   |     | | |
 | & Square Corners |  '--. .--'  /              \       |---+---|     '-)-'       .--------.
 '--+------------+-'  .--. |     '-------+--------'      |   |   |       |        / Search /
    |            |   |    | '---.        |               '-------'       |       '-+------'
    |<---------->|   |    |      |       v                Interior                 |     ^
    '           <---'      '----'   .-----------.              ---.     .---       v     |
 .------------------.  Diag line    | .-------. +---.              \   /           .     |
 |   if (a > b)     +---.      .--->| |       | |    | Curved line  \ /           / \    |
 |   obj->fcn()     |    \    /     | '-------' |<--'                +           /   \   |
 '------------------'     '--'      '--+--------'      .--. .--.     |  .-.     +Done?+-'
    .---+-----.                        |   ^           |\ | | /|  .--+ |   |     \   /
    |   |     | Join        \|/        |   | Curved    | \| |/ | |    \    |      \ /
    |   |     +---->  o    --o--        '-'  Vertical  '--' '--'  '--  '--'        +  .---.
 <--+---+-----'       |     /|\                                                    |  | 3 |
                      v                             not:line    'quotes'        .-'   '---'
  .-.             .---+--------.            /            A || B   *bold*       |        ^
 |   |           |   Not a dot  |      <---+---<--    A dash--is not a line    v        |
  '-'             '---------+--'          /           Nor/is this.            ---
```

</div>
````

<div style="width: 800px">

```goat
+-------------------+                           ^                      .---.
|    A Box          |__.--.__    __.-->         |      .-.             |   |
|                   |        '--'               v     | * |<---        |   |
+-------------------+                                  '-'             |   |
                       Round                                       *---(-. |
  .-----------------.  .-------.    .----------.         .-------.     | | |
 |   Mixed Rounded  | |         |  / Diagonals  \        |   |   |     | | |
 | & Square Corners |  '--. .--'  /              \       |---+---|     '-)-'       .--------.
 '--+------------+-'  .--. |     '-------+--------'      |   |   |       |        / Search /
    |            |   |    | '---.        |               '-------'       |       '-+------'
    |<---------->|   |    |      |       v                Interior                 |     ^
    '           <---'      '----'   .-----------.              ---.     .---       v     |
 .------------------.  Diag line    | .-------. +---.              \   /           .     |
 |   if (a > b)     +---.      .--->| |       | |    | Curved line  \ /           / \    |
 |   obj->fcn()     |    \    /     | '-------' |<--'                +           /   \   |
 '------------------'     '--'      '--+--------'      .--. .--.     |  .-.     +Done?+-'
    .---+-----.                        |   ^           |\ | | /|  .--+ |   |     \   /
    |   |     | Join        \|/        |   | Curved    | \| |/ | |    \    |      \ /
    |   |     +---->  o    --o--        '-'  Vertical  '--' '--'  '--  '--'        +  .---.
 <--+---+-----'       |     /|\                                                    |  | 3 |
                      v                             not:line    'quotes'        .-'   '---'
  .-.             .---+--------.            /            A || B   *bold*       |        ^
 |   |           |   Not a dot  |      <---+---<--    A dash--is not a line    v        |
  '-'             '---------+--'          /           Nor/is this.            ---
```

</div>

ちなみにこれをHTMLにレンダリングしたものをみると、とんでもないコードができあがっている。


---
### Google検索結果に表示させる {#google-search}
Google検索に引っ掛けるには、Webページのサイトマップを作成し、[Google Search Console](https://search.google.com/search-console/about?hl=ja)で登録する必要があります。
サイトマップはHUGOによってデフォルトで作成されるようになっているので、特に設定することはありません。  
カスタマイズする場合は[こちら](https://juggernautjp.info/templates/sitemap-template/)を参照。

<br>

まずはGoogle Search Consoleの登録。  
URLプレフィックスの方に登録するサイトのURLを入力し、続行します。

<br>

![sitemap01](/images/sitemap01.png)

<br>

Google Analyticsに登録していなかったり、DNSレコードが関連付けられていない場合は、サイトの所有権が確認できていない状態なので、確認をとる必要があります。  
ファイルをダウンロードし、デプロイ先のリポジトリ直下にhtmlファイルを配置したあと、確認ボタンを押してください。

<br>

![sitemap02](/images/sitemap02.png)

<br>

そのままだとHTMLがファイルが消えた場合に関連付けができなくなってしまうので、サイト内にメタタグを埋め込みます。  
設定からHTMLタグを選択し、表示されたコードをコピー。  
\<head\>タグ内にペーストし、サイトに適用されたら確認ボタンを押します。

<br>

![sitemap03](/images/sitemap03.png)

<br>

**https://[ユーザー名].github.io/[リポジトリ名]/sitemap.xml**のURLでサイトマップが作成されていると思うので、  
Google Search Consoleのサイトマップから、サイトマップのパスを入力して送信します。

<br>

![sitemap04](/images/sitemap04.png)

<br>

これで2, 3日待てばGoogle検索に引っかかるようになります。