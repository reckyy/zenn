---
title: "Jujutsu (jj)を1ヶ月使ってみた：便利だった点とハマった点"
emoji: "🥋"
type: "tech"
topics: ["jujutsu", "jj", "git", "vcs", "技術記事を書く技術"]
published: false
---

## はじめに

reckyです。

最近、Gitの代わりとなるバージョン管理システムであるJujutsu(jj)にふれる機会があり、僕自身1か月ほど使ってみました。
使いはじめて間もないですが、使いやすさに惚れ込んで使い続けているので、便利な点と躓いた点に分けて書きます。

きっかけは、以前『りあクト！』シリーズを購入していた縁でboothから届いた新刊メールでした。同じ著者から『じゅじゅちゅ！ jj newで始めるJujutsu × AIワークフロー』（以下『じゅじゅちゅ！』）が出るという案内を見て、新しいもの好きのため「Gitより使いやすそうだから試してみよ」くらいの軽い気持ちで購入。今ではすっかり普段使いしています。

:::message
本記事は**Jujutsuがまだ初耳〜なんとなく知っている**くらいの方を想定して書いています。
Jujutsuを網羅的に解説する記事ではありませんが、記事を読み進めるのに必要な前提知識は本文中で都度補足しているので、未使用の方もそのまま読めるはずです。
:::

### 動作環境

- macOS 15.7.7（M1）
- Jujutsu (jj) 0.41.0
- jjui 0.10.3
- gh CLI 2.92.0

本記事にもし間違いがあれば、優しく教えてください。🙂‍↕️

## Jujutsuの基本: Gitと何が違うのか

ここから先の話を読むうえで前提となる、Jujutsuの基本を3つだけ紹介します。
Gitとの違いを意識した最小限の説明なので、ここを押さえれば以降の内容は多分頭に入るはずです。

### commitの代わりにchange、branchの代わりにbookmark

Jujutsuには、Gitとは別の名前で似たような概念がいくつかあります。対応表にするとこんな感じです。

| Git | Jujutsu | 補足 |
|---|---|---|
| commit | change | Jujutsuの作業の最小単位 |
| branch | bookmark | 特定のchangeを指す名前付きポインタ |
| HEAD | `@` | 今いるchangeを指す記号 |
| `git add` | （不要） | 保存だけで自動反映 |
| `git commit --amend` | （不要） | `@`への編集は自動で反映される |
| `git stash` | （不要） | 別のchangeに飛ぶだけ |

このうち、特に押さえておきたい違いが2つあります。

**changeは何度でも書き換えられる**
Gitでは一度commitすると書き換えるには`--amend`や`rebase -i`が必要でした。Jujutsuのchangeは何度でも気軽に書き換えられるもので、`@`の中身を編集して保存すれば自動で反映されます。これが履歴を後から整える操作の手軽さにつながります。

**bookmarkは自動で動かない**
Gitのbranchは現在のbranchに新しいcommitを積むと、branchも一緒に先に進みます。Jujutsuのbookmarkはそういった自動追従をしません。bookmarkは「特定のchangeに付けておく目印（ラベル）」であって、自分で`jj bookmark move`のようなコマンドで動かさない限り元の位置のままです。

最初は不便に感じるかもしれませんが、いま作業中の場所に必ずしも名前を付けなくていいというメリットにつながります（後の節で触れます）。

### ステージングエリアがなく、保存即記録

Gitでは、ファイルの変更が履歴に記録されるまでに3つの状態を経由します。

```
[Working Tree] --git add--> [Staging Area] --git commit--> [Repository]
```

Jujutsuではこうです。

```
[Working Copy] -------(自動)-------> [Repository]
```

図の通り、ステージングエリア（index）に相当するものがそもそもありません。
代わりに、`jj`コマンドを実行したタイミングで、その時点のworking copyの状態が自動的にスナップショットされ、履歴に記録されます。決してリアルタイム監視で勝手にcommitされるわけではなく、何らかの`jj`コマンドを叩いた瞬間に取り込まれる、という仕組みです。

そもそも何らかのアクションを起こしたいときは、必ず`jj`コマンドを叩くのでいつ保存されるかということは気にしなくても大丈夫です。
僕のような神経質、心配性な人間には認知負荷がひとつ減るだけでも日頃の作業がずっと楽に感じます。

```bash
# Git
$ git add app/models/post.rb
$ git commit -m "feat: Postに公開状態を追加"

# Jujutsu
# （ファイルを保存するだけで自動的にchangeに反映済み）
$ jj describe -m "feat: Postに公開状態を追加"
```

## 環境構築とツール紹介

具体的なインストール手順は公式や既存記事に記載されてるので、本記事では下記に絞ってのみ簡単に紹介します。

- 何を入れたか
- なぜそれを使うのか

### 入れたもの

| ツール | 役割 | 入れ方 |
|---|---|---|
| Jujutsu | Jujutsu本体 | [公式: Installation and setup](https://docs.jj-vcs.dev/latest/install-and-setup/) |
| jjui | Jujutsuの履歴表示や操作に便利なTUI | [GitHub: idursun/jjui](https://github.com/idursun/jjui) |
| Ghostty | シンプルで軽量なターミナル | [公式](https://ghostty.org/) |
| gh CLI | GitHubのPR操作などに使う（Jujutsuに必須ではない） | [公式](https://cli.github.com/) |

どのツールもインストールから初期設定まで簡単です。
jjuiもGhosttyも『じゅじゅちゅ！』内でおすすめされていたのをそのまま採用しただけなので、ほかにも同機能のツールはあります。それぞれ好みのツールを使用してください。

### jjuiとは

[jjui](https://github.com/idursun/jjui) は、Jujutsuの履歴をターミナル上でグラフィカルに表示できるTUIツールです。
僕がこのツールを愛用している理由は、コマンドを打たなくて（覚えなくて）良いからです！！
見た目も見やすくて、コマンド操作が苦手な自分でも快適に使えてます。どんなシーンで使用してるかは、後述します。

### Ghostty(ターミナル)を使うと便利

『じゅじゅちゅ！』では以下のように述べられています。

> jjuiは常駐させてこそ真価を発揮するので、画面分割ができるターミナルがあるとさらに快適です。

僕は今の所常駐させてはいますが、そこまで頻繁に使用しているわけではないのであまり恩恵を感じられていません。。いつか感じたら追記します！
ただついでにインストールしたGhosttyは見た目、設定ともにシンプルで好みなので使い続けています。背景の透明度を下げれるのが地味にいいです。何となくですが。

## 良いと感じたところ

### 作業前にbookmark名を考えないでいい
まずは作業に取り掛かりたいのに、branchの命名を最初に考えないといけないのは面倒です。まずは手を動かしてやる気を上げていきたい時にはより問題です。
Jujutsuなら、bookmark名を考えるのは作業の途中でも作業終了後でもいつでもいいので、ノイズがなくなった感覚がして好みです。

### ステージングエリア不要で`git add`しなくていい

前述の「ステージングエリアがなく、保存即記録」のとおり、Jujutsuではファイルを保存しただけで現在のchangeに自動で取り込まれます。

```bash
# Git
$ git add .
$ git commit -m "feat: ボタンを追加"

# Jujutsu
# （保存するだけで反映済み。追加するファイルを選ばなくていい）
$ jj describe -m "feat: ボタンを追加"
```

### 再pushの時`--force-with-lease`が自動オプション

push済みのcommitを修正して再pushするとき、Gitだとこうなります。

```bash
$ git add .
$ git commit --amend --no-edit
$ git push origin <branch> --force-with-lease
```

これがJujutsuだと、修正してファイルを保存したあと、以下のコマンドだけで済みます。

```bash
$ jj git push
```

force push相当が安全に自動で走ります（内部的には常にforce-with-lease相当）。楽でいいです。
自動で走ってくれるのがありがたいな、という地味な話でした。はい。
でも塵が積もれば何とやらです。

### stash不要、changeを切り替えるだけ

レビュー中に、現在作業中の内容をいったん置いておいてほかのブランチの内容を確認したい、という場面がよくあります。実際僕はチーム開発の時よくありました。（作業中にレビュー依頼が来て、終わりそうにない自分の作業よりレビューを先に済ませたい的な）
Gitでは、コミットしていない変更があるときはブランチを切り替えられないので、以下のような作業が必要です。

```bash
$ git stash
$ git checkout main
# ... 確認
$ git checkout -
$ git stash pop
```

Jujutsuだと、現在のchangeはそのまま残しておいて、別のchangeに飛ぶだけでOKです。

```bash
$ jj edit main             # mainを直接見にいく
$ jj edit <元のchange-id>  # 戻る
```

Jujutsuでは`jj edit`で別のchangeに移動すると、その瞬間にworking directoryのファイルの中身もそのchangeの状態に切り替わります。元のchangeに戻れば、作業中だったファイルの状態もそのまま復元されます。
stashの存在を忘れてpopできない事故も起きません。

### コミット粒度を後から自由に整えられる

`jj split`（changeを2つに分ける）、`jj squash`（changeを統合する）、`jj edit <change-id>`（過去のchangeを直接編集）など、履歴を後から整える操作が圧倒的にやりやすいです。

Gitの`rebase -i`でeditを選んでamendしてcontinueして、、というのが面倒でしたがJujutsuはこれだけです。

```bash
$ jj edit <change-id>         # そのchangeに移動
# 編集して保存（自動で反映、子孫も自動rebase）
$ jj edit <最新のchange-id>    # 最新に戻る
```

子孫のchangeが自動でrebaseされるので、conflictが起きないかぎり何もしなくていいです。

実際に、コメント機能（モデル・コントローラ・ビュー）を1つのchangeに詰めて作ったあと、`jj split`で3つに分割した例がこちらです。

> 📸 [ここに分割前後の`jj log`のスクショを貼る予定]

これもチーム開発時に使いたかった。。

### `jj undo`で何でも戻せる
実際これが役立った時がないので、もしそういうシチュエーションがあった時追記します。（実感していないことは書けないので）
ただ、いつでも手軽に状態を戻せると考えたら心理的ハードルも下がるため、一応良いポイントとして記載しました。

## つまずいたところ

便利な反面、つまずいたポイントが数個あったので、共有します。

### gh CLIが今いるブランチを認識しない

Jujutsu環境では内部のGit HEADが常にmainを指しているため、`gh pr create`を引数なしで実行すると以下のエラーになります。

```
must be on a branch named differently than "main"
```

結論としては、`--head <bookmark名>`を明示してあげればいいです。

```bash
$ gh pr create --head feat/add-button
```

`gh pr merge`も同様にPR番号かbookmark名を引数で渡す必要があります。

```bash
$ gh pr merge 100                    # PR番号
$ gh pr merge feat/add-button        # bookmark名
```

参考: https://cli.github.com/manual/gh_pr_merge

### bookmark自動生成のデフォルト名が不親切

ブランチ名を自動生成できると見たので、`jj git push -c @`でpushしたら、`push-otpzyxmmoksz`という名前でpushされてしまいました。

```
$ jj git push -c @
Creating bookmark push-otpzyxmmoksz for revision otpzyxmmoksz
Changes to push to origin:
  Add bookmark push-otpzyxmmoksz to 711b5eb034da
remote:
remote: Create a pull request for 'push-otpzyxmmoksz' on GitHub by visiting:
remote:      https://github.com/reckyy/tsundoku-backend/pull/new/push-otpzyxmmoksz
remote:
```

[公式ドキュメント](https://www.jj-vcs.dev/latest/config/#generated-bookmark-names-on-push)を見ると、`templates.git_push_bookmark`でデフォルトを変えられるとのこと。
ただし、変更できるのは**prefix部分のみ**で、`change_id`は一意性のために含める必要があります。

つまり`feat/add-button`のような意味のある名前にはできません。意味のある名前を付けたいなら自分で命名するか、生成AIに任せるかです。

```bash
# 最初から名前を指定してpush
$ jj git push --named feat/add-button=@
```

ちなみに`--named <bookmark名>=@`は「`@`（現在のchange）に`<bookmark名>`という名前のbookmarkを付けて実行して」という意味です。
bookmark名をpushの際についでに作成もできるオプションです。

### 機密ファイルの自動取り込みに注意

これはどちらかというと気を付けたほうがいい点になります。

Jujutsuはファイルを保存しただけで自動的にchangeに取り込まれます。便利な反面、**`.env`やAPIキーのような機密ファイルを誤って取り込みやすい**という側面もあります。`.gitignore`の運用がGit以上に大事です。プライベートリポジトリだと被害は軽減されるかもしれませんが、注意するに越したことはありません。

詳しくはこちらの記事がわかりやすかったので、リンクを掲載します。

[自動トラッキングに関する注意と事故防止](https://zenn.dev/nttdata_tech/articles/33ec2c7972373e#%E8%87%AA%E5%8B%95%E3%83%88%E3%83%A9%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E6%B3%A8%E6%84%8F%E3%81%A8%E4%BA%8B%E6%95%85%E9%98%B2%E6%AD%A2)

## jjuiの便利な使い方

前半で軽く紹介した[jjui](https://github.com/idursun/jjui) 、日常的にどう使っているかを少しだけ詳しく書いておきます。

### diffがめちゃくちゃ見やすい（jjui）

jjuiを別ペインで起動しておけば、各changeのdiffをその場でサッと見られます。レビューの時、差分が大きい場合上から下までスクロールしたりファイルを畳んだり面倒なので、ファイルごとにdiffが表示されるこのUIはいいと感じました。
操作も直感的で、`j` / `k`でchange間を移動、`p`でプレビューパネルを開く、といった具合です。操作方法がわからなくても、`?`でその場で実行可能なキーが一覧表示されます！

![jjuiのpreview](/images/jujutsu_gifs/jjui_preview.png)

プレビューはデフォルトだと右側に出るのですが、『じゅじゅちゅ！』で「下に表示するのがおすすめ」と紹介されていたので僕もそうしています。確かに行が折り返されにくくて見やすいです。
`~/.config/jjui/config.toml`に以下を書き込むだけです。

```toml
[preview]
position = "bottom"
```

### bookmark / descriptionの変更が容易

jjuiで対象を選択するだけで、bookmarkの付け直しやdescriptionの編集ができます。
例えば、descriptionの編集は該当changeを選択し、入力するだけです！

![descriptionを編集しているGIF](/images/jujutsu_gifs/edit_description.gif)

## まとめ
使い始めてまだ1ヶ月かつ、内容も既出のものが含まれてますが伊藤淳一さんの『技術記事を書く技術』を読了し、せっかくだし記事書いてみるかぁ！というモチベで書き上げました。普段あまり構成とか考えなかった分疲れましたが、自分で書く文章より遥かに読みやすくなったかなとは感じています。
技術記事の執筆も、Jujutsuの仕様も継続していきます！

## 参考

- [Jujutsu公式ドキュメント](https://docs.jj-vcs.dev/latest/)
- [じゅじゅちゅ！ `jj new`で始めるJujutsu × AIワークフロー](https://booth.pm/ja/items/8169264)
- [Gitの代わりにJujutsuを使い始めて1ヶ月 - STORES Product Blog](https://product.st.inc/entry/2025/12/09/191758)
- [Gitの次へ。jj（Jujutsu）が変えるバージョン管理の常識](https://zenn.dev/yamitake/articles/jj-jujutsu-modern-vcs-guide)
- [git の次の時代のバージョン管理システム jj (jujutsu)](https://zenn.dev/nttdata_tech/articles/33ec2c7972373e)
- [jjui (TUI)](https://github.com/idursun/jjui)
