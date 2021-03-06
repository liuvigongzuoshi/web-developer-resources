# JS 面试常见手写工具



## 1. 节流 throttle

```javascript
function throttle(fn, threshhold = 250) {
  let timerId, last

  return function(...args) {
    const now = Date.now()
    if (last && last - now < threshhold) {
    	clearTimeout(timerId)
      timerId = setTimeout(() => {
        fn.apply(this, args)
      }, threshhold)
    } else {
      last = now
      fn.apply(this, args)
    }
  }
}

// 使用
const task = () => { console.log('run task') }
const throttleTask = throttle(task, 250)
window.addEventListener('scroll', throttleTask)
```



## 防抖 debounce

```javascript
function debounce(fn, interval = 250) {
  let timerId = null;

  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args)
    }, interval)
  }
}

// 使用
const task = () => { console.log('run task') }
const debounceTask = debounce(task, 1000)
window.addEventListener('scroll', debounceTask)
```



## 2. 柯里化

柯里化是指这样一个函数，它接收函数 A，并且能返回一个新的函数，这个新的函数能够处理函数 A 的剩余参数

- 参数不固定

```javascript
function currying(fn) {
  let argArr = [];
  let closure = function(...args) {
    if(args.length > 0) {
      argArr = [...argArr, ...args];
      return closure;
    }
    return fn(...argArr);
  }
  return closure;

}
```

- 参数固定长度

```javascript
function currying(fn) {
  return function (...args) {
    return args.length >= fn.length
      ? fn.apply(this, args)
      : currying(fn.bind(this, ...args));
  }
}
```



## 3. 数据缓冲器分片处理

```javascript
function multistep(steps, args, callback){
    let tasks = steps.concat();

    setTimeout(function(){
        let task = tasks.shift();
        task.apply(null, args || []);   //调用 Apply 参数必须是数组

        if(tasks.length > 0){
            // https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments/callee
            setTimeout(arguments.callee, 25);
        }else{
            callback();
        }
    }, 25);
}
```



## 4. imageLazyLoad

```javascript
// <img src="default.png" data-src="https://xxxx/real.png">
function isVisible(el) {
  const position = el.getBoundingClientRect()
  const windowHeight = document.documentElement.clientHeight
  const topVisible = position.top > 0 && position.top < windowHeight;
  const bottomVisible = position.bottom < windowHeight && position.bottom > 0;
  return topVisible || bottomVisible;
}

function imageLazyLoad() {
  const images = document.querySelectorAll('img')
  for (let img of images) {
    const realSrc = img.dataset.src
    if (!realSrc) continue
    if (isVisible(img)) {
      img.src = realSrc
      img.dataset.src = ''
    }
  }
}

// 使用
window.addEventListener('scroll', throttle(imageLazyLoad, 1000))
```



## 5. flatten

```javascript
function flattenArr(arr, depth = 1) {
  let res = [];
  for (let idx = 0; idx < arr.length; idx++) {
    let i = arr[idx];
    if (Array.isArray(i) && (depth > 0 || depth === Infinity)) {
      res.push(...flattenArr(i, --depth));
    } else {
      res.push(i);
    } 
  }
  return res;
}

function flatten(arr) {
    return arr.reduce(function(acc, next){
        let isArray = Array.isArray(next)
        return acc.concat(isArray ? flatten(next) : next)
    }, [])
}

if (!Array.prototype.flatten) {
    Array.prototype.flatten = function() {
        return flatten(this)
    }
}

// 使用
let arr = [1,2,[3,4, [5,6, [7, [8, 9, 10]]]]]
const source = [1, 2, [3, 4, [5, 6]], 7]
console.log(arr.flatten(source));
```



## 6. objectFlat

```javascript
function objectFlat(obj = {}) {
  const result = {}
  function flat(item, preKey = '') {
    Object.entries(item).forEach(([key, val]) => {
      const newKey = preKey ? `${preKey}.${key}` : key
      if (val && typeof val === 'object') {
        flat(val, newKey)
      } else {
        result[newKey] = val
      }
    })
  }
  flat(obj)
  return result
}

// 使用
const source = { a: { b: { c: 1, d: 2 }, e: 3 }, f: { g: 2 } }
console.log(objectFlat(source));
```



## 7. EventEmitter

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }

  on(event, callback) {
    const callbacks = this.events[event] || [];
    callbacks.push(callback);
    this.events[event] = callbacks;

    return this;
  }

  off(event, callback) {
    const callbacks = this.events[event];
    this.events[event] = callbacks && callbacks.filter(fn => fn !== callback);

    return this;
  }

  emit(event, ...args) {
    const callbacks = this.events[event];
    callbacks.forEach(fn => {
      fn(...args);
    });

    return this;
  }

  once(event, callback) {
    const wrapFun = function(...args) {
      callback(...args);

      this.off(event, wrapFun);
    };
    this.on(event, wrapFun);

    return this;
  }
}

// 使用
const eventBus = new EventEmitter()
const task1 = () => { console.log('task1'); }
const task2 = () => { console.log('task2'); }
eventBus.on('task', task1)
eventBus.on('task', task2)
setTimeout(() => {
  eventBus.emit('task')
}, 1000)
```



## 8. 观察者与发布订阅模式

### 观察者模式

```js
let observer_ids=0;
let observed_ids=0;
//观察者类
class Observer {
   constructor() {
      this.id = observer_ids++;
   }
   //观测到变化后的处理
   update(ob){
      console.log("观察者" + this.id + `-检测到被观察者${ob.id}变化`);
   }
}
//被观察者列
class Observed {
   constructor() {
      this.observers = [];
      this.id=observed_ids++;
   }
   //添加观察者
   addObserver(observer) {
      this.observers.push(observer);
   }
   //删除观察者
   removeObserver(observer) {
      this.observers = this.observers.filter(o => {
         return o.id != observer.id;
      });
   }
   //通知所有的观察者
   notify(ob) {
      this.observers.forEach(observer => {
         observer.update(ob);
      });
   }
}

let mObserved=new Observed();
let mObserver1=new Observer();
let mObserver2=new Observer();
mObserved.addObserver(mObserver1);
mObserved.addObserver(mObserver2);
mObserved.notify();
```



### 发布订阅模式



```js
//发布者
class Pub{
   constructor(dispatcher){
       this.dispatcher=dispatcher;
       this.id=observed_ids++;
   }
   /**
    * @description: 发布方法
    * @param {type} 通知类型
    */
   publish(type){
      this.dispatcher.publish(type,this)
   }
}
//订阅者
class Subscriber{
    constructor(dispatcher){
      this.dispatcher=dispatcher;
      this.id=observer_ids++;
    }
    subscribe(type){
       this.dispatcher.subscribe(type,this);
    }
    doUpdate(type,arg){
        console.log("接受到消息"+arg)
    }
}
//调度中心
class Dispatcher{
   constructor(){
      this.dispatcher={};
   }
   //订阅
   subscribe(type,subscriber){
      if(!this.dispatcher[type]){
         this.dispatcher[type]=[];
      }  
      this.dispatcher[type].push(subscriber);
   }
   //退订
   unsubscribe(type, subscriber) {
      let subscribers = this.dispatcher[type];
      if (!subscribers || !subscribers.length) return;
      this.dispatcher[type] = subscribers.filter(item =>{ 
         return item.id !== subscriber.id
      });
  }
  //发布
  publish(type, args) {
      let subscribers = this.dispatcher[type];
      if (!subscribers || !subscribers.length) return;
      subscribers.forEach(subscriber=>{
         subscriber.doUpdate(type,args);
      });        
   }
}
class Reader extends Subscriber{
   constructor(name,dispatcher){
      super(dispatcher);
      this.name=name;
   }
    doUpdate(type,st){
      //   super.update(st);
        console.log(this.name+`阅读了--${type}--公众号的文章`);
    }
}
class WeiX extends Pub{
    constructor(name,dispatcher){
       super(dispatcher);
       this.name=name;
    }
    publishArticle(type){
       this.publish(type)
    }
}

let dispatcher=new Dispatcher();
//公众号
let wei1=new WeiX("前端",dispatcher);
let wei2=new WeiX("数据库",dispatcher);
//读者们
let reader1=new Reader("小玲",dispatcher);
let reader2=new Reader("小明",dispatcher);
let reader3=new Reader("小李",dispatcher);
//读者订阅公众号
reader1.subscribe("前端");
reader2.subscribe("数据库");
reader3.subscribe("数据库");
//公众号发布文章
wei1.publishArticle("前端");
wei1.publishArticle("数据库");
```

## 9. Promise 并发限制

```js
class Scheduler {
  constructor(limit) {
    this.queue = [];
    this.limit = limit;
  }
  // promise版    
  add(promiseCreator) {
    return new Promise((resolve, reject) => {
      if (this.limit > 0) {
        this.limit--;
        resolve(promiseCreator());
      } else {
        this.list.push([resolve, promiseCreator]);
      }
    }).then(() => {
      if (this.list.length) {
        let item = this.list.shift();
        item[0](item[1]());
      } else {
        this.limit++;
      }
      return Promise.resolve();
    })
  }
  
  // promise callback 版   
  add2(promiseCreator) {
    const callback = (result) => {
      if (this.list.length) {
         let item = this.list.shift();
         item[0](item[1]());
      } else {
        this.limit++;
      }
      return result
    if(this.limit > 0) {
      this.limit --
      return promiseCreator.then(callback)
    } else {
      return new Promise((resolve, reject) => {
        this.list.push([resolve, promiseCreator]);
      }).then(callback)
    }
  }

  // await版
  async add3(promiseCreator) {
    if (this.limit <= 0) {
      await new Promise((resolve) => {
        this.queue.push(resolve);
      })
    }
    this.limit--;
    await promiseCreator();
    this.limit++;

    if (this.queue.length) this.queue.shift()();
  }
}

const timeout = (time) => new Promise(resolve => {
  setTimeout(resolve, time)
})
const scheduler = new Scheduler();
const addTask = (time, order) => {
  scheduler.add(() => timeout(time)).then(() => console.log(time, 'time, order', order))
}

addTask(1000, '1');
addTask(500, '2');
addTask(300, '3');
addTask(400, '4');
// output: 2 3 1 4
// 一开始，1、2两个任务进入队列
// 500ms时，2完成，输出2，任务3进队
// 800ms时，3完成，输出3，任务4进队
// 1000ms时，1完成，输出1
// 1200ms时，4完成，输出4
```

## 10. LRUCache
```js
class LRUCache {
  constructor(capacity = 5) {
    this.cache = new Map()
    this.capacity = capacity
  }

  /**
   * @param {number} key
   * @return {number}
   */
  get(key) {
    if (this.cache.has(key)) {
      // 存在即更新
      const temp = this.cache.get(key)
      this.cache.delete(key)
      this.cache.set(key, temp)
      return temp
    }
    return -1
  }

  /**
   * @param {number} key
   * @param {number} value
   * @return {void}
   */
  put(key, value) {
    if (this.cache.has(key)) {
      // 存在即更新（删除后加入）
      this.cache.delete(key)
    } else if (this.cache.size >= this.capacity) {
      // 不存在即加入
      // 缓存超过最大值，则移除最近没有使用的
      this.cache.delete(this.cache.keys().next().value)
    }
    this.cache.set(key, value)
  }
}
```

