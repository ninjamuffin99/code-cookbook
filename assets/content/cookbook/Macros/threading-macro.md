[tags]: / "pipe,macro,thread,operator,clojure"

# Threading macro like Clojure and pipe operator

## Introduction

Threading macros (or arrow macros) are used in Clojure for compose many function calls.


Other language like Elixir, LiveScript or F# use a pipe operator &#124;&gt; for the same functionality.

In Haxe we can create the same utility with a simple macro. Thanks to the static analyzer we can eliminate the temp variables 
used for memorize the intermediate values.

## The code macro

```haxe
import haxe.macro.Expr;
import haxe.macro.Context;
using haxe.macro.ExprTools;
using haxe.macro.MacroStringTools;


class FunctionThread {
  public static macro function thread(exprs:Array<Expr>) {
    var exprs = [ for (expr in exprs) macro var _ = $expr  ];
    exprs.push(macro _);
    return macro $b{exprs};
  }
}
```

## A simple example

```haxe
import FunctionThread.thread;

class Main {
  static function sum(a, b) return a + b;

  static function main() {
    var x = 0;
    var res = thread(
      sum(x,1),
      sum(_,1),
      sum(_,2),
      sum(3,_)
    );
    trace(res);
  }
}
```

## The code generated in JavaScript without the static analyzer

```js
// Generated by Haxe 3.3.0
(function () { "use strict";
var HxOverrides = function() { };
HxOverrides.iter = function(a) {
    return { cur : 0, arr : a, hasNext : function() {
        return this.cur < this.arr.length;
    }, next : function() {
        return this.arr[this.cur++];
    }};
};
var Main = function() { };
Main.sum = function(a,b,c) {
    if(c == null) {
        c = 0;
    }
    return a + b + c;
};
Main.main = function() {
    var x = 0;
    var _ = Main.sum(x,1);
    var _1 = Main.sum(_,1,5);
    var _2 = Main.sum(_1,2);
    var _3 = Main.sum(3,_2,7);
    var res = _3;
    console.log(res);
};
Main.main();
})();
```

## The code generated with the static analyzer

In this case thanks to the static analyzer we don't need of the temp variables

```js
// Generated by Haxe 3.3.0
(function () { "use strict";
var HxOverrides = function() { };
HxOverrides.iter = function(a) {
	return { cur : 0, arr : a, hasNext : function() {
		return this.cur < this.arr.length;
	}, next : function() {
		return this.arr[this.cur++];
	}};
};
var Main = function() { };
Main.sum = function(a,b) {
	return a + b;
};
Main.main = function() {
	console.log(Main.sum(3,Main.sum(Main.sum(Main.sum(0,1),1),2)));
};
Main.main();
})();
```

## With inline function 

With inline of sum and static analyzer the output is:
```
console.log(7);
```