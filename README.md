# 入出力ファイル一括取得プログラム

当プログラムは機械学習用に指定したワークフローの計算結果を取得する。

## 概要　

このプログラムは二段階で実行する。まず、ワークフローIDを指定してこのワークフローを実行したランの情報を全て取得する。その後その情報からラン毎の入出力ファイル内容を取得する。

必要なものは以下となる。

* ワークフローID
* APIトークンまたはログイン情報
* 環境名
* siteID
* 入出力情報の変換対応テーブル

あれば便利または詳細な指定が可能なファイル。
* 絞り込み用ラン一覧リスト
  + 空白区切りで４カラムめにRxxxxxyyyyyyyyyyというラン番号が格納されているファイル。
  + ```runlist:<ファイル名>``` で指定する。

## システム構成

本スクリプトを実行するために必要なシステム構成を記述する。

* OS
  + python3.6が実行できれば特定のOSに縛られない。
* python
  + Version3.6以降
  + requests package
  + misrc_workflow_python_lib(from gitlab)


## 使い方

本プログラムは２段階で使用する。そのためにmodeをしてする必要がある。modeによって与えるパラメータも違ってくる。
詳細は、[リポジトリのWIKIページ](https://gitlab.mintsys.jp/midev/extract_run_results/-/wikis/%E5%8B%95%E4%BD%9C%E4%BB%95%E6%A7%98)を参照。

### 使い方の例
* ラン情報の取得
```
$ python3.6 ~/extract_run_results/run_results_m.py token:６４文字のトークン misystem:dev-u-tokyo.mintsys.jp workflow_id:W000020000000300 mode:iourl csv:results.csv siteid:site00002
```

tokenパラメータの指定が無い場合は、ログインプロンプトで対応する。

* 編集
  + 対象ファイル
  ```
  $ vi table_template.tbl
  ```
  + 編集内容
  ```
  {"<カラム名（ポート名）>":"csv", "default": None, "ext": ""}
  ```
  + カラム名
    - スカラーデータはcsvのまま。
    - テキストファイルに残す場合は、fileを指定する。
    - バイナリファイルに残す場合は、filebを指定する。
    - この項目がいらない場合は、deleteを指定する。
  + default
    - csvの時かつ非必須でパラメータ指定しなかったとき、デフォルト値として値を指定すると、こちらが使われる。
    - パラメータ指定していた場合でも、ここがNone以外の時はそちらが使われる。
  + ext
    - カラム名の指定が、fileまたはfilebの時の拡張子
    - ファイル名は、```<ラン番号>_カラム名.<指定した拡張子>``` となる。

* 機械学習データ構築
```
$ python3.6 ~/extract_run_results/run_results_m.py mode:file csv:results.csv table:table_template.tbl dat:W000020000000300.csv
```
 
## ヘルプの表示

```
ワークフローIDの計算結果データを取得する。

Usage:  $ python /home/misystem/extract_run_results/run_results_m.py workflow_id:Mxxxx token:yyyy misystem:URL mode:[iourl/file] [options]

必須パラメータ
               mode   : 動作モード。
                        iourl : 入出力名をヘッダーとしたランIDごとのCSVファイルを作成する。
                                各カラムは計算結果データをGPDBへのURLが格納される。
                        file : iourlモードで作成したテーブルと別途用意した構成ファイルを使い、
                                機械学習向けのCSVファイルを作成する。 
                csv   : iourlモードで作成されるCSVファイルの名前
                        fileモードではiourlモードで作成したCSVとして指定する。
               conf   : いくつかのパラメータを書いておける便利な構成ファイル
                        README.mdを参照

     mode を iourlと指定したとき
          workflow_id : Wで始まる15桁のワークフローID
               token  : 64文字のAPIトークン。指定しない場合ログイン問い合わせとなる。
             misystem : dev-u-tokyo.mintsys.jpのようなMIntシステムのURL
              siteid  : siteと＋５桁の数字。site00002など
              thread  : API呼び出しの並列数（デフォルト10個）
             usecash  : 次回以降キャッシュから読み込みたい場合に指定する。
                        未指定で実行すればキャッシュは作成される。

     mode を fileと指定したとき
               table  : iourlで取得したGPDB情報を変換するテーブルの指定
                dat   : fileモードで作成される結果ファイル。機械学習用
非必須のパラメータ
            runlist   : modeがiourlの時に指定する。
                        workflow_execute.pyが出力するランリスト。
                        空白区切りで4カラム目にIDがあれば他はどうなっていても問題無し。
                        このランリストに該当するランのみを処理対象とする。
                        指定が無い場合は該当する全ランが対象となる。

```

