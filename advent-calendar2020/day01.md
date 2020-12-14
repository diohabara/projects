# Day01

このアドベントカレンダーでは､RDBMS(Relational DataBase Management System)を作ろうと思います｡

実装に使う言語はC++で､ビルドツールとしてBazelを使います｡レポジトリは[こちら](github.com/diohabara/simpleDB)です｡12/25までには完成させます｡

Day1である今日はまずはRDBMSに関係のない開発全般に関する部分について述べます｡次回からはRDBMSに関連する話+実装をしていきます｡

## お手本

RDBMSを個人で実装する上で最も困るのは実装しようにも手本となるような小さなソースコードがほとんど見つからないことでしょう｡作ってみた系の本も知りません｡しかし､お手本がないことはなく､CMU DB groupが[BusTub](https://github.com/cmu-db/bustub/)という教育用のDBを公開しています｡こちらを参考にして書いていこうと思います｡

ちなみにCMU DB groupは講義資料等も公開しており､[Database Systems](https://15445.courses.cs.cmu.edu/fall2020/)という授業を参考にします｡

## 技術選定

お手本があるので最悪完成はするでしょう｡次はRDBMSをどのような技術で作るかという問題です｡お手本はC++で作られています｡

最初は「今どきはRustで作り直すべきかな〜」と思いました｡しかし､C++でなにかをゴリゴリに実装するという経験が個人的に必要になったのでC++で実装する運びとなりました｡お手本もC++なのでちょうど良いでしょう｡

C++を使うということはビルドツールも選ぶ必要があります｡(これまた)ちょうど夏のインターン先で使っていた[Bazel](https://github.com/bazelbuild/bazel)というソフトウェアを個人開発でも使いたいと思っていたので､これを使ってビルドすることにしました｡まあ､CMakeとか今更書きたくないですよね〜

次にどのようにテストをしようかと考えました｡C++を今まで使ったので競技プログラミングくらいなのでまともに`class`もわかりません｡そのため､当然のようにテストフレームワークもわかりませんでした｡しかし､調べてみると[GoogleTest](https://github.com/google/googletest)という良さげなフレームワークがあります｡Googleという名前から来る安心感もあり､こちらを使うことにしました｡

つまり最終的にC++､Bazel､GoogleTestを使って自作RDBMSを作ります｡もちろん他のツールは基本的に使いません｡formatterやlinterなどは別として(そもそもlinterがあるのかさえ知らない)｡

## RDBMSを書く前に

RDBMSを書く前にツールの使い方を確認するために小さなプログラムを書いてみます｡

### Bazel & GoogleTest

Bazelの詳しい使い方は[公式ページ](https://bazel.build/)を読むのが最適です｡ここでは必要最低限だけを説明します｡ 

下のようにBazelで扱うプロジェクトのrootに`WORKSPACE`というファイルを置きます｡そして､C++のファイルを入れるディレクトリ(ここでは`src`と`test`)内に`BUILD.bazel`もしくは`BUILD`というファイルを作ります｡

名前はどちらでも良いですが､個人的には`BUILD`よりは`BUILD.bazel`の方が明示的かつ他のツールと名前が被ることが無いのでおすすめです｡

```shell
.
├── src
│  ├── BUILD.bazel
│  ├── greet.cc
│  └── greet.h
├── test
│  ├── BUILD.bazel
│  └── test_greet.cc
└── WORKSPACE
```

GoogleTestを使うには`WORKSPACE`にその旨を書く必要があります｡

### `WORKSPACE`

以下のように記述することで`gtest`として､GoogleTestのレポジトリをダウンロードし､プロジェクト内で使えるように出来ます｡あとはGoogleTestを使ってテストとコードを書くだけです｡

```starlark
load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")

git_repository(
    name = "gtest",
    branch = "v1.10.x",
    remote = "https://github.com/google/googletest",
)
```

### `src/BUILD.bazel`

`cc_library`を下のように書くことで`greet.h`をヘッダーファイルとする`greet.cc`を`greet`というライブラリとして使えます｡

実行可能ファイルを作りたい場合は`cc_binary`に同じようなコードを書きます｡

簡単ですね？

```starlark
cc_library(
    name = "greet",
    srcs = [
        "greet.cc",
    ],
    hdrs = [
        "greet.h",
    ],
    includes = [
        ".",
    ],
    visibility = [
        "//visibility:public",
    ],
)

```

### `src/greet.{h, cc}`

下のようにヘッダーファイルとソースファイルを書きます｡これで`greet`という関数が完成です｡

```cpp:greet.h
#include <string>

std::string greet(std::string name);
```

```cpp:greet.cc
#include <greet.h>

std::string greet(std::string name) { return "Hello " + name; }
```

### `test/BUILD.bazel`

次に`test`側のコードを書きます｡

ここではテストの名前を`unit_test`として､テスト用のソースファイルを`test_greet.cc`としています｡ また､C++のコンパイルオプションとして`-std=c++17`を使います｡

dependencyにさっき作った`greet`ライブラリを入れ､`gtest`と名付けたGoogleTestを使っています｡

```starlark
cc_test(
    name = "unit_test",
    srcs = [
        "test_greet.cc",
    ],
    copts = [
        "-std=c++17",
    ],
    deps = [
        "//:greet",
        "@gtest",
        "@gtest//:gtest_main",
    ],
)

```

### `test/test_greet.cc`

そして､このようにテストファイルを書きます｡これで`bazel test //test/...`とすると､`test`ディレクトリ内にあるテスト用のコードがすべて実行されます｡

```cpp
#include "greet.h"
#include "gtest/gtest.h"

TEST(Hello, HelloBazel) { EXPECT_EQ(greet("Bazel"), "Hello Bazel"); }
```

非常に簡単ですね？

実装ではこのような単純なbinary targetとbinary target用のlibraryとテストを書いていきます｡そうすることでCMakeで書くような複雑な依存関係を自分の頭に入れなくても良く､実装の見通しも簡単になります｡C++は思ったより近代的なのだなと感心します｡RustだとCargoだけで済みますが､このような利点も他の言語に触れることが感じられます｡常に先人の知恵に感謝ですね｡

## Day01まとめ

- RDBMSを自作する
- [BusTub](https://github.com/cmu-db/bustub/)というお手本がある
- C++/Bazel/GoogleTestを使う
- 開発方法を速習した
