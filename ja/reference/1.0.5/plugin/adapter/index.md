---
title: 適合フェーズでのプラグインAPI
layout: ja
---

{% comment %}
##############################################
  THIS FILE IS AUTOMATICALLY GENERATED FROM
  "_po/ja/reference/1.0.5/plugin/adapter/index.po"
  DO NOT EDIT THIS FILE MANUALLY!
##############################################
{% endcomment %}


* TOC
{:toc}


## 概要 {#abstract}

各々のDroonga Engineプラグインは、それ自身のための*アダプター*を持つことができます。適合フェーズでは、アダプターは入力メッセージ（Protocol AdapterからDroonga Engineへ送られてきたリクエストに相当）と出力メッセージ（Droonga EngineからProtocol Adapterへ送られるレスポンスに相当）の両方について変更を加えることができます。


### アダプターの定義の仕方 {#howto-define}

例えば、「foo」という名前のプラグインにアダプターを定義する場合は以下のようにします：

~~~ruby
require "droonga/plugin"

module Droonga::Plugins::FooPlugin
  extend Plugin
  register("foo")

  class Adapter < Droonga::Adapter
    # このアダプターを設定するための操作
    XXXXXX = XXXXXX

    def adapt_input(input_message)
      # 入力メッセージを変更するための操作
      input_message.XXXXXX = XXXXXX
    end

    def adapt_output(output_message)
      # 出力メッセージを変更するための操作
      output_message.XXXXXX = XXXXXX
    end
  end
end
~~~

アダプターを定義するための手順は以下の通りです：

 1. プラグイン用のモジュール（例：`Droonga::Plugins::FooPlugin`）を定義し、プラグインとして登録する。（必須）
 2. [`Droonga::Adapter`](#classes-Droonga-Adapter)を継承したアダプタークラス（例：`Droonga::Plugins::FooPlugin::Adapter`）を定義する。（必須）
 3. [アダプターを適用する条件を設定する](#howto-configure)。（必須）
 4. 入力メッセージに対する変更操作を[`#adapt_input`](#classes-Droonga-Adapter-adapt_input)として定義する。（任意）
 5. 出力メッセージに対する変更操作を[`#adapt_output`](#classes-Droonga-Adapter-adapt_output)として定義する。（任意）

[プラグイン開発のチュートリアル](../../../tutorial/plugin-development/adapter/)も参照して下さい。


### アダプターはどのように操作するか {#how-works}

アダプターは以下のように動作します：

 1. Droonga Engineが起動する。
    * アダプタークラス（例：`Droonga::Plugins::FooPlugin::Adapter`）の唯一のインスタンスが作られ、登録される。
      * 入力のマッチングパターンおよび出力のマッチングパターンが登録される。
    * Droonga Engineが起動し、入力メッセージを待ち受ける。
 2. 入力メッセージがProtocol AdapterからDroonga Engineへ送られてくる。
    この時点で（入力メッセージ用の）適合フェーズが開始される。
    * そのメッセージが[入力のマッチングパターン](#config)にマッチするアダプターについて、アダプターの[`#adapt_input`](#classes-Droonga-Adapter-adapt_input)が呼ばれる。
    * このメソッドは、[入力メッセージ自身が持つメソッド](#classes-Droonga-InputMessage)を通じて入力メッセージを変更することができる。
 3. すべてのアダプターが適用された時点で、入力メッセージ用の適合フェーズが終了し、メッセージが次の立案フェーズに送られる。
 4. 出力メッセージが前の集約フェーズから送られてくる。
    この時点で（出力メッセージ用の）適合フェーズが開始される。
    * そのメッセージ外貨の両方の条件を満たす場合に、アダプターの[`#adapt_output`](#classes-Droonga-Adapter-adapt_output)が呼ばれる：
      - そのメッセージが、そのアダプター自身によって処理された入力メッセージに起因した物である。
      - そのメッセージが、アダプターの[出力のマッチングパターン](#config)にマッチする。
    * このメソッドは、[出力メッセージ自身が持つメソッド](#classes-Droonga-OutputMessage)を通じて出力メッセージを変更することができる。
 5. すべてのアダプターが適用された時点で、出力メッセージ用の適合フェーズが終了し、メッセージがProtocol Adapterに送られる。

上記の通り、Droonga Engineは各プラグインのアダプタークラスについて、インスタンスを全体で1つだけ生成します。
対になった入力メッセージと出力メッセージのための状態を示す情報をアダプター自身のインスタンス変数として保持してはいけません。
代わりに、状態を示す情報を入力メッセージのbodyの一部として埋め込み、対応する出力メッセージのbodyから取り出すようにして下さい。

アダプター内で発生したすべてのエラーは、Droonga Engine自身によって処理されます。[エラー処理][error handling]も併せて参照して下さい。


## 設定 {#config}

`input_message.pattern` ([マッチングパターン][matching pattern], 省略可能, 初期値=`nil`)
: 入力メッセージに対する[マッチングパターン][matching pattern]。
  パターンが指定されていない（もしくは`nil`が指定された）場合は、すべてのメッセージがマッチします。

`output_message.pattern` ([マッチングパターン][matching pattern], 省略可能, 初期値=`nil`)
: 出力メッセージに対する[マッチングパターン][matching pattern]。
  パターンが指定されていない（もしくは`nil`が指定された）場合は、すべてのメッセージがマッチします。

## クラスとメソッド {#classes}

### `Droonga::Adapter` {#classes-Droonga-Adapter}

これはすべてのアダプターに共通の基底クラスです。独自プラグインのアダプタークラスは、このクラスを継承する必要があります。

#### `#adapt_input(input_message)` {#classes-Droonga-Adapter-adapt_input}

このメソッドは、[`Droonga::InputMessage`](#classes-Droonga-InputMessage)でラップされた入力メッセージを受け取ります。
入力メッセージは、メソッドを通じて内容を変更することができます。

この基底クラスにおいて、このメソッドは何もしない単なるプレースホルダとして定義されています。
入力メッセージを変更するには、以下のようにメソッドを再定義して下さい：

~~~ruby
module Droonga::Plugins::QueryFixer
  class Adapter < Droonga::Adapter
    def adapt_input(input_message)
      input_message.body["query"] = "fixed query"
    end
  end
end
~~~

#### `#adapt_output(output_message)` {#classes-Droonga-Adapter-adapt_output}

このメソッドは、[`Droonga::OutputMessage`](#classes-Droonga-OutputMessage)でラップされた出力メッセージを受け取ります。
出力メッセージは、メソッドを通じて内容を変更することができます。

この基底クラスにおいて、このメソッドは何もしない単なるプレースホルダとして定義されています。
出力メッセージを変更するには、以下のようにメソッドを再定義して下さい：

~~~ruby
module Droonga::Plugins::ErrorConcealer
  class Adapter < Droonga::Adapter
    def adapt_output(output_message)
      output_message.status_code = Droonga::StatusCode::OK
    end
  end
end
~~~

### `Droonga::InputMessage` {#classes-Droonga-InputMessage}

#### `#type`, `#type=(type)` {#classes-Droonga-InputMessage-type}

入力メッセージの`"type"`の値を返します。

以下のように、新しい文字列値を代入することで値を変更できます：

~~~ruby
module Droonga::Plugins::MySearch
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "my-search"]

    def adapt_input(input_message)
      p input_message.type
      # => "my-search"
      #    このメッセージは「my-search」というメッセージタイプに
      #    対応したプラグインによって処理される。

      input_message.type = "search"

      p input_message.type
      # => "search"
      #    メッセージタイプが変更された。
      #    このメッセージはsearchプラグインによって、
      #    通常の検索リクエストとして処理される。
    end
  end
end
~~~

#### `#body`, `#body=(body)` {#classes-Droonga-InputMessage-body}

入力メッセージの`"body"`の値を返します。

以下のように、新しい値を代入したり部分的に値を代入したりすることで、値を変更することができます：

~~~ruby
module Droonga::Plugins::MinimumLimit
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "search"]

    MAXIMUM_LIMIT = 10

    def adapt_input(input_message)
      input_message.body["queries"].each do |name, query|
        query["output"] ||= {}
        query["output"]["limit"] ||= MAXIMUM_LIMIT
        query["output"]["limit"] = [query["output"]["limit"], MAXIMUM_LIMIT].min
      end
      # この時点で、すべての検索クエリが"output.limit=10"の指定を持っている。
    end
  end
end
~~~

別の例：

~~~ruby
module Droonga::Plugins::MySearch
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "my-search"]

    def adapt_input(input_message)
      # 独自形式のメッセージからクエリ文字列を取り出す。
      query_string = input_message["body"]["query"]

      # "search"型の内部的な検索リクエストを組み立てる。
      input_message.type = "search"
      input_message.body = {
        "queries" => {
          "source"    => "Store",
          "condition" => {
            "query"   => query_string,
            "matchTo" => ["name"],
          },
          "output" => {
            "elements" => ["records"],
            "limit"    => 10,
          },
        },
      }
      # この時点で、"type"と"body"は両方とも完全に置き換えられている。
    end
  end
end
~~~

### `Droonga::OutputMessage` {#classes-Droonga-OutputMessage}

#### `#status_code`, `#status_code=(status_code)` {#classes-Droonga-OutputMessage-status_code}

出力メッセージの`"statusCode"`の値を返します。

以下のように、新しいステータスコードを代入することで値を変更できます： 

~~~ruby
module Droonga::Plugins::ErrorConcealer
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "search"]

    def adapt_output(output_message)
      unless output_message.status_code == StatusCode::InternalServerError
        output_message.status_code = Droonga::StatusCode::OK
        output_message.body = {}
        output_message.errors = nil
        # この時点で、内部的なサーバーエラーはすべて無視されるため
        # クライアントは通常のレスポンスを受け取る事になる。
      end
    end
  end
end
~~~

#### `#errors`, `#errors=(errors)` {#classes-Droonga-OutputMessage-errors}

出力メッセージの`"errors"`の値を返します。

以下のように、新しいエラー情報を代入したり値を部分的に書き換えたりする事ができます：

~~~ruby
module Droonga::Plugins::ErrorExporter
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "search"]

    def adapt_output(output_message)
      output_message.errors.delete(secret_database)
      # 秘密のデータベースからのエラー情報を削除する。

      output_message.body["errors"] = {
        "records" => output_message.errors.collect do |database, error|
          {
            "database" => database,
            "error" => error
          }
        end,
      }
      # エラー情報を、"error"という名前の擬似的な検索結果に変換する。
    end
  end
end
~~~

#### `#body`, `#body=(body)` {#classes-Droonga-OutputMessage-body}

出力メッセージの`"body"`の値を返します。

以下のように、新しい値を代入したり部分的に値を代入したりすることで、値を変更することができます：

~~~ruby
module Droonga::Plugins::SponsoredSearch
  class Adapter < Droonga::Adapter
    input_message.pattern = ["type", :equal, "search"]

    def adapt_output(output_message)
      output_message.body.each do |name, result|
        next unless result["records"]
        result["records"].unshift(sponsored_entry)
      end
      # これにより、すべての検索結果が広告エントリを含むようになる。
    end

    def sponsored_entry
      {
        "title"=> "SALE!",
        "url"=>   "http://..."
      }
    end
  end
end
~~~


  [matching pattern]: ../matching-pattern/
  [error handling]: ../error/
