# jQuery.each()

## 简要介绍

一个通用的迭代器，即可以用于对象，也可以用于数组。详细说明请参考官网 - [jQuery.each()](http://api.jquery.com/jQuery.each/)。

## 代码解析

首先我们看一下方法签名，有三个参数，第一个是要遍历的目标对象（无论其为数组还是普通对象），第二个是迭代的回调方法，至于第三个，看官网的API中似乎并没有关于这第三个参数的内容。而看源代码中方法前面的注释我们可以知道，这第三个参数是仅用于程序内部的。虽然javascript并没有什么方法可以阻止我们在外部代码使用这第三个参数，但还是建议尽量不使用这第三个参数。因为对于没写进API的“隐藏方法”，开发者随时都可以在下一个版本废弃或者改变签名。

```
// args is for internal usage only
each: function( obj, callback, args ) {
	// 内部代码在下面续讲
}
```

方法内部首先声明了一些变量，并初始化了length，用于指示所迭代数组的长度（如果所迭代对象为普通对象则用不到），以及isArray，指示所迭代对象是否具有数组形态，使用了内部方法[isArrayLike(obj)]()。

```
var value,
	i = 0,
	length = obj.length,
	isArray = isArraylike( obj );
```

声明完成之后，后面的代码主要分为两部分，第一部分是处理有传args的情况，第二部分是处理被外部程序调用的常见情况。我们看有传args的情况下，如果目标对象是具有数组形态的对象，那么使用数字下标遍历直到等于length。遍历的时候向回调函数传递目标对象在该下标的值以及args，然后判断回调函数返回值，如果为false则跳出循环；如果目标对象不是具有数组形态的对象，如普通对象的话，那么就使用in关键字遍历对象中的属性，遍历的时候向回调函数传递目标对象当前属性中的值以及args，然后也是根据返回值判断是否继续下去。

```
if ( args ) {
	if ( isArray ) {
		for ( ; i < length; i++ ) {
			value = callback.apply( obj[ i ], args );

			if ( value === false ) {
				break;
			}
		}
	} else {
		for ( i in obj ) {
			value = callback.apply( obj[ i ], args );

			if ( value === false ) {
				break;
			}
		}
	}
}
```

其实不传递args的情况下的代码也是跟上面的类似，唯一的不同就是向回调函数传的参数，在这里我们传的分别是当前index以及当前下标或属性下的值。注意这里调用回调函数所使用的是call方法，而不是上面的apply，而且call方法的第一个参数为目标对象当前下标或属性下的值，这意味着我们在回调函数中使用this时，它的值将会是当前下标或属性下的值，与回调函数中传给我们的第二个参数一样。

```
// A special, fast, case for the most common use of each
else {
	if ( isArray ) {
		for ( ; i < length; i++ ) {
			value = callback.call( obj[ i ], i, obj[ i ] );

			if ( value === false ) {
				break;
			}
		}
	} else {
		for ( i in obj ) {
			value = callback.call( obj[ i ], i, obj[ i ] );

			if ( value === false ) {
				break;
			}
		}
	}
}
```

最后返回目标对象，可用于后续链式调用。

```
return obj;
```
