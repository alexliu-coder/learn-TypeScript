# TypeScirpt编程

### unknown

unknown是类似于any，但是需要在后续的代码中做**细化类型(p154)**，通过分支语句来细化类型；

- ts不会推到unknown类型，必须显示注解

### const

const声明的简单数据时，ts会把特定的值推导为类型，但是声明对象时，对象的键值对的类型不会缩窄；

### ts中对象的键的声明

```typescript
let a: {
  b: number,                     // 表示必须有
  c?: string,                    // 表示可选
  [key: number]: boolean,        // 表示任意个
  [seatNumber: string]: string,  // 不一定是key，可以取别的名字
  readonly firstName: string     // 不可以更改值的键
}
```

## 类型的变量声明

```type Age = number```
使用type来声明一个类型的变量，同一变量名不能声明两次，而且和let，const一样，type声明的类型变量也具有块级作用域

## 类型的并集和交集

类型的交并集使用& |来表示

- ’|’表示两个类型中的任意一个或者全是都可以，但必须要满足两个中的其中一个的类型
- ‘&’表示必须是两个类型的全部合并产生的新的类型，必须包含两个类型中的所有类型

```typescript
type Cat = {
	name: string,
	meow: boolean
};

type Dog = {
	name: string,
	bark: boolean
}

type CatOrDog = Cat | Dog   // 只要满足任意一个就可以，也可以都满足
type CatAndDog = Cat & Dog  // 必须满足连个类型中的所有
```

## 数组和元祖

数组的两种声明方式：```number[]``` 和 ```Array<number>```
数组的类型推导：

```typescript
let a = [];
a.push(1)      // number[]
a.push('abc')  // (number | string)[]
```

数组会在离开当前的定义域以后确定类型，不再扩张；

#### 元祖

元祖是array的子类型，长度固定，索引位置的类型固定

```typescrit
let a: [number] = [1];
let trainFares: [number, number?][]; // 元祖组成的数组，元祖也支持可选元素
let friends: [string, …string[]];    // 元祖支持剩余参数，定义最小长度
```

#### 只读的元祖和数组

只读的数组只能使用非型变的方法（.concat .slice），不能使用型变的方法（.push, .splice）;

```typescript
// array
type A = readonly string[];
type B = ReadonlyArray<string>;
type C = Readonly<string[]>;

// tuple
type D = readonly [number, number];
type E = Readonly<[number, number]>;
```

## null, undefined, void, never

null表示缺少值
undefined表示尚未定义
void表示函数不会返回值，比如执行一些打印类的函数，没有return
never表示函数被错误中断，或者是进入死循环，所以不会返回值

## 函数

默认参数无需放在末尾
可选参数必须放在末尾

```typescript
function test(message: string = '', id?: number): void {
	//
}
```

剩余参数：

```typescript
function test(…number: number[]): number {
	//
}
```

剩余参数可以解决任意数量参数传入时的类型校验，剩余参数必须放在末尾

## 函数签名

```typescript
// 签名的简写
type Log = (a: number, b: number) => number;

// 签名的正常写法
type ticketFare = {
    (from: Date, to: Date, destination: string): boolean;
    (from: Date, destination: string): boolean;
}

```

函数重载后还是需要将对应的参数做兼容处理，所以为什么不直接将兼容参数的函数作为函数签名呢？

## 泛型

多态的类型参数，相当于是未知类型的变量

```typescript
type Filter = {
	(array: number[], f: (n: number) => boolean): number[];
	(array: string[], f: (n: string) => boolean): string[];

	// 泛型可以只写一个来包含上面的两条
	<T>(array: T[], f: (n: T) => boolean): T[];
}

// 泛型实现filter函数
type Filter = {
    <T>(array: T[], f: (n: T) => boolean): T[] 
}

let filter: Filter = (arr, f) => {
    const result = []
    for (let i = 0; i < arr.length; i++) {
        f(arr[i]) && result.push(arr[i]);
    }
    return result
}

let ar = [1, 'a']

// 给定了arr以后，传入的函数n的类型也就会自动推导出来
filter(ar, (n) => typeof n === 'number')
```

补充：**泛型的第二个写法**

```typescript
type Filter<T> = {
	(array: T[], f: (item: T) => boolean): T[];
}

let filter: Filter<number> = ...
```

map函数的类型:

```typescript
type Map = {
	<T, U>(array: T[], f: (item: T) => U): U[];
}
```

promise的泛型写法:

```typescript
let promise = new Promise<number>(resolve => {  // 在promise后边显式注解
	resolve(34)
})

promise.then(res => {...});   // res此时的类型为number
```

泛型别名:

```typescript
// p94
type MyEvent<T> = {
	target: T,
	type: string
}

type MyButtonEvent = MyEvent<HTMLButtonElement>;
type TimedEvent<T> = {
	event: MyEvent<T>,
	from: number,
	to: number
}

// 函数后面必须声明T，不然在MyEvent后面的T就是未定义的
function triggerEvent<T>(event: MyEvent<T>): void {...}

triggerEvent({
	target: document.querySelector('#my-button'),
	type: 'click'
})
```

受限的多态

```typescript
type TreeNode = {
	value: string
}

type LeafNode = TreeNode & {
	isLeaf: true
}

type InnerNode = TreeNode & {
	children: [LeafNode] | [LeafNode, LeafNode]
}

// 规定泛型至少是TreeNode
function map<T extends TreeNode>(node: T, f: (value: string) => string): T {
	return {
		...node,
		value: f(node.value)
	}
}
```

使用受限的多态来模拟变长参数 p99

```typescript
function call(f: (...args: unknown[]) => unknown, ...args: unknown[]): unknown {
	return f(...args)
}

function fill(length: number, value: string): string[] {
	let result = [];
	for (let i = 0; i < length; i++) {
		result.push(value)
	}
	return result
}

// 使用泛型来模拟
function call<T extends unknown[], R>(f: (...args: T[]) => R, ...args: T[]): R {
	return f(...args)
}

call(fill, 2, 'abc')  // 因为fill的参数类型已经给定，所以T[]的类型也确定了，call函数的剩余参数也被确定了
```

泛型的默认类型

```typescript
// 泛型
type MyEvent<T> = {
	target: T,
	type: string
}
// 限制泛型
type MyEvent<T extends HTMLElemet> = {
	target: T,
	type: string
}
// 限制泛型并默认
type MyEvent<T extends HTMLElement = HTMLElement> = {
	target: T,
	type: string
}
```

*注意：有默认类型的泛型需要放在没有默认类型的泛型后面*

exercises: [Hard] Update our call implementation from earlier in the chapter (Using
Bounded Polymorphism to Model Arity) to only work for functions whose second
argument is a string. For all other functions, your implementation should
fail at compile time.

```typescript
function call<T extends [unknown, string, ...unknown[]], R>(
	f: (...args: T) => R,
	...args: T
): R {
	return f(args);
}
```

## 类和继承

构造方法中的修饰符

```typescript
type Color = 'write' | 'black';
class Piece {
	constructor (
		private readonly color: Color
	) {}
}

class Piece {
	protected color: Color;
	constructor (color: string) {
		this.color = color
	}
}
```

public 任何地方都可以访问的属性和方法
protected 只能是当前类及其子类访问
private 只能是当前类访问

抽象类

```typescript
abstract class Piece {
	constructor () {};
	moveTo (position: [number, number]) {...};
	abstract move(position: [number, number]): boolean;
}
```

抽象类不能初始化，但是还是可以在抽象类中新增方法
子类继承了抽象类以后，必须实现抽象类的方法

ts中也可以使用super关键字来访问父类的方法

#### 以this返回的类型

this可以作为js的值，也可以作为类型，对类来说，可以用于注解方法的返回类型

```typescript
class Set {
	add(value: any): this {
		// ...
	}
}

class MutableSet extends Set {
	// mutable set的add方法需要返回的是mutableSet，所以add方法需要返回this
}

```

## 接口

接口和类型关键字type类似

```typescript
type food = {
	name: string
}
type susi = food & {
	salty: boolean
}

interface food = {
	name: string
}
```

interface 和 type 的区别：

1. type的右边可以是任何类型，包括类型的表达式，运算符号；而interface的右边必须为结构
2. 扩展接口的时候ts将检查扩展的接口是否可赋值给被扩展的接口；使用type和&扩展的时候，会使函数重载
3. 同一个作用域的多个同名接口将自动合并，但是同一个作用域的多个类型别名会导致编译错误(p115声明合并)

```typescript
// 2.接口扩展示例
interface A {
	good(x: number): string
	bad(x: number): string
}

interface B extends A {
	good(x: string | number): string   // correct (x: string | number): string 可以赋值给 (x: number): string
	bad(x: string): string             // error (x: string): string不可以赋值给 bad(x: number): string
}
```

## 类的实现

声明类的时候使用implements关键字来指明该类满足的接口, 一个类可以实现多个接口

```typescript
interface Animal {
	eat(food: string): void
	sleep(hours: number): void
}

interface DogType {
	bark(): void
}
class Dog implements Animal, DogType {
	bark() {}
	eat(food) {}
	sleep(hours) {}
}
```

#### 类的泛型

1. 在声明类的时候声明泛型，此处声明的泛型可以在实例方法和实例属性使用
2. 构造方法不能声明泛型
3. 实力方法也可以有自己的泛型，也可以使用一级的泛型
4. 静态方法不能访问类的泛型

```typescript
class MyMap<T, U> {
	constructor(key: T, value: U) {
		
	}
	get(key: T): U {
		//...
	}
	set(key: T, value: U): void {

	}
	merge<K1, K2>(map: MyMap<T, U>): MyMap<T | K1, U | K2> {
		
	}
	static of<T, U>(k: T, v: U): MyMap<T, U> {

	}
}
```

## 混入

```typescript
type ClassConstructor<T> = new(...args: any[]) => T;

function EZDebug<C extends ClassConstructor<{ getDebugValue(): object>>(Class: C) {
	return class extends Class {
		debug() {
			let name = Class.constructor.name;
			let value = this.getDebugValue();
			return name + value
		}
	}
}
```

## 类型进阶

#### 子类和超类

any => Object => Array => Tuple => never 右边是左边的子类，左边是右边的超类

A >: B   B是A的子类
A <: B   B是A的超类

#### 函数的型变

1. 函数A的参数数量小于函数B
2. 函数A的this类型微指定，或者>:B的this
3. 函数A的各个参数类型>:函数B的相应参数
4. 函数A的返回类型<:函数B的返回类型

满足上述4个条件，则函数A是函数B的子类型；(p144)

#### 类型拓宽

Typescript在推导类型时，会推到出一个更宽的类型

let 和 var 赋值的时候，类型会拓宽
const 赋值的时候类型会锁定

const作为类型断言

```typescript
let c = { x: 3 } as const;

let e = [1, {x: 2}] as const;  // readonly [1, {readonly x: 2}]
// const会阻止类型拓宽并且递归将成员设置为readonly
```

```typescript
type Option = {
    baseURL: string
    cacheSize?: number
    tier?: 'prod' | 'dev'
}
class API {
    constructor(private option: Option) {
    }
}

new API({baseURL: '123', badTier: 'dev'} as Option) // 让ts不进行属性检查


let badOption = {
    baseURL: '123',
    badTier: 'dev'
}
// 把选项对象赋值给变量badOption，typescript不在把它视为新鲜对象，因此不做多余的检查
new API(badOption) // ok 不会报错
```

使用分支来推到类型，辨别并集类型
handle不可以区分，handle2可以区分出text和mouse事件

通过标记来辨别类型的原则:

1. 标记需要再并集类型的各组成部分的相同位置上
2. 使用字面量类型(字符串、数字、布尔值等)，最好使用同一种字面量
3. 不要使用泛型
4. 要互斥

```typescript
type UserTextEvent = {
    type: 'input'
    value: string
    target: HTMLInputElement
}

type UserMouseEvent = {
    type: 'click'
    value: [number, number]
    target: HTMLElement
}

type UserEvent = UserMouseEvent | UserTextEvent;

function handle(event: UserEvent) {
    if (typeof event.value === 'string') {
        console.log(event.value)
        console.log(event.target)
        return
    }
    console.log(event.value)
    console.log(event.target)
}

function handle2(event: UserEvent) {
    if (event.type === 'input') {
        console.log(event.value)
        console.log(event.target)
        return
    }
    console.log(event.value)
    console.log(event.target)
}

```

#### 键入

通过键来获取类型

```typescript
type UserInfoRes = {
    user: {
        name: string
        id: string
        friendList: {
            count: number
            friend: {
                firstName: string
                lastName: string
            }[]
        }
    }
}

type friendList = UserInfoRes['user']['friendList']

type friend = friendList['friend'][number] // number是‘键入’数组类型的方式，如果是元祖，可以使用0、1或其他数字字面量
```

#### keyof 运算符

```typescript
type Test = {
    a: string
    b: number
    c: boolean
}

// K extends keyof O 表示K至少是 'a' | 'b' | 'c' 
function get<O extends object, K extends keyof O>(o: O, k: K): O[K] {
    return o[k];
}

let test: Test = {
    a: '123',
    b: 123,
    c: true
}

get(test, 'b') // return number
```

#### Record关键字

```typescript
type WeekDay = 'mon' | 'tue' | 'wen' | 'thu' | 'fri'

type Day = WeekDay | 'sat' | 'sun'

let nextDay: Record<WeekDay, Day> = {
    mon: 'tue',
    tue: 'ddd'  // error!!! 'ddd' is not assignable to type 'Day'
}
```

Record<T, U>表示键和值的对应关系，键的类型是T，值的类型是U

in 关键字：

1. 作为js的判断使用，k in obj判断obj[k]是否存在
2. 在ts中遍历UnionType来使用
<https://stackoverflow.com/questions/50214731/what-does-the-in-keyword-do-in-typescript>

keyof 关键字：
接口一个对象，返回键组成的UnionType
<https://www.typescriptlang.org/docs/handbook/2/keyof-types.html>

```typescript
type Record<K extends keyof any, U> = {
	[P in K]: U
}
```

#### 映射类型

映射类型的功能比Record功能更强大，配合‘键入’还可以约束特定键的类型

```typescript
type Account = {
	id: number
	name: string
	notes: string[]
}

type OptionalAccount = {
	[Key in keyof Account]?: Account[Key]
}

type AccountCanNull = {
	[Key in keyof Account]: Account[Key] | null
}

type ReadonlyAccount = {
	readonly [Key in keyof Account]: Account[key]
}

type Account2 = {
	-readonly [Key in keyof ReadonlyAccount]: ReadonlyAccount[Key]
}

type NoOptionalAccount = {
	[Key in keyof OptionalAccount]-?: OptionalAccount[Key]
}
```

内置的映射类型：

1. ```Record<Keys, Values>```
2. ```Partial<Object>```, Object内的键全部为可选
3. ```Required<Object>```, Object内的键全部必填
4. ```Readonly<Object>```, 返回只读
5. ```Pick<Object, Keys>```, Object的子类，只包含Keys
