> 1 道网上看到的面试题，觉得有趣，做个记录

```
实现一个LazyMan，可以按照以下方式调用:
LazyMan(“Hank”)输出:
Hi! This is Hank!

LazyMan(“Hank”).sleep(10).eat(“dinner”)输出
Hi! This is Hank!
//等待10秒..
Wake up after 10
Eat dinner~

LazyMan(“Hank”).eat(“dinner”).eat(“supper”)输出
Hi This is Hank!
Eat dinner~
Eat supper~

LazyMan(“Hank”).sleepFirst(5).eat(“supper”)输出
//等待5秒
Wake up after 5
Hi This is Hank!
Eat supper

以此类推。
```

**分析**

1. 没看到`new`构造调用，也就是需要自行返回`new LazyMan`对象
2. 链式调用，`sleep eat`等函数都需要返回已有的`lazyMan`对象
3. 每个方法都等待上一个方法执行完毕才执行，需要任务列表以及通过`next`函数自动触发下一步

**完整代码**

```javascript
// 工厂，返回一个lazyman对象
function LazyMan(name) {
  var lazyMan = new _LazyMan(name);
  lazyMan.hello();
  return lazyMan;
}

// 模拟一个类，this放属性，原型链上放方法
function _LazyMan(name) {
  this.name = name;
  this.task = []; // 任务队列
  observeTaskPush.call(this);
}

// sleep公共方法
function sleep(time) {
  return function() {
    setTimeout(
      function() {
        console.log("Wake up after " + time);
        this.next();
      }.bind(this),
      time * 1000
    );
  }.bind(this);
}

// 监听push方法
function observeTaskPush() {
  var arr = [];
  var that = this;
  var original = Array.prototype.push;
  arr.push = function() {
    var res = original.apply(this, arguments);

    // 当task任务列表从0变为1时，自动触发next
    if (that.task.length === 1) {
      setTimeout(function() {
        that.next();
      }, 0);
    }

    // 继续响应原生方法
    return res;
  };
  this.task.__proto__ = arr;
}

// 执行任务队列中的第一个
_LazyMan.prototype.next = function() {
  var fn = this.task.shift();
  fn && fn();
};

_LazyMan.prototype.hello = function() {
  var fn = function() {
    console.log("Hi This is " + this.name);
    this.next();
  }.bind(this);

  this.task.push(fn);

  return this;
};

_LazyMan.prototype.sleep = function(time) {
  var fn = sleep.call(this, time);
  this.task.push(fn);
  return this;
};

_LazyMan.prototype.sleepFirst = function(time) {
  var fn = sleep.call(this, time);
  this.task.unshift(fn);

  return this;
};

_LazyMan.prototype.eat = function(foot) {
  var fn = function() {
    console.log("Eat " + foot);
    this.next();
  }.bind(this);

  this.task.push(fn);

  return this;
};

// LazyMan('Hank')
//   .sleep(10)
//   .eat('dinner');

// LazyMan('Hank')
//   .eat('dinner')
//   .eat('supper');

// LazyMan('Hank')
//   .sleepFirst(5)
//   .eat('supper');
```
