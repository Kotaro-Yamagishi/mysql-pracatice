# CPUベンチマーク

### コマンド

```sysbench --test=cpu --cpu-max-prime=20000 run```

--cpu-max-prime=20000は素数計算を複数スレッドで同時に実行することで、CPU負荷体制のチェックをしている

### 出力結果例

#### 良い例

```
Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:(一秒間に難解処理できるか)
    events per second: 15722015.72

General statistics:（10秒間で何回処理できたか）
    total time:                          10.0001s
    total number of events:              157224072

Latency (ms):
         min:                                    0.00
         avg:                                    0.00
         max:                                    1.49
         95th percentile:                        0.00（95%が0.00秒で処理完了）
         sum:                                 2681.82

Threads fairness:
    events (avg/stddev):           157224072.0000/0.00（avg:各スレッドが平均して処理したイベント数、stddev:スレッド間の処理料のばらつき）
    execution time (avg/stddev):   2.6818/0.00

```

今回は標準偏差が0.00で各スレッドが完全に同じ量の処理を同じ時間で実行できていることがわかる

events:sysbench:実行した処理の回数

execution time:そのイベントを処理するのに要した時間

#### 悪い例

```
General statistics:
    total time:                          300.0003s
    total number of events:              23954569819

Latency (ms):
         min:                                    0.00
         avg:                                    0.00
         max:                       18446744073709.55(64bit整数のオーバーフロー。異常値)
         95th percentile:                        0.00
         sum:                               776148.55

Threads fairness:
    events (avg/stddev):           2994321227.3750/3857167.64
    execution time (avg/stddev):   97.0186/0.26
```

標準偏差：約386万回のばらつき

実行時間：約0.26秒のばらつき

### ばらつきがあった場合の対処法

- ばらつきの原因
  - ハードウェア要因
    - CPUコアの性能差
    - メモリ帯域の制限
    - キャッシュサイズの違い
  - ソフトウェア要因
    - OSのスケジューリング
    - 他のプロセスの干渉
    - メモリ割り当ての不均等
- システム設計への影響
  - スケーラビリティ
    - スレッド数を増やしても効果が限定的
  - リソース計画
    - 過大なリソース確保が必要
- 対処法
  - スレッド数の調整
  - プロセス優先度の設定
  - CPUアフィニティの設定
  - 不要なプロセスの終了
- ばらつきの意味と影響
  - イベント数のばらつき
    - 1000万回以上は悪い
    - 効率的なCPU使用かCPUリソースの無駄遣い
      - 一部のコアが遊んでる
      - 処理時間が不安定
  - 実行時間のばらつき
    - １秒以上は悪い
    - 予測可能な性能か全体の処理時間が長くなるか

### 用語

- TPS(transactions per second)
  一秒間に処理できるトランザクション
- QPS(Queries per second)
  一秒間に処理できるクエリ数
- レイテンシー
  レスポンス時間
- エラー率
  接続セラーやタイムアウトの発生率

### ユースケース

- サーバーリソースの適正化
- スケールアップ、スケールアウト判断
- コスト最適化

# ファイルI/Oベンチマーク

### コマンド

準備

```
sysbench fileio --file-test-mode=seqwr --file-total-size=1G --file-num=4 prepare
```

#### 締め

```
sysbench --test=fileio --file-total-size=150G cleanup  
```

### 順次書き込みテスト

```
sysbench fileio --file-test-mode=seqwr --file-total-size=1G --file-num=4 --time=30 --threads=4 run
```

結果

```
General statistics:
    total time:                          30.0033s
    total number of events:              1067767

Latency (ms):
         min:                                    0.00
         avg:                                    0.11
         max:                                 1189.56
         95th percentile:                        0.00
         sum:                               119907.59

Threads fairness:
    events (avg/stddev):           266941.7500/2048.27
    execution time (avg/stddev):   29.9769/0.00
```

### 順次読み込みテスト

```
sysbench fileio --file-test-mode=seqrd --file-total-size=1G --file-num=4 --time=30 --threads=4 run
```

結果

```
General statistics:
    total time:                          30.0003s
    total number of events:              1491824

Latency (ms):
         min:                                    0.00
         avg:                                    0.08
         max:                                   10.87
         95th percentile:                        0.00
         sum:                               119806.35

Threads fairness:
    events (avg/stddev):           372956.0000/250.18
    execution time (avg/stddev):   29.9516/0.00
```

### ランダム読み書きテスト

```
sysbench fileio --file-test-mode=rndrw --file-total-size=1G --file-num=4 --time=30 --threads=4 run
```

結果

```
General statistics:
    total time:                          30.0037s
    total number of events:              2174120

Latency (ms):
         min:                                    0.00
         avg:                                    0.06
         max:                                  574.93
         95th percentile:                        0.00
         sum:                               119745.91

Threads fairness:
    events (avg/stddev):           543530.0000/3940.84
    execution time (avg/stddev):   29.9365/0.00
```

### スループットと秒間の処理量の計算

#### 例）順次読み込みテストの計算

基本情報として

ブロックサイズ：16KB

実行時間：30秒（コマンドにて指定）

総イベント数：1491824回（テスト結果）

reads/sの計算

`reads/s = 1,491,824 ÷ 30 = 49,726.36回/秒`

スループットの計算

```
1回の読み込み = 16KB
1秒間の読み込み = 49,726.36 × 16KB = 795,621.76KB
MiB/s = 795,621.76KB ÷ 1024 = 776.97 MiB/s(1KB=1024bype)
```

### 性能評価ポイント

- 順次読み込み
  - 500+ MiB/s
- 順次書き込み
  - 400+ MiB/s
- ランダムアクセス
  - 300+ MiB/s
- レイテンシー
  - 1ms以下

### スループットとばらつき

スループットとは

大量データの高速配信

- 処理速度
- データ転送量
- 処理能力
- 効率性

スループット重視の場合

- 大量データの読み込み/書き込み
- バッチ処理
- データ移行
- 静的ファイル配信
- 画像・動画配信
- CDN用途
- ビルド処理
- テスト実行
- デブロイメント

ばらつき

データの整合性

- 処理の一貫性
- 応答時間の安定
- 予測可能な性能
- データの信頼性

ばらつき重視の場合

- オンライントランザクション
- リアルタイム処理
- ユーザー体験が重要なアプリ
- 動的コンテンツ生成
- APIサーバー
- ユーザーセッション管理

### スループットのみ確認

```

sysbench fileio --file-test-mode=seqrd --file-total-size=1G --file-num=4 --time=30 run | grep "read, MiB/s"

 read, MiB/s:                  10663.04
```

### ばらつきのみ確認

```
sysbench fileio --file-test-mode=rndrw --file-total-size=1G --file-num=4 --time=60 run | grep -A 5 "Threads fairness"

hreads fairness:
    events (avg/stddev):           3559917.0000/0.00
    execution time (avg/stddev):   59.7398/0.00
```

ばらつきは0.00

# OLTPベンチマーク

### 準備

```
sysbench --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=user --mysql-password=passw0rd --mysql-db=sakila --table-size=10000 --tables=10 --threads=1 oltp_common prepare
```

### OLTP読み書きテスト

```
sysbench --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=user --mysql-password=passw0rd --mysql-db=sakila --table-size=10000 --tables=10 --threads=4 --time=60 --report-interval=10 oltp_read_write run
```

結果

```
SQL statistics:
    queries performed:
        read:                            461342
        write:                           131812
        other:                           65906
        total:                           659060
    transactions:                        32953  (549.07 per sec.)（総トランザクション数と一秒間に処理した数）
    queries:                             659060 (10981.49 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)


General statistics:
    total time:                          60.0154s
    total number of events:              32953

Latency (ms):
         min:                                    4.48
         avg:                                    7.28
         max:                                  120.83
         95th percentile:                        0.00
         sum:                               239991.20

Threads fairness:
    events (avg/stddev):           8238.2500/12.93
    execution time (avg/stddev):   59.9978/0.00
```

### OLTP読み取り専用テスト

```
sysbench --db-driver=mysql --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-user=user --mysql-password=passw0rd --mysql-db=sakila --table-size=10000 --tables=10 --threads=4 --time=30 oltp_read_only run
```

結果

```
SQL statistics:
    queries performed:
        read:                            402570
        write:                           0
        other:                           57510
        total:                           460080
    transactions:                        28755  (958.20 per sec.)
    queries:                             460080 (15331.15 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          30.0093s
    total number of events:              28755

Latency (ms):
         min:                                    2.88
         avg:                                    4.17
         max:                                   51.83
         95th percentile:                        0.00
         sum:                               119997.13

Threads fairness:
    events (avg/stddev):           7188.7500/4.32
    execution time (avg/stddev):   29.9993/0.00
```

### 評価基準

#### TPS(Transactions Per Second)

優秀: 500+ TPS
良好: 200-500 TPS
一般的: 50-200 TPS

#### QPS（Query Per Second）

優秀: 10,000+ QPS
良好: 5,000-10,000 QPS
一般的: 1,000-5,000 QPS

#### レイテンシー

優秀: avg < 10ms
良好: avg 10-50ms
一般的: avg 50-100msLatency (ms):

### 結果が悪い時の対処法

* TPS
  * MySQL設定の最適化
    * バッファプールサイズ
    * ログファイルサイズ
      * ログファイルを大きくすると、ログファイルの切り替えも少なく、ディスクIOが減少する
    * 同時接続数
      * メモリ使用量の制御が可能
    * クエリキャッシュ
  * インデックスの最適化
  * ハードウェアリソースの確認
* レイテンシー
  * クエリの最適化
  * 接続プールの設定
    * 接続プールを作ることで接続の再利用が可能でオーバーヘッドが少ない
  * ネットワーク遅延の確認
* ばらつき
  * スレッド数の調整
    * 過剰なスレッド数は競合が発生し、オーバーヘッドに繋がるが、少なすぎると処理をするのに時間かかる
  * システム負荷の軽減
  * CPUアフィニティの設定
    * プロセスを特定のコアに固定すること
      * プロセスのコンテキストスイッチが減る
    * キャッシュ効率がよくなり、キャッシュヒット率が向上
    * メモリアクセスパターンが同じ
* エラー率が高い
  * 接続設定の調整
  * メモリ設定の最適化

#### 補足

#### キャッシュの階層構造

CPUコア1: L1キャッシュ + L2キャッシュ
CPUコア2: L1キャッシュ + L2キャッシュ
CPUコア3: L1キャッシュ + L2キャッシュ
CPUコア4: L1キャッシュ + L2キャッシュ

全CPUコアで共有: L3キャッシュ

#### キャッシュの種類

* L1キャッシュ: 32KB（CPUコアごと）

  * 容量: 32KB（データ用）+ 32KB（命令用）
    速度: 1-2サイクル（最速）
    場所: CPUコア内（最も近い）
    用途: 最も頻繁にアクセスされるデータ
    特徴: 各CPUコア専用
  * 実際の動作例

    * クエリプラン
    * where句の条件
    * 結果セットの一部
  * 最適化のポイント

    - 小さなデータ構造を使用
    - ループの最適化
    - 局所性の向上
* L2キャッシュ: 256KB（CPUコアごと）

  * 容量: 256KB - 1MB
    速度: 10-20サイクル
    場所: CPUコア内（L1の外側）
    用途: L1に収まらないデータ
    特徴: 各CPUコア専用
  * 実際の動作例
    * テーブルのインデックス
    * 結果セットの大部分
    * メタデータ
  * 最適化のポイント
    * データ構造のサイズ調整
    * アクセスパターンの最適化
    * メモリレイアウトの改善
* L3キャッシュ: 8MB（CPU間で共有）

  * 容量: 8MB - 32MB
    速度: 40-80サイクル
    場所: CPUダイ内（全コアで共有）
    用途: 大きなデータセット
    特徴: 全CPUコアで共有
  * 実際の動作例
    * テーブルのデータ
    * 大きな結果セット
    * 共有データ
  * 最適化のポイント
    - 大きなデータセットの分割
    - 並列処理の活用
    - メモリ使用量の削減
* メインメモリ: 16GB（システム全体）

#### CPUの構造

CPUチップ
├── 制御ユニット（命令の解読・実行制御）
├── 演算ユニット（ALU: 計算処理）
├── レジスタ（高速な記憶装置）
├── キャッシュメモリ（L1、L2、L3）
└── バスインターフェース（外部との通信）
