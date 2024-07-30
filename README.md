# 現場の実務でよく使うgit操作のユースケース

## 目次

- [現場の実務でよく使うgit操作のユースケース](#現場の実務でよく使うgit操作のユースケース)
  - [目次](#目次)
  - [Featureブランチについて](#featureブランチについて)
    - [ユースケース](#ユースケース)
    - [手順](#手順)
  - [複数のgitリポジトリを扱う場合の.vscode/settings.jsonの管理方法](#複数のgitリポジトリを扱う場合のvscodesettingsjsonの管理方法)
    - [ユースケース](#ユースケース-1)
    - [手順](#手順-1)
  - [作業したファイルをgit diff --name-onlyで一覧化する方法](#作業したファイルをgit-diff---name-onlyで一覧化する方法)
    - [ユースケース](#ユースケース-2)
    - [手順](#手順-2)
  - [本番リリース時に切り戻しブランチを作成するためのgit revert方法](#本番リリース時に切り戻しブランチを作成するためのgit-revert方法)
    - [ユースケース](#ユースケース-3)
    - [手順](#手順-3)
  - [FTPでファイル更新する現場での差分ファイルのコピー方法](#ftpでファイル更新する現場での差分ファイルのコピー方法)
    - [ユースケース](#ユースケース-4)
    - [手順](#手順-4)

---

## Featureブランチについて

gitの管理方法として**git flow**という手法が使われていることがある(特に古い現場で多い)

特長として、新機能の開発ブランチを作成する時に頭に**feature/**をつけてブランチを作成する

※他にhotfixやreleaseなどをつける場合もあるが、形骸化していることが多い

### ユースケース

- 新しい機能の開発を開始する際に使用する
- チームメンバーが独立して作業できるようにする

### 手順

1. Featureブランチの作成

    ```bash
    git checkout -b feature/新機能名
    ```

2. 開発作業を行い、コミット

    ```bash
    git add .
    git commit -m "新機能の説明"
    ```

3. リモートリポジトリへのプッシュ

    ```bash
    git push origin feature/新機能名
    ```

---

## 複数のgitリポジトリを扱う場合の.vscode/settings.jsonの管理方法

複数のディレクトリをvscodeで開いて管理する際、タブの設定やvscodeの背景色を個別で管理したいが、`.vscode/settings.json`がgitの管理対象に含まれて困る場合などに使用する

注意点として、プロジェクトの管理体制上、vscodeのワークスペースごとの設定も含めて管理対象になっている場合は実施しないこと

### ユースケース

- 複数のリポジトリを扱うプロジェクトで、VSCodeの設定ファイルを個別に管理したくない場合
- 運用上`.gitignore`の追加に制限がある、リーダーに打診しづらい場合

### 手順

1. リポジトリのルートで`.git/info/exclude`ファイルを確認する

    ```bash
    $ ls .git/info/exclude
    gitの管理対象のルートに存在するはず
    ```

2. `.git/info/exclude`に`./vscode`を追記する

    ```bash
    $ echo "./vscode" >> .git/info/exclude
    ファイル末尾に追加されること
    ```

3. 追跡対象から外れていることを確認

    ```bash
    $ git status
    ./vscode以下のファイルが追跡対象になっていた場合、表示されなくなること
    ```

---

## 作業したファイルをgit diff --name-onlyで一覧化する方法

ローカルで開発していて、コミットを作りすぎて元のブランチから全体でどのファイルを修正したかわからなくなった場合

### ユースケース

- 変更内容を確認したい場合
- レビュー用に変更ファイル一覧を取得したい場合

### 手順

1. 作業ブランチとmasterブランチの差分を確認

    ```bash
    git diff --name-only master..作業ブランチ名
    ```

---

## 本番リリース時に切り戻しブランチを作成するためのgit revert方法

### ユースケース

- 本番環境にデプロイ後、問題が発生した場合に元の状態に戻す

### 手順

1. 本番環境での最新のコミットIDを確認

    ```bash
    git log
    ```

2. 巻き戻し用のブランチを作成

    ```bash
    git checkout master // 開発ブランチの元ブランチ
    git checkout -b feature/xxx_rb // 巻き戻しブランチは_rbなどをつけて開発ブランチと同名で切ることが多い
    ```

3. 変更をリセットするためのrevertを実行

    ```bash
    コミット履歴が一つの場合
    git revert コミットID

    コミット履歴が複数の場合
    $ git revert HEAD   // 直前のcommitをrevert
    $ git revert HEAD^  // 一つ前まで(直前とその前)をrevert
    $ git revert HEAD~n // n個前までのcommitをrevert
    $ git revert HEAD~コミットID // 直前からコミットIDの一つ後までのcommitをrevert
    ```

4. リモートリポジトリにプッシュ

    ```bash
    git push
    その後、マージリクエストを作成する
    ```

---

## FTPでファイル更新する現場での差分ファイルのコピー方法

### ユースケース

- FTPを使用して手動でファイルを更新する現場で、差分のみをアップロードしたい場合

### 手順

1. masterブランチと開発ブランチの差分を抽出

    例) masterブランチと開発中のブランチの差分を、任意のテキストに出力

    ```bash
    gitリポジトリ内で実行
    git diff --name-only master..開発ブランチ名 > changed_files.txt
    ```

2. 差分ファイルをコピーするためのスクリプトを作成（例：bashスクリプト）

    ```bash
    #!/bin/bash
    while read file; do
        cp --parents "$file" /path/to/directory/ # 任意のコピー先ディレクトリ(フルパス)
    done < changed_files.txt # 差分一覧ファイルはバッチから読み取れる場所を指定
    ```

    ※cpコマンドのparentsオプションはlinuxだと実行可能(windows git bashだと使える、macだと使えない可能性あり)

3. スクリプトを実行して差分ファイルをコピー

    ```bash
    ./copy_diff_files.sh
    ```
