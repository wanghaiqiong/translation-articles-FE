
译自：https://levelup.gitconnected.com/9-tricks-for-kickass-javascript-developers-in-2019-eb01dd3def2a

又一年过去了，JavaScript 也一直在改变。不过有些技巧可以帮助你写出简洁高效可伸缩的代码，即便是（或者说特别是）2019 年。下面 9 条实用小技巧能助你成为一个更好的开发者。

# 1.async / await
如果你仍深陷**回调地狱**，那么你应该还在写 2014 年之前的老古董代码吧。除非很有必要，比如遵守代码库要求或者出于性能原因，否则不要使用回调方式。Promise 还行，但如果你的代码日渐庞大，Promise 就显得有些尴尬了。我现在的首选方案是 `async / await`，它让代码的阅读与改进都变得简洁很多。事实上，你可以 `await` JavaScript 中的每一个 Promise，如果你用的库函数返回一个 Promise，就可以简单地 `await` 之。其实，`async / await` 只是使用 Promise 的语法糖。想让你的代码正常工作起来，你只需要在 funcion 前增加 `async` 关键字。如下是一个简单例子：

```js
async function getData() {
    const result = await axios.get('https://dube.io/service/ping')
    const data = result.data

    console.log('data', data)

    return data
}
getData()
```

注意，在最顶层没办法使用 `await`，只能在 `async` 函数中使用。

*`async / await` 是在 ES2017 中引入的，所以记得转译你的代码。*


# 2.async control flow（异步控制流）
实际开发中不可避免地经常会遇到这种情况，我们要**获取多个数据项然后分别对它们进行某些处理（`for…of`）**，或者需要**在所有异步调用都得到返回值后再完成某项任务（Promise.all）**。

## for…of

比方说我们要获取页面中几个 Pokemon 的具体信息，我们并不想等待所有调用全部完成，尤其是有时候并不知道具体有多少次调用，但我们想只要一有返回数据就立即更新数据项。这时候我们就可以用 `for...of` 来遍历数组，在循环体内执行 async 代码，代码的执行会被暂停，直到每次调用成功。必须注意的是如果你在代码中如示例这样做，可能会带来性能瓶颈，但把这个技巧收藏你的工具箱里还是非常有用的。示例如下：

```js
import axios from 'axios'

let myData = [{id: 0}, {id: 1}, {id: 2}, {id: 3}]

async function fetchData(dataSet) {
    for(entry of dataSet) {
        const result = await axios.get(`https://ironhack-pokeapi.herokuapp.com/pokemon/${entry.id}`)
        const newData = result.data
        updateData(newData)

        console.log(myData)
    }
}

function updateData(newData) {
    myData = myData.map(el => {
        if(el.id === newData.id) return newData
        return el
    })
}

fetchData(myData)
```

*注：这些示例都可有效运行，可随意复制粘贴到你喜欢的代码沙盒内执行（如 jsfiddle、jsbin、codepen）。*


## Promise.all

如果想并行获取所有 Pokemon 的信息又该如何实现呢？既然 `await` 可以用在所有 Promise 上，很简单，用 `Promise.all`：

```js
import axios from 'axios'

let myData = [{id: 0}, {id: 1}, {id: 2}, {id: 3}]

async function fetchData(dataSet) {
    const pokemonPromises = dataSet.map(entry => {
        return axios.get(`https://ironhack-pokeapi.herokuapp.com/pokemon/${entry.id}`)
    })

    const results = await Promise.all(pokemonPromises)

    results.forEach(result => {
        updateData(result.data)
    })

    console.log(myData)
}

function updateData(newData) {
    myData = myData.map(el => {
        if(el.id === newData.id) return newData
        return el
    })
}

fetchData(myData)
```

*注：`for...of` 与 `Promise.all` 都是在 ES6+ 引入的，所以（必要的话）记得转译你的代码。*

# 3.Destructuring & default values（解构赋值与默认值）

让我们返回到上一示例中，我们是这样做的：

```js
const result = axios.get(`https://ironhack-pokeapi.herokuapp.com/pokemon/${entry.id}`)
const data = result.data
```

有一种更简便的做法，采用解构的方式从某个对象或者数组中获取一个或者一些值。像这样：
```js
const { data } = await axios.get(...)
```

耶！变成一行代码了。甚至还能把变量重命名：
```js
const { data: newData } = await axios.get(...)
```

另一个很妙的技巧是解构赋值的时候给出默认值。这样就确保了你再也不会得到 `undefined`，而且也不必费心去手动检测变量。

```js
const { id = 5 } = {}
console.log(id) // 5
```

这些技巧同样适用于函数参数，如下所示：
```js
function calculate({operands = [1, 2], type = 'addition'} = {}) {
    return operands.reduce((acc, val) => {
        switch(type) {
            case 'addition':
                return acc + val
            case 'subtraction':
                return acc - val
            case 'multiplication':
                return acc * val
            case 'division':
                return acc / val
        }
    }, ['addition', 'subtraction'].includes(type) ? 0 : 1)
}

console.log(calculate()) // 3
console.log(calculate({type: 'division'})) // 0.5
console.log(calculate({operands: [2, 3, 4], type: 'multiplication'})) // 24
```

这个例子乍一看可能有点令人困惑，别急慢慢来。当我们不传任何参数时，函数将使用默认值。一旦开始传递参数，只有没传的参数会使用默认值。

*注：解构赋值在 ES6 中被引入，确保先转译你的代码。*


# 4.Truthy & falsy values（检测真假值）

在确定是否要取默认值的时候，我们往往会先对给定的值进行检查，其中的某些检查现在来说已经没有必要了，将成为历史。无论如何，知道如何处理 `真值`（truthy values）和 `假值`（falsy values）总是非常好的。它能帮助我们改进代码，省去一些表达式，让代码更清晰明白。我经常看到有人这样做：

```js
if(myBool === true) {
  console.log(...)
}
// OR
if(myString.length > 0) {
  console.log(...)
}
// OR
if(isNaN(myNumber)) {
  console.log(...)
}
```

这些都可以简写成：
```js
if(myBool) {
  console.log(...)
}
// OR
if(myString) {
  console.log(...)
}
// OR
if(!myNumber) {
  console.log(...)
}
```

想要真正用好这些，你需要理解 `真值` 和 `假值` 的含义。这里有个小总结：

**假值**
+ 长度为 0 的字符串
+ 数字 `0`
+ `false`
+ `undefined`
+ `null`
+ `NaN`

**真值**
+ 空数组
+ 空对象
+ 所有其他的东西

注意在检测真假值时，这里进行的是非严格比较，也就是说用的是 `==` 而不是 `===`。一般说来，二者行为相同，但在某些特定情况下会出现 bug。对我来说，常发生在数字 `0` 上。

# 5.Logical and ternary operators（逻辑运算符与三元运算符）

同样，这也是精简代码的好方法。通常都能帮我们简化代码，但也会带来一些混乱，尤其是链式使用时。

## Logical operators

逻辑运算符主要用于连接两个表达式，计算返回 `true`，`false` 或者与之匹配的值，`&&` 表示逻辑与，`||` 表示逻辑或。如下：

```js
console.log(true && true) // true
console.log(false && true) // false
console.log(true && false) // false
console.log(false && false) // false
console.log(true || true) // true
console.log(true || false) // true
console.log(false || true) // true
console.log(false || false) // false
```

我们把逻辑运算符与上一个知识点**真假值**结合起来理解。当使用逻辑运算符时，遵从如下规则：

+ `&&`：返回第一个假值，如果没有，则返回最后一个真值
+ `||`：返回第一个真值，如果没有，则返回最后一个假值

```js
console.log(0 && {a: 1}) // 0
console.log(false && 'a') // false
console.log('2' && 5) // 5
console.log([] || false) // []
console.log(NaN || null) // null
console.log(true || 'a') // true
```

## Ternary operator

三元运算符与逻辑运算符类似，但有三个部分：

1. 比较表达式，计算返回真值或者假值
2. 第一个返回值，用于表达式计算为真值时返回
3. 第二个返回值，用于表达式计算为假值时返回

示例如下：

```js
const lang = 'German'
console.log(lang === 'German' ? 'Hallo' : 'Hello') // Hallo
console.log(lang ? 'Ja' : 'Yes') // Ja
console.log(lang === 'French' ? 'Bon soir' : 'Good evening') // Good evening
```

# 6.Optional chaining（可选链式调用）

你是否遇到过这种问题，想要访问嵌套对象的属性，然而并不知道该对象或其中一个子属性是否存在？你很可能会写出类似这样的代码：

```js
let data
if(myObj && myObj.firstProp && myObj.firstProp.secondProp && myObj.firstProp.secondProp.actualData)
data = myObj.firstProp.secondProp.actualData
```
这太啰嗦了，有个更好的方法，至少是一种更好的提议（继续往下看如何用它）。它就是可选链式调用(optional chaining) ，用法如下：

```js
const data = myObj?.firstProp?.secondProp?.actualData
```

我认为，这是一种让代码更清晰的检查嵌套属性的有效方法。

*注：目前可选链式调用 (optional chaining) 还不是官方规范的一部分，是处于 stage-1 的实验性特性。你需要在你的 balelrc 中添加插件 @babel/plugin-proposal-optional-chaining 来使用。*

# 7.Class properties & binding（类属性与绑定）

函数绑定在 JavaScript 中十分常见。随着 ES6 规范中箭头函数的引入，我们现在有办法自动绑定函数到定义时的上下文了，这种方法非常好用，被 JavaScript 开发者广泛使用。Class（类）刚刚引入的时候，你并不能真正的使用箭头函数，因为类方法需要一种特定的声明方式。我们要在其他地方绑定函数，如在构造器中（React.js 的最佳实践）。我一直觉得先定义类方法然后再绑定的流程很烦人，一段时候过后再看更感觉莫名其妙。有了类属性语法，我们又可以用箭头函数获得自动绑定的好处。箭头函数现在可以在类内使用了。示例如下，重点看 `_increaseCount` 是如何绑定的：

```js
class Counter extends React.Component {
    constructor(props) {
        super(props)
        this.state = { count: 0 }
    }

    render() {
        return(
            <div>
                <h1>{this.state.count}</h1>
                <button onClick={this._increaseCount}>Increase Count</button>
            </div>
        )
    }

    _increaseCount = () => {
        this.setState({ count: this.state.count + 1 })
    }
}
```

*注：目前，class properties 并不是正式官方规范的一部分，是处于 stage-3 的一个实验性特性。需要在你的 balelrc 中添加插件 @babel/plugin-proposal-class-properties 来使用。*

# 8.Use parcel
做为前端开发者，你肯定遇到过打包和转译代码的情况。wepback 成为事实标准已经有很长一段时间了。我最初使用 webpack 时它还处于第一个版本，那时候很痛苦。我花了无数个小时去处理各种不同的配置项，让项目打包运行。一旦能跑起来，我就再也不会去动它们，怕又给弄坏了。几个月前偶然发现的 parcel，让我松了口气。它提供的所有功能开箱即用，同时还允许我们在必要时做出更改。它像 webpack 或者 babel 一样支持插件系统，并且速度极快。如果你还没听过 parcel，墙裂建议去看看！

# 9.Write more code yourself
这是个很好的话题。关于这个问题，我有过很多不同的讨论。即使是 CSS，有很多人也会倾向于使用组件库，比如 bootstrap。JavaScript 的话，也有不少人使用 jQuery 和一些轻量代码库处理验证、滑动效果等。虽然用库也可以理解，但我还是墙裂建议自己编写更多的代码，而不是盲目地安装 npm 包。对于那些整个团队维护构建的大型代码库（或者框架），如 moment.js、react-datepicker，我们个人尝试去编写是没有什么意义的。但可以多写一些只是自己项目使用的代码。这样对自己有三大好处：

1. 你能确切地知道代码中都做了什么
2. 在某种程度上，帮助自己开始真正理解什么是编程以及程序底层是如何运作的
3. 防止代码库变得更加臃肿

一开始，用 npm 包会显得更简单，自己去实现某些功能反而更费时间。但万一这个包并没有像预期的那样工作，然后你不得不换另一个，花更多的时间去阅读如何使用新的 API。如果是自己实现，你可以按自己的使用情况 100% 量身定制。
