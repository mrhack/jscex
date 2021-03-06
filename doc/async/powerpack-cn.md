# Jscex异步增强模块

[Jscex异步模块](README-cn.md)中提供了Jscex支持异步编程的核心类库，例如Task类型或是异步构造器。有了异步模块，我们就可以编写异步方法，或是将某种异步操作封装为Task对象等等。但是，异步模块除了提供了核心功能以外，并没有对日常异步开发中的常见任务给与足够的支持。而[Jscex异步增强模块](../../src/jscex-async-powerpack.js)便对常见的异步开发模式，或是异步操作的绑定模式提供了支持。

## 引入Jscex异步增强模块

如果您要使用Jscex异步增强模块，首先必须[引入Jscex异步模块](README-cn.md)，此时您会得到了一个Jscex根对象，之后再基于这个对象初始化异步增强模块。

### Node.js

目前Jscex异步增强模块尚未发布至npm，因此您需要[下载该文件](../../src/jscex-async-powerpack.js)，保存在合适的位置，并使用`require`方法引入该本地模块并初始化Jscex根对象：

    require("./jscex-async-powerpack").init(Jscex);

该方法也适用于非[Node.js](http://nodejs.org/)，但实现了[CommonJS规范](http://www.commonjs.org/)的JavaScript运行环境。

### 浏览器

如果您要在浏览器里使用Jscex异步增强模块，请[下载该文件](../../src/jscex-async-powerpack.js)，保存在合适的位置，并在页面中引入该文件即可：

    <script src="jscex-async-powerpack.js"></script>

此时异步增强模块会自动为根上的Jscex对象添加异步增强模块相关的成员。该方法也适用于各类**没有**实现CommonJS规范的JavaScript运行环境。

## 异步方法

异步增强模块中提供了一些常见的异步方法，可直接使用。这些异步方法都定义在`Jscex.Async`模块之上。

### sleep(delay, [ct])

`sleep`方法用于将当前的异步方法暂停一段时间。该方法接受两个参数：

1. `delay`：表示暂停时长，单位为毫秒。
2. `ct`：可选参数，用于取消暂停操作的CancellationToken对象。

该异步方法没有返回值。

使用示例：

    var ct = new Jscex.Async.CancellationToken();

    var printEverySecondAsync = eval(Jscex.compile("async", function (texts, ct) {        for (var i = 0; i < texts.length; i++) {            $await(Jscex.Async.sleep(1000, ct));            console.log(texts[i]);        }    }));

### onEvent(obj, eventName, [ct])

`onEvent`方法用于监听某个对象上某个事件的“下一次”触发。该方法接受三个参数：

1. `obj`：目标对象。
2. `eventName`：事件名。
3. `ct`：可选参数，用于取消监听操作的CancellationToken对象。

为了保证兼容性，`onEvent`会使用目标对象上的`addEventListener`、`addListener`或是`attachEvent`方法来监听事件，并在操作结束或是取消之后使用`removeEventListener`、`removeListener`或是`detachEvent`方法来取消监听。`onEvent`将会返回事件对象，即事件触发时传递给监听器的第一个参数。

使用示例：

    var Async = Jscex.Async;    var ct = new Async.CancellationToken();    var drawLinesAsync = eval(Jscex.compile("async", function (board, ct) {        var currPoint = $await(Async.onEvent(board, "click", ct));
        while (true) {            var nextPoint = $await(Async.onEvent(board, "click", ct));                        drawLine(                { x: currPoint.clientX, y: currPoint.clientY },                { x: nextPoint.clientX, y: nextPoint.clientY });                            currPoint = nextPoint;        }    }));

## 任务协作

Jscex异步增强模块也包含了一些常见的任务协作方式，它们作为静态方法定义在`Jscex.Async.Task`类型上。

### whenAll(t0[, t1[, t2[, …]]])

`whenAll`方法接受一系列的Task对象，并返回一个新的Task对象，该新对象只有在所有Task都成功结束之后才会结束，并使用数组返回所有Task对象的结果，其顺序与输入Task的顺序一一对应。

使用示例：

    var Task = Jscex.Task;

    var getUserItemsAsync = eval(Jscex.compile("async", function (userId) {

        var results = $await(Task.whenAll(
            queryUserAsync(userId),
            queryItemsAsync(userId)
        ));

        return {
            user: results[0],
            items: results[1]
        };
    });

在启动`whenAll`返回的新Task对象时，会立即启动所有尚未开始执行的输入Task对象。当任意一个输入Task对象发生错误时，新Task对象也会立即地直接抛出该异常，而其余正在运行的Task不受任何影响，无论成功还是失败都会执行完毕。

### whenAll(taskArray)

`whenAll`方法亦可接受一个Task对象数组`taskArray`作为参数，其余表现与上述重载完全相同。

使用示例：

    var Task = Jscex.Task;

    var getUserItemsAsync = eval(Jscex.compile("async", function (userId) {

        var tasks = [queryUserAsync(userId), queryItemsAsync(userId)];
        var results = $await(Task.whenAll(tasks));

        return {
            user: results[0],
            items: results[1]
        };
    });

### whenAll(taskMap)

`whenAll`方法的第三个重载可以接受一个对象`taskMap`作为参数，该对象上的每个字段都对应一个Task对象。该异步对象的返回值也是一个对象，包含所有Task对象的运行结果，并与输入对象的字段一一对应。该方法的其他表现与之前的两个重载完全相同。

使用示例：

    var Task = Jscex.Task;

    var getUserItemsAsync = eval(Jscex.compile("async", function (userId) {

        return $await(Task.whenAll({
            user: queryUserAsync(userId),
            items: queryItemsAsync(userId)
        }));
    });

### whenAny(t0[, t1[, t2[, …]]])

### whenAny(taskArray)

### whenAny(taskMap)

## 异步操作绑定

## 相关链接

* [源代码](../../src/jscex-async-powerpack.js)
* [异步模块](README-cn.md)
* [Node.js异步增强模块](node-cn.md)
* [浏览器异步增强模块](browser-cn.md)