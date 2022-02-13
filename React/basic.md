## jsx语法规则
- 定义虚拟DOM时，不要写引号
- 标签中混入`js表达式`时要用{}
- 样式的类名指定不要用class，要用ClassName
- 内联样式，要用style = {{key:value}} 的形式去写
- 只有一个根标签
- 标签必须闭合
- 标签首字母
    - 若小写字母开头，则将该标签转为html中同名元素；若HTML中无该标签对应的同名元素，则报错
    - 若大写字母开头，react就去渲染对应的组件，若组件没有定义，则报错
- 注意事项：区分【js语句（代码）】与【js表达式】
    - 表达式：一个表达式会产生一个值，可以放在任何一个需要值的地方
    ```js
    // 下面这些都是表达式：
        a
        a+b
        demo(1)
        arr.map()
        function test() {}
    ```
    - 语句（代码）：
    ```js
    // 下面这些是语句
        if(){}
        for() {}
        switch(){case: xxx}
    ```

## 模块与组件、模块化与组件化
- 模块：向外提供特定功能的js程序，一般是一个js文件
- 组件：用来实现`局部功能效果的代码和资源的集合`
- 模块化：当应用的js都以模块来编写的，该应用就是一个模块化的应用
- 组件化：当应用是以多组件的方式实现，这个应用就是一个组件化的应用


### 函数式组件
```js
// 创建函数式组件
function MyComponent() {
    console.log(this) // 此处的this是undefined，因为babel编译之后开启了严格模式
    return <h2>我是函数定义的组件（适用于【简单组件（无状态state）】的定义）</h2>
}
// 渲染组件到页面
ReactDOM.render(<MyComponent />, document.getElementById('test'))

/*
    执行了ReactDOM.render(<MyComponent />.....)之后发生了什么？
    1. React解析组件标签，找到了MyComponent组件
    2. 发现组件是使用函数定义的，随后调用该函数，将返回的虚拟DOM转为真实DOM，随后呈现在页面中
*/
```

### 类
- 类中的构造器不是必须写的，要对实例进行一些初始化的操作，如添加指定属性时才写
- 如果A类继承了B类，且A类中写了构造器，那么A类构造器中的super是必须调用的
- 类中所定义的方法，都是放在了类的原型对象上，供实例去使用

### 类式组件
```js
// 创建类式组件
class MyComponent extends React.Component {
    render() {
        // render是放在那里的？ -- MyComponent的原型对象上，供实例使用
        // render中的this是谁？-- MyComponent的实例对象 <=> MyComponent组件实例对象
        console.log('render中的this', this)
        return <h2>我是类定义的组件（适用于【复杂组件（有状态state）】的定义）</h2>
    }
}

// 渲染组件到页面中
ReactDOM.render(<MyComponent />, document.getElementById('test'))

/*
    执行了ReactDOM.render(<MyComponent />.....)之后发生了什么？
    1. React解析组件标签，找到了MyComponent组件
    2. 发现组件是使用类定义的，随后new 出来该类的实例，并通过该实例调用到原型上的render方法
    3. 将返回的虚拟DOM转为真实DOM，随后呈现在页面中
*/
```

### state 简写
```js

class Weather extends React.Component {
    // 初始化状态
    state = {isHot: false,wind: '微风'}

    render() {
        const {isHot, wind} = this.state
        return <h1 onClick={this.changeWeather}> 今天天气很{isHot?'炎热':'凉爽'}，{wind}
    }
    // 自定义方法 -- 要用赋值语句 + 箭头函数  this指向实例本身
    changeWeather = () => {
        const isHot = this.state.isHot;
        this.setState({isHot: isHot})
    }
}

ReactDOM.render(<Weather/>, document.getElementById('test))

```
- 总结：
    - state是组件对象最重要的属性，值是对象（可以包含多个key-value的组合）
    - 组件被称为“状态机”， 通过更新组件的state来更新对应页面显示（重新渲染组件）
    - 组件中`render方法的this为组件实例对象`
    - 组件自定义方法中的this为undefined，如何解决？
        - 强制绑定this：通过函数对象的bind()
        - 箭头函数
    - 状态数据，不能直接修改或更新

### props


### refs
- 字符串形式的ref--尽量避免
- 回调形式的ref  
    - 内联形式的回调形式的ref函数
    - 非内联形式的回调形式的ref函数
- createRef 调用后可以返回一个容器，该容器可以存储被ref所标识的节点，该容器是专有的

### 受控组件 vs 非受控组件
- 受控组件：input框通过onChange的回调接收控制变化的数据，存入state里面
- 非受控组件：通过ref接收数据

### 高阶函数
- 一个函数如果符合下面任意一项就是高阶函数
    - 一个函数接收的参数是一个函数，那么该函数就可以称之为高阶函数
    - 一个函数返回值是一个函数，那么该函数可以称之为高阶函数


### 函数科里化
- 通过函数调用继续返回函数的方式，实现多次接收参数，最后统一处理的函数编码形式

### 生命周期函数 生命周期钩子函数  旧生命周期
- 1
    - componentWillMount 组将即将挂载
    - componentDidMount  组件挂载完毕调用  *常用：开启定时器，发送网络请求，订阅消息*
    - render  初始化渲染  状态更新之后
- 2
    - shouldComponentUpdate 组件将要更新  控制组件是否更新的阀门  
    - render 重新渲染
    - componentDidUpdate  组件更新完毕
    - componentWillUnmount 组件将要卸载时调用    *常用： 关闭定时器，取消订阅消息*
- 3
    - forceUpdate 强制更新， 不受‘阀门’状态影响
    - render 重新渲染
- 4
    - componentWillReceiveProps 组件非首次接收参数时调用
    - shouldComponentUpdate 组件将要更新  控制组件是否更新的阀门  
    - render 重新渲染
    - componentDidUpdate  组件更新完毕
    - componentWillUnmount 组件将要卸载时调用

### 生命周期函数 生命周期钩子函数  新生命周期
- 
