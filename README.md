# JS-self-answer

## 待解决
- 如何处理nodejs应用的状态管理（全局变量）
- 访问`window.a`,返回`undefined`，但如果直接访问变量`a`，则报错：`Uncaught ReferenceError: a is not defined`，为什么？
- 在使用Vue的`mixin`特性时，一个`mixin`文件a中需要使用另一个`mixin`文件b的method，怎么检测b文件的依赖已经被主文件引入？（不能暴力的在当前a文件下使用`mixin`特性引入b文件，禁止套娃[雾]）

## 已解决
Q：有以下代码：`var b`，当判断变量`b`的值是否为`undefined`时，为什么我们应该使用`typeof b === 'undefined'`，而不推荐使用`b === undefined`?  
A：在非严格模式下，我们可以为全局标识符`undefined`赋值，比如：`undefinded = 2`，这时用`b === undefined`判断的结果并不正确，所以推荐使用`typeof b === 'undefined'`， 顺便一提，`null`是一个特殊关键字，不是标识符，却不能将其当做变量来使用和赋值。  

Q：`const`关键字在声明对象时，不变的部分是哪里？  
A：`const`关键字的作用是声明不可变的变量，对于JS而言，变量即值，JS的值的类型分为基本类型和复合类型（对象）。因此`const`声明不可变变量即声明不可变的值，对于基本类型而言，表示变量的值不可变；而对于复合类型，就有点区别了，复合类型值的变量实际上拿到的是该值的引用（即内存地址），因此我们不能对`const`声明的变量重新赋值，但如果是复合类型值的变量，则可操作该引用，比如：
```js
const obj = {}
obj.fn = () => console.log('it is me|!') // 操作引用地址未变，正常的操作
obj = [] // 错误操作，会报错
```
以上代码看似对`obj`变量进行了新的赋值操作，但实际上该变量的引用并没有变化，所以是正确的操作。而对于基本类型的值，变量拿到的是确确实实的值，而不是引用地址，要想改变其值，只能重新赋值，对于`const`关键字声明的基本类型的值，则会报错。  

Q: 有如下字符串：`./src/utils/timeFormat.js, ./src/components/input.vue, ./src/styles/default.less, ./src/readme.md`，且有如下正则表达式:
```javascript
var reg = /^\.\/.+\.(?!less|md)$/i
```
在使用`require.context`方法时，该正则表达式不能达到排除less,md文件，只选择其他文件的功能。  
A: 原因是语法`x(?!y)`匹配项并不包含`y`的值，而该正则在*负向后瞻*后紧跟了$ end标识符，从而匹配失败。所以需要注意负向后瞻与$结束符的配合使用

Q: 有某不定项的数组，会被渲染为li元素，渲染规则为：三个为一组，渲染为一行，依行渲染，最后剩余项渲染为一行。  
问题是如何给最后一行的元素（注：最后一行的元素为0，1，2个）设置特别的样式。如果用css的`nth-child`语法能达到要求吗？
A: 答案是**不能**，因为要选定最后一行的元素必须明确知道传入数组的成员个数，css很难轻易做到这一点（？存疑）。所以只能在现代web框架中在渲染li元素时动态的设置该项元素的`className`属性。判断该项元素是否为最后一行元素：
```javascript
// arr: 给定的数据，index：该元素在数组中的索引
index + 1 > parseInt(arr.length / 3) * 3 // 值为true表示该元素是最后一行的元素
```

Q: 在DOM元素上绑定事件的回调函数时，为什么回调函数中的`this`关键字指向的是触发该事件的目标元素？  
A：事件绑定函数时，该函数会以当前元素为作用域执行,所以回调函数中的this是当前的DOM元素。（参考链接： [https://leohxj.gitbooks.io/front-end-database/javascript-basic/events.html#事件的回调函数](https://leohxj.gitbooks.io/front-end-database/javascript-basic/events.html#事件的回调函数)）

Q: 在Vue框架下使用vuex库时，有以下代码：
```javascript
// action.js
getCascade(context) {
    return API.getcascade().then(res => {
        let data = []
        res.data.forEach(item => {
            data.push(handleItem(item))
            
            context.commit('updateCascade', data)
        })
        
         return res
    })
}
```
报错：*Error: [vuex] Do not mutate vuex store state outside mutation handlers.*，为什么会报错？  
A: 因为`data`是一个引用类型的变量，而在`res.data.forEach`的迭代回调中，每次执行`context.commit`都使用同一个`data`对象，而从第二次迭代开始，`data.push`语法会改变`data`的值，隐式的使得`state`中的`data`发生变化，而这种变化不是由mutation引发的，所以会报如此错误。  
另说，在`forEach`中执行`context.commit`语法真是一种愚蠢的写法，正确的做法是把`context.commit`语法放到`forEach`迭代结束后执行。

Q: 如何在浏览器环境下设计一个简单的文本摘要算法？
A: [设计一个简单的[JS]文本摘要算法](https://blog.csdn.net/ccaoee/article/details/102810150)，大致思路是：加盐->获取到每个字符的码点然后做凯撒字符偏移->将处理后的码点转为2进制->分块->以块为单位转为16进制->将16进制原始字符数据返回
