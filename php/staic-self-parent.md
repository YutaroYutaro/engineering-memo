## `static::` や `new static()` について
* 遅延静的束縛という機能で`static`や`::`とは関係がない

[遅延静的束縛 (Late Static Bindings)](https://www.php.net/manual/ja/language.oop5.late-static-bindings.php)

## `parent::`について
* 親クラスのメソッドと静的プロパティを呼び出す
* 親クラスのメソッドは静的メソッドじゃなくても呼べる
  * `::` の定義を逸脱している
  * オーバーライドされていないメソッドは本来であれば `parent->` で呼び出されるべき?
* 親クラスのプロパティは静的プロパティじゃないとエラー

```php
<?php

class P
{
    public $foo;
    public static $staticFoo = 'P: staticFoo';

    public function __construct($foo)
    {
        $this->foo = $foo;
    }

    public function Foo()
    {
        echo 'P: Foo Method' . PHP_EOL;
    }

    public static function StaticFoo()
    {
        echo 'P: StaticFoo Method' . PHP_EOL;
    }

    public function Bar()
    {
        echo 'P: Bar Method' . PHP_EOL;
    }
}

class C extends P
{
    public static $staticFoo = 'C: staticOverRideFoo';

    public function Foo()
    {
        parent::Foo();                      // オーバーライドされたメソッド
        parent::StaticFoo();                // 静的メソッド
        parent::Bar();                      // 普通のメソッド
        echo parent::$staticFoo . PHP_EOL;  // 静的プロパティ
        echo $this->foo . PHP_EOL;

        // echo parent::$foo . PHP_EOL;        // fotal error
    }
}
```
```php
$c = new C('Foo');
$c->Foo();
// 出力
// P: Foo Method
// P: StaticFoo Method
// P: Bar Method
// P: staticFoo
// Foo
```

## `{class}::`について
* 静的メソッドではなくても参照できる
  * エラー出力レベルをあげると `PHP Deprecated` が出る
* インスタンスを生成しているわけではないので `$this` は参照できない
* インスタンスから静的メソッドは参照できない
  * `$a->StaticFoo()`
* インスタンスからメソッドを参照した時はメソッド内で静的プロパティを参照できない

```php
<?php

class A
{
    public $bar;
    public static $staticBar = 'A: staticBar Property';

    public function __construct($bar)
    {
        $this->bar = $bar;
    }

    public function Foo()
    {
        echo 'A: Foo Method' . PHP_EOL;
    }

    public static function StaticFoo()
    {
        echo 'A: StaticFoo Method' . PHP_EOL;
    }

    public function ThisBar()
    {
        echo $this->bar . PHP_EOL; // fatal error
    }

    public function SelfBar()
    {
        echo self::$bar . PHP_EOL; // fatal error
    }

    public function SelfStaticBar()
    {
        echo self::$staticBar . PHP_EOL;
    }
}
```
```php
// 出力
// A: Foo Method
// PHP Deprecated
// 静的じゃないメソッドを呼び出すべきではない
A::Foo();

// 出力
// A: StaticFoo Method
A::StaticFoo();

// Fatal Error
// インスタンスが生成されていないので$thisが存在しない
A::ThisBar();

// Fatal Error
// $barが静的プロパティではない
A::SelfBar();

// 出力
// A: staticBar Property
// PHP Deprecated
// 静的じゃないメソッドを呼び出すべきではない
A::SelfStaticBar();

// Fatal error
// $barが静的プロパティじゃない
echo A::$bar;

// 出力
// A: staticBar Property
echo A::$staticBar;


$a = new A('bar');

// 出力
// A: Foo Method
$a->Foo();

// 出力
// A: StaticFoo Method
$a->StaticFoo();

// 出力
// bar
$a->ThisBar();

// Fatal Error
// $barが静的プロパティではない
$a->SelfBar();

// 出力
// A: staticBar Property
$a->SelfStaticBar();

// 出力
// bar
echo $a->bar . PHP_EOL;

// PHP Notice
// $staticBar は静的プロパティだからA::$staticBarで参照して
echo $a->staticBar . PHP_EOL;
```