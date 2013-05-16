# jQuery.extend()

## 简要介绍

主要用于将两个或多个对象的内容合并到第一个对象上。详细API请参考jQuery官网 - [jQuery.extend()](http://api.jquery.com/jQuery.extend/)。

## 代码解析

首先，在方法的头部定义方法里面用到的变量，这是编程的一个好习惯。这时我们暂时先把第一个参数当作我们的目标对象，也就是最后得到所有对象内容的对象。

```
var options, name, src, copy, copyIsArray, clone,
    		target = arguments[0] || {},
    		i = 1,
    		length = arguments.length,
    		deep = false; 
```

按照API文档，当第一个参数是布尔类型的时候，这时用的其实是这个方法的另一种重载形式，即用第一个参数指示是否为深度拷贝，而原有参数则向后移。表现在代码中就是目标对象变为了第二个，同时指示变量i也后移了一位。（题外话：之前一直觉得javascript没有方法重载有些遗憾，使用起来不大方便。不过现在看来也算是有重载，用代码也能勉强实现。只不过，重载的参数设计就得遵循一些套路了，免得处理起来过于复杂。）

```
// Handle a deep copy situation
  if ( typeof target === "boolean" ) {
		deep = target;
		target = arguments[1] || {};
		// skip the boolean and the target
		i = 2;
	}
```

接下来就是对目标对象的处理。如果目标对象既不是对象也不是方法的话，那就把它变成空对象。我想这里可能有人会疑问为什么这里没有判断 `target` 为 `null` 或者为 `undefined` 等情况，这是因为前面 `target = arguments[i] || {}` 的时候已经对可能隐式转为false的值做了处理。

```
// Handle case when target is a string or something (possible in deep copy)
  if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
		target = {};
	}
```

再来就是再另外一种作用的方法重载处理了。按照API文档，当我们只是传递一个目标对象，没有其它对象与之合并的时候，即表示要将这个对象合并到jQuery对象上，此时目标对象就变成了jQuery，而原来的目标对象则变成了要被合并的对象。这是一个经常被使用的方法，无论是在jQuery源码本身，还是在开发者扩展jQuery的时候，都可以使用这个方法为jQuery增加自己需要的方法。回到代码本身，`length` 在这里是指示参数的个数， `i` 则是第一个非目标对象的待合并对象的下标。因此当这两个参数相等时，就是说只传递了一个目标对象，而没有后面其它的对象了。这个时候目标对象变为 `this` ，即jQuery。`i`后移一位，指向原来的目标对象，使之变成待合并对象。

```
// extend jQuery itself if only one argument is passed
  if ( length === i ) {
		target = this;
		--i;
	}
```
接上面，因为`i`是待合并对象的下标指示器，因此for循环直到最后一个参数。然后在循环内部判断，只对不为`null`的参数做处理。再然后又是一个循环，遍历参数的所有属性用于复制。这里的一句 `src = target[ name ];`  我个人觉得 `src` 这个变量名取得不好，第一次看有点丈二和尚摸不着头脑，因为看代码这 `src` 明显是用来保存目标对象上该属性的值的，却取的名字含义为“源”。再看这 `copy` ，是用于保存待合并对象上该属性的值的，取的名字似乎也有待商榷。好，先不管这名字，总之我们在这里是各自取出了目标对象和待合并对象上对应于该属性的值。

接下来这段代码有点废脑筋。当待合并对象该属性的值是指向目标对象的时候，跳过这一属性。什么意思？我个人理解是为了避免使得合并后的对象出现自引用。不过在我们的日常使用中，目标对象和待合并对象往往是两个完全不相关的对象，出现这种情况的几率较小。

```
for ( ; i < length; i++ ) {
  	// Only deal with non-null/undefined values
		if ( (options = arguments[ i ]) != null ) {
			// Extend the base object
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// Prevent never-ending loop
				if ( target === copy ) {
					continue;
				}

				// 判断是做遍历还是直接拿值，因代码太长不便阅读，故放到下面续讲。
			}
		}
	}
```

继续上面两个循环里面的代码。做了防止死循环的检验之后，接下来要做一个判断，假如是深拷贝，而且待合并对象该属性的值不为null，而且该值是普通对象或者是数组的话，进入深拷贝流程；如果不是，而且待合并对象该属性的值绝对不等于 `undefined` 的话，就将该值直接复制给目标对象。注意这里是绝对不等于 `undefined` ，如果为 `null` 的话，是可以的。

至于深拷贝流程，首先判断是为数组还是对象，只有这两种可能，然后构造一个待拷贝的对象。待合并对象属性值为数组时，如果目标对象原来有值且值为数组，那么就用原来的数组，不然建一个新的空数组给待拷贝对象；为对象时，如果目标对象原来有值且值为对象，那么就用原来的对象，不然新建一个新的对象给待拷贝对象。最后，再递归调用 `jQuery.extend` 方法，目标对象为我们刚刚构造的待拷贝对象，等构造完成后，传递给目标对象，便完成了。

个人觉得递归调用里面的第一个参数 `deep` 直接写字面量 `true` 更加直观点，毕竟都经过外层if的判断了，进到这里来的肯定是要深度拷贝的。

```
// Recurse if we're merging plain objects or arrays
  			if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
					if ( copyIsArray ) {
						copyIsArray = false;
						clone = src && jQuery.isArray(src) ? src : [];

					} else {
						clone = src && jQuery.isPlainObject(src) ? src : {};
					}

					// Never move original objects, clone them
					target[ name ] = jQuery.extend( deep, clone, copy );

				// Don't bring in undefined values
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
```
最后，就是返回目标对象了。

```
  // Return the modified object
	return target;
```
