### コンテナの作成と起動

1. mysqlのdockerコンテナをビルド＆起動
```
cd docker/mysql
docker compose up -d
```


### コンテナ内にアクセス
docker exec -it **コンテナID** mysql -u root -p 

### コンテナ削除
docker compose down -v


### MySQLコマンド
SHOW DATABASE：データベース一覧を見る
USE データベース名：データベースを選択する
SHOW TABLES：テーブル一覧を見る
DESCRIBE テーブル名：テーブルの構造を見る
EXIT：MySQLコンソールを修了する

### テストデータのインポート

SQLファイルが置いてあるディレクトリにcdコマンドで移動してから以下のコマンドを実行
```
$ mysql -u root -p
パスワード入力
mysql> source [ファイル名].sql  
```