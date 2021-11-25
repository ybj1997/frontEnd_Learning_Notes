# 一、JSX

## 优势

- onclick添加的事件处理函数是在全局环境下执行的，这污染了全局环境，很容易产生意料不到的后果；JSX中挂在的函数控制在组件范围内，不会污染全局环境
- JSX中一个组件使用onClick，没有直接产生使用onclick的HTML，而是通过事件委托，无论出现多少个onClick，最后都只在DOM树上添加一个事件处理函数，挂在最顶层DOM节点上。
- React控制了组件的生命周期，在unmount的时候自然能够清除相关的所有事件处理函数，内存泄露也不再是一个问题

# 二、Virtual DOM

## DOM（文档对象模型）

​	DOM是结构化HTML文档的抽象模型。HTML中的每一个元素对应DOM中的一个节点，由于HTML元素是逐级包含关系，DOM节点自然就构成了DOM树。

​	页面渲染时，HTML由解析器解析成DOM树后，与CSSOM树合成渲染树，经过Layout回流和Painting重绘后，由GPU进程展示到页面上。

​	由于改变DOM会引起页面的回流和重绘，因此性能优化的原则之一：**尽量减少DOM操作**

## Virtual DOM（虚拟DOM）

​	Virtual DOM是对DOM树的抽象，不会触及浏览器部分，只存在于JS空间的树形结构。

​	每次自身而下渲染React组件时，会对比本次产生的VDOM和上一次渲染的VDOM，修改DOM树时，只修改变化部分。

## Keys

​	React中通过对比新旧VDOM进行最小力度的修改通过的就是key

​	Keys是React用于追踪那些元素发生改变的辅助标识。

​	在开发过程中，我们需要保证key的唯一性，React Diffing算法中React借助元素的key值来判断该元素的由来，从而减少不必要的元素渲染。

# 三、 Props

父子组件传递参数

# 四、State 

1. 修改state

	- 对象式setState（{}，cb）
	- 函数式setState（()=>{}，cb）

	回调函数会在 setState 函数调⽤完成并且组件开始重渲染的时候被调⽤ 

2. 初始化state

	在constructor里初始化，因为生命周期里constructe最先被调用

# 五、Refs

# 六、生命周期

### 挂载（Mount）过程

1. ### constructor

	用于初始化状态和绑定自定义函数的this环境

	```javascript
	Class  Demo  Extends React.component{
		constructor(){
	        this.state={show:true}
	        this.test = this.test.bind(this)
	    }
	    
	    function test(){
	        console.log(this.state.show);
	    }
	}
	```

	

2. ### getInitialState和getDefaultProps

	只有在用React.createClass（）方法创建组件实例才发生作用，ES6语法不产生作用

3. ### componentWillMount

	<font color="red">新版本将被废弃</font>

4. ### render

	render函数并不做实际的渲染动作，它只是<font color="pink">返回一个JSX描述的结构</font>，最终由React来操作渲染过程。组件在某些情况下选择没有东西可画，那就让render函数返回一个null或者false，等于告诉React，这个组件这次不需要渲染任何DOM元素。

5. ### componentDidMount

### 运行过程

1. ### shouldComponentUpdate()

	控制组件是否更新，常用于性能优化

2. ### render（）

3. ### getSnapshotBeforeUpdate（）

	 在最近一次渲染输出（提交到 DOM 节点）之前调用 ，它能够使组件发生改变之前从DOM中捕获信息（例如滚动位置）

4. ### componentDIdUpdate（）

### 销毁过程

​	**componentWillUnmount（）**

​		销毁之前调用



# Redux（集中状态管理）

用于管理全局共享数据

![1637843247881](C:\Users\AllenYan\AppData\Roaming\Typora\typora-user-images\1637843247881.png)

## 基础使用

### 1.创建Store（一个项目只有一个）

store.js文件

```javascript
//引入createStore，专门用于创建Redux的核心store对象
import {createStore} from 'redux';

//引入一个为目标组件的Reduce
import keyReduce from 'path'

//创建一个store向外暴露store
export default createStore(keyReduce)

```

### 2.创建一个为key组件服务的Reducer

key_Reducer.js文件

```javascript
/*
	创建为当前组件服务的reducer，本质是一个函数(用于完成最基本的目的)
	preState:之前状态（初始值为undefined，可以自定义）;actionObj:动作对象{type:,data:}
*/
function(preState,actionObj){
	//获取action对象的内容
    const {type,data} = action;
    
    //根据type，处理data
    switch (type) {
        case type1 :
            return 
        case type2 :{}
        ...
        default:
        	return preState的默认值
    }
}
```

### 3.在目标组件中使用

```javascript
import store from '/path/store.js'

//调用reducer，传入参数actionObj为{type:type1,date:value}
store.dispatch({type:type1,date:value})

/*
使用redux的核心store来调用reducer改变数据后：
	*如果使用setState修改state数据时，默认会调用render，React更新DOM;
	*redux中不会调用render，因此需要自主更新
*/
componentDidMount(){
    //监测store的状态变化，变化了就调用回调
    store.subscribe(()=>{
        this.setState({})
    })
}

//获取reducer中的prestate值
store.getState()
```

