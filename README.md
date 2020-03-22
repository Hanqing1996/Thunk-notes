#### 函数的参数到底应该何时求值。
* 传值调用
在进入函数体之前，就计算 x + 5 的值（等于6），再将这个值传入函数 f 。C语言，js 就采用这种策略。
* 传名调用
直接将表达式 x + 5 传入函数体，只在用到它的时候求值。Hskell语言采用这种策略。


#### Thunk 函数
> 编译器的"传名调用"实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做 Thunk 函数。
```
function f(m){
  return m * 2;     
}

f(x + 5);

// 等同于
var thunk = function () {
  return x + 5;
};

function f(thunk){
  return thunk() * 2;
}
```
#### JS 的 Thunk 函数
1. 将一个多参数函数封装为单参数函数
2. 且封装后的函数只接受回调函数作为参数。
```
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
var Thunk = function (fileName){
  return function (callback){
    return fs.readFile(fileName, callback); 
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

#### Thunkify 模块
1. TJ 发明的
2. 回调函数只执行一次
```
function f(a, b, callback){
  var sum = a + b;
  callback(sum);
  callback(sum);
}

var ft = thunkify(f);
ft(1, 2)(console.log); 
// 3
```
#### thunk 配合 Generator 实现流程管理
```
// thunk 
function thunk (fn) {
    return function (){
        var args = Array.prototype.splice.call(arguments);
        return function (cb) {
            args.push(cb);
            return fn.apply(this, args);
        }
    }
}
//Generator
var step = thunk(function(value) {
    return value + 1;
});
function* gen() {
    var value1 = yield step();
    var value2 = yield step(value1);
    var value3 = yield step(value2);
    var value4 = yield step(value3);
}

function run(fn){
   var gen = fn();
   function next(value){
     var result = gen.next(value);
     if (result.done){
        return;
     }
     result.value(next)
   }
  next()
}
run(gen);
```
