
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [2章 エンドポイントの設計とリクエストの形式](#2章-エンドポイントの設計とリクエストの形式)
  - [2.1 APIとして公開する機能を設計する](#21-apiとして公開する機能を設計する)
  - [2.1.1 モバイルアプリケーション向けAPIに必要な機能](#211-モバイルアプリケーション向けapiに必要な機能)
  - [2.2 API エンドポイントの考え方](#22-api-エンドポイントの考え方)
    - [2.2.1 エンドポイントの基本的な設計](#221-エンドポイントの基本的な設計)
      - [2.2.1.1 短く入力しやすいURI](#2211-短く入力しやすいuri)
      - [2.2.1.2 人間が読んで理解できるAPI](#2212-人間が読んで理解できるapi)
      - [2.2.1.3 大文字小文字が混在していないURI](#2213-大文字小文字が混在していないuri)
      - [2.2.1.4 改造しやすい(Hackableな)URI](#2214-改造しやすいhackableなuri)
      - [2.2.1.5 サーバー側のアーキテクチャが反映されていないURI](#2215-サーバー側のアーキテクチャが反映されていないuri)
      - [2.2.1.6 ルールが統一されたURI](#2216-ルールが統一されたuri)
  - [2.3 HTTPメソッドとエンドポイント](#23-httpメソッドとエンドポイント)
    - [2.3.1 GETメソッド](#231-getメソッド)
    - [2.3.2 POSTメソッドｖ](#232-postメソッドv)
    - [2.3.3 PUTメソッド](#233-putメソッド)
    - [2.3.4 DELETEメソッド](#234-deleteメソッド)
    - [2.3.5 PATCHメソッド](#235-patchメソッド)
      - [2.3.5.1 X-HTTP-Method-Overrideヘッダ](#2351-x-http-method-overrideヘッダ)
  - [2.4 APIのエンドポイント設計](#24-apiのエンドポイント設計)
    - [2.4.1 リソースにアクセスするためのエンドポイントの設計の注意点](#241-リソースにアクセスするためのエンドポイントの設計の注意点)
      - [2.4.1.1 複数形の名詞を利用する](#2411-複数形の名詞を利用する)
      - [2.4.1.2 利用する単語に気をつける](#2412-利用する単語に気をつける)
      - [2.4.1.3 スペースやエンコードを必要とする文字を使わない](#2413-スペースやエンコードを必要とする文字を使わない)
      - [2.4.1.4 単語をつなげる必要がある場合はハイフンを利用する](#2414-単語をつなげる必要がある場合はハイフンを利用する)
  - [2.5 検索とクエリパラメータの設計](#25-検索とクエリパラメータの設計)
    - [2.5.1 取得数と取得位置のクエリパラメータ](#251-取得数と取得位置のクエリパラメータ)
    - [2.5.2 相対位置を利用する問題点](#252-相対位置を利用する問題点)
    - [2.5.3 絶対位置でデータを取得する](#253-絶対位置でデータを取得する)
    - [2.5.4 絞り込みのためのパラメータ](#254-絞り込みのためのパラメータ)
      - [2.5.4.1 URIに「Search」という単語を入れるべきか](#2541-uriにsearchという単語を入れるべきか)
    - [2.5.5 クエリパラメータとパスの使い分け](#255-クエリパラメータとパスの使い分け)
  - [2.6 ログインとOAuth 2.0](#26-ログインとoauth-20)
    - [2.6.1 アクセストークンの有効期限と更新](#261-アクセストークンの有効期限と更新)
    - [2.6.2 その他のGrant Type](#262-その他のgrant-type)
    - [補足：自分の情報へのエイリアス](#補足自分の情報へのエイリアス)
  - [2.7 ホスト名とエンドポイントの共通部分](#27-ホスト名とエンドポイントの共通部分)
  - [2.8 SSKDsとAPIデザイン](#28-sskdsとapiデザイン)
  - [まとめ](#まとめ)

<!-- /code_chunk_output -->


# 2章 エンドポイントの設計とリクエストの形式
## 2.1 APIとして公開する機能を設計する
- 最初に、APIの機能として何を公開するのかを決めなければならない
- SNSサービスを作ると仮定して、実際にどんなAPIを作るべきか考えてみる。主な機能は以下の通り
  - ユーザー登録、編集
  - 友達の検索、追加、削除
  - 友人間のメッセージのやりとり   
-  まず、データベースのテーブルを直接操作するようなものを作るべきではない。そのようなAPIは使いづらく、DBの内部仕様の理解が必要で、かつセキュリティ上の問題がある
- ポイントは、公開したAPIがどのように使われるのか、そのユースケースをきちんと考えること。今回は「モバイルアプリケーションのバックエンド向けのAPI」を題材に考えてみる

## 2.1.1 モバイルアプリケーション向けAPIに必要な機能
- 設計を考えるあたっては、まず、クライアントアプリの画面とその遷移を考える
- 図を見ながら、どんな機能をAPIとして提供しなければならないかを考える
  - ユーザー登録
  - ログイン
  - 自分の情報の取得
  - ........etc
- このリストと図を見比べながら過不足がないかを考える。これらをひとつひとつ個別のAPIとして実装するのではなく、まずは整理してみる

## 2.2 API エンドポイントの考え方
- APIの整理はエンドポイントを考えながら行う。Web APIにおけるエンドポイントとは、APIにアクセスするためのURIのことを意味する
  
### 2.2.1 エンドポイントの基本的な設計
- まず「良いURIの設計とは何か」が重要になってくる。良いURIとは、**「覚えやすく、どんな機能を持つURIなのかが一目でわかる」** もの  
- もう少し具体的にいうと、一般的に重要なのは以下のような点
  - 短く入力しやすい
  - 人間が読んで理解できる
  - 大文字小文字が混在していない
  - 改造しやすい(Hackableである)
  - サーバー側のアーキテクチャが反映されていない
  - ルールが統一されている
  
- 以下で詳細をのべる

#### 2.2.1.1 短く入力しやすいURI
- 例えば以下のようなURIは、不要な情報が入っていたり意味が重複していたりする
  ```
  http://api.example.com/service/api/search
  ``` 
- これは例えば次のように修正できる。このように短くしても含んでいる情報は変わらないため、同じことを表すならシンプルにすべきである
  ```
  http://api.example.com/search
  ```

#### 2.2.1.2 人間が読んで理解できるAPI
- 以下のようなURIは明らかにわかりにくい
  ```
  http://api.example.com/sv/u
  ```
- こうしたわかりにくいURIを生み出さないためには以下の点が大事
  1. **むやみに省略型を使わない。** productsをprod, weekをwkと略すなど  
     - ただし、国名を表す「jp」「jpn」 などは別。このようにISOなどで標準化され「コード」として体系化されたものは、むしろこちらを使う
  2. **APIでよく使われている英単語を使う。** 例えば検索型のAPIでよく使われるのは「find」ではなく「search」であるなど
     - 実際に他のAPIを見てみるのが一番よい。
  3. **スペルミスをしない。** 「calender」を「carender」としたりなど。複数形や過去形も間違いやすいので注意する
     - 有名な間違いに、「登録する」に「regist」を使うというものがある。正しくはregisterであり、resistは意味が違う

#### 2.2.1.3 大文字小文字が混在していないURI
- 以下のようにはしない。基本的にはすべて小文字にするべき
  ```
  http://api.example.com/Users/12345
  ```
- では、大文字を混ぜたURIでアクセスした場合はどう処理を行うべきか
  - 以下の処理が考えられる
    - どちらでも同じ結果を返す
    - 小文字だけのURIにリダイレクトする
    - 正しいURIとして認識せず、Not Foundを返す
    - 小文字でないと認識しないことをエラーメッセージで返す
  - 既存のサイトでは、単にNot Foundを返す場合が多い
    - Foursquare, Github, Tumblerなどが例
- そもそもHTTPにおいてURIは「スキーマとホスト名を除いては大文字と小文字は区別される」と仕様に書かれている(RFC 7230)。したがって大文字を混ぜた場合エラーになるのは当然ともいえる
- なお、一般的なサイトではステータスコード301でリダイレクトするのが最もよいとされているが、APIは検索エンジンと無関係のため、あまり問題にならない。
#### 2.2.1.4 改造しやすい(Hackableな)URI
- Hackableとは「URIを修正して別のURIにするのが容易である」という意味
- 例えば
  ```
  http://api.example.com/v1/items/123456
  ```
- あまりドキュメントを見なくても使える構造にするのが理想
- Hackableにさせる必要がない、という意見もある。例えばHATEOASというRESTを拡張する概念においてなど
  - HATEOASについては本章の最後に述べる

#### 2.2.1.5 サーバー側のアーキテクチャが反映されていないURI
- 例えば以下のようなエンドポイントはアーキテクチャの特定を容易にし、攻撃を受ける可能性が上がる
  ```
  http://api.example.com/cgi-bin/get_user.php?user=100
  ```

#### 2.2.1.6 ルールが統一されたURI
- 以下のようなURIは悪い例
  - 友達の情報の取得
     ```http://api.example.com/friends?id=100```
  - メッセージの投稿
    ```http://api.example.com/friend/100/message```

## 2.3 HTTPメソッドとエンドポイント
- URIとメソッドの関係は、操作するものと操作方法の関係である
  - URIは「操作する対象=リソース」、HTTPメソッドは「何をするか」
- 1つのURIのエンドポイントに異なるメソッドでアクセスすることで、リソースとその扱い方を分離して考えることができる
  - これは本来のHTTPの考え方に合致しており、現在のWeb APIの設計の主流

### 2.3.1 GETメソッド
- GETメソッドは「情報の取得」を表し、URIで指定されたリソースを取得するために使う
- GETメソッドに大してサーバー側の情報を変更する処理を書くのは「御法度」 

### 2.3.2 POSTメソッドｖ
- POSTメソッドは、指定したURIに属する新しいリソースを送信する(新しい情報を登録する)ために利用する
  
### 2.3.3 PUTメソッド
- PUTメソッドはPOSTと同じくサーバー側の情報を変更するためのメソッドだが、URIの指定の方法が違う
- POSTでは、送信したデータは指定したURIに「従属(subscribe)」したものになる
  - ディレクトリやカテゴリを表すURIに対してPOSTを行うと、新しいデータがその配下に作られるイメージ 
- PUTでは、更新したいリソースのURIそのものを指定し、その内容を書き換える

### 2.3.4 DELETEメソッド
- DELETEメソッドは、URIで指定したリソースを削除するメソッド

### 2.3.5 PATCHメソッド
- PATCHメソッドは、PUTと同じく指定したリソースを更新するメソッドだが、PUTと違い「一部を変更する」ことを明示したメソッド

#### 2.3.5.1 X-HTTP-Method-Overrideヘッダ
- GET/POST以外のメソッドが使えない場合を想定するときはどうするか？X-HTTP-Method-Overrideヘッダを用いることで、POSTでPUTやDELETEを利用することができる。GoogleのGoogle Data Protocolなどがこれを採用している

## 2.4 APIのエンドポイント設計
- SNSアプリケーションにおけるエンドポイントの例

| 目的 | エンドポイント | メソッド |
| ---- | ---- |----|
| ユーザーの一覧取得       |http://api.example.com/v1/users   |GET   |
| ユーザーの新規登録       |http://api.example.com/v1/users   |POST   |
| 特定のユーザーの情報の取得|http://api.example.com/v1/users/:id|GET   |
| ユーザーの情報の更新     |http://api.example.com/v1/users/:id|PUT/PATCH   |
| ユーザーの情報の削除     |http://api.example.com/v1/users/:id|DELETE   |

- ユーザーの検索は、ユーザー一覧取得APIに対してクエリパラメータで絞り込みを行うことで実現する
- `/users`はユーザーの集合、`/users/:id`は個々のユーザーを表すエンドポイント
- このように、**「データの集合」と「個々のデータ」をエンドポイントとして表現し、それに対してHTTPのメソッドで操作を表す**のが、Web APIの基本中の基本。これはデータベースにおけるテーブル名とレコードの関係と同じだといえる

### 2.4.1 リソースにアクセスするためのエンドポイントの設計の注意点
#### 2.4.1.1 複数形の名詞を利用する
- `users, friends, updates`など。データベースのテーブル名に複数形を用いるのが適切であるとされるのと同様

#### 2.4.1.2 利用する単語に気をつける
- エンドポイントで利用する単語には適切なものを選ぶ。ProgrammableWeb(https://www.programmableweb.com/) などで複数の例を参考にするとよい
  
#### 2.4.1.3 スペースやエンコードを必要とする文字を使わない
- URIでは利用できない文字があり、そういった文字はパーセントエンコーディングを利用する必要がある(例：%E3%81%82)。APIのエンドポイントにはパーセントエンコーディングされた文字が入らないようにする

#### 2.4.1.4 単語をつなげる必要がある場合はハイフンを利用する
- エンドポイント中での2つ以上の単語の繋ぎ方には、大きく分けて3つの方法がある
  
1. `http://api.example.com/v1/users/12345/profile-image`
2. `http://api.example.com/v1/users/12345/profile_image`
3. `http://api.example.com/v1/users/12345/profileImage`

- 上から順にスパイナルケース(チェインケース)、スネークケース、キャメルケースとよぶ。APIの設計においてどれを選べばよいかは、特に決定的なものはない
- **迷ったらハイフンにしておくのが無難。** 理由として、ホスト名(ドメイン名)にはアンダースコアが使えず、大文字小文字の区別がないので、そのルールに統一するとハイフンが適切だといえるから

## 2.5 検索とクエリパラメータの設計
- クエリパラメータとは、URIに対して`?`の後ろに付けることのできるパラメータのこと
  
### 2.5.1 取得数と取得位置のクエリパラメータ
- たくさんあるデータの一部を取得するとき、どういったパラメータで取得数と取得位置を取得するのがよいか？(ページネーション=pagenationの実現方法)
  
| サービス名 | 取得数 | 取得位置(相対位置) | 取得位置(絶対位置) |
| ---- | ---- |----|----|
| Twitter |count|cursor| max_id | 
| Linkedln |count|start| - | 
| Tumbler |limit|offset| since_id | 
| Disqus |limit|offset| - |
| YouTube |maxResults| pageToken | publishedBefore/publishedAfter| 
| Flickr |per_page|page| max_upload_date |
| GitHub |per_page|page| - |

- 主に一般的なのは`page/per_page`の組み合わせと、`offset/limit`の組み合わせ
- 1ページ50アイテム存在していた場合で3ページ目(101アイテム目から)取得する場合は以下のようになる
  - `per_page=50&page=3` 
  - `limit=50&offset=100`
- pageは1から(1-based)、offsetは0から(0-based)数え始めるのが一般的

### 2.5.2 相対位置を利用する問題点
相対的な取得位置でデータを取得する方法には問題がある
- 1つにはパフォーマンスで、例えばMySqlなどでは相対位置で取得する場合先頭から数えるため、データ数が膨大になると速度が落ちる
- また、更新頻度の高いデータにおいて、データに不整合が生じる場合がある。例えば最初の20件を取得してから次の20件を取得する間にデータが更新されると、実際に取得したい情報と取得された情報にズレが生じる

### 2.5.3 絶対位置でデータを取得する
- 指定したIDより前、あるいは指定した日時より前、という方法で指定を行うもの。上で述べた問題を解決できる
- TwitterのAPIにおけるmax_idなどがこれに相当する

### 2.5.4 絞り込みのためのパラメータ
- LinkedInのPeople Search APIなど(https://developer.linkedin.com/docs/v1/people/people-search-api)
  
`https://api.linkedin.com/v1/people-search`

#### 2.5.4.1 URIに「Search」という単語を入れるべきか
- これはURIをリソースとするデザインからはやや外れる。が、リソースが多すぎる場合は全てを取得することは現実的でなく、一覧が取れない検索用のAPIであることを明示するためには間違った考え方ではない
- 特にTwitterではSearch APIとAPIを独立した形で公開してい

### 2.5.5 クエリパラメータとパスの使い分け
- パラメータをクエリパラメータに入れるかパスに入れるかの判断基準には、主に以下がある
  - 一意なリソースを表すのに必要な情報かどうか
    - 例えばユーザーIDは、ユーザーIDを指定する事で参照したい情報が一意に定まるためパスに入れるほうがよい
    - 逆にアクセストークンなどは利用者の認可が目的で、リソースとは無関係なので、クエリパラメータのほうが適している
  - 省略可能かどうか
    - リストや検索の際のoffset, limitなど、省略するとデフォルトの値が利用される場合などはクエリパラメータを使う
  

## 2.6 ログインとOAuth 2.0
- OAuth2.0を利用したログイン寺のエンドポイント(トークンエンドポイント)について考える

|  サービス  |  エンドポイント  |
| ---- | ---- |
| Twitter | `/oauth2/token` |
| Dropbox | `/oauth2/authorize` |
| Facebook | `/oauth2/access_token` |
| Google | `/o/oauth2/token` |

(https://qiita.com/TakahikoKawasaki/items/200951e5b5929f840a1f)

- Oauthで認証を行う際、エンドポイントに対して以下のデータを`application/x-www-form-urlencoded`(Form送信の形式)、UTF-8で送る

|  キー  |  内容  |
| ---- | ---- |
| grand_type | "password" |
| username | ログインするユーザー名 |
| password | ログインするパスワード |
| scope | アクセスのスコープを指定(省略可能) |

### 2.6.1 アクセストークンの有効期限と更新
- `expires_in`はアクセストークンの有効期限を表す。これが切れると、サーバーは`invalid_token`というエラーをステータスコード401で返すことになっている
- `invalid_token`が発生した際はリフレッシュトークンを使って再度アクセストークンを要求できるが、リフレッシュトークンは返さないことも可能
- リフレッシュの際のリクエストは`grant_type`に`reflesh_token`を指定し、`reflesh_token`とともに返す

### 2.6.2 その他のGrant Type
- OAuth 2.0で定められている4種の認証フローをGrant Typeと呼ぶ
- 上で述べたのはResource Owner Password CredentialsというGrant Typeで、他の三つはAuthorization Code, Implicit, Client Credentials
- Authorization CodeとimplicitはFacebookやTwitterが提供するような、第三者(別のサービス)が、あなたのサービスのリソースへのアクセス許可を得るために利用するもの

### 補足：自分の情報へのエイリアス
- アクセスしているユーザー自身の情報を取得するとき、いちいち自身のユーザーIDが必要になるのは不便なので、`me`や`self`などのキーワードを用いがエンドポイントが利用されることが多い
- このように認証が必要な処理自体を分けることで、他人の個人情報が漏洩するなどのバグを防ぐこともできる

|  キー  |  内容  | 例 |
| ---- | ---- | ---- |
| Instagram | `self` | `/users/self/media/liked` |
| Google Calendar | `me` | `/users/me/calendarList` |
| Xing | `me` | `/users/me` |
| Zendesk | `me` | `/users/me.json` |
| Blogger | `self` | `/users/self` |

## 2.7 ホスト名とエンドポイントの共通部分
- ホスト名は、`api.example.com`のように"API"という名前を入れるのが主流
- Googleなどは`googleapis.com`という専用のドメインを用意しているが、筆者の意見としては、サービスが別のドメイン名で提供されているのであればAPIもそのドメイン名に従うべきと考える。`example.com`なら`api.example.com`にするなど

## 2.8 SSKDsとAPIデザイン
- ここまで述べてきた内容は、どちらかというとLSUDs向けのもの(1.8節参照)
- SSKDs向けのAPIで重視すべきは、エンドユーザーにとってのユーザー体験
- 例えばECサイトのスマートフォン向けアプリを作るとして、そのAPI設計について考える。ホーム画面を表示するには、新着、人気の商品、ユーザー情報が表示されるが、これらをLSUDs向けのように異なるAPIとしてしまうと、クライアント側で何度も異なるAPIにアクセスしなければならない。この場合、「ホーム画面表示用API」を作成し、それに一回アクセスするだけで表示できるようにするのがよく、これがSSKDs向けのAPIデザインである
- 「1スクリーン1APIコール、1セーブ1APIコール」一つの画面表示に一回のAPIコール、セーブも然り

## まとめ
- [Good]覚えやすく、どんな機能を持つかがひと目でわかるエンドポイントにする
- [Good]適切なHTTPメソッドを利用する
- [Good]適切な英単語を利用し、単数形、複数形にも注意する
- [Good]認証にはOAuth2.0を使う