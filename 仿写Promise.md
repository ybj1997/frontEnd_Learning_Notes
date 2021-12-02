`Promise`构造函数实现

- 创建一个promise实例对象，有状态属性和值属性，以及回调函数数组（状态改变和产生回调函数有先后顺序）
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

-  返回一个Promise，由执行结果决定Promise的状态

   返回一个不为promise的值，则下一个回调函数默认调用成功的回调；

   抛出错误，则下一个回调默认调用失败的回调；

   返回一个promise对象，则下一个回调函数执行由该promise的状态决定

-  异常穿透，失败回调接收上一级传来的reason，如果本级没有处理失败的回调函数,抛出错误给下一级的失败回调，如果下一级也不是就继续抛直到找到失败回调

-  值的传递，成功回调接收上一级传来的value，如果本级没有处理失败的回调函数，传给下一级的成功回调，如果下一级也不是就做值的传递

```javascript
Promise.prototype.then = function(onResolved,onRejected){
	/*异常穿透与值的传递*/
    onResolved = typeof onResolved === 'function' ? onResolved : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason};
    
    /*返回一个promise对象*/
    const _this = this;
    return new Promise((resolve,reject)=>{
        	/*封装改变promise状态的函数*/
            function handlePromise(cb) {
                try {
                    let result = cb(_this.data);
                    if (result instanceof Promise) {
                        result.then(resolve, reject);
                    } else {
                        resolve(result)
                    }
                } catch (err) {
                    reject(err);
                }
            }
        if(_this.status === 'pending'){//先产生回调函数，后改变状态
            _this.callbacks.push({
                onResolved(){
                    handlePromise(onResolved)
                },
                onRejected(){
                    handlePromise(onRejected)
                }
            })
        }
        else if(_this.status === 'resolved'){//先改变状态为成功，调用成功的回调
            setTimeout(() => {
                 handlePromise(onResolved);
            })
        }else{//改变状态为失败，调用失败的回调
            setTimeout(() => {
                 handlePromise(onRejected);
            })
        }
    })
}
```

`Promise.prototype.catch`的实现

- 接收失败的回调进行处理
- 可以在链式调用的中间调用，后面的then方法默认为成功，value为undefined

```javascript
Promise.prototype.catch = function(onRejected){
	return Promise.prototype.then(undefined,onRejected);
}
```

