# const 与 final

[TOC]

概念上说，final 表示不可变，只能赋值一次，初次赋值之后就不能再赋其它值了，是只读变量。而 **const** 表示常量，更准确地说是编译时常量，因此 **const** 也是只读的、不可变的。**const** 和 final 都可以表示不可变，但又有细微的差异。 

**区别一**：final 要求变量只能初始化一次，并不要求赋的值一定是编译时常量，可以是常量也可以不是。而 **const** 要求在声明时初始化，并且赋值必需为编译时常量。

**区别二**：final 是惰性初始化，即在运行时第一次使用前才初始化。而 **const** 是在编译时就确定值了。

 **区别三**：const 可以修饰变量，也可以修饰值（value）。而 final 只用来修饰变量。

### 变量声明

var 或者类型可以声明变量，而 **const** 和 final 也可以用于声明变量。声明 var 的地方可以用 **const** 或 final 代替。声明类型的地方可以与 **const** 或 final 组合使用。例如下面这些声明都是可以的： 

```dart
var a = 1;

const a = 1;

final a = 1;



int a = 1;

const int a = 1;

final int a = 1;
```

这里 **const** 和**final **没有区别，因为变量 a 被立即赋值为一个数字常量。



### 编译时常量

**编译时常量**指的是：字面量（如数字、bool、字符串、List的字面量形式）、其它常量 或者 常量的算术运算，也可以是这些的组合形式（递归要求），简单地说常量就是可以在编译时确定的值。

```dart
const a = 8;

const b = false;

const c = a;

const d = 5 * 3;

const e = a * d + 2;
```

而其它形式，比如一个函数调用的返回值、实例化一个新的对象就不是编译时常量了，它们在运行时才能获得结果。 

```dart
final x = new DateTime.now();//ok

const x = new DateTime.now();//error


final y = sin(90);//ok

const y = sin(90);//error
```



### final,const 实例成员

类中的 final 成员要么在声明时直接初始化，要么在初始化列表中初始化： 

```dart
class X {
  final a;
  X(this.a){
  }
}

//或

class X {
  final a;
  X(a): this.a = a{
  }
}

//或

class X {
  final a = 0;
  X(){
  }
}
```

**const 实例成员**只能直接声明。 

```dart
class X {
  const a = 0;
  X(){
  }
}
```



### 常量对象和常量构造函数

编译时常量还包括常量对象，也就是通过 **const** **构造函数**创建的对象。

如果一个类的所有实例成员都是**final **或 **const** 实例成员，那么该类所创建的对象的状态就是不可变的。这种情况下就可以定义 **const** **构造函数**，**const** **构造函数**所创建的对象就是常量对象。在语法上，**const** **构造函数**要在**构造函数**前面加上 **const** 修饰符，同时用 **const** 代替 new 来调用**构造函数**。也可以用 new 调用 **const** **构造函数**，但那样就不能用 **const** 声明该变量了。此外，**const** **构造函数**也不能有函数体。

```dart
class X {

  // all instance variables are final or const
  const a = 1;
  final b;

  const X(this.b); // no body

  static const m = const X(3);//ok, a static const member

}



main(){

  const x = const X(3);// ok

  const x = new X(3);// error

  final x = const X(3);// ok

  final x = new X(3);// ok

}
```

常量对象还有一个特点，相同的常量对象始终只有一个。 

```dart
identical(const X(3), const X(3)) // =>true

getConst() => const [1, 2]; 
main() { 
  var a = getConst(); 
  var b = getConst(); 
  identical(a, b); // =>true 
} 
```



### const修饰变量值

const 可以修饰变量，也可以修饰值（value）。而 final 只用来修饰变量。 

```dart
const list = [1, 2, 3]; // error

const list = const [1, 2, 3]; // ok

const list = const [new DateTime.now(), 2, 3]; // error, because new DateTime.now() is not const

const list = const [const X(3), 2, 3]; // ok



var x = 5;

const list = const[x]; //error



const x = 5;

const list = const[x];//ok



//final、var或类型定义的变量都不要求右边必需是 const，但右边的 const 修饰符要求值必需是常量

final list = [1, 2, 3]; // ok

final list = const [1, 2, 3]; // ok

final list = const [new DateTime.now(), 2, 3]; // error, because new DateTime.now() is not const
```

**const** 修饰值的时候，要求值必需是常量或由常量组成。**var、final**等在左边定义变量时，并不关心右边是不是常量。但如果右边用了 **const**，那么不管左边是什么要求，右边都必需是常量。 











