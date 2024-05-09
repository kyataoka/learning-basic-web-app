---
marp: true
theme: mytheme
paginate: true
---

<!--
_class: title
_paginate: false
-->

# Web 開発入門

### 最小構成の Web アプリ開発を通した基礎知識の習得

---

<!--
_class: page
-->

# 目次

- 本資料について
- Step 0 - 使用ツール
- Step 1 - 学習環境
- Step 2 - データベース
- Step 3 - バックエンド(API)
- Step 4 - フロントエンド

---

<!--
_class: title
-->

# 本資料について

---

<!--
_class: page
-->

# 本資料について

本資料は簡単なサンプルを通じて Web の基本構成を
学ぶことを目的としたものである。

最小構成の Web アプリケーションを学ぶことで

- データベース
- バックエンド
- フロントエンド

と言った Web アプリケーションを構成する要素に対する
理解を深められることを期待する。

---

<!--
_class: title
-->

# Step 1 - 学習環境

---

<!--
_class: page
-->

# Step 1 - 学習環境

本資料での学習は基本的に下記ソフトウェアの利用を学習とする。
各種ソフトウェアの利用方法については本資料では割愛する。

- Windows (たぶんそれ以外でも ok)
- Visual Studio Code (以下 VSCode)
- Docker
- Google Chrome

また、開発においては下記ツールを利用する。

- PostgreSQL (データベース管理システム)
- FastAPI

---

<!--
_class: page
-->

# Step 1 - 学習環境

各ステップに必要な環境は各"step_X"ディレクトリに格納されている。

ターミナル上で対象の Step のディレクトリに移動し、
下記コマンドにて Docker 環境を立ち上げた上で学習を進める。
`$ docker compose up -d`

Docker 環境を停止させる場合は **_ctrl + c_** を押下する。

また、各環境を修了させるには下記コマンドを実行する。
`$ docker compose down`

また、本資料の環境においては
環境を終了させると DB 上のデータは全て初期化される。

---

<!--
_class: page
-->

# Step 1 - 学習環境

---

<!--
_class: title
-->

# Step 2 - データベース

---

<!--
_class: page
-->

# Step 2 - データベース

本章では Web アプリケーションのデータを管理するデータベースについて学習する。

データベースには下記のような種類がある。

- RDB (Relational Database: リレーショナルデータベース)
- キーバリュー型
- カラム指向型
- ドキュメント型
- グラフ型

本資料では現状一般的に普及している RDB を扱う。

---

<!--
_class: page
-->

# Step 2 - データベース

## RDB (RDBMS) とは

日本語では関係データベースと訳され、
複数の表でデータを管理しそれらを関連付けることにより
複雑な処理を実現するデータベースである。

また、RDBMS (Relational Database Management System)とは
RDB を管理するためのツールを意味し、
データベースの定義、操作、制御などを行う。

一般的に使われる RDBMS としては
PostgreSQL / MySQL / Oracle Database
などがある。

本資料では PostgreSQL を扱う。

---

<!--
_class: page
-->

# Step 2 - データベース

## SQL とは

RDB は基本的に SQL(Structured Query Language: 構造問い合わせ言語)と呼ばれる構造文を RDBMS に対して実行することで操作を行う。
SQL を用いることにより、データ構造の定義、
データの作成(Create) / 読み出し(Read) / 更新(Update) / 削除(Delete)などを行うことができる。
また、これらのデータ操作のことを頭文字を取って **_CRUD_** と表現することがある。

---

<!--
_class: page
-->

# Step 2 - データベース

## 0.下準備

データベース学習環境を作成するため、
ターミナルにて下記コマンドを実行する。

```bash
# 作業ディレクトリへ移動
$ cd step_2
# Docker環境の立ち上げ
$ docker compose up -d
$ docker compose exec db bash

# PostgreSQLの管理コンソールの立ち上げ
$ psql -d mydb -U postgres
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 1.テーブルの作成

まずはデータを管理するためのテーブルを作成する。
下記 SQL を実行する。

```SQL
CREATE TABLE employee (id integer, name varchar(10), dept_code integer);
```

カッコの中はテーブルに持たせるカラム名とデータ型をカンマ区切りで列挙する。
上記の SQL は id / name / dept_code という 3 つのカラムを持つ
employee というテーブルを作成するものである。

上記コマンド実行後、下記により作成したテーブルを確認する。

```SQL
\dt --テーブルの一覧を表示
\d employee --作成したテーブル:employeeの詳細を表示
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 1.テーブルの作成

下記のような出力が得られればテーブルの作成に成功である。

```
mydb=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner
--------+----------+-------+----------
 public | employee | table | postgres
(1 row)

mydb=# \d employee
                      Table "public.employee"
  Column   |         Type          | Collation | Nullable | Default
-----------+-----------------------+-----------+----------+---------
 id        | integer               |           |          |
 name      | character varying(20) |           |          |
 dept_code | integer               |           |          |
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 2.データの作成(Create)

下記 SQL を実行し、先ほど作成したテーブル:employee にデータを新規作成する。

```sql
--1レコードを追加
INSERT INTO employee VALUES (1, 'yamada', 1);
--複数レコードを追加
INSERT INTO employee VALUES (2, 'sato', 2), (3, 'suzuki', 2), (4, 'takahashi', 3);
```

下記 SQL により、上記データが正常に追加できていることを確認する。
(SQL の詳細は次にて解説)

```
mydb=# SELECT * FROM employee ;
 id |   name    | dept_code
----+-----------+-----------
  1 | yamada    |         1
  2 | sato      |         2
  3 | suzuki    |         2
  4 | takahashi |         3
(4 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 2.データの読み込み(Read)

次は先ほど追加したデータを取得する。
下記 SQL を実行する。

```sql
SELECT * FROM employee;
```

次のような出力が得られる。

```
mydb=# SELECT * FROM employee ;
 id |   name    | dept_code
----+-----------+-----------
  1 | yamada    |         1
  2 | sato      |         2
  3 | suzuki    |         2
  4 | takahashi |         3
(4 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 3.データの読み込み(Read)

今度はカラムを絞って取得する。
先ほどの例では SELECT の後に **_\*_** を指定したが、その代わりに具体的なカラム名を指定することで
そのカラムのデータのみを取得できる。

```sql
SELECT id, name FROM employee;
```

次のような出力が得られる。

```
mydb=# SELECT id, name FROM employee ;
 id |   name
----+-----------
  1 | yamada
  2 | sato
  3 | suzuki
  4 | takahashi
(4 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 3.データの読み込み(Read)

最後に取得するデータの条件による絞り込みを行う。
取得する条件を指定するためには WHERE を用いる。

```sql
SELECT * FROM employee WHERE dept_code = 2;
```

次のような出力が得られる。

```
mydb=# SELECT * FROM employee WHERE dept_code = 2;
 id |  name  | dept_code
----+--------+-----------
  2 | sato   |         2
  3 | suzuki |         2
(2 rows)
```

dept_code が 2 であるデータのみが取得できる。

---

<!--
_class: page
-->

# Step 2 - データベース

## 4.データの更新(Update)

既存のデータに対して値の書き換えを行うには UPDATE 句を用いた下記の SQL を実行する。

```sql
UPDATE employee SET dept_code = 4 WHERE name = 'takahashi';
```

更新後にデータを取得すると
name = 'takahashi'のデータの dept_code が書き換わっていることが確認できる。

```
mydb=# SELECT * FROM employee ;
 id |   name    | dept_code
----+-----------+-----------
  1 | yamada    |         1
  2 | sato      |         2
  3 | suzuki    |         2
  4 | takahashi |         4
(4 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 5.データの削除(Delete)

テーブルからデータを削除するには DELETE 句を用いる。

```sql
DELETE FROM employee WHERE id = 3;
```

削除後にデータを取得すると
id = 3 のデータがテーブルから削除されていることが確認できる。

```
mydb=# SELECT * FROM employee ;
 id |   name    | dept_code
----+-----------+-----------
  1 | yamada    |         1
  2 | sato      |         2
  4 | takahashi |         4
(3 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 5.データの削除(Delete)

また、先ほどの SQL では WHERE 句を用いて条件を指定していたが、
これを指定しない場合は対象のテーブルの全データを削除するため
注意が必要である。

```sql
DELETE FROM employee ;
```

削除後にデータを取得すると
すべてのデータがテーブルから削除されていることが確認できる。

```
mydb=# SELECT * FROM employee ;
 id | name | dept_code
----+------+-----------
(0 rows)
```

---

<!--
_class: page
-->

# Step 2 - データベース

## 6.章末問題

1. 下記を満たすテーブルを作成してください。
   テーブル名: todo
   カラム
   | カラム名 | データ型 |
   | --------- | ----------- |
   | id | integer |
   | name | varchar(10) |
   | done | boolean |
   | create_at | date |
   | done_at | date |

---

<!--
_class: page
-->

# Step 2 - データベース

## 6.章末問題

2. 問 1 で作成したテーブルに対して、下記データを作成してください。
   | id | name | done | create_at | done_at |
   | ---- | ---- | ---- | ---- | ---- |
   | 1 | shopping | true | 2024/5/1 | 2024/5/2 |
   | 2 | homework | false | 2024/5/1 | null |
   | 3 | cleaning | true | 2024/5/2 | 2024/5/4 |
   | 4 | haircut | false | 2024/5/3 | null |

3. 問 2 で追加したデータのうち、未完了のもののみを取得してください。

4. 問 2 で追加したデータのうち、homework を 2024/5/5 に完了したという更新をしてください。

5. 問 4 が完了した状態で完了している全てのデータを削除してください。

---

<!--
_class: page
-->

# Step 2 - データベース

## 7.章末問題(解答)

1.  ```sql
    CREATE TABLE todo (id integer, name varchar(10), done boolean, create_at date, done_at date);
    ```

2.  ```sql
    INSERT INTO
      todo
    VALUES
      (1, 'shopping', TRUE, '2024-05-01', '2024-05-02'),
      (2, 'homework', FALSE, '2024-05-01', null),
      (3, 'cleaning', TRUE, '2024-05-02', '2024-05-04'),
      (4, 'haircut', FALSE, '2024-05-03', null);
    ```

3.  ```sql
    SELECT * FROM todo WHERE NOT done;
    ```

---

<!--
_class: page
-->

# Step 2 - データベース

## 7.章末問題(解答)

4.  ```sql
    UPDATE
      todo
    SET
      done = TRUE,
      done_at = '2024-05-05'
    WHERE
      name = 'homework';
    ```

5.  ```sql
    DELETE FROM todo WHERE done;
    ```

---

<!--
_class: page
-->

# Step 2 - データベース

## 8.まとめ

本章では基本的な SQL を解説した。

ここで読者は

> psql を終了させられない

や

> ctrl + c を押してもターミナルに操作が戻らない

と焦っている頃かと思うが、
落ち着いて `\q` してみよう。

本章が終了したら学習環境を終了させることも忘れずに行うこと。

```bash
$ docker compose down
```

---

<!--
_class: title
-->

# Step 3 - バックエンド(API)

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

本章では Web アプリケーションのバックエンドについて説明する。

バックエンドの実装には下記のような言語が主に使用される。

- Python
- JavaScript / TypeScript (Node.js)
- Java
- Go
- Ruby (Ruby on Rails)
- C# (.NET Framework)
- PHP

本資料では Python を用いた解説を行う。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## バックエンドとは

バックエンドとは、サーバサイドともいい
主にデータ処理を担当する。
クライアント(ユーザのブラウザ)からのリクエストを受け取り、
適切なデータ処理やデータベース処理を行い
その結果をクライアントへ送り返す。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## フレームワークとは

フレームワークとは、Web アプリケーション以外の分野でもよく用いられ、
効率的かつ組織的にアプリケーションを開発する仕組みを提供する。
フレームワークを利用するメリットとしては下記のようなものがある。

- コードの再利用
- 開発スピードの向上

各言語における主要な Web フレームワークには下記のようなものがある。

| Python                         | JavaScript             | Java   | Ruby          | PHP                      |
| ------------------------------ | ---------------------- | ------ | ------------- | ------------------------ |
| FastAPI <br> Django <br> Flask | Express.js <br> NestJS | Spring | Ruby on Rails | Laravel <br> Codeigniter |

本資料では記述方法の簡潔さから Python の FastAPI を用いた解説を行う。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## Web API とは

Web API とは、Web アプリケーションにおいてサーバとクライアント(ブラウザ)が情報をやりとりするためのインターフェースである。
一般的には HTTP プロトコルを使用し、JSON や XML 形式のデータのやり取りを行う。
Web アプリケーションのみならず、モバイルアプリケーションとサーバ間や、サーバ同士での通信にもよく用いられる。

また、Web API の設計として REST アーキテクチャが採用されることが多い。
REST アーキテクチャ(RESTful API)は HTTP メソッドを利用してリソースの状態を操作する。

HTTP メソッドには POST / GET / PUT / DELETE などが存在し、
それぞれが CRUD における Create / Read / Update / Delete に相当する。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 0.下準備

バックエンド学習環境を作成するため、ターミナルにて下記コマンドを実行する。

```bash
# 作業ディレクトリへ移動
$ cd step_3

# Docker環境の立ち上げ
$ docker compose up -d
$ docker compose exec backend bash

# Docker環境内での作業ディレクトリ確認
$ pwa #/appであればok
```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 1.単純なデータを返す GET メソッドの実装

まずは固定の値を返す GET メソッドを実装する。
これは割り当てられた Route (URL)に対してクライアントからリクエストが来た際に、
特定の文字列を返すものである。

Docker 環境上に下記ファイルを作成する。

```python
# /app/simple_get.py

from fastapi import FastAPI
app = FastAPI()

@app.get("/")
async def get_hello_world():
  return 'Hello World!'
```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 1.単純なデータを返す GET メソッドの実装

下記コマンドでバックエンドサーバを起動する。
(コードを書き換えるたびに下記コマンドの再実行が必要、実行を停止するには ctrl + c を押下)

```bash
$ uvicorn simple_get:app --host 0.0.0.0 --port 8000
```

別のターミナル(あるいはコマンドプロンプト等)を開き書きコマンドを実行する

```bash
# Docker環境をもう1つ開く
$ docker compose exec backend bash
# サーバへリクエストを送る
$ curl -w '\n' http://localhost:8000
```

curl コマンドはブラウザなどと同様にサーバに対してリクエストを送信し、
レスポンスを出力する CLI 版のクライアントである。
コマンド実行の結果として下記が得られることを確認する。

```
"Hello World!"
```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 1.単純なデータを返す GET メソッドの実装

先ほどのコードを解説する。

1. このコードで Python の Web フレームワークである FastAPI よ読み込む。
   ```python
   from fastapi import FastAPI
   ```
2. 読み込んだフレームワークのインスタンスを生成する
   ```python
   app = FastAPI()
   ```
3. http\://localhost:8000/への GET リクエストに対する処理を実装する
   (ここでは固定文字列をそのまま返す)
   ```python
   @app.get("/")
   async def get_hello_world():
     return 'Hello World!'
   ```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## コラム - uvicorn ってなーに？？

一般的に Python で書いたプログラムを CLI 上で実行する際には python コマンドを用いる。
ではなぜ先ほど書いた simple_get.py は uvicorn というコマンドで実行したのか。

uvicorn とは Web サーバと呼ばれるソフトウェアに分類されるものである。
Web サーバとは、サーバに対して送られてきた HTTP リクエストを受け取り、
それをバックエンドアプリケーションに渡し、その結果(HTML や画像、各種データなど)を
クライアントへ返す役割を持つソフトウェアである。

一般的に HTTP リクエストはヘッダやペイロードといった様々なデータを持つものであり、
それを効率的に処理してアプリケーション側に渡し、
アプリケーションの実行結果を改めて HTTP レスポンスとして整形しクライアントへ返すのが
Web サーバである。

類似ソフトウェアには Apache や Nginx ("えんじんえっくす"って読むよ！)などがある。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 2.ルーティングを分岐

一般的な Web サーバは 1 つの処理だけではなく、 リクエストされた URL に応じて様々な処理を行う。
ここではリクエストされた URL に応じて処理を追加する方法を解説する。

Docker 環境上に下記ファイルを作成する。

```python
# /app/routing.py

from fastapi import FastAPI
app = FastAPI()

@app.get("/foo")
async def get_foo():
  return 'Hello Foo!'

@app.get("/bar")
async def get_bar():
  return 'Hello Bar!'
```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 2.ルーティングを分岐

先ほど同様に下記コマンドで実行する。

```bash
$ uvicorn simple_get:app --host 0.0.0.0 --port 8000
```

別ターミナル側で下記コマンドを実行する。

```bash
# サーバへリクエストを送る
$ curl -w '\n' http://localhost:8000/foo
# "Hello Foo!"と表示される

# サーバへリクエストを送る
$ curl -w '\n' http://localhost:8000/bar
# "Hello Bar!"と表示される
```

このように URL に対応した処理を振り分けることをルーティング (Routing) と呼ぶ。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 3.DB との接続

これまでは単純な固定値のみを返していたが、実際の Web アプリケーションとしては
DB に問い合わせをかけ、その結果をクライアントに返すことが多い。

ここでは実際に Python のバックエンドプログラムから DB に問い合わせを行い、
その結果を JSON 形式でクライアントへ返信するプログラムを作成する。

まずは Python から PostgreSQL へのアクセス情報を設定する。
本資料では asyncpg というライブラリを利用する。
今回提供する Docker 環境ではすでにインストール済みだが、実際には各自で環境を用意する必要がある。

```python
# /app/main.py
import asyncpg
import os

async def create_db_connection():
  return await asyncpg.connect(
    host='db',
    port='5432',
    user='postgres',
    password='password',
    database='mydb'
  )
```

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 3.DB との接続

次に実際に DB に対して Query を実行する部分の実装を行う。

```python
# /app/main.py

import asyncpg
import os
from fastapi import FastAPI

async def create_db_connection():
# 中略

app = FastAPI()
@app.get('/employee')
async def get_all_employee():
  conn = await create_db_connection()
  items = await conn.fetch('SELECT * FROM employee')
  conn.close()
  return items
```

なお、本プログラムは解説用となるため例外処理等は省いている。

---

<!--
_class: page
-->

# Step 3 - バックエンド(API)

## 3.DB との接続

修正が完了したら下記コマンドで実行する。

```bash
$ uvicorn simple_get:app --host 0.0.0.0 --port 8000
```

別ターミナルでの Docker 環境にて、下記コマンドを実行し結果を取得する。

```bash
curl -w '\n' http://localhost:8000/employee
```

結果として下記 JSON 形式の文字列が確認できる。(可読性のため一部整形済み)

```json
[
  { "id": 1, "name": "Yamada", "dept_code": 1 },
  { "id": 2, "name": "Sato", "dept_code": 2 },
  { "id": 3, "name": "Suzuki", "dept_code": 2 },
  { "id": 4, "name": "Takahashi", "dept_code": 3 }
]
```

---

<!--
_class: eof
_paginate: false
-->

# EOF :sleepy:
