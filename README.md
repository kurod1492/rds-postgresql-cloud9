# RDS 上に PostgreSQL インスタンスを作成し、Cloud9 から接続する

## 概要

RDS 上に PostgreSQL の データベース(DB)インスタンスを作成します。
Cloud9 から作成した DB インスタンスに接続できるように設定します。
PostgreSQL のバージョンは 11 を使用します。

## RDS セットアップ

https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html

セキュリティグループを作成します。
AWS サービスの VPC にアクセスします。
左のメニューの「セキュリティグループ」をクリックします。
「セキュリティグループを作成」をクリックします。

セキュリティグループ名
  MyDBSecurityGroup
説明
  DB access
  
同じ画面で「インバウンドルール」の「ルールを追加」をクリックします。

タイプ
  PostgreSQL
ソース
  虫めがねをクリックして Cloud9 のセキュリティグループをクリックします。

「セキュリティグループを作成」をクリックします。

## RDS でDB インスタンスの作成

Amazon RDS の画面を表示します。
「データベースの作成」をクリックします。

データベースの作成方法を選択
  簡易作成
設定
  エンジンのタイプ
    PostgreSQL
  DBインスタンスサイズ
    無料利用枠
  DBインスタンス識別子
    database-1 (デフォルトのまま)
  マスターユーザー名
    postgres (デフォルトのまま)
  「パスワードの自動生成」にチェック

「データベースの作成」ボタンをクリックします。

DB インスタンス一覧画面に遷移します。
「認証情報の詳細を表示」ボタンをクリックします。
マスターユーザー名とマスターパスワードが表示されるのでコピペして保存しておきます。

DB インスタンスの作成には時間がかかることがあります。
RDS の画面はこのままにしておいて、待っている間に Cloud9 のセットアップを行います。

## Cloud9 に PostgreSQL 11 の psql コマンドをセットアップする

Cloud9 上で  PostgreSQL 11 の psql コマンドが実行できるようにセットアップします。
postgresql.org に Ubuntu 用のパッケージが用意されているので利用します。
postgresql-11 というパッケージをインストールします。

参考
[Apt \- PostgreSQL wiki](https://wiki.postgresql.org/wiki/Apt)

```
sudo apt install curl ca-certificates gnupg
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt install postgresql-11
```

psql が実行できることを以下のコマンドを実行して確認します。

```
sudo -u postgres psql
```

以下のような画面出力になればうまくいっています。

```
$ sudo -u postgres psql
psql (11.8 (Ubuntu 11.8-1.pgdg18.04+1))
Type "help" for help.

postgres=# 
```

```postgres=# ``` のプロンプトで ```\q``` と入力しエンターを押して終了します。

## DB インスタンスの設定

RDS の画面に戻り、DB識別子が「database-1」のDB インスタンスのステータスが「利用可能」になっていたら、DB インスタンスの作成が完了しています。
DB識別子欄の「database-1」をクリックして詳細情報を表示します。
エンドポイントとポートを確認します。

エンドポイント
  database-1.hogehoge.us-east-1.rds.amazonaws.com
ポート
  5432

セキュリティグループの設定を追加します。
「database-1」を選択した状態で「変更」をクリックします。
「ネットワーク & セキュリティ」の「セキュリティグループ」で「MyDBSecurityGroup」を追加します。
次へをクリックします。
「変更のスケジュール」で「すぐに適用」を選択して「DBインスタンスの変更」をクリックします。
これで設定は完了です。

## Cloud9 上の psql コマンドで RDS に接続確認する

Cloud9 上の psql コマンドを実行して、RDS 上の DB インスタンスに接続してみます。

psql --host=DB_instance_endpoint --port=port --username=master_user_name --password --dbname=database_name

 --host=エンドポイント
 --port=ポート
 --username=マスターユーザー名
 --dbname=データベース名

ここで「データベース名」とは、DB インスタンンスの中に作られる実際のデータベースのことです。
まだ作成していないので、```--dbname`` は指定しないで接続してみます。
以下のコマンドを実行します。

```
psql --host=database-1.hogehoge.us-east-1.rds.amazonaws.com --port=5432 --username=postgres --password
```

「Password: 」と表示されますのでパスワードを入力してエンターを押します。入力内容は画面に表示されないことに注意してください。
以下のような画面が表示されたらうまくいっています。

```
psql (11.8 (Ubuntu 11.8-1.pgdg18.04+1), server 11.6)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=> 
```

```postgres=# ``` のところに ```\q``` と入力しエンターを押して終了します。

