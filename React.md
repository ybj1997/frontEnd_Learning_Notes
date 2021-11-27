# 一、JSX

## 优势

- onclick添加的事件处理函数是在全局环境下执行的，这污染了全局环境，很容易产生意料不到的后果；JSX中挂在的函数控制在组件范围内，不会污染全局环境
- 在JSX中看到一个组件使用了onClick，但并没有产生直接使用onclick（注意是onclick不是onClick）的HTML，而是使用了事件委托(event delegation)的方式处理点击事件，无论有多少个onClick出现，其实最后都只在DOM树上添加了一个事件处理函数，挂在最顶层的DOM节点上。
- React控制了组件的生命周期，在unmount的时候自然能够清除相关的所有事件处理函数，内存泄露也不再是一个问题

# 二、Virtual DOM

## DOM（文档对象模型）

​	DOM是结构化HTML文档的抽象模型（本质是一个Object对象）。HTML中的每一个元素对应DOM中的一个节点，由于HTML元素是逐级包含关系，DOM节点自然就构成了DOM树。

​	页面渲染时，HTML由解析器解析成DOM树后，与CSSOM树合成渲染树，经过Layout回流和Painting重绘后，由GPU进程展示到页面上。

​	由于直接操作改变DOM会引起页面的回流和重绘，因此性能优化的原则之一：**尽量减少DOM操作**

## Virtual DOM（虚拟DOM）

​	Virtual DOM是对DOM树的抽象本质是一个Obj对象，不会触及浏览器部分，只存在于JS空间的树形结构。

​	每次至上而下渲染React组件时，会对比本次产生的VDOM和上一次渲染的VDOM，修改DOM树时，只修改变化部分。

## React  diffing算法

### 工作原理

​	  每次React调用  `render()` 方法，会创建一棵由 React 元素组成的树，与一次产生的树进行比较。

- 将树形结构按层级分解，同级之间进行比较
- 列表结构设置key来进行比较。更新列表时， 在子元素列表末尾新增元素时，更新开销比较小（因此不能使用索引所谓key）
- React进行差异合并并重新绘制。

## Key

**什么是key？**

​	  Keys是 React ⽤于追踪哪些列表元素被修改、被添加或者被移除的辅助标识 。 <u>只需要保证同一数组中的兄弟元素的key是唯一的，不需要在整个组件或者单一组件中保持唯一。</u>

​	React中通过对比新旧VDOM进行最小力度的修改通过的就是key

**key的作用？**

​	 key 帮助 React 识别出同级数组中被修改、添加或删除的 item。 

​	在开发过程中，我们需要保证key的唯一性，React Diffing算法中React借助元素的key值来判断该元素的由来，从而减少不必要的元素渲染。

# 三、 Props（父子组件通信）

父子组件传递参数

## props.children

每个组件（函数或者类式）都可以获取到props.children

```jsx
//1.使用组件
<Welcome>Hello world!</Welcome>

//2.函数组件内部的props.children
function Weilcom(props){
    return (
    	console.log(props.children);//Hello world!
    )
}

//3.类式组件内部的props.childeren
class Welcome extends React.component{
    
    render(){
        return (
        	console.log(this.props.children);//Hello world!
        )
    }
}
```



# 四、State 

## 初始化state

​	组件的State应该在constructor函数中初始化，因为组件生命周期里constructe最先被调用

## 更新state

​	`setState()` 会对一个组件的 `state` 对象安排一次更新。当 state 改变了， 就会重新渲染此组件及其子组件。

​	<font color="pink">setState()并不总是立即更新</font>，为了更好的感知性能，它会批量推迟更新。这样会让获取this.state存在隐患，为了消除隐患就必须在应用更新后才能获取this.state，因此可以使用 `componentDidUpdate` 或者 `setState` 的回调函数（`setState(updater, callback)`） 。

- 对象式setState（updater：obj，cb）

- 函数式setState（updater：func，cb）

	```javascript
	//updater为函数时，state参数不应该直接改变
	this.setState((state, props) => {
	  return {counter: state.counter + props.step};
	});
	```

	

​    回调函数会在 `setState` 完成合并并重新渲染组件后执行。通常，更建议使`componentDidUpdate()` 来代替此方式。 

## 状态提升（兄弟组件通信）

​	**应用场景**：当两个子组件各自独立存在相同的state，想要共享并关联两个子组件的state，便可以进行变量提升，使两个子组件共享父组件的state。

​	**原理**：状态提升后父组件的state通过props传递给两个子组件进行共享，但是props不能直接修改，那么就需要依赖另一个知识点：<font color="pink">受控组件</font>，父组件通过props不仅要传递state给子组件，还应该传递修改父组件state的方法给子组件，子组件通过调用该方法就可以修改父组件的state，从而全局的state发生变化。

# 五、Refs与事件处理

## Refs

### 创建Refs

 Refs 是使用 `React.createRef()` 创建的，并通过 `ref` 属性附加到 React 元素（可以是DOM元素，也可以是自定义组件）。 

### 访问Refs

- ref属性用于HTML元素时，接收当前DOM元素作为其current属性。（this.myref.current）
- ref属性用于自定义类组件时，接收组件<font color="pink">实例对象</font>为其current属性。
- `ref` 属性不能用在函数组件上，因为函数组件没有<font color="pink">实例对象</font>。但可以在函数组件内部使用ref属性，只要它指向DOM元素或者类式组件。

### 将DOM  Refs转发给父组件

利用`React.forwardRef()`

```jsx
//向下转发ref到DOM元素上
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

const ref = React.createRef();//1.创建一个ref
<FancyButton ref={ref}>Click me!</FancyButton>;//2.ref作为参数，传递给forwardRef 内函数 (props, ref) => ...，作为其第二个参数。
```





## React中事件处理逻辑

​	React对浏览器原生事件进行封装，忽略了底层浏览器之间的细节差异，达到跨浏览器的兼容性问题。同时当对VDOM的元素进行事件监听的时候，没有对一一绑定事件监听，而是利用事件委托将所有统一在顶层DOM元素进行监听。

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

	在组件挂载完毕后进行AJAX请求，如果先进行AJAX请求，获取到的数据应用到组件上，由于组件为挂载完，则会报错；而在挂载之后请求数据，应用到组件上只需要setState就可以了。

### 运行过程

1. ### shouldComponentUpdate()

  控制组件是否更新，常用于性能优化

  ![./images/](E:\自学前端\Notes\Notes\images\React-shouldComponentUpdate.png)

  ```javascript
  //比较前后props和state是否更新来决定，是否进行更新，默认为true
  shouldComponentUpdate(nextProps, nextState) {
      if (this.props.color !== nextProps.color) {
        return true;
      }
      if (this.state.count !== nextState.count) {
        return true;
      }
      return false;
    }
  ```

  

2. ### render（）

3. ### getSnapshotBeforeUpdate（）

	 在最近一次渲染输出（提交到 DOM 节点）之前调用 ，它能够使组件发生改变之前从DOM中捕获信息（例如滚动位置）

4. ### componentDIdUpdate（）

### 销毁过程

​	**componentWillUnmount（）**

​		销毁之前调用，当React更新DOM的时候，如果一个DOM节点被删除时，也会触发该生命周期函数



# Redux（集中状态管理）（任意组件间通信）

用于管理全局共享数据

![1637843247881](C:\Users\AllenYan\AppData\Roaming\Typora\typora-user-images\1637843247881.png)

## 基础使用

### 1.创建Store（一个项目只有一个）

store.js文件

```javascript
//引入createStore，专门用于创建Redux的核心store对象
import {createStore} from 'redux';

//引入一个为目标组件的Reduce
import keyReduce from '/path'

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

