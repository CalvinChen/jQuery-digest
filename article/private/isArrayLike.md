# isArrayLike(obj)

## 简要介绍

jQuery内部方法，用于判断一个对象是不是具有数组的形态，或者说，能不能当作一个数组一样使用。

## 代码解析

函数开头处首先声明了两个变量，length获取目标对象的length属性，以及目标的类型。在这里获取目标的类型使用了[jQuery.type(obj)]()方法。

接下来，使用[jQuery.isWindow( obj )]()判断目标对象是否为全局的window对象。在这里做这样一个排除主要是因为浏览器window对象虽然不为array like对象，但是它也具有length这个array like对象必备的属性。

接着检测目标对象是否具有nodeType这个属性且其值为1，如果这个条件成立表明此目标对象是一个DOM元素，且与此同时如果它还具有length这个属性的话，那么就确定是array like对象了。

最后，执行最通用的判断。如果目标对象的类型为数组，那么毋庸置疑返回true；又或者目标对象的类型绝对不为函数的话，而且length绝对为0或者length肯定为数字且length大于0且目标对象中存在着length-1下标的属性，那么也返回true。其余情况皆返回false。

最后的通用判断有些难懂，但多读几遍，理解这里面的各种可能情况，慢慢也就可以理解了。

```
function isArraylike( obj ) {
	var length = obj.length,
		type = jQuery.type( obj );

	if ( jQuery.isWindow( obj ) ) {
		return false;
	}

	if ( obj.nodeType === 1 && length ) {
		return true;
	}

	return type === "array" || type !== "function" &&
		( length === 0 ||
		typeof length === "number" && length > 0 && ( length - 1 ) in obj );
}
```
