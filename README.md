# 手動テストのコードカバレッジ
## 背景
+ 開発者とテスターに距離を感じる
+ 開発者がテスターが何をテストしているのか, やってほしいテストが網羅されているのかを

## 思ったこと
+ 手動テストにおいても, コードカバレッジがでたら一つの目安になるのではないか？
+ もちろん, 単体テストにおけるコードカバレッジと同様に, それが100%だからといって全てを網羅したテストができているわけではないが, 一つの目安にはなる.

## やること
+ PHPにおいてそういう機能があるらしい
  + 
  + これをやってみて, 挙動と仕組みを把握する.
+ Goにおいて同様のものがあるか調べる.
+ なかったら, コードカバレッジとプロファイラについて調べて, 消耗のものを作る.

## 履歴
### [codeception](https://piccagliani.github.io/Codeception.docs.ja_JP/01-Introduction.html)を調べてみる
+ [カバレッジ](https://piccagliani.github.io/Codeception.docs.ja_JP/11-Codecoverage.html)


### phpのサンプルプロジェクトを準備する
+ 適当なフレームワークのサンプルでいいか.
  + Laravelで.
+ 動かす環境をDockerで作りたい
  + [Laradoc](https://github.com/LaraDock/laradock)で, Laravel動かす環境を作れる.

### [Laradoc動かしてみる](https://www.google.co.jp/search?safe=off&ei=IkmCW9faFoLWhwP6t7eQCg&q=Laravel+Laradock&oq=Laravel+Laradock&gs_l=psy-ab.3..0j0i7i30k1l3j0i30k1j0i8i30k1l2j0i7i5i30k1.1221.2063.0.2225.6.6.0.0.0.0.115.537.5j1.6.0....0...1c.1.64.psy-ab..0.2.211...0i8i7i30k1.0.GthUDU-Fb1A)
+ Laradoc起動
  ```
  mkdir laravelwork
  cd laravelwork
  git clone https://github.com/LaraDock/laradock.git
  cd laradock
  ```
+ .env準備して起動
  + いろんなものが入っていて, すごく長い...
  + 度のサービスを使うか指定する形なのね.
  ```
  cp env-example .env
  docker-compose up -d workspace
  ```
+ composerを実行して, プロジェクトを作る
  ```
  docker exec -it laradock_workspace_1 bash
  cd /var/www/laravel
  composer create-project laravel/laravel laraveltest --prefer-dist
  ```
+ なんとなくした修正
  + laraveltest, 一つ上に移した
  + .envの `APP_CODE_PATH_HOST` も修正
+ サービスを起動
  + 動いていることはなんか見える.
  ```
  docker-compose up -d php-fpm nginx mysql phpmyadmin
  open http://localhost
  open http://localhost:8080
  ```

### [quickstartのタスクリスト](https://readouble.com/laravel/5.1/ja/quickstart.html)
+ clone
  ```
  git clone https://github.com/laravel/quickstart-basic quickstart
  ```
+ Laradockのenv修正
  ```
  APP_CODE_PATH_HOST=../quickstart
  ```
+ quickstartのenv修正
  ```
  DB_HOST=mysql
  DB_DATABASE=default
  DB_USERNAME=default
  REDIS_HOST=redis
  ```
+ 起動
  ```
  docker-compose down;
  docker-compose up -d php-fpm nginx mysql phpmyadmin redis
  ```
+ composer install
  ```
  docker exec -it laradock_workspace_1 bash
  composer install
  php artisan migrate
  ```
+ テストの実行
  ```
  docker exec -it laradock_workspace_1 bash
  phpunit
  ```

#### トラブル
+ php artisan migrateでエラー
  ```
  SQLSTATE[HY000] [2054] The server requested authentication method unknown to the client
  ```
  + phpmyadminでは入れることを確認
  + [この辺](http://blog.websandbag.com/entry/2018/05/17/121730)
    + `default_authentication_plugin=mysql_native_password` をつけてみろとか.
      + Laradockのmysqlを修正してbuildし直してみる.
    + docker-entrypoint-initdb.dにユーザ書く処理を入れるのは微妙なので, mysqlのversionを落とす.
      + mysql/Dockerfileを, ARG MYSQL_VERSION=5.7
      + にしなくても, laradock/.envを修正して, docker-compose build mysql
+ version切り替え後, なんかmysqlが起動しない.
  ```
  Table flags are 0 in the data dictionary but the flags in file
  ```
  + エラー検索したら, [これ](https://www.lancard.com/blog/2017/08/09/docker%E3%81%8B%E3%82%89amazon-ec2%E3%81%B8%E3%81%AE%E5%A5%AE%E9%97%98%E8%A8%98/)


### [codeception](https://codeception.com/for/laravel)をインストールしてみる
+ codeception
  ```
  composer require codeception/codeception --dev
  ```
  + phpunit, 4.8じゃ古いらしく, composer updateした

+ unitテストを準備
  ```
  composer exec codecept generate:test unit Example
  ```

+ functionalてすとを準備
  + テストの作成
    + [テストのリファレンス](https://codeception.com/docs/modules/Phalcon#dontSee)
  ```
  composer exec codecept g:cest functional Task
  ```
  + テストの実行
  ```
  composer exec codecept run -v
  ```

+ [コードカバレッジ](https://piccagliani.github.io/Codeception.docs.ja_JP/11-Codecoverage.html)
  + 

+ xdebugを有効にする.
  + https://qiita.com/yamazaki/items/d8b583a0fee70cadfc7b






