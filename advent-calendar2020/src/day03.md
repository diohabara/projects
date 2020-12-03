# Day03

今回は自作RDBMSで使う型について考えていこうと思います｡

## データと型

RDBMSが扱っているデータは､当然コンピュータが扱っているので型があります｡

[Orableの公式ドキュメント](https://docs.oracle.com/cd/E13214_01/wli/docs92/rdbmseg/datatypes.html#wp1066348)によるとOracleは以下のようなデータを扱っているようです｡

| Oracle                                                     |
| ---------------------------------------------------------- |
| BLOB: 可変長バイナリ                                       |
| CHAR: 固定長の文字                                         |
| CLOB: 任意長の巨大文字列(フィールド定義に用いられるらしい) |
| DATE: 日付                                                 |
| DECIMAL: 10進数                                            |
| FLOAT: 浮動小数                                            |
| INTEGER: 整数                                              |
| NUMBER: 数                                                 |
| NUMERIC: 大きな整数                                        |
| RAW: 可変長バイナリ                                        |
| REAL: 有理数                                               |
| SMALLINT: 小さな整数                                       |
| NCLOB: CLOBのUnicode版                                     |
| NVARCHAR2: (Unicode)可変長の文字                           |
| NCHAR: (Unicode)固定長の文字                               |

色々ありますね｡初心者なので量減らしてくれないかなと思っているんですけど､ダメですかそもそもこれを全部自分で実装するのも面倒そうです｡

今回はこのうち重要そうなものに絞って(本当は全部重要なのだけど自分の実装が簡単にできるように)実装します｡

## RDBMS命名

その前に...RDBMSの名前をまだ決めていなかったのでその名の通り"simpleDB"という名前にします｡理由は大したものなので名前を付けるのも恥ずかしいからです｡

では､また型について考えていきます｡

## simpleDB内部の型

こういうときは参考にすると言っていたBusTubの型の実装を見るのが一番です｡

- https://github.com/cmu-db/bustub/tree/master/src/include/type
- https://github.com/cmu-db/bustub/tree/master/src/type

これを見た所最悪

- BOOLEAN
- INTEGER
- STRING
- TIMESTAMP

があれば大丈夫そうだな？と思ったのでこちらを実装していこうと思います｡

## simpleDBの構成

simpleDBは以下のような構成となっています

`Bazel`関連のディレクトリや､使い方を確認したために適当に作ったファイルが入っていますが､こんな感じになっています｡

```sh
.
├── bazel-bin -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out/darwin-fastbuild/bin
├── bazel-out -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out
├── bazel-simpleDB -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__
├── bazel-testlogs -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out/darwin-fastbuild/testlogs
├── bin
│  ├── BUILD.bazel
│  └── format.sh
├── lib
│  ├── BUILD.bazel
│  ├── greet.cc
│  └── greet.h
├── README.md
├── src
│  ├── BUILD.bazel
│  ├── includes
│  │  ├── add.h
│  │  └── BUILD.bazel
│  └── main.cc
├── test
│  ├── BUILD.bazel
│  └── test_greet.cc
└── WORKSPACE
```

`src`､`src/includes` 以下に`type`ディレクトリを作って､そこに必要な型を書いていきます｡

```sh
simpleDB/src on  type 
❯ mkdir includes/type type

simpleDB/src on  type
❯ touch includes/type/{type_id.h,type.h,boolean_type.h,integer_type.h,string_type.h,timestamp_type.h}

simpleDB/src on  type
❯ touch type/{type.cc,boolean_type.cc,integer_type.cc,string_type.cc,timestamp_type.cc}
```

`type_id.h`は型を列挙したものです｡`type.{h,cc}`は基底クラス､型全般に関するファイルです｡

そんな感じで色々作って最低限の部分だけ書きます｡必要になったら書き足せば良いのでまあ良いでしょう｡

いらないファイルを削って最終的にこんな感じになります｡

```sh
.
├── bazel-bin -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out/darwin-fastbuild/bin
├── bazel-out -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out
├── bazel-simpleDB -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__
├── bazel-testlogs -> /private/var/tmp/_bazel_jio/f5658d19d2bd7d6d5e686a125c69dd0f/execroot/__main__/bazel-out/darwin-fastbuild/testlogs
├── bin
│  ├── BUILD.bazel
│  └── format.sh
├── README.md
├── src
│  ├── includes
│  │  └── type
│  │     ├── boolean_type.h
│  │     ├── BUILD.bazel
│  │     ├── integer_type.h
│  │     ├── string_type.h
│  │     ├── timestamp_type.h
│  │     ├── type.h
│  │     └── type_id.h
│  └── type
│     ├── boolean_type.cc
│     ├── BUILD.bazel
│     ├── integer_type.cc
│     ├── string_type.cc
│     ├── timestamp_type.cc
│     └── type.cc
├── test
│  └── test_greet.cc
└── WORKSPACE

```

`test`ディレクトリに何も書き込んでいないので､ファイルは残しています(すぐ`Bazel`の使い方忘れるので)

実際のファイルは下のような感じ｡

### `src/includes/type/BUILD.bazel`

```starlark
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "boolean",
    hdrs = ["boolean_type.h"],
    visibility = ["//src:__subpackages__"],
    deps = ["//src/includes/type"],
)

cc_library(
    name = "integer",
    hdrs = ["integer_type.h"],
    visibility = ["//src:__subpackages__"],
    deps = ["//src/includes/type"],
)

cc_library(
    name = "string",
    hdrs = ["string_type.h"],
    visibility = ["//src:__subpackages__"],
    deps = ["//src/includes/type"],
)

cc_library(
    name = "timestamp",
    hdrs = ["timestamp_type.h"],
    visibility = ["//src:__subpackages__"],
    deps = ["//src/includes/type"],
)

cc_library(
    name = "type_id",
    hdrs = ["type_id.h"],
    visibility = ["//src:__subpackages__"],
)

cc_library(
    name = "type",
    hdrs = ["type.h"],
    visibility = ["//src:__subpackages__"],
    deps = ["//src/includes/type:type_id"],
)

```

### `src/includes/type/type.h`

```cpp
#include <cstdint>

#include "src/includes/type/type_id.h"

namespace simple_db {
class Type {
 public:
  explicit Type(TypeId type_id) : type_id_(type_id) {}

  virtual ~Type() = default;

  inline TypeId GetTypeId() const { return type_id_; }

 protected:
  TypeId type_id_;
  static Type *k_types[14];
};
}  // namespace simple_db
```

### `src/includes/type/type_id.h`

```cpp
namespace simple_db {
enum TypeId { INVALID = 0, BOOLEAN, INTEGER, STRING, TIMESTAMP };
}
```

### `src/includes/type/boolean_type.h`

```cpp
#include "src/includes/type/type.h"

namespace simple_db {
class BooleanType : public Type {
 public:
  ~BooleanType() override = default;
  BooleanType();
};
}  // namespace simple_db
```

こんな感じにヘッダーファイルは書きました(本当に最低限)｡

次にソースファイルです｡こちらも最小限に留めました｡

### `src/type/BUILD.bazel`

```starlark
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "boolean",
    srcs = [
        "boolean_type.cc",
    ],
    copts = [
        "-std=c++17",
    ],
    deps = [
        "//src/includes/type:boolean",
    ],
)

cc_library(
    name = "integer",
    srcs = [
        "integer_type.cc",
    ],
    copts = [
        "-std=c++17",
    ],
    deps = [
        "//src/includes/type:integer",
    ],
)

cc_library(
    name = "string",
    srcs = [
        "string_type.cc",
    ],
    copts = [
        "-std=c++17",
    ],
    deps = [
        "//src/includes/type:string",
    ],
)

cc_library(
    name = "timestamp",
    srcs = [
        "timestamp_type.cc",
    ],
    copts = [
        "-std=c++17",
    ],
    deps = [
        "//src/includes/type:timestamp",
    ],
)

```

### `src/type/type.cc`

```cpp
#include "src/includes/type/boolean_type.h"
#include "src/includes/type/integer_type.h"
#include "src/includes/type/string_type.h"
#include "src/includes/type/timestamp_type.h"

namespace simple_db {
Type *Type::k_types[] = {
    new Type(TypeId::INVALID), new BooleanType(),   new IntegerType(),
    new StringType(),          new TimestampType(),
};
}
```

### `src/type/boolean_type.cc`

```cpp
#include "src/includes/type/boolean_type.h"

namespace simple_db {
BooleanType::BooleanType() : Type(TypeId::BOOLEAN) {}
}  // namespace simple_db
```

このように基本的な型の実装をしました｡おそらく実装を進めるうちにこんなに適当ではまずくなると思いますが､それはそのときに考えることにしましょう｡

## Day03まとめ

- データを扱う所に型あり
- 自作RDBMSの型を作った
- C++難しい...(ソースコードが読めない)

## 次回予告

- MUST: RDBMSのStorageに関して説明
- SHOULD: Storageに関する部分を実装
