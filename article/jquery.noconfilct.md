# jQuery.noConflict()

## 简要介绍

让出jQuery对“$”变量的控制权，当可选布尔参数值为true时，连同变量“jQuery”的控制权也让出。详细API请参考官网[jQuery.noConflict()](http://api.jquery.com/jQuery.noConflict/)。

## 代码解析

关于jquery.noconflict方法的代码有两段，一段自然是noconflict的方法体，另一段在jquery代码的头部，jquery定义一系列局部变量以供自身使用的地方。在其中我们可以看到这两个变量的声明和定义，“_jquery”变量保存之前“window.jQuery”变量的值，“_$”则保存“window.$”，无论这些全局变量之前的内容是什么，都作为备份保存起来，防止被覆盖，以便后面调用noconflict的时候可以恢复。

```
// Map over jQuery in case of overwrite
_jQuery = window.jQuery,

// Map over the $ in case of overwrite
_$ = window.$,
```

了解了jquery初始化变量时做的这些工作，那么关于noconflict方法所做的事情应该也就能猜到一二了。没错，它所做的事情就是将当前的“$”变量内容还原为原先存储的“_$”的内容。当然，这得是在确认当前“$”变量确实是指向jQuery对象的时候。假如当前“$”变量的控制权并不在jQuery手上（即“$”变量并不指向jQuery时），则不会将“$”变量还原。为什么呢？因为jQuery认为，当“$”变量的控制权不在自己手上的时候，强制还原其值只会引起多个“$”使用者之间的混乱。

另外，假如有传入真值参数的话，那么也对“jQuery”变量进行判断和让出操作，其过程与“$”变量的让出过程一致，不赘述。

最后，返回jQuery变量。这里的jQuery变量是jQuery匿名函数中的局部变量，其存储的值肯定为jQuery对象自身。当noconflict方法被传入真值对象调用的时候，这里的返回值将成为当前jQuery库的jQuery对象的唯一引用（假如之前没有备份jQuery引用的话）。因此一般会将这个返回值用另一个变量保管以作后用，除非是以后再也不使用jQuery相关变量。

```
noConflict: function( deep ) {
    if ( window.$ === jQuery ) {
        window.$ = _$;
    }

    if ( deep && window.jQuery === jQuery ) {
        window.jQuery = _jQuery;
    }

    return jQuery;
}
```