# PHP7の例外

## 種類
### `Throwable`
  * PHP7ので追加されたインターフェース
  * `Error` と `Exception` に実装されている

### `Error`
* 概要
    * PHP7で追加されたクラス
    * 言語レベルに近いエラーに当たる例外
    * 基本的にバグ
    * PHP側から投げられる

* `TypeError`
    * 型が違う
    * 引数の数が少ない

* `AssertionError`
  * アサーションが `false`

* `ParseError`
  * `eval()` などのパースが失敗

* `ArithmeticError`
  * 数値に関わるエラー
  * マイナスのビットシフトやintの範囲を超えた数値
  * 0割り


### `Exception`
* 概要
  * 今までのPHPで扱われていた例外
  * 自分で定義したり，ライブラリなどから投げられる
  * 種類は8種類

* `RuntimeException`
  * 実行時で防ぎようのない例外
  * データベースの接続が切れたなど

* `LogicException`
  * ロジック系の例外
  * バグ

* `ErrorException`
  * PHPのエラーを例外にしたもの
  * PHP側から投げられる

### 例外に関わる関数
* `set_error_handler()`
  * PHPのエラーを例外として扱える
  * `ErrorException`

* `set_exception_handler()`
  * 例外をキャッチできなかった時の処理
  * 想定外の例外が発生

## 注意点
* 例外安全
  * DBトランザクションやゾンビキャッシュ
  * グローバル変数
* ポケモン問題
  * 広い範囲でキャッチする
  * 例外ごとに正しい後始末をしなければならない
* 例外の投げなおし
  * 無意識に握りつぶしてしまう可能性
  * `throw()` の第三引数を利用する

## エラーとの区別
* 例外は必ず処理しなければならない
* エラーは無視することができる
* 無視したい時にエラーは有効
  * 非推奨機能など
  * `trigger_error()` などで `e_deprecated` など

## Codeigniterのエラーや例外
* `Codeigniter.php` で `set_error_handler()` と `set_exception_handler()` を呼び出す
* `Common.php` で 呼び出される関数を定義
  * 処理を止めるエラーレベルの設定
    * `E_ERROR | E_PARSE | E_COMPILE_ERROR | E_CORE_ERROR | E_USER_ERROR`
  * ログの出力
    * ログの出力は `application/log` 配下
    * `DEBUG`, `INFO` レベルも出力されている
  * エラー画面の表示
* `Exception.php` で `Common.php` から呼び出されるクラスを定義
  * エラー画面の表示

## 提案
* 例外作成
  * レスポンスコードと例外コードを紐づける
  * こちらで処理するべきものだけに絞る
  * `Exception.php` をオーバーライドして，エラー画面は非表示にし，レスポンスを返すようにする


## サンプル

```php
<?php
declare(strict_types=1);

set_error_handler(function($errno, $errstr, $errfile, $errline) {
    throw new ErrorException($errstr, $errno, 0, $errfile, $errline);
});

set_exception_handler(function ($exception) {
    echo "Uncaught exception: " . $exception->getMessage() . "\n";
    exit;
});

try {
    // TypeError
    typeError('1', 1);
    typeError(1, '1a');

    // AssertionError
    assert_options(ASSERT_EXCEPTION, 1);
    assert(false);

    // ParseError
    eval('$content = (100 - );');

    // ErrorException
    echo HELLO;
    $arr = ['hoge' => 'foo'];
    echo $arr['bar'];

    // Error
    $test = null;
    $test->test();
    
    // Stack Throw
    catchExceptionTest();
}catch (AssertionError $e) {
    do {
        printf("%s:%d %s (%d) [%s]\n", $e->getFile(),$e->getLine() $e->getMessage(), $e->getCode(),get_class($e));
    } while ($e = $e->getPrevious());
    exit;
} catch (ParseError $e) {
    do {
        printf("%s:%d %s (%d) [%s]\n", $e->getFile(),$e->getLine() $e->getMessage(), $e->getCode(),get_class($e));
    } while ($e = $e->getPrevious());
    exit;
} catch (ErrorException $e) {
    do {
        printf("%s:%d %s (%d) [%s]\n", $e->getFile(),$e->getLine() $e->getMessage(), $e->getCode(),get_class($e));
    } while ($e = $e->getPrevious());
    exit;
} catch (TypeError $e) {
    do {
       printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(),$e->getMessage(), $e->getCode(), get_class($e));
    } while ($e = $e->getPrevious());
} catch (Error $e) {
    do {
       printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(), $e->getMessage(), $e->getCode(), get_class($e));
    } while ($e = $e->getPrevious());
} catch (Exception $e) {
    do {
       printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(), $e->getMessage(), $e->getCode(), get_class($e));
    } while ($e = $e->getPrevious());
} catch (Throwable $e) {
    do {
       printf("%s:%d %s (%d) [%s]\n", $e->getFile(), $e->getLine(), $e->getMessage(), $e->getCode(), get_class($e));
    } while ($e = $e->getPrevious());
}

function helloUndefined() {
    echo HELLO;
}

function typeError(int $a, string $b) {
    echo $a + $b;
}

function nestExceptionTest() {
    throw new Exception('nestExceptionTest');
}

function catchExceptionTest() {
    try {
        nestExceptionTest();
    } catch (Exception $e) {
        throw new Exception('catchExceptionTest', 0, $e);
    }
}

```


## 参考
* [PHP7 で堅牢なコードを書く - 例外処理、表明プログラミング、契約による設計](https://speakerdeck.com/twada/php-conference-2016)
* [PHPにおける例外クラスの設計考察](http://blog.tojiru.net/article/455279557.html)
* [お前は PHP 7 における Fatal Error / Catchable Fatal Error / Error / ErrorException / Exception の違いを言えるか？](https://qiita.com/mpyw/items/c69da9589e72ceac470c)
* [複雑さに潜り込む - 大規模PHPアプリケーションにおける例外・モニタリング・ロギング](https://suzuken.hatenablog.jp/entry/2016/12/15/122130)
* [CodeIgniter3のエラー・例外処理](https://qiita.com/atwata/items/b088a8f0621f227d4294)
