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
