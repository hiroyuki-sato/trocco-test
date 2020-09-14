
## この文書について

この文書は、Embulkのマネージドサービス[trocco](https://trocco.io/lp/)をつかってみた体験記です。

## そもそものきっかけ

私は、Embulkがリリースされた2015年1月から、Embulkのまとめを作ってその動向をみてきました。そういう中でプライムナンバーの小林様と知り合いました。Embulkのマネージドサービスということで、troccoには前々から興味があったのですが、小林様の好意でtroccoを使わせていただくことができました。

## troccoとは

Embulk + αのマネージドサービスが利用できるプライムナンバー社のサービスです。

## Embulkとは

2015年にfluendやなどのサービスを開発したトレジャーデータ社の古橋さんが開発したオープンソースソフトウェアです。詳しくは私がメンテナンスをしている[Fluentdのバッチ版Embulk(エンバルク)のまとめ
](http://qiita.com/hiroysato/items/397f36c4838a0a93e352) をみてください。

## 対象読者

なるべくこの資料をみると「troccoのイメージが掴めた」と思えるような文書にしたいと思っています。以下のような方にご覧いただけるページにしているつもりです。

* これからデータ基盤を作る予定だが、サーバの管理とかの経験がなくマネージドサービスを検討しているかた
* troccoに興味のあるかた
* Embulkを使ったことある人

## troccoでデータを登録するまでのステップ

troccoを利用する前に次のことを実行します。

* 連携サービスのアカウント作成: ex) Google Bigqueryのアカウントを取得
  * BigQueryにはサンドボックスという無料で利用できるサービスがありますが、このサービスをtroccoで利用するとエラーになってしまうようです。(これはbigqueryのプラグインの問題です。)
* [無料トライアル](https://trocco.io/inquiry_trial/new)を申し込み

## troccoのアカウントができたら

* 連携サービスの認証情報の登録(今回割愛)
* 転送ジョブの登録

## 実際のデータ投入

#### 転送元・転送先の設定

自分のデスクトップにあるファイルを、Bigqueryにアップロードするのをやってみます。サンプルに利用するファイルはEmbulkで例として使っているcsvファイルを利用します。このファイルはembulk exampleコマンドで生成することができます。また[こちら](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/samples/001_embulk_load/sample_01.csv.gz)でも同じファイルをダウンロードすることができますので、デスクトップに保存をしてください。

* 入力元: 様々なサービスが出てくるのでLocalFileを選択します。
* 出力先: bigqueryを選択します。
* 「この内容で作成」を押し具体的な設定を行います。

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/001.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/002.png)

### 概要設定

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/003.png)

* 概要設定
  * 名前: この転送の名前を指定します。(初めてのtrocco)
  * メモ: この転送の説明を記述することができます。
  * 共有するグループ: グループを使っているとこの転送情報を設定するようです。(今回はスキップ)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/004.png)


* 転送元ローカルアップロードの設定
  * ファイル: アップロードするファイルを指定します。
  * 入力ファイル形式: `CSV`, `JSON`などから
  * ファイル名には変数を利用することができるようです。日時で生成される売り上げファイルなどはこちらの変数を使うことができるようです(これは後日使ってみたいと思います。)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/005.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/006.png)

* Bigqueryの接続設定
  * Bigqueryの接続情報: 事前に作成した接続情報が一覧表示されるので、利用する接続設定を選択します。
  * データセット: [BigQuery](https://cloud.google.com/bigquery/docs/datasets-intro?hl=ja)で指定するデータセットの名前を指定します。選択した接続情報で利用可能なデータセットが一覧表示されます。まだBigQueryにデータセットがない場合は、作成したいデータセットの名前を入力します。**これをしないとエラーになります**
  * テーブル: アップロードするデータを登録するテーブル名を指定します。このテーブルも既存のテーブルは一覧表示されて選択をすることができます。データ登録時にテーブルを一緒に作成する場合は、作成するテーブル名を入力します。
  * データセットのリージョン: テーブルを登録する先のリージョンを指定します。
  * データセットの自動作成オプション: データセットが未作成の場合は、datasetを自動で作成するを選択します。**重要**
  * 転送モード: データを追加登録する場合は、append、既存のデータを入れ替えする場合: replaceを選択します
* 保存して自動設定・プレビューへボタンを押します。

### データプレビュー・詳細設定

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/007.png)

* 前画面で指定したファイルの中身を類推して、カラム情報を自動作成します。(embulk guessを実行します。)
  * 実行した結果はマネージドサービスらしく、Webインタフェースでみやすくなっています。
* プレビュー画面のしたには、次のタブが表示されており入出力データ・ジョブの設定ができるようになっているようです
  * データ設定
  * 入力オプション(LocalFileの設定をおこないます)
  * 出力オプション(BigQueryの設定をおこないます。)
  * ジョブ設定(並列度などの設定をおこなうことができまる)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/007.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/007.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/007.png)

* カラム定義
  * もし自動類推されたカラム情報が期待するカラムと異なる場合、カラムの情報を指定します。
  * カラム名: 指定したカラム名がBigQueryに登録した際のカラム名になります。
  * 元カラム名: csvファイルの先頭行に記載されてるカラム名が表示されます。
  * データ型: 利用するデータ型を一覧から選択します
  * 日付フォーマット: 入力データが例えば2020年8月30日 14時3分05秒で、2020-08-30 14:03:05のようになっている場合、`%Y-%m-%d %H:%M:%S`のように記述します。
  * カラム追加:

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/008.png)

* フィルター設定
  * 行の絞り込みを行えるようです。(今回は未設定)
  * 画面はembulk-filter-rowを使って、条件式に合致するデータのみ抽出することができるようです。
  * 例えば、2020年度9月のデータのみ抽出等
  * その他のフィルタも利用できるようです。

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/009.png)

* マスキング設定
  * メールアドレスを `bob@example.com`を`***@****.com`のようにマスクできるようです。

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/010.png)

* 転送時刻カラム設定
  * データを登録した時刻を記録するカラムを追加できるようです。
  * 多分`embulk-filter-add_time`を利用しているようです

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/011.png)

* 文字列置き換え
  * 正規表現を使って文字列の置き換えをすることができるようです。

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/012.png)

* カラム暗号化
  * 設定したカラムの文字列を、SHA-256でハッシュ化できるようです。

### データプレビュー・入力オプション

* LocalFileで指定したCSVファイル区切り文字などが指定できるようです。(今回はデフォルトのまま)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/012.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/013.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/014.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/015.png)

### データプレビュー・出力オプション

* 転送先のBigQueryの設定を指定できるようです。(今回はデフォルト)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/016.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/017.png)

### データプレビュー・ジョブ設定

* 転送時の並列数を指定して転送速度を速くしたり、エラーが発生した際に再実行する回数を指定できるようです(今回はデフォルト)

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/018.png)


### データプレビュー・ジョブ設定

* 保存して実行を完了するといよいよデータを実行の段階になるようです。
* ジョブの実行は次の二つがあるようです。
  * 現在時刻で展開: 今すぐ登録したジョブを実行する
  * 指定時刻で展開: 日時を指定してジョブを実行
* 今回は、現在時刻で実行を選択します。設定を終えたらジョブを実行ボタンを押します

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/018.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/020.png)
![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/021.png)

* ステータスが実行完了になりました

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/022.png)

### 登録データの確認

* BigQueryのコンソールで、データが登録されているか確認をすると、指定したデータが登録されていることが確認できました。

![](https://raw.githubusercontent.com/hiroyuki-sato/trocco-test/master/images/001_embulk_load/023.png)

## 感想

* Embulkの環境を用意することもなく、ファイルをアップロードして、GUIで設定をするだけで簡単にBigQueryにデータを登録することができました。
* 様々な入出力先に対応しているので、同様の操作で他社のサービスにも簡単にデータ登録をすることができそうです。
* Embulk本体にはない、時刻指定のアップロードや、再実行(プラグインのリトライではなく)機能も搭載されておいいるようで、ネットワークのトラブルなどにも強いサービスのようです。

## 最後に

* 今回は、embulkのサンプルデータをtroccoに登録してみました。
* 次回は、もう少しちゃんとしたデータを登録していろいろ確認をしてみたいと思います。
