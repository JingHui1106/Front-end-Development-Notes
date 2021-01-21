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

而call()，apply()，bind()的作用就是改变这个this

# ES6

## 面向对象

### q1：父类的构造函数中声明的属性跟子类的构造函数有什么联系？

答：为什么子类中还要再写一个构造函数？不写这个构造函数的话也能用啊，子类中的构造函数肯定有一些用，但是不知道它到底是干什么的？网上说的都是如果不写constructor或者不调用super方法的时候会报错，但是我这也没报错啊，该传值还是传值，该用还是可以用。无论是超类还是子类，都会默认自带一个constructor(){}函数，即使不写，这个类中也有，就像是每个对象都有一个原型一样，在超类中，如果需要在超类中自定义属性，就必须在超类的constructor(){}中声明，否则是真的会报错，而超类中的constructor(){}的作用不只是声明，它还有一个初始化超类中自定义属性的作用，虽然可以在自定义属性时直接定义值，但是通用语法是要写在constructor(){}中，在子类中，则不需要去写constructor(){}函数，因为extends已经把超类中的所有属性都继承过来了，但是并没有为其赋值，除非在超类中已经定义好了，这时候在子类的constructor(){}中就可以为其继承过来的属性初始化值，初始化之后，即使在创建子类实例时传入了值，也仍然不起作用

### q2：对象继承

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

但是这个方法的缺点是构造函数的每个属性和方法都会在实例对象中重新创建一遍，dog实例和cat实例都会创建（this.eat = new Function('')）一个eat方法，但两个方法并不相等，不同实例上的同名函数是不相等的，这样会同时占用两块内存，造成内存浪费，或许我们也可以把eat方法设置成全局函数试一下

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

我们先来分析一下new的时候到底做了什么事，声明声明一个Animal构造函数

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

三、原型实现继承：上面提到无论是直接new出一个实例对象还是单独创建一个全局函数对象，都有相应的缺点和弊端，有没有更好一些的方法呢？在”**二、构造函数实现继承**“中我们提到当创建一个函数对象时，函数对象中是会有一个prototype属性的，而一般对象中是没有这个属性的，所以我们可以通过函数对象的原型去找到对应的方法和属性

