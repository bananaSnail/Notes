### typescript 与 javascript 的优势
- typescript 是javascript 的超集，本质上是为javascript增加了静态类型声明；typescript 最终也会被编译成javascript，在浏览器、node等环境中使用
- 编译期行为
- 不引入额外开销
- 不改变运行时行为

### const 和 readonly的区别
- const 用于变量， readonly 用于属性
- const 在运行时检查， readonly 在编译时检查
- const 声明的变量不得改变值，这意味着，const 一旦声明变量，就必须立即初始化，不能留到以后赋值; readonly 修饰的属性能确保自身不能修改属性，但是当你把这个属性交给其它并没有这种保证的使用者(允许出于类型兼容性的原因)，他们能改变
- const 保证的不是变量的值不得改动，而是变量指向的那个内存地址不得改动，例如使用 const 变量保存的数组，可以使用 push ， pop 等方法。但是如果使用 ReadonlyArray 声明的数组不能使用 push ， pop 等方法。

### 枚举和常量枚举的区别
- 枚举会被编译时会编译成一个对象，可以被当作对象使用
- const 枚举会在 typescript 编译期间被删除，const 枚举成员在使用的地方会被内联进来，避免额外的性能开销
```js
// 枚举 
enum Color { 
  Red, 
  Green, 
  Blue 
} 
 
var sisterAn = Color.Red 
// 会被编译成 JavaScript 中的 var sisterAn = Color.Red 
// 即在运行执行时，它将会查找变量 Color 和 Color.Red 

// 常量枚举 
const enum Color { 
  Red, 
  Green, 
  Blue 
} 
 
var sisterAn = Color.Red 
// 会被编译成 JavaScript 中的 var sisterAn = 0 
// 在运行时已经没有 Color 变量 

```
### 如何获取到函数返回值类型
- Exclude<T, U> -- 从T中剔除可以赋值给U的类型。
- Extract<T, U> -- 提取T中可以赋值给U的类型。
- NonNullable<T> -- 从T中剔除null和undefined。
- ReturnType<T> -- 获取函数返回值类型。
- InstanceType<T> -- 获取构造函数类型的实例类型。

### 打包后enum是存在的，如何使用enum中某个值，并使之打包后不存在这个枚举类型值

### any 和 unknown、 never区别
- any 表示任意类型，不做任何类型约束，编译时会跳过对其类型检查
- void 表示无任何类型，一般用定义函数返回值。变量也可以申明为 void 类型，只不过只能给这个变量分配 undefined, null（非严格模式下） 和 void 类型的值
- unknown 表示未知类型，一般用于从服务器接口返回的数据或者JSON.parse() 返回结果等；相当于any的安全版本。unknown能被赋值为任何类型，但不能赋值给除了any 和 unknown之外的类型，并且不允许执行unknown类型变量的方法
```js
let a: unknown = 123;
a = '123'; // ok
a = true; // ok
a.toString(); // Error

let b: string = a; // Error 
```
- never 表示永不存在的值，从程序运行角度考虑一些函数执行时抛出了异常，这个函数就变成永远不会有值了；**never是所有类型的子类型，但没有任何类型是never的子类型，除了never自身外。**
```js
let a: never
let b: never
a = b; // ok

let c: any = 'str';
a = c; // Error

```


### 介绍下泛型，一般用于什么场景
- 泛型指定义函数、接口或类的时候，不预先指定具体的类型，而在使用的时候再指定类型的一种特性
- 使用场景：使用泛型的关键目的是在`成员之间提供有意义的约束`，如类的实例成员、类的方法、函数参数、函数返回值
- 滥用泛型：泛型仅用于单个参数的位置时是完全没必要用泛型的，此时相当于一个类型断言as
- 坑： 除了泛型接口，我们还可以创建泛型类。 注意，`无法创建泛型枚举和泛型命名空间。`

### 介绍下interface
- 在面向对象语言中，接口是对行为的抽象，而具体如何行动需要由类去实现，typescript中的接口可以对`类的一部分行为进行抽象，也可用于对对象的形状进行描述`

### d.ts是什么
- .d.ts文件扩展名指定这个文件是一个`声明文件`；如果一个文件有扩展名.d.ts，意味着每个顶级的声明都必须以declare关键字为前缀，`声明文件描述一个 JavaScript 模块（广义上的）内所有导出接口的类型信息`

### typescript是如何编译的


### namespace/module
- 一个文件就是一个module，即一个模块
- namespace是命名空间，在全局空间中具有唯一性


# TypeScript
TypeScript 是 JavaScript 的一个超集，主要提供了类型系统和对 ES6 的支持。

TypeScript 只会进行静态检查，编译为 js 之后，并没有什么检查的代码被插入进来。如果编译时发现有错误，就会报错，但还是会生成编译结果。

### TS优势

 - TypeScript 增加了代码的可读性和可维护性
   - 类型系统实际上是最好的文档，大部分的函数看看类型的定义就可以知道如何使用了
   - 可以在编译阶段就发现大部分错误，这总比在运行时候出错好
   - 增强了编辑器和 IDE 的功能，包括代码补全、接口提示、跳转到定义、重构等
 - TypeScript 非常包容
   - TypeScript 是 JavaScript 的超集，.js 文件可以直接重命名为 .ts 即可
   - 即使不显式的定义类型，也能够自动做出类型推论
   - 可以定义从简单到复杂的几乎一切类型
   - 即使 TypeScript 编译报错，也可以生成 JavaScript 文件
   - 兼容第三方库，即使第三方库不是用 TypeScript 写的，也可以编写单独的类型文件供 TypeScript 读取
 - TypeScript 拥有活跃的社区
   - 大部分第三方库都有提供给 TypeScript 的类型定义文件
 - TypeScript 的缺点
   - 有一定的学习成本
   - 短期可能会增加一些开发成本，毕竟要多写一些类型的定义，不过对于一个需要长期维护的项目，TypeScript 能够减少其维护成本
   - 集成到构建流程需要一些工作量
   - 可能和一些库结合的不是很完美


### 声明文件
通常我们会把声明语句放到一个单独的文件中，声明文件必需以 `.d.ts` 为后缀。
```ts
// src/jQuery.d.ts
declare var jQuery: (selector: string) => any;
```

更推荐的是使用 `@types` 统一管理第三方库的声明文件。

@types 的使用方式很简单，直接用 npm 安装对应的声明模块即可，以 jQuery 举例：`npm install @types/jquery --save-dev`

可以在[这个页面](https://microsoft.github.io/TypeSearch/)搜索你需要的声明文件。

#### 书写声明文件

 - 以`npm install @types/xxx --save-dev`安装的，则不需要任何配置。
 - 如果是将声明文件直接存放于当前项目中，则建议和其他源码一起放到 src 目录下
 - 如果没有生效，可以检查下 tsconfig.json 中的 files、include 和 exclude 配置，确保其包含了 jQuery.d.ts 文件
 - 语法
   - declare var/let/const 
     - 声明全局变量，一般用const
   - declare function 
     - 声明全局方法的**类型**
   - declare class 
     - 声明全局类，只定义不实现
   - declare enum 
     - 声明全局枚举类型
   - declare namespace 
     - 它用来表示全局变量是一个对象，包含很多子属性。
     - 可嵌套定义
   - interface 和 type
     - 声明全局类型
 - 防止命名冲突
   - 暴露在最外层的 interface 或 type 会作为全局类型作用于整个项目中，我们应该尽可能的减少全局变量或全局类型的数量。故应该将他们放到 namespace 下。在使用这个 interface 的时候，也应该加上 命名空间 前缀了
 - npm 包
   - 已存在配置
     - 与该 npm 包绑定在一起。
     - 发布到了 `@types` 里。
   - 自己为它写声明文件
     - 创建一个 types 目录，专门用来管理自己写的声明文件，将 foo 的声明文件放到 types/foo/index.d.ts 中。
   - `export`
     - 在 npm 包的声明文件中，使用 declare 不再会声明一个全局变量，而只会在当前文件中声明一个局部变量。只有在声明文件中使用 export 导出，然后在使用方 import 导入后，才会应用到这些类型声明。


### 内置对象

内置对象是指根据标准在全局作用域（Global）上存在的对象。这里的标准是指 ECMAScript 和其他环境（比如 DOM）的标准。

在 [TypeScript 核心库](https://github.com/Microsoft/TypeScript/tree/master/src/lib)的定义文件中定义了JS的内置对象。


### 类型别名
类型别名用来给一个类型起个新名字。类型别名常用于联合类型。

### 字符串字面量类型
类型别名与字符串字面量类型都是使用 type 进行定义。

### 类
 - 类(Class)：定义了一件事物的抽象特点，包含它的属性和方法
 - 对象（Object）：类的实例，通过 new 生成
 - 面向对象（OOP）的三大特性：封装、继承、多态
 - 封装（Encapsulation）：将对数据的操作细节隐藏起来，只暴露对外的接口。外界调用端不需要（也不可能）知道细节，就能通过对外提供的接口来访问该对象，同时也保证了外界无法任意更改对象内部的数据
 - 继承（Inheritance）：子类继承父类，子类除了拥有父类的所有特性外，还有一些更具体的特性
 - 多态（Polymorphism）：由继承而产生了相关的不同的类，对同一个方法可以有不同的响应。
 - 存取器（getter & setter）：用以改变属性的读取和赋值行为
 - 修饰符（Modifiers）：修饰符是一些关键字，用于限定成员或类型的性质。比如 public 表示公有属性或方法
 - 抽象类（Abstract Class）：抽象类是供其他类继承的基类，抽象类不允许被实例化。抽象类中的抽象方法必须在子类中被实现
 - 接口（Interfaces）：不同类之间公有的属性或方法，可以抽象成一个接口。接口可以被类实现（implements）。一个类只能继承自另一个类，但是可以实现多个接口

#### TS中的类

 - `public` 修饰的属性或方法是公有的，可以在任何地方被访问到，默认所有的属性和方法都是 public 的
 - `private` 修饰的属性或方法是私有的，不能在声明它的类的外部访问
 - `protected` 修饰的属性或方法是受保护的，它和 private 类似，区别是它在子类中也是允许被访问的

#### 抽象类
`abstract` 用于定义抽象类和其中的抽象方法。
 - 抽象类是不允许被实例化
 - 抽象类中的抽象方法必须被子类实现


### 类与接口
有时候**不同类之间可以有一些共有的特性，这时候就把特性提取成接口（interfaces）**，用 implements 关键字来实现。 

 - 一个类可实现多个接口
 - 接口与接口之间可以是继承关系
 - 接口也可以继承类
 - 混合类型（有时候，一个函数还可以有自己的属性和方法）


### 泛型
在函数名后添加了 `<T>`，其中 `T` 用来指代任意输入的类型，在后面的输入 `value: T` 和输出 `Array<T>` 中即可使用了。
```ts
function createArray<T>(length: number, value: T): Array<T> {
    let result: T[] = [];
    for (let i = 0; i < length; i++) {
        result[i] = value;
    }
    return result;
}
createArray(3, 'x'); // ['x', 'x', 'x']
```

定义泛型的时候，可以一次定义多个类型参数：
```ts
function swap<T, U> (tuple:[T, U]) : [U, T] {
  return [tuple[1], tuple[0]];
}

swap([7, 'seven']); // ['seven', 7]
```

 - 泛型约束
   - 即函数内部使用泛型变量的时候，由于事先不知道它是哪种类型，所以不能随意的操作它的属性或方法，这时，我们可以对泛型进行约束。**即让泛型继承一个接口**
   - 多个类型参数之间也可以相互约束，如下例：其中要求 T 继承 U，这样就保证了 U 上不会出现 T 中不存在的字段。
 - 泛型接口
   - 在使用泛型接口的时候，需要定义泛型的类型
 - 泛型类
 - 泛型参数的默认类型
   - 当使用泛型时没有在代码中直接指定类型参数，从实际值参数中也无法推测出时，这个默认类型就会起作用。

### 用过最复杂的ts API，用过哪些ts API
- 泛型  类型断言  
- 题目：第一个参数是不定类型，如 type 或 enum这种  第二个参数在第一个参数确定下来后跟第一个参数类型保持一致
- 解析：
  1. `函数参数间的约束考虑用泛型`，`注意：是无法创建泛型枚举和泛型空间的`，因此考虑type类型。
  2. `infer类型推断：`由于第二个参数依赖于第一个参数的类型
```js
// 方法一
function name<T extends Test, U extends T>(a: T, b: U) {
  console.log(a, b);
}
enum Test {
  A = 1,
  B = 'string'
}

name(Test.A, 'xxxx') // 报错，类型不兼容
name(Test.A, 1) // 可

// 方法二
type Type = Number | String // 注意要大写，小写的number | string 是ts 里面定义值类型的，Number和String 是其中的包装对象；定义值的话就必须得一样了
type ParamType<T> = T extends (param: infer P) ? P : T 

testFunc<T extends Type>(a: T, b: ParamType<T>) {
  console.log(a, b)
}

testFunc('aaa', 111) // 报错，Argument of type 'number' is not assignable to parameter of type 'string'.
testFunc('aaa', 'bbbb')
```

### 声明合并
如果定义了两个相同名字的函数、接口或类，那么它们会合并成一个类型。
```ts
interface Alarm {
    price: number;
    alert(s: string): string;
}
interface Alarm {
    weight: number;
    alert(s: string, n: number): string;
}
```
相当于：
```ts
interface Alarm {
    price: number;
    weight: number;
    alert(s: string): string;
    alert(s: string, n: number): string;
}
```


### is关键字
可以用来判断一个变量属于某个接口|类型

例如，此时有一个接口A
```js
interface IAProps {
  name: string
  js: any
}
```
现在需要判断一个变量是否为该类型

定义规则：
```js
// 属于接口A
let isAProps = (props: any): props is IAProps =>
  typeof (props as IAProps)['js'] !== 'undefined'
```
若isAProps(props)返回true则断定参数props为IAProps类型


### narrowing：TypeScript 类型收窄就是从宽类型转换成窄类型的过程。类型收窄常用于处理联合类型变量的场景，而不是通过类型断言as去处理
- typeof 类型保护
- 真值收窄
- 等值收窄
- in操作符收窄
- instance of收窄
- 赋值语句收窄
- 控制流分析、if、Switch
- 类型谓词 pet is Fish就是类型谓词
- function isFish(pet: Fish | Bird): pet is Fish { return 'swim' in pet; }
- 自定义类型守卫：返回布尔值的条件表达式赋予类型守卫的能力， 只有当函数返回 true 时，形参被确定为 A 类型

