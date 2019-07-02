#### String

```
if(Object.prototype.toString.call('我是个string') === '[object String]') {
  //String
}
```

#### Array

```
if(Object.prototype.toString.call([1,2,3]) === '[object Array]') {
  //Array
}
```

#### Number

```
if(Object.prototype.toString.call(45) === '[object Number]') {
  //Number
}
```

#### Boolean

```
if(Object.prototype.toString.call(false) === '[object Boolean]') {
  //Boolean
}
```

#### function

```
function isFunction(functionToCheck) {
 var getType = {};
 return functionToCheck && getType.toString.call(functionToCheck) === '[object Function]';
}

var a = function() {};

console.log(isFunction(a)); //true
```
