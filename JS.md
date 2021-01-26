# ES5

## 函数调用

### q1：call()，apply()，bind()的用法

答：官方解释是这样的

call()：调用对象的方法，用另一个对象替换当前对象

apply()：调用函数，用指定的对象替换函数的this值，用指定的数组替换函数的参数

bind()：对于给定的函数，创建与原始函数具有相同主体的绑定函数。绑定函数的this对象与指定对象关联，并具有指定的初始参数

大致看一下意思好像没什么区别，总结一句话：改变当前函数对象this指向

我们都知道如果在没有模块化划分的前提下，函数对象中的this是指向window对象的，即全局对象

```javascript
function fn() {
	console.log(this);
};
fn(); // Window {window: Window, …}
```

又或者在一个一般对象中声明一个函数对象，此时函数对象中的this是指向这个一般对象的

```javascript
var obj = {
	fn: function() {
		console.log(this);
	};
};
obj.fn(); // {fn: ƒ}
```

而call()，apply()，bind()的作用就是改变this的指向对象，下面我们分别执行这三个方法试一下

```javascript
function fn() {
	console.log(this);
};

var obj = {};

fn.call(obj); // {}
fn.apply(obj); // {}
fn.bind(obj)(); // {}
```

我们发现call()，apply()方法都重复执行了一遍它们自己的调用对象（fn函数对象）中的语句，而bind()的官方定义为返回一个新的函数，所以就在后面添加了一个调用的语句，结果依然是执行了fn函数对象中的语句，而他们的共同点也显而易见，在fn单独执行时，函数中的this指向Window对象，而增加了这三个方法后，并且在方法中传了一个obj普通对象作为参数，fn中的this指向了obj普通对象，这样就证明了这三个方法不仅会先改变原函数对象中this的指向，而且又会执行一遍原函数对象的语句，下面贴一个伪代码增强理解

```javascript
fn.call(obj); ==> (function obj() {
                  	  console.log(this); // fn函数对象中的语句
                  })();
```

那么如果我们在fn方法对象中增加几个形参呢？在执行这三个方法时会有影响吗？下面我们来仔细分析一下

```javascript
function fn(a, b) {
	console.log(this + '参数：' + a + ' ' +  b);
};

var obj = {};

fn.call(obj); // [object Object]参数：undefined undefined
fn.apply(obj); // [object Object]参数：undefined undefined
fn.bind(obj)(); // [object Object]参数：undefined undefined
```

我们在fn函数对象中传了a和b两个形参，但是在调用这三个方法时仅仅传了一个obj作为参数，没有对这两个参数进行赋值，所以打印出undefined，下面我们按照官方定义的方式对这三个方法的参数进行赋值

```javascript
function fn(a, b) {
	console.log(this + '参数：' + a + ' ' +  b);
};

var obj = {};

fn.call(obj, '我是a', '我是b'); // [object Object]参数：我是a 我是b
fn.apply(obj, ['我是a', '我是b']); // [object Object]参数：我是a 我是b
fn.bind(obj, '我是a', '我是b')(); // [object Object]参数：我是a 我是b
```

我们发现除了apply()外，其他两个方法都可以依次为a和b进行赋值，并且同样重复执行了一遍fn函数对象中的语句，而apply()的官方要求第二个参数必须为一个数组，当传入实参时，该方法会把这个数组拆开分别赋给对应的形参

# ES6

## 面向对象

### q1：父类的构造函数中声明的属性跟子类的构造函数有什么联系？

答：为什么子类中还要再写一个构造函数？不写这个构造函数的话也能用啊，子类中的构造函数肯定有一些用，但是不知道它到底是干什么的？网上说的都是如果不写constructor或者不调用super方法的时候会报错，但是我这也没报错啊，该传值还是传值，该用还是可以用。无论是超类还是子类，都会默认自带一个constructor(){}函数，即使不写，这个类中也有，就像是每个对象都有一个原型一样，在超类中，如果需要在超类中自定义属性，就必须在超类的constructor(){}中声明，否则是真的会报错，而超类中的constructor(){}的作用不只是声明，它还有一个初始化超类中自定义属性的作用，虽然可以在自定义属性时直接定义值，但是通用语法是要写在constructor(){}中，在子类中，则不需要去写constructor(){}函数，因为extends已经把超类中的所有属性都继承过来了，但是并没有为其赋值，除非在超类中已经定义好了，这时候在子类的constructor(){}中就可以为其继承过来的属性初始化值，初始化之后，即使在创建子类实例时传入了值，也仍然不起作用

### q2：对象原型实现继承

答：

一、`__proto__`实现继承：每个对象都有一个特殊的属性：`__proto__`，可以用这个属性去关联另一个对象实现继承

```javascript
var animal = {
	name: "animal",
	eat: function() {
		console.log(this.name + " is eating");
	}
};

var dog = {
	name: "dog",
	__proto__: animal
};
dog.eat(); // dog is eating
```

但是这个方式的缺点是`__proto__`属性直到ES6才添加到规范中，存在兼容性问题，并且直接使用`__proto__`来改变原型链对性能不太友好

二、构造函数实现继承：在`__proto__`之上做了一些变通，让JS可以像Java那样new出来对象

```javascript
function Animal(name, sex) { // 构造函数
    this.name = name;
    this.sex = sex;
    this.eat = function() {
        console.log(this.name + ' is eating', 'sex：' + this.sex);
    };
}

var dog = new Animal('dog', 'male');
dog.eat(); // dog is eating sex：male

var cat = new Animal('cat', 'female');
cat.eat(); // cat is eating sex：female

console.log(dog.eat === cat.eat); // false
```

但是这个方法的缺点是构造函数的每个属性和方法都会在实例对象中重新创建一遍，我们实际需要的只是name属性和sex属性不同而已，在new的时候dog实例和cat实例都会创建（this.eat = new Function('')）一个eat方法，但两个方法并不相等，不同实例上的同名函数是不相等的，这样会同时占用两块内存，造成内存浪费，或许我们也可以把eat方法设置成全局函数试一下

```javascript
function Animal(name, sex) { // 构造函数
	this.name = name;
	this.sex = sex;
	this.eat = eat;
}

function eat() {
	console.log(this.name + ' is eating', 'sex：' + this.sex);
}

var dog = new Animal('dog', 'male');
dog.eat(); // dog is eating sex：male

var cat = new Animal('cat', 'female');
cat.eat(); // cat is eating sex：female

console.log(dog.eat === cat.eat); // true
```

这时候虽然只占用了一块内存，并且两个实例对象上的方法相等了，但是又出现了一个新问题，在全局作用域上定义的函数对象应该只能被某个唯一的对象调用，这样做就造成了全局污染问题，如果实例对象需要定义很多方法，那么就要定义很多个全局函数，那么又怎么能保证方法名不会重复呢？

我们先来分析一下new的时候到底做了什么事

```javascript
function Animal(name, sex) { // 构造函数
    this.name = name;
    this.sex = sex;
    this.eat = function() {
        console.log(this.name + ' is eating', 'sex：' + this.sex);
    };
    console.log(this); // 没有new出实例对象前，this指向window，实例化后会改变this指向，指向实例对象
}
```

1.创建一个新的对象

```javascript
var dog = {};
```

在我们正常声明一个一般对象时，对象中有`__proto__`属性但是没有prototype属性，但是当我们声明一个函数对象时，对象中不仅有`__proto__`属性同时还有了一个prototype属性

```javascript
var obj = {};
console.log(obj.__proto__) // {constructor: ƒ, …}
console.log(obj.prototype); // undefined

var fn = function() {}
console.log(fn.__proto__); // ƒ () { [native code] }
console.log(fn.prototype); // {constructor: ƒ}
```

2.将构造函数的作用域赋给新对象并执行构造函数中的代码

```javascript
dog.__proto__ = Animal.prototype;
Animal.call(dog);
```

Animal.prototype相当于是”**一、`__proto__`实现继承**“中的父类

3.返回新对象

```javascript
return dog;
```

下面拟写一个new方法

```javascript
function Animal(name, sex) { // 构造函数
    this.name = name;
    this.sex = sex;
    this.eat = function() {
        console.log(this.name + ' is eating', 'sex：' + this.sex);
    };
    console.log(this); // 没有new出实例对象前，this指向window，实例化后会改变this指向，指向实例对象
}

function myNew(Parent, ...prop) {
    var Child = {};
    Child.__proto__ = Parent.prototype;
    Parent.apply(Child, prop);
    return Child;
}

var dog = myNew(Animal, 'dog', 'male'); // Animal {name: "dog", sex: "male", eat: ƒ}
dog.eat(); // dog is eating sex：male
```

三、原型实现继承：上面提到无论是把方法放在构造函数内部还是在构造函数外部声明一个全局方法，都有相应的缺点和弊端，有没有更好一些的方法呢？在”**二、构造函数实现继承**“中我们提到当创建一个函数对象时，函数对象中是会有一个prototype属性的，而一般对象中是没有这个属性的，所以我们可以通过函数对象的原型去找到对应的方法和属性

```javascript
function Animal(name, sex) {
	this.name = name;
	this.sex = sex;
}
Animal.prototype = {
	eat: function() {
		console.log(this.name + ' is eating', 'sex：' + this.sex);
	};
};

var dog = new Animal('dog', 'male');
dog.eat(); // dog is eating sex：male
```

