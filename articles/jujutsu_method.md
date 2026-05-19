---
title: "Git使いがJujutsu (jj)を1ヶ月使ってみた：便利だった点とハマった点"
emoji: "🥋"
type: "tech"
topics: ["jujutsu", "jj", "git", "vcs", "技術記事を書く技術"]
published: false
---

## はじめに

reckyです。

最近、Gitの代わりとなるバージョン管理システムであるJujutsu(jj)にふれる機会があり、僕自身1か月ほど使ってみました。
使いはじめて間もないですが、Gitより使いやすく使い続けているので、特に「これは便利だ」と感じたポイントと、逆に「ここでハマった」というポイントをまとめます。

きっかけは、以前『りあクト！』シリーズを購入していた縁でboothから届いた新刊メールでした。同じ著者から『じゅじゅちゅ！ jj newで始めるJujutsu × AIワークフロー』が出るという案内を見て、新しいもの好きのため「Gitより使いやすそうだから試してみよう」くらいの軽い気持ちで購入。読みながら手を動かしているうちに、いつの間にか普段使いするようになっていました。

:::message
本記事は**普段Gitを使っていてJujutsuがまだ初耳〜なんとなく知っている**くらいの方を想定して書いています。
Jujutsuを網羅的に解説する記事ではありませんが、記事を読み進めるのに必要な前提知識は本文中で都度補足しているので、未使用の方もそのまま読めるはずです。
:::

### 動作環境

- macOS 15.x（Apple Silicon）
- Jujutsu (jj) 0.41.0
- jjui 0.10.3
- gh CLI 2.92.0

本記事にもし間違いがあれば、優しく教えてください。🙂‍↕️

## Jujutsuの基本: Gitと何が違うのか

ここから先の話を読むうえで前提となる、Jujutsuの基本を3つだけ紹介します。
Gitとの違いを意識した最小限の説明なので、ここを押さえれば以降の内容はスッと頭に入るはずです。

### commitの代わりにchange、branchの代わりにbookmark

Jujutsuには、Gitとは別の名前で似たような概念がいくつかあります。最初は戸惑うので対応表にしておきます。

| Git | Jujutsu | 補足（Jujutsu） |
|---|---|---|
| commit | **change** | Jujutsuの作業の最小単位 |
| branch | **bookmark** | 特定のchangeを指す名前付きポインタ |
| HEAD | **`@`** | 今いるchangeを指す記号 |
| `git add` | （不要） | 保存だけで自動反映 |
| `git commit --amend` | （不要） | `@`への編集は自動で反映される |
| `git stash` | （不要） | 別のchangeに飛ぶだけ |

このうち、Gitの感覚で読むときに**特に押さえておきたい違い**が2つあります。

**changeは何度でも書き換えられる**
Gitのcommitは「一度作ったらハッシュが固定で、書き換えるには`--amend`や`rebase -i`が必要」でした。Jujutsuのchangeは**何度でも気軽に書き換えられる**もので、`@`の中身を編集して保存すれば自動で反映されます。これが「履歴を後から整える操作の手軽さ」につながります。

**bookmarkは自動で動かない**
Gitのbranchは「現在のbranch」に新しいcommitを積むと、branchも一緒に先に進みます。Jujutsuのbookmarkは**そういう「自動追従」をしません**。bookmarkは「特定のchangeに付けておく目印（ラベル）」であって、自分で`jj bookmark move`のようなコマンドで動かさない限り元の位置のままです。

これは最初は不便に感じるかもしれませんが、「いま作業中の枝に必ずしも名前を付けなくていい」というメリットにつながります（後の節で触れます）。

### ステージングエリアがなく、保存即記録

Gitでは、ファイルの変更が履歴に記録されるまでに**3つの状態**を経由します。

```
[Working Tree] --git add--> [Staging Area] --git commit--> [Repository]
   作業中のファイル              ステージング領域            履歴データベース
```

これがJujutsuでは**2つに減ります**。

```
[Working Copy] -------(自動)-------> [Repository]
   作業中のファイル                     履歴データベース
```

Jujutsuには、Gitでいうステージング領域（index）に相当するものがありません。
代わりに、**`jj`コマンドを実行したタイミングで、その時点のworking copyの状態が自動的にスナップショットされ、履歴に記録**されます。`jj`は常駐デーモンではないので、リアルタイム監視で勝手にcommitされるわけではなく、何らかの`jj`コマンドを叩いた瞬間に取り込まれる、という仕組みです。

実用上は「ファイルを保存しておけば、次に`jj`コマンドを打ったときに勝手にchangeへ反映されている」と覚えておけばOKです。`git add` → `git commit`の2ステップが不要になります。

```bash
# Git
$ git add app/models/post.rb
$ git commit -m "feat: Postに公開状態を追加"

# Jujutsu
# （ファイルを保存するだけで自動的にchangeに反映済み）
$ jj describe -m "feat: Postに公開状態を追加"
```

「あ、`git add`忘れたからcommitに入ってない！」という事故がそもそも起きません。

この「**ステージングがない・保存即記録**」が、Jujutsuのいちばんの根っこです。

### `@`が動くとファイルも動く

最後にもうひとつ。Jujutsuを使ううえで一番大事な感覚です。

Jujutsuでは、**「今いるchange（`@`）の中身」と「working directoryのファイル中身」が常に同期**しています。

```
@ の中身  ⇄  working directory のファイル中身
```

たとえば`jj edit <change-id>`で`@`を別のchangeに移動すると、**その瞬間にファイルの中身もそのchangeの状態に書き換わります**。`jj new`で新しいchangeを作って飛んでも同じです。

これだけだと抽象的なので、シンプルな例で確認しておきます。

```bash
# 現在のchange (CHANGE_A) で memo.md を作成
$ echo "# memo" > memo.md
$ cat memo.md
# memo

# 別のchangeに飛ぶ（@が動く）
$ jj new main -m "別の作業"

# memo.md は消えている
$ cat memo.md
cat: memo.md: No such file or directory

# 元のchangeに戻る（@を戻す）
$ jj edit CHANGE_A

# memo.md が戻ってきている
$ cat memo.md
# memo
```

> 📸 [ここに検証フェーズ1-0-2の3ペアスクショを貼る予定]

Gitの`git checkout <commit>`に近い感覚ですが、Jujutsuでは**これがあらゆる操作の前提**になっています。`jj edit`も`jj new`も、コマンドを実行した瞬間に `@`が動いてファイルも切り替わる。

この大原則を頭に入れておくと、後で出てくる**「ハマり: PRマージ直後にファイルが消えたように見える」**が「なるほど、`@`が古い位置に取り残されているからファイルもその時点のものになっているのか」と自然に納得できるはずです。

## 環境構築とツール紹介

具体的なインストール手順は公式や既存記事に記載されてるので、本記事では「何を入れたか」と「なぜそれを使うのか」だけ簡単に紹介します。

### 入れたもの

| ツール | 役割 | 入れ方 |
|---|---|---|
| **Jujutsu (jj)** | Jujutsu本体 | [公式: Installation and setup](https://docs.jj-vcs.dev/latest/install-and-setup/) |
| **jjui** | jjの履歴を常時表示できるTUI | [GitHub: idursun/jjui](https://github.com/idursun/jjui) |
| **Ghostty** | シンプルで軽量なターミナル | [公式](https://ghostty.org/) |
| **gh CLI** | GitHubのPR操作などに使う（Jujutsuに必須ではない） | [公式](https://cli.github.com/) |

MacならjjもjjuiもHomebrewで`brew install jj` / `brew install jjui`の一発で入ります。
jjもjjuiも前述の書籍内でおすすめされていたのをそのまま採用しただけなので、ほかにも同機能のツールはあります。それぞれ好みのツールを使用してください。

### jjuiとは

[jjui](https://github.com/idursun/jjui) は、Jujutsuの履歴をターミナル上でグラフィカルに表示できるTUIツールです。

Jujutsuは履歴を自由に行き来できる設計なので、**「今どこにいて、ツリーがどうなっているか」を常に見ておきたい**シーンが多くあります。とはいえ、いちいち`jj log`を実行するのは面倒。jjuiを別ペインで起動しておくと、コマンドを実行するたびにツリーがリアルタイムで更新されるので、操作と確認の往復が圧倒的に減ります。

詳しい使い方は記事の後半[jjuiの便利な使い方](#jjuiの便利な使い方)で紹介しますが、**Jujutsuを使うならjjuiもセットで入れるのを強くおすすめ**します。

### Ghostty(ターミナル)を使うと便利

jjuiは常駐させてこそ真価を発揮するので、**画面分割ができるターミナル**があるとさらに快適です。
僕は[Ghostty](https://ghostty.org/)を使っていて、左ペインでコマンドを打ち、右ペインでjjuiを表示しっぱなしにしています。お使いのものをどうぞ。

## 良いと感じたところ

### 作業前にbookmark名を考えないでいい
まずは作業に取り掛かりたいのに、branchの命名を最初に考えないといけないのは面倒です。
Jujutsuなら、bookmark名を考えるのは作業の途中でも作業終了後でもいつでもいいので、ノイズがなくなった感覚がして好みです。

### ステージングエリア不要で`git add`漏れがなくなる

前述の「ステージングエリアがなく、保存即記録」のとおり、Jujutsuではファイルを保存しただけで現在のchangeに自動で取り込まれます。

```bash
# Git
$ git add .
$ git commit -m "feat: ボタンを追加"

# Jujutsu
# （保存するだけで反映済み）
$ jj describe -m "feat: ボタンを追加"
```

Gitだとファイルを編集して `git add`でstagedにしてから`git commit`で記録という2ステップが必要です。正直、得した経験があんまりなくて、毎回`git add`する時間が無駄に感じていました。

Jujutsuでは「あ、`git add`忘れたからcommitに入ってない！」という事故がそもそも起きません。

> 📸 [ここに`jj st`で複数ファイルが自動的に検知されているスクショを貼る予定]

### 再pushで`--force-with-lease`を意識しなくていい

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

**force push相当が安全に自動で走ります**（内部的には常にforce-with-lease相当）。
`--force`のタイプミスで事故を起こす心配もなく、毎回オプションを思い出す必要もありません。チーム開発のときに使えていたら楽だっただろうな〜と感じました。

> 📸 [ここに2回目のpushログを貼る予定]

### stash不要、changeを切り替えるだけ

レビュー中に、現在作業中の内容をいったん置いておいてほかのブランチの内容を確認したい、という場面がよくあります。
Gitでは、コミットしていない変更があるときはブランチを切り替えられないので、以下のような作業が必要です。

```bash
$ git stash
$ git checkout main
# ... 確認
$ git checkout -
$ git stash pop
```

Jujutsuだと、**現在のchangeはそのまま残しておいて、別のchangeに飛ぶだけ**でOKです。

```bash
$ jj edit main             # mainを直接見にいく
$ jj edit <元のchange-id>  # 戻る
```

これも前述の「`@`が動くとファイルも動く」の応用ですね。`@`を別のchangeに移動するだけで、ファイルの中身も切り替わります。元のchangeに戻れば、作業中だったファイルの状態もそのまま復元されます。

stashの存在を忘れてpopできない事故も起きないので、これも地味に嬉しいです。

> 📸 [ここに「飛ぶ前」「飛んだ直後」「戻った直後」の3枚スクショを貼る予定]

### コミット粒度を後から自由に整えられる

`jj split`（changeを2つに分ける）、`jj squash`（changeを統合する）、`jj edit <change-id>`（過去のchangeを直接編集）など、**履歴を後から整える操作が圧倒的にやりやすい**です。

Gitの`rebase -i`でeditを選んでamendしてcontinueして...というのが苦手だったのですが、Jujutsuはこれだけです。

```bash
$ jj edit <change-id>          # そのchangeに移動
# 編集して保存（自動で反映、子孫も自動rebase）
$ jj edit <最新のchange-id>    # 最新に戻る
```

**子孫のchangeが自動でrebaseされる**ので、conflictが起きないかぎり何もしなくていいです。

実際に、コメント機能（モデル・コントローラ・ビュー）を1つのchangeに詰めて作ったあと、`jj split`で3つに分割した例がこちらです。

> 📸 [ここに分割前後の`jj log`のスクショを貼る予定]

これもチーム開発時に使いたかった……！

### conflictがブロックされず、後から解決できる

Gitでconflictが発生すると、操作が強制的にブロックされます。`git merge` や `git rebase` の途中でconflictに当たると、それを解消するまで先に進めません。

Jujutsuはここが根本的にちがって、**conflictはただの「状態」として扱われ、操作をブロックしません**。Jujutsuの公式ドキュメントでは[First-class conflicts](https://docs.jj-vcs.dev/latest/conflicts/)と呼ばれています。

たとえば`jj rebase`の途中でconflictが起きても、rebaseコマンド自体は成功します。conflictは新しくできたchangeに`(conflict)`というマークがついた状態で記録されるだけで、解決はあとで好きなタイミングでやればOKです。

#### conflictマーカーについて

ちなみにJujutsuのconflictマーカーは独特で、Gitのもの（`<<<<<<<` `=======` `>>>>>>>`）よりも情報量が多い分、人間が素で読み解くのはしんどいです。

```
<<<<<<< conflict 1 of 1
%%%%%%% diff from: vpxusssl 38d49363 "merge base"
\\\\\\\        to: rtsqusxu 2768b0b9 "commit A"
 apple
-grape
+grapefruit
 orange
+++++++ ysrnknol 7a20f389 "commit B"
APPLE
GRAPE
ORANGE
>>>>>>> conflict 1 of 1 ends
```

実際の解決手段としては、`jj resolve`コマンドで設定済みのマージツール（VS Code標準のmerge editorやMeldなど）を起動して、Gitと同じ感覚で3-wayマージできます。

「conflictが起きても操作はブロックされず、解決は後回しでOK」というだけでも、Gitに慣れた身からするとかなり気持ちが軽くなる体験でした。

> 不慣れなうちにVS Code上でconflict解決したい人には、[JJ View](https://marketplace.visualstudio.com/items?itemName=jj-view.jj-view)というVS Code拡張もありますが、僕はまだ使ってないので詳細は割愛します。使うタイミングがあれば、この記事に追記します。

### `jj undo`で何でも戻せる

Jujutsuには、**ローカルでのあらゆる操作をundoできる**`jj undo`というコマンドがあります。

これがどれくらい強力か、実際に「うっかり大事なchangeを`jj abandon`で消してしまった」状況を再現してみます。

```bash
# 大事なchangeを作る
$ jj new main -m "feat: 大事な機能"
$ echo "important content" > important.txt
$ jj log
@  abcd1234 (...) feat: 大事な機能
◆  main efgh5678 (...)

# うっかり abandon
$ jj abandon abcd1234

$ jj log
# → さっきのchangeが消えている
$ ls important.txt
ls: important.txt: No such file or directory
```

> 📸 [ここに abandon 直後の`jj log`スクショを貼る予定]

ここで`jj undo`を叩きます。

```bash
$ jj undo

$ jj log
@  abcd1234 (...) feat: 大事な機能   ← 復活！
◆  main efgh5678 (...)
$ cat important.txt
important content
```

> 📸 [ここに`jj undo`後の`jj log` + `cat important.txt`復活のスクショを貼る予定]

完全に元通りです。

「いやもっと前」というときは、`jj op log`で操作履歴をたどって任意の時点に戻せます。

```bash
$ jj op log
@  ab2b0fa8704e undo: restore to operation ...
○  cd5c9de7e65b abandon commit abcd1234...
○  2c3ab596b5f2 ...
$ jj op restore <operation-id>
```

> 📸 [ここに`jj op log`のスクショを貼る予定]

Git にも`reflog`がありますが、`reflog`を読み解いて安全に元に戻すには Git の内部構造への知識が必要で、パニック状態だと使いこなすのは正直しんどいです。`jj op log`はもっとシンプルで、操作単位で記録されているので「あの操作の前に戻す」が直感的にできます。

**何をやってもとりあえずundoできる**という安心感のおかげで、Jujutsuだと「とりあえずやってみる」のハードルが圧倒的に下がりました。

## ハマったところ

便利な反面、Gitの感覚で進めるとひっかかるポイントがいくつかあったので、共有します。

### gh CLIが「今いるブランチ」を認識しない

Jujutsu環境では内部のGit HEADが常にmainを指しているため、`gh pr create`を引数なしで実行すると以下のエラーになります。

```
must be on a branch named differently than "main"
```

> 📸 [ここに`gh pr create`のエラーメッセージのスクショを貼る予定]

結論としては、`--head <bookmark名>`を明示するのが正解です。

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

`jj git push -c @`で誘導されるままpushしたら、`push-otpzyxmmoksz`という名前でpushされてしまいました。

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
ただし、**変更できるのはprefix部分のみ**で、`change_id`は一意性のために含める必要があります。

つまり「`feat/add-button`のような意味のある名前」にはできません。意味のある名前を付けたいなら、**最初から `--named <bookmark名>=@`を使うべき**、というのが結論です。

```bash
# 最初から名前を指定してpush
$ jj git push --named feat/add-button=@
```

`--named <bookmark名>=@`は「`@`（現在のchange）に`<bookmark名>`という名前のbookmarkを付けて実行して」という意味です。

### PRマージ直後に「設定ファイルが消えた」ように見える

これは個人的に一番焦ったハマりで、Jujutsuのメンタルモデルが頭に入っていないと原因がわかりません。

#### 何が起きたか

ある日、機能ブランチをPRにしてマージ（`gh pr merge <PR番号> -d`）した直後、ファイルを見たら**さっき書いたはずのコードが消えているように見えました**。

```bash
$ gh pr merge 123 -d --merge
# マージ成功

$ cat config/routes.rb | grep "search"
# → 何も出ない！ さっき書いたはずのルーティングが消えている？？
```

> 📸 [ここに「ファイルが消えて見える」VS Codeのファイル状態のスクショを貼る予定]

#### 原因: `@`が古い位置に取り残されている

原因は、先述の「**`@`が動くとファイルも動く**」を思い出すとわかります。

PRをマージした時点で、リモートのmainは新しい位置に進みました。一方、**ローカルの `@` はまだ古い位置（PRを作る前のmainの直上）に残ったまま**です。

```
マージ前:
◆  main (古い位置)
│
@  feat/add-search ← @はここ。search アクションあり
```

```
マージ後（jj log で見ると）:
◆  main (古い位置、リモートはもっと先に進んでいるけどローカルはまだ追いついていない)
│
@  (空change) ← @はここ。古いmainの直上にぽつんと残されている
```

先述のとおり、**`@`の中身とworking directoryのファイル中身は常に同期**しています。`@` が古いmainの直上の空changeに取り残されているということは、**ファイルの中身も「マージ前のmainの状態」**になっている、というわけです。だからsearchアクションが「消えた」ように見えた、というわけです。

> 📸 [ここに`jj log`でマージ直後の@の位置を示すスクショを貼る予定。`@`の位置と「ローカルのmain」の位置に、画像上で矢印や囲みを入れて読者が一目でわかるようにする]

#### 解決: fetchして `jj new main` する

最新のmainに`@`を移動させれば、ファイルも一緒に最新状態になります。

```bash
$ jj git fetch            # リモートの最新を取り込む（mainが進む）
$ jj new main             # 最新のmainの上に新しいchangeを作って@を移動
$ cat config/routes.rb | grep "search"
# → search アクションが復活！
```

> 📸 [ここに`jj git fetch`後 → `jj new main`後 の各`jj log`とcat出力のスクショを貼る予定。`@`が古い位置から最新mainの直上に移動した様子を、画像上で矢印で示す]

`jj git fetch`でリモートの最新位置を取り込み、`jj new main`で`@`を最新mainの直上に移動。`@`が動けばファイルも追従するので、消えていたコードが戻ってきます。

#### 前提: `jj bookmark track main --remote=origin`を最初にやっておく

上の解決手順がそのまま使えるのは、ローカルの main bookmarkがリモートを**track（追跡）**している場合です。書籍では`jj git clone`で始めるシナリオが紹介されていて、その場合は最初からtrack済みになります。

ただ、僕のように**既存のGitリポジトリで`jj git init`する**場合、デフォルトではtrackされません。そのまま`jj git fetch`しても、リモートの`main@origin`だけが進んで**ローカルのmainは古い位置のまま**になってしまいます。

なので、`jj git init`した直後に一度、track設定をしておくのがおすすめです。

```bash
$ jj bookmark track main --remote=origin
```

これを最初にやっておけば、PRマージ後も`jj git fetch && jj new main`で正しく追従できます。

### 機密ファイルの自動取り込みに注意

これは「ハマった」というより「気を付けたほうがいい」点ですが、ハマる前提知識として共有しておきます。

Jujutsuはファイルを保存しただけで自動的にchangeに取り込まれます。便利な反面、**`.env`やAPIキーのような機密ファイルを誤って取り込みやすい**という側面もあります。`.gitignore`の運用がGit以上に大事です。プライベートリポジトリだと被害は軽減されるかもしれませんが、注意するに越したことはありません。

詳しくはこちらの記事がわかりやすかったので、引用させていただきます。

[自動トラッキングに関する注意と事故防止](https://zenn.dev/nttdata_tech/articles/33ec2c7972373e#%E8%87%AA%E5%8B%95%E3%83%88%E3%83%A9%E3%83%83%E3%82%AD%E3%83%B3%E3%82%B0%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E6%B3%A8%E6%84%8F%E3%81%A8%E4%BA%8B%E6%95%85%E9%98%B2%E6%AD%A2)

## jjuiの便利な使い方

「環境構築とツール紹介」で軽く紹介した [jjui](https://github.com/idursun/jjui) 、日常的にどう使っているかを少しだけ詳しく書いておきます。

### diffがめちゃくちゃ見やすい（jjui）

jjuiを別ペインで起動しておけば、各changeのdiffをその場でサッと見られます。`jj log`でツリーを眺めつつ、気になるchangeの中身を瞬時に確認できるのは、ターミナルで `git diff` を打ちつづけていた頃と比べて圧倒的に楽です。

操作も直感的で、`j` / `k`でchange間を移動、`p`でプレビューパネルを開く、といった具合にVimライクなキーバインドで完結します。

> 🎬 [ここにjjuiでchangeを選択して`p`キーでpreviewパネルを開き、diffが表示されるまでのGIFを貼る予定]

### bookmark / descriptionの変更が容易

jjuiで対象を選択するだけで、bookmarkの付け直しやdescriptionの編集ができます。`jj bookmark move ...`のようなコマンドを覚えなくても、ピルをドラッグするだけです。

> 🎬 [ここにjjuiでbookmarkをドラッグして移動する操作のGIFを貼る予定]

### Ghostty + jjuiの常駐スタイル

jjuiの真価は**常駐させてリアルタイムにツリーを見続けられること**にあります。僕は[Ghostty](https://ghostty.org/)で画面を縦に分割して、左ペインでコマンドを打ち、右ペインにjjuiを表示しっぱなしにしています。

> 🎬 [ここにGhostty画面分割で、左ペインで`jj`コマンドを実行すると右ペインのjjuiがリアルタイムに更新される様子のGIFを貼る予定]

コマンドを実行するたびに右ペインのツリーが更新されるので、「いま何が起きたか」が常に視覚で確認できます。Jujutsuは履歴を自由に動き回るVCSなので、この**常時可視化**の効果は本当に大きいです。

なお、jjuiの設定で`[preview] position = "bottom"`にしておくと、プレビューパネルが下半分に表示されて行が折り返されにくくなります。`~/.config/jjui/config.toml`に書いておくのがおすすめです。

```toml
[preview]
position = "bottom"
```

## まとめ

Jujutsuに乗り換えて1ヶ月、感じたことを箇条書きでまとめます。

- Gitの`add` / `--amend` / `--force-with-lease` / `stash`まわりがまるごと不要になって、日常の摩擦が減った
- 履歴を後から整える操作が圧倒的にやりやすく、レビュー対応が楽になった
- conflictがブロックされず、後回しにできるのが本当に気持ちいい
- `jj undo`で何でも戻せる安心感のおかげで、試行錯誤のハードルが下がった
- gh CLIまわりやbookmarkの挙動、PRマージ後の`@`の動きなど、Git感覚だとハマるポイントもあるが、**「`@`が動くとファイルも動く」というメンタルモデル**さえ押さえれば原理から納得できる
- **チームのほかのメンバーがGitのままでも、自分だけJujutsuを使える**のが導入の最大の決め手

特に最後の点が大きく、共同開発の環境でも一人で試せます。興味がある方はまずは個人のプロジェクトでぜひ始めてみてください。最初に`jj bookmark track main --remote=origin`だけ忘れずに、あとは`jj undo`を信じて遠慮なく試行錯誤するのがおすすめです。

「Gitの不満を地味に解消してくれる相棒」という感覚で、これからも使いつづけたいです。

## 参考

- [Jujutsu公式ドキュメント](https://docs.jj-vcs.dev/latest/)
- [じゅじゅちゅ！ `jj new`で始めるJujutsu × AIワークフロー](https://booth.pm/ja/items/8169264) ※書籍
- [Gitの代わりにJujutsuを使い始めて1ヶ月 - STORES Product Blog](https://product.st.inc/entry/2025/12/09/191758)
- [Gitの次へ。jj（Jujutsu）が変えるバージョン管理の常識](https://zenn.dev/yamitake/articles/jj-jujutsu-modern-vcs-guide)
- [git の次の時代のバージョン管理システム jj (jujutsu)](https://zenn.dev/nttdata_tech/articles/33ec2c7972373e)
- [君のレポジトリを領域展開 - 次世代バージョン管理システムJujutsu (jj-vcs/jj)の世界](https://zenn.dev/zetamatta/books/c1e309aea68960/)
- [jjui (TUI)](https://github.com/idursun/jjui)
