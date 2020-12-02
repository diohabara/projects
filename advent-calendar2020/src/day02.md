# Day02

[前回](./day01.md)は開発全般に関する話だったので､今回はRDBMS全般に関する話をします｡そのため､実装はないです｡まあ､ないものは書けないです｡

## Why to choose RDBMS

そもそもなぜRDBMSにしたのかと言うとそもそも自分はデータに興味があり､データを格納し､引き出すというデータベスに興味があったからです｡Relational Database Management Systemには特別な興味があるというわけではないのです､その中で最も有名なものが一番**カンタン**にできそうだったので選択しました｡

調べると全然簡単そうではないですけどね｡

## How to make RDBMS

では､どうやってRDBMSを作っていきましょう｡ちょっとその前にRDBMSがどのようなものかをサラっと確認していきます｡

### Table

以下のように`tuple`と呼ばれる属性の集合があります｡`("Sato" Taro, 1999, "Japan")`みたいなやつです｡これらはそれぞれ`(name, year, country)`に対応しています｡このような`tuple`が集まったものがテーブルというのを作ります｡例えば下は人のテーブルと考えて良いでしょう｡

| name          | year | country  |
| ------------- | ---- | -------- |
| Sato Taro     | 1999 | Japan    |
| JC            | 1954 | HongKong |
| J. J. Sakurai | 1933 | U.S.     |

そして､これらのテーブルには`tuple`を識別するために一意定まるキーが本来付いています｡以下はそのキーを追加したものです｡

| id      | name          | year | country  |
| ------- | ------------- | ---- | -------- |
| 114514  | Sato Taro     | 1999 | Japan    |
| 1198964 | JC            | 1954 | HongKong |
| 1975430 | J. J. Sakurai | 1933 | U.S.     |

このidを外部のテーブルから参照することができ､これによってテーブル同士を関連させられます｡これが"Relational"の由来です｡

### SQL

では､どうやってテーブルを操作するのでしょう？実は2つの流派(？)があります｡

- Procedural
  DBMSがデータを取り出す手順まで含めて指示する流派(Relational Algebra)
- Non-Procedural
  DBMSに欲しいデータを要求するだけで手順までは指示しない流派(Relational Calculus)

私の流派(私のDBMSの流派？)はProceduralです｡

そして､この手順を記述するのに使われているのがSQLです｡"Structured Query Language"の略です｡"Structured"とは要は上のように`id`などで関連付けられたデータのことを指します｡

### Architecture

RDBMSからデータをどのように取ってくるのかを考えると以下のようになります｡ただし､上がデータを取り出す側､下が保存される側｡

- Query Planning
- Operator Execution
- Access Methods
- Buffer Pool Manager
- Disk Manager

DBMSは基本的には揮発性メモリ(RAM)上のデータを不揮発性メモリ(Storage)に入れるdisk-oriented architectureを採用しており上のような構造になっています｡

下から紹介すると､"Disk ManagerはそのStorageとの接点となる部分です｡OSがやれよと思うかもしれませんが､複数のスレッドで単一のファイルを扱おうと思った際にOSの機能だけでは不足するのでDBMSでどうにかしたくなったりもするのでDBMSでまとめて上手くやります｡

"Buffer Pool Manager"はメモリ上でデータを扱うための仕組みです｡

"Access Methods"は文字通りデータに辿り着く仕組みです｡

"Operator Execution"はSQLを処理する仕組みです｡

"Query Planning"はどうデータを取得するかで人間が考える部分ですね｡

## Day02まとめ

- なぜRDBMSを自作するかを把握した
- RDBMSの概要を把握した

## 次回予告

- MUST: RDBMSで使う型を定義する
- SHOULD: Storageに関する実装をする