---
title: Gitコマンド整理
date: 2017-05-11 13:15:09
tags:
- Git
id: git-commands
---

以下のような感じでGitコマンドを整理する。

- ローカル環境設定
- ローカル操作
- リモート操作
- ブランチ操作

<!-- more -->

# ローカル環境設定

- `git init`
  - ローカルリポジトリの作成
- `git clone`
  - リモートリポジトリからローカルリポジトリを複製
- `git config`
  - Gitクライアントの設定


# ローカル操作

Gitの以下の保存領域がある。

- <span style="color:red">ワーキングディレクトリ</span> / <span style="color:red">ワークツリー</span>
  - 作業スペース。編集中のファイルがある。
- <span style="color:red">ステージングエリア</span> / <span style="color:red">インデックス</span>
  - コミットする内容をた保存するスペース。
- <span style="color:red">ローカルリポジトリ</span>
  - コミットするとステージングエリアの内容が保存される。
- <span style="color:red">リモートリポジトリ</span>
  - ローカル以外のすべてのリモートリポジトリ。

ワーキングディレクトリからローカルリポジトリまでには以下の操作がある。

- `git add`
  - ステージング（ワーキングディレクトリ → ステージングエリア）
  - ワーキングディレクトリの変更をステージングエリアに反映する
- `git commit`
  - コミット（ステージングエリア → ローカルリポジトリ）
  - ステージングエリアの内容をローカルリポジトリに反映する
- `git status`
  - ワーキングディレクトリとステージングエリアの変更状況を表示する
    - <span style="color:red">赤</span>はワーキングディレクトリ、<span style="color:blue">青</span>はステージングエリアの変更
- `git log`
  - コミットによるローカルリポジトリの変更履歴を表示する
- `git diff`
  - *ステージングエリア* と *ローカルリポジトリ* を比較する
- `git checkout`
  - 状態を戻す
- `git commit --amend`
  - 直前のコミットを削除してステージングエリアが１つ前の状態に戻る
- `git revert`
  - 過去のコミットを削除する（ワーキングディレクトリの内容も戻る）
  - `git revert HEAD^`
    - HEADの一個前を消す
  - `git revert HEAD^^`
    - HEADの二個前を消す
  - 「 *HEAD* 」は現在のブランチの先頭、最新コミットへのポインタを指す
- `git reset`
  - コミット履歴をさかのぼって、それ以降のコミットをなかったことにする
    - `git revert` は打消し処理も履歴として残るが、 `git reset` は取消しの履歴も残らない
  - `git reset <file>`
    - ステージングエリアに追加されたファイルをステージングエリアから削除する
    - ワーキングディレクトリはそのまま
  - `git reset --hard HEAD^`
    - 現在のブランチHEADの１つ前の状態に戻る（HEADが１つ戻り、前のHEADはなくなる）
    - ワーキングディレクトリも戻る
  - `git reset <commit id>`
    - 現在のブランチの位置を `<commit id>` まで戻しつつステージングエリアもそのときの状態に戻す
    - ワーキングディレクトリはそのまま
    - `--hard` をつけるとワーキングディレクトリも戻る
  - resetには「 *soft* 」「 *mixed* 」「 *hard* 」の３つのモードがありそれぞれ下表のようになる

|モード|HEADの位置|ステージングエリア|ワーキングディレクトリ|
|:----:|:--------:|:----------------:|:--------------------:|
|soft  |変更する  |変更しない        |変更しない            |
|mixed |変更する  |変更する          |変更しない            |
|hard  |変更する  |変更する          |変更する              |


# リモート操作

- `git push`
  - リモートリポジトリにローカルリポジトリの変更を反映する
- `git pull`
  - リモートリポジトリから変更を取得する（マージまで行う）
  - 「`git fetch` + `git merge origin/master`」に同じ
- `git fetch`
  - リモートリポジトリから最新の変更をローカルリポジトリに持ってくる（マージは行わない）

## *master* と *origin/master* の違い

*master* はローカルリポジトリのmasterブランチを指し、 *origin/master* はリモートリポジトリのmasterブランチを指す。  
ただし、 *origin/master* はリモートリポジトリと結びついているブランチであって、ローカルに存在する。  
`git fetch` することによって、ローカルの *origin/master* が更新される。  
つまり、 `git fetch` した後に `git merge origin/master` すると、「ローカルの *master* ブランチに最新状態に更新した *origin/master* の変更をマージする」ということになる。


# ブランチ操作

- `git branch`
  - ブランチの一覧を表示する
  - `git branch <branchname>`
    - ブランチを作成する（現在のブランチのリビジョンで作成される）
  - `git branch -d <branchname>`
    - ブランチを削除する
- `git checkout`
  - ブランチの切り替え（元のブランチ名はmaster）／リビジョンへの切り替えもできる
  - `git branch <branchname>/<revision>`
- `git merge`
  - マージ（masterからnewBranchを取り込む）
  - `git merge <branch>`
    - 今のブランチに指定したブランチをマージする
    - コンフリクトが発生するとコンフリクトが発生したファイルを修正してaddとcommit（メッセージは自動生成）を実行する必要がある
- `git reset`
  - 変更（マージ）の取り消し、指定したリビジョンまで戻る
  - `git reset --hard HEAD^`
    - 現在のブランチHEADの１つ前の状態まで戻る（HEADが１つ戻り、前のHEADはなくなる）
  - 前述したがコミットの取り消しにも使用できる

- `git rebase <branchname>`
  - 指定したブランチのHEADに現在のブランチの変更を追加する
  - `git rebase master`
    - 現在のブランチをmaster(HEAD)から作成したことにする
  - 競合が発生した場合は修正の後`git rebase --continue`を実行する
