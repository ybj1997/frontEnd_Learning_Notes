`Promise`构造函数实现

- 创建一个promise实例对象，有状态属性和值属性，以及回调函数数组
- 参数是excutor执行器函数，并在创建promise对象实例的时候同步执行excutor
- 有resolve、reject、抛异常三种形式改变实例的状态
- 回调函数的执行需要两个条件：1.状态发生改变    2.指定回调函数，两个条件先后都有可能

```javascript
function Promise(excutor){
    /*初始化实例对象状态*/
	this.status = 'pending';
    this.data = undefined;
    this.callbacks = [];//可能先产生会函数，也可能先改变状态
    
    function resolve(value){
        if(this.status !== 'pending') return;
        this.status = 'resolved';
        this.data = value;
        setTimeout(()=>{//利用定时器模拟异步微任务
            if(this.callbacks.length > 0){
                this.callbacks.forEach( cbObj => {
                    cbObj.onResolved(value);
                })
            }
        })
    }
    function reject(reason){
        if(this.status !== 'pending') return;
        this.status = 'rejected';
        this.data = value;
        setTimeout(()=>{//利用定时器模拟异步微任务
            if(this.callbacks.length > 0){
                this.callbacks.forEach( cbObj => {
                    cbObj.onRejected(reason);
                })
            }
        })
    }
    
    /*同步执行执行器函数，并解决抛出异常的结果*/
    try{
        excutor(resolve,reject);
    }catch(err){
        reject(err);
    }
}
```

`Promise.prototype.then()`实现

-  :exclamation: then（）调用是同步执行的，内部的回调函数是异步执行的

```javascript
Promise.prototype.then = (onResolved,onRejected)=>{
	this.callbacks.push({onResolved,onRejected})
}
```

