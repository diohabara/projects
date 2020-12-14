# Day05

前回終わらなかったStorage周りの話を再びしていこうと思います｡

## Log-Structured FIle Organization

前回紹介したのは`slotted pages`という構成で､`Page`に`Tuple`を保存していました｡これに対して`log`の記録を取るという`log-structured file organization`があります｡

システムが`log`の記録をファイルのどんどん付け加えていき､それを辿ることでデータの復元が出来ます｡そして､定期的に`log`を小さくする工夫をします｡

今回この構成は実装しないので簡単に触れる程度にします｡

## Data Representation

データを保持するのはRDBMSですが､例えば浮動小数点を"native"なC/C++の型として扱うと誤差の問題があります｡そのためこれを実装してあげる必要があります｡

Day02に型の実装を(適当に)したのですが､これをそのうち改善する必要があるということです｡

また､RDBMSは`Tuple`が`Page`の大きさを超えることを認めていません｡そのため､巨大なデータを扱う際は`Page`を内に`Tuple`の`Attribute`を入れる必要があります｡または､`BLOB`などのように外部ファイル内にデータを入れます｡

このように保存するデータのための実装を追加する必要がありそうですね｡

## System Catalogs

DBMSはその内部カタログにDBに関するメタ情報を持っています｡例えばテーブル､カラム､インデックス権限､内部統計情報などなど...

ということで､これらを実装します｡

## Storage Models

DBには`On-Line Transaction Processing(OLTP)`と`On-Line Analytical Processing(OLAP)`という処理方法があります｡

`OLTP`は小さなデータの読み書きをする高速な処理を毎回行う処理で､`OLAP`は複数のデータを計算するために大きなデータを読み込む複雑な処理です｡

加えてそれらを使った`Hybrid Transaction + Analytical Processing(HTAP)`があります｡

### N-ary Storage Model(NSM)

今回実装するRDBMSはこれらの処理を念頭に入れ､`NSM`というものにします｡これはある`Page`内にある`Tuple`の`Attribute`は全て連続して同`Page`内にあるというモデルです｡ 

#### 利点

利点は高速なファイル処理で全ての`Tuple`を必要とするようなクエリには良いです｡

#### 欠点

欠点は`Table`の大部分を読み込むような際に非効率な点です｡

### (おまけ)Decomposition Storage Model(DSM)

今回は実装しませんが､こういうモデルもあるという話です｡

このモデルではDBMSは全ての`Tuples`のある`Attribute`の値全てを`Page`内に連続して格納します｡このモデルは`OLAP`に適しています｡`Column Store`とも呼ばれるようです｡利点・欠点は`NSM`の反対ですね｡

`Tuple`を識別するには固定長のオフセットを使う方法と`Tuple`のIDを`Attribute`に振るという方法があります｡

## 今日のまとめ

- データ保存のために色々実装する必要がある
- RDBMSのメタ情報を取るための実装も必要
- `NSM`を実装するRDBMSには適用する

## 次回予告

- テスト前なので明日はRDBMS関係ない小話でもします