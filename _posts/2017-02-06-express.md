---
layout: post
title: 路由实现解读
---

# express.js 路由实现解读
关于express.js的实现源码解读，版本为 4.14。主要为路由部分。

一个Web框架最重要的模块是路由功能，该模块的目标是：能够根据method、path匹配需要执行的方法，并在定义的方法中提供有关请求和回应的上下文。

## 模块声明

`express`中的路由模块由**Router**完成，通过完成调用`Router()`得到一个`router`的实例，`router`既是一个对象，也是一个函数，原因是实现了类似C++中的`()`重载方法，实质指向了对象的`handle`方法。`router`的定义位于<u>router/index.js</u>中。

```javascript
// router/index.js - line 42
var proto = module.exports = function(options) {
  var opts = options || {};

  // like operator() in C++
  function router(req, res, next) {
    router.handle(req, res, next);
  }
  //...
}
```

## 接口定义

`router`对外（即开发者）提供了路由规则定义的接口：`get`、`put`等对应于HTTP method类别，函数签名都是`$method(path, fn(req, res), ...)`，接口的方法通过元编程动态定义生成，可以这样做的根本原因是方法名可以使用变量的值定义和调用，Java中的反射特性也可间接实现这点，从而大量被应用于Spring框架中。

```javascript
// router/index.js - line 507
// create Router#VERB functions
// --> ['get', 'post', 'put', ...].foreach
methods.concat('all').forEach(function(method){    
  // so that we can write like 'router.get(path, ...)'
  proto[method] = function(path){     
    // create a route for the routing rule we defined
    var route = this.route(path)      
    // map the corresponding handlers to the routing rule
    route[method].apply(route, slice.call(arguments, 1));
    return this;
  };
});
```

## 路由定义

在规则定义的接口中，路由规则的定义需要`router`保存路由规则的信息，最重要的是方法、路径以及匹配时的调用方法（**下称handler**），还有其他一些细节信息，这些信息（也可以看做是配置）的保存由**Router对象**完成，一个Route对象包含一个路由规则。`Route`对象通过`router`对象的`route()`方法进行实例化和初始化后返回。

```javascript
// router/index.js - line 491
proto.route = function route(path) {
  // create an instance of Route.
  var route = new Route(path);    
  // create an instance of Layer.
  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
  }, route.dispatch.bind(route));
  // layer has a reference to route.
  layer.route = route;
  // router has a list of layers which is created by 'route()'
  this.stack.push(layer);
  return route;
};
```

`Route`的成员变量包括路径`path`，以及HTTP method的路由配置接口集，这里和`router`中一样的技巧提供了method所有类别的注册函数，此处无关紧要，只要`route`能够得到路由配置的method值即可，将method作为一个参数传入或者作为方法名调入都可以。

`route()`方法除了实例化一个`Route`外，还是实例化了一个**`Layer`**，这个的`Layer`相当于是对应`Route`的总的调度器，封装了handlers的调用过程，先忽略。

真正将handlers传入到`route`中发生在510行，也即上述`route`提供的注册函数。由于一条路由设置中可以传入多个handler，因此需要保存有关handler的列表，每一个handler由一个`Layer`对象进行封装，用以隐藏异常处理和handler调用链的细节。因此，`route`保存了一个`Layer`数组，按handler在参数中的声明顺序存放。这里体现`Layer`的第一个作用：**封装一条路由中的一个handler，并隐藏链式调用和异常处理等细节**。

```javascript
// router/route.js - line 190    
for (var i = 0; i < handles.length; i++) {
  var handle = handles[i];
  /* ... */
  // create a layer for each handler defined in a routing rule
  var layer = Layer('/', {}, handle);
  layer.method = method;

  this.methods[method] = true;
  // add the layer to the list.
  this.stack.push(layer);
}
```

返回到`router`中，最初实例化一个`route`的方法`route`中，还实例化了一个`Layer`，并且`router`保存了关于这些Layer的一个列表，由于我们可以在`router`定义多个路由规则，因此这是Layer的第二个作用：**封装一条路由中的一个总的handler，同样也封装了链式调用和异常处理等细节**。这个总的handler即是遍历调用route下的所有的handler的过程，相当于一个总的Controller，每一个handler实际上是通过对应的小的`Layer`来完成handler的调用。

由`route()`方法可知，总的handler定义在`route`的`dispatch()`方法中，该方法中，的确在遍历`route`对象下的`Layer`数组（成员变量`stack`以及方法中的`idx++`）。

```javascript
// router/index.js - line 491
proto.route = function route(path) {
  var route = new Route(path);    
  
  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
  // the 'big' layer's handler is the method 'dispatch()' defined in route 
  }, route.dispatch.bind(route));

  layer.route = route;
  
  this.stack.push(layer);
  return route;
};
```

## 路由匹配

整理路由配置过程，思考每个路由配置信息的保存位置，有：

- 路由规则，一条对应于一个`Route`中，并包装一个`Layer`。
- 所有路由规则保存在`Router`中的`stack`数组中。
- 对于一个路由规则：
  - 路径在`Route`和`Layer`的成员变量`path`。
  - HTTP method在`Route`下每个handler对应的`Layer`中的`method`成员变量，以及`Route`下的成员变量`methods`标记了各个method是否有对应的`Layer`。
  - handler，每一个都包装成一个`Layer`，所有的`Layer`保存在`Route`中的`stack`数组中。

有了如上信息，当一个请求进来需要寻找匹配的路由变得清晰。路由匹配过程定义在`Router`的`handle()`方法中（<u>router/index.js</u> 135行）（**回顾**：`Router()`方法实际上调用了`handle()`方法。)

`handle()`方法中，不关注解析url字符串等细节。从214行可发现，**不考虑异常情况**，寻找匹配路由的过程其实是遍历所有`Layer`的过程：

1. 对于每个`Layer`，判断`req`中的`path`是否与`layer`中的`path`匹配，若不匹配，继续遍历（path匹配过程后述）；

2. 若path匹配，则再取`req`中的`method`，通过`route`的`methods`成员变量判断在该`route`下是否存在匹配的method，若不匹配，继续遍历。

3. 若都匹配，则提取路径参数（形如`/:userId`的通配符），调用关于路径参数的handler。(通过`router.param()`设置的中间件)

4. 调用路由配置`route`的handlers，这又是遍历`route`下的小的`Layer`数组的过程。

5. 决定是否返回1继续遍历。返回到`stack`的遍历是通过**尾递归**的方式实现的，注意到`next`被传入`layer.handle_request`的方法中，`handle_request`中处理事情最后向`handler`传入`next`，从而是否继续遍历取决于handler的实现是否调用的`next()`方法。express的实现大量使用尾递归\尾调用的模式，如`process_params()`方法。



简化版的路由匹配过程如下所示：

```javascript
   // router/index.js - line 214
   proto.handle = function handle(req, res, out) {
    
     // middleware and routes
     var stack = self.stack;
     next();
     
     // for each layer in stack     
     function next(err) {  
       // idx is 'index' of the stack
       if (idx >= stack.length) {            
         setImmediate(done, layerError);
         return;
       }
       // get pathname of request
       var path = getPathname(req);
       
       // find next matching layer
       var layer;
       var match;
       var route;
       while (match !== true && idx < stack.length) {
         layer = stack[idx++];
         
         // match the path ?
         match = matchLayer(layer, path);    
         route = layer.route;      
         if (match !== true) {
           continue;
         }
         
         // match the method ?
         var method = req.method;
         var has_method = route._handles_method(method);
         if (!has_method && /**/) {          
           match = false;
           continue;
         }           
       }

       // no match
       if (match !== true) {
         return done(layerError);
       }

       // Capture one-time layer values
       // get path parameters.
       req.params = /*...*/;      
       
       // this should be done for the layer
       // invoke relative path parameters middleware, or handlers 
       self.process_params(layer, paramcalled, req, res, function (err) {
         if (route) {
           // invoke all handlers in a route
           // then invoke the 'next' recursively
           return layer.handle_request(req, res, next);    
         }
       });
     }
   }
```

## 特殊路由

在路由匹配的分析中，省略了大量细节。

- 通过`Router.use()`配置的**普通中间件**：默认情况下，相当于配置了一个`path`为`'/'`的路由，若参数提供了`path`，则相当于配置了关于`path`的全method的路由。不同的是，handlers不使用`route`封装，每一个handler直接使用一个大的`Layer`封装后加入到`Router`的`stack`列表中，`Layer`中的`route`为`undefined`。原因是`route`参杂了有关http method有关的判断，不适用于全局的中间件。

- 通过`Router.use()`配置的**子路由**， `use()`方法可以传入另一个`Router`，从而实现**路由模块化**的功能，处理实际上和普通中间件一样，但此时传入handler为`Router`，故调用Router()时即调用`Router`的`handle()`方法，使用这样的技巧实现了子路由的功能。

  ```javascript
  // router/index.js - line 276
  // if it is a route, invoke the handlers in the route. 
  if (route) {
  	return layer.handle_request(req, res, next);
  }
  // if it is a middlewire (including router), invoke Router().
  trim_prefix(layer, layerError, layerPath, path);
  ```

  子路由功能还需要考虑父路径和子路径的提取。这在`trim_prefix`方法（<u>router/index.js</u> 212行），当`route`为`undefined`时调用。直接将`req`的路径减去父路由的`path`即可。为了能够在子路由结束时返回到父路由，需要从子路径恢复到带有父路径的路径（信息在`req`中)，结束时调用`done()`，`done`指向`restore()`方法，用于恢复`req`的属性值。

  ```javascript
  // router/index.js - line 602
  // restore obj props after function
  function restore(fn, obj) {
    var props = new Array(arguments.length - 2);
    var vals = new Array(arguments.length - 2);
    // save vals.
    for (var i = 0; i < props.length; i++) {
      props[i] = arguments[i + 2];
      vals[i] = obj[props[i]];
    }

    return function(err){
      // restore vals when invoke 'done()'
      for (var i = 0; i < props.length; i++) {
        obj[props[i]] = vals[i];
      }

      return fn.apply(this, arguments);
    };
  }
  ```

- 通过`app`配置的**应用层路由和中间件**，实际上由`app`里的成员变量`router`完成。默认会载入`init`和`query`中间件（位于<u>middleware/</u>下），分别用于初始化字段操作以及将`query`解析放在`req`下。

- 通过`Router.param()`配置的**参数路由**，`router`下`params`成员变量存放`param`映射到`array[: handler]`的map，调用路由前先调用匹配参数的中间件。

## 路径参数

现在考虑带有参数通配符的路径配置和匹配过程。细节在`Layer`对象中。

路径的匹配实际上是通过正则表达式的匹配完成的。将形如

```javascript
'/foo/:bar'
```

转为

```javascript
/^\/foo\/(?:([^\/]+?))\/?$/i
```

正则的转换由第三方模块`path-to-regex`完成。解析后放在`req.params`中。

## 链式调用和异常处理

在handler的调用中都使用了尾调用\尾递归模式设计（也可以理解为责任链模式、管道模式），包括：

- `Router`中的`handle`方法调用匹配路由的总handler和中间件。
- `Router`中的路径参数路由（`params`）的调用过程。
- `Route`中`dispatch`方法处理所有的handlers和每一个`Layer`中的handle配合。


链式调用示意图：

- 每一个节点都不了解自身的位置以及前后关系，调用链只能通过`next()`调用下一个，若不调用则跳过，并调用`done()`结束调用链。
- 调用链的一个环节仍可以是一个调用链，形成层次结构（**思考上述提到的大`Layer`和小`Layer`的关系**）
- 子调用链中的`done()`方法即是父调用链中的`next()`方法。
- 出现异常则：
  1. 若能够接受继续进行，不中断调用链，则可以继续调用`next`方法，带上`err`参数，即`next(err)`。最终通过`done(err)`将异常返回给父调用链。
  2. 若不能接受，需要中断，则调用`done`方法，，带上`err`参数，即`done(err)`。

![Capture](image\Capture.PNG)

-- Fin -- 

---

## 进阶

- 视图渲染模块 render实现，在<u>applications.js</u> 和 <u>view.js</u> 中。
- 对`req`和`res`的扩展，header处理。
- express从0.1、1.0、2.0、3.0、4.0的变化与改进思路。
- 与koa框架的对比

## 感想

- express的代码其实不多。
- 路由部分其实写得还是比较乱，大量关于细节的if、else判断，仍是过程式的风格，功能的实现并没有特别的算法技巧，尤其是路由，直接是一个一个试的。框架的实现并不都是所想的如此神奇或者高超。
- 一些不当的代码风格，如route.get等API中没有在函数签名中写明handler参数，直接通过argument数组取slice得到，而且为了实现同一函数名字的不同函数参数的重载，不得不在函数中判断参数的类型再 if、 else 。（js不支持函数重载）