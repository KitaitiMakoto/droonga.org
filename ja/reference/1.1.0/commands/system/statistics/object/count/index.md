---
title: system.statistics.object.count
layout: ja
---

{% comment %}
##############################################
  THIS FILE IS AUTOMATICALLY GENERATED FROM
  "_po/ja/reference/1.1.0/commands/system/statistics/object/count/index.po"
  DO NOT EDIT THIS FILE MANUALLY!
##############################################
{% endcomment %}


* TOC
{:toc}

## 概要 {#abstract}

`system.statistics.object.count` コマンドは、データセット内の物理的なオブジェクト数を数えて報告します。

## APIの形式 {#api-types}

### HTTP {#api-types-http}

リクエスト先
: `(ドキュメントルート)/droonga/system/statistics/object/count`

リクエストメソッド
: `GET`

リクエストのURLパラメータ
: [パラメータの詳細](#parameters)を参照。

リクエストのbody
: なし。

レスポンスのbody
: [レスポンスメッセージ](#response)。

### REST {#api-types-rest}

対応していません。

### Fluentd {#api-types-fluentd}

形式
: Request-Response型。コマンドに対しては必ず対応するレスポンスが返されます。

リクエストの `type`
: `system.statistics.object.count`

リクエストの `body`
: [パラメータ](#parameters)のハッシュ。

レスポンスの `type`
: `system.statistics.object.count.result`

## パラメータの構文 {#syntax}

    {
      "output": [
        "tables",
        "columns",
        "records"
      ]
    }

または

    {
      "output": [
        "total"
      ]
    }

## 使い方 {#usage}

このコマンドは、指定された対象の物理的な数を数えて報告します。
例：

    {
      "type" : "system.statistics.object.count",
      "body" : {
        "output": [
          "tables",
          "columns",
          "records",
          "total"
        ]
      }
    }
    
    => {
         "type" : "system.statistics.object.count.result",
         "body" : {
           "tables":  2,
           "columns": 0,
           "records": 1,
           "total":   3
         }
       }


## パラメータの詳細 {#parameters}

全てのパラメータは省略可能です。

### `output` {#parameter-output}

概要
: 個数を報告する対象。

値
: 計数対象の配列。指定された対象のみが計数されます。
  取り得る値：
  
   * `tables`
   * `columns`
   * `records`
   * `total`

既定値
: `[]`


## レスポンス {#response}

このコマンドは以下のようなハッシュを `body` 、`200` を `statusCode` としたレスポンスを返します。以下はその一例です。。

    {
      "tables":  <テーブルの総数>,
      "columns": <カラムの総数>,
      "records": <レコードの総数>,
      "total":   <全てのオブジェクトの総数>,
    }

`tables`
: データセット内の物理的なテーブル数。
  複数のスライスがある場合、テーブルの個数も多くなります。
  例えば、2つのテーブルを定義していて2つのスライスがある場合、テーブルの個数は`4`となります。

`columns`
: データセット内の物理的なカラム数。
  複数のスライスがある場合、カラムの個数も多くなります。
  例えば、2つのテーブルにそれぞれ2つずつのカラムを定義していて、2つのスライスがある場合、カラムの個数は`8`となります。

`records`
: データセット内の物理的なレコード数。
  複数のスライスがある場合、fact表のレコード数の個数も多くなります。
  例えば、1つのレコードを持つ通常のテーブルと、1つのレコードを持つfact表があり、2つのスライスがある場合、レコードの個数は`3`となります。
  （1つは通常のテーブルのレコード、2つはfact表のレコード）

`total`
: `tables`、`columns`、`records`の合計。
  全てのオブジェクトの数の合計だけを知りたい場合は、それぞれの対象を個別に計数するよりも、こちらの方が高速です。

## エラーの種類 {#errors}

このコマンドは[一般的なエラー](/reference/message/#error)を返します。