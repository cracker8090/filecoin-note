





## go与其他语言区别

复合类型

interface,struct,function,map

Go不支持继承，而是支持组合

*Go does not support inheritance, but rather composition* 



## interface接口

在Go中，接口主要用于封装目的，并允许我们编写更简洁，更可靠的代码。通过这样做，我们仅在程序中公开方法和行为。如上一节所述，方法集将行为添加到一个或多个类型。

接口类型定义一个或多个方法集。因此，可以说一种类型通过实现其方法来实现接口。因此，接口使我们能够编写具有共同行为的自定义类型。

在Go中，接口是隐式的。这意味着，如果属于一个接口类型的方法集的每个方法都由一个类型实现，则称该类型来实现该接口。



如果不止一种类型实现相同的方法，则方法集可以形成接口类型。这样，我们就可以将该接口类型作为参数传递给打算实现该接口行为的函数。这样，可以实现多态



根据经验，当我们在程序包中开始具有多个类型来实现相同的方法签名时，便可以开始重构代码并使用接口类型。这样做避免了早期的抽象。






































