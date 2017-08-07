>本文档翻译自 https://github.com/Reactive-Extensions/RxJS 

## RxJS 

### RxJS是什么？

RxJS是一套使用可观察的集合(observable collections)和[数组的一种扩展风格](https://blogs.msdn.microsoft.com/ie/2010/12/13/ecmascript-5-part-2-array-extras/)(Array#extras style[注1])组合构成的异步的和基于事件的依赖库。

这个项目的主要开发者来自微软，同时也有来自开源社区的很多开发者参与贡献。

### 使用RxJS的必要

网页应用在这些年发生了很大的变化，由最初最简单的静态页，到有动画的DHTML，再到ajax，每次我们都为我们的应用增加了更多的复杂性，更多的数据和更多的异步行为。那么我们如何管理这一切和扩展呢？实际上，通过使用事件驱动的，弹性的和可响应的“响应式体系”(Reactive Architectures)，包括RxJS提供的响应式扩展，你可以构建这样的系统。

### 关于RxJS

上文已经简单介绍了RxJS，就使用来讲，开发者可以通过使用Observables来表示异步数据流，使用RxJS提供的多种运算来查询异步数据流，并且使用Scheduler来参数化异步数据流的并发。简单的说，**RxJS = Observables + Operators + Scheduler**

开发者在使用javascript来构建网页应用或者使用node来构建服务端应用的时候，都需要同异步和事件驱动编程打交道，当然，现在已经有了比如Promise风格的编程方式，但是处理异常，取消和同步等诸如此类的难题还是比较容易出错(笔者注:在一些不是特别复杂的场景中，Promise完全够用了)。

使用RxJS，你可以表示多个异步数据流（来自不同的源，例如股票报价，推文，计算机事件和Web服务请求)，并且使用Observer对象来订阅事件流，Observable会在事件发生时通知订阅的Observer实例。

因为可观察事件的序列是数据流，所以你可以使用由Observable类型实现的标准查询运算符来查询它们，也因此，你可以使用这些运算符轻松地过滤、计算、聚合、写入和执行同时基于多个事件和时间的操作。

此外，还有许多其他的响应流允许特定的写入者编写强大的查询语句。你也可以使用Observable对象上的方法来处理取消、异常和同步。

但是最好的消息是，实际上你应该本来就了解如何使用这种方式进行编程，以下面的代码为例，我们取得一些股票数据，然后对结果进行一些处理和迭代：

```
/* Get stock data somehow */
const source = getAsyncStockData();

const subscription = source
  .filter(quote => quote.price > 30)
  .map(quote => quote.price)
  .forEach(price => console.log(`Prices higher than $30: ${price}`));
```

现在，如果这些数据来自某种事件，例如一个流(比如WebSocket)，那么我们几乎可以写相同的查询来迭代我们的数据，改变很少。

```
/* Get stock data somehow */
const source = getAsyncStockData();

const subscription = source
  .filter(quote => quote.price > 30)
  .map(quote => quote.price)
  .subscribe(
    price => console.log(`Prices higher than $30: ${price}`),
    err => console.log(`Something went wrong: ${err.message}`)
  );

/* When we're done */
subscription.dispose();
```

唯一的区别是我们可以处理我们订阅的错误，当我们不再有兴趣在数据流中接收数据时，我们可以在我们的订阅中进行处理。(注意使用subscribe而不是forEach，虽然我们也可以使用forEach，这是一个subscribe的别名，但是我们强烈建议使用subscribe)

### RxJs和Promise等其他

你也许会问自己这样一个问题，为什么要使用RxJS?Promise是否已经够用了? 实际上，Promise非常便于用来处理异步请求，例如使用XMLHttpRequest请求一个服务API，其预期的返回是一个值，然后完成这个过程。而RxJS统一了Promises、回调以及事件数据，比如DOM的输入、Web Workers和WebSocket。统一这些概念可以实现丰富的组合(rich composition)。

为了让你进一步了解所谓的rich composition，我们可以创建一个自动完成的服务，它获取用户输入进文本框的内容然后限制查询(防止用户不断的改变文本框的内容而导致调用频率太高)，

首先我们引入RxJS的依赖文件(注意: RxJS并不直接依赖jQuery，但是为了之后编码方便，我们这里还是一起引入jQuery)

```
<script src="https://code.jquery.com/jquery.js"></script>
<script src="rx.lite.js"></script>
```

接下来，我们取得用户的输入，并且通过`Rx.Observable.fromEvent`绑定用户的keyup事件，这个方法会依次查询项目是否有 jQuery, Zepto, AngularJS, Backbone.js and Ember.js 中的事件绑定方法，如果有就优先使用，如果没有就平稳回退到普通的事件绑定(之所以这样使用，是因为为了让你思考的时候可以意识到这是和RxJS强关联的内容，并没有其他特殊考虑)

```
const $input = $('#input');
const $results = $('#results');

/* Only get the value from each key up */
var keyups = Rx.Observable.fromEvent($input, 'keyup')
  .pluck('target', 'value')
  .filter(text => text.length > 2 );

/* Now debounce the input for 500ms */
var debounced = keyups
  .debounce(500 /* ms */);

/* Now get only distinct values, so we eliminate the arrows and other control characters */
var distinct = debounced
  .distinctUntilChanged();
```

接下来，我们通过维基百科提供的API进行查询，我们可以通过`Rx.observable.fromPromise`绑定符合[Promise A+](https://github.com/promises-aplus/promises-spec)的任何一个实现，直接返回即可：

```
let searchWikipedia = (term) => {
  return $.ajax({
    url: 'https://en.wikipedia.org/w/api.php',
    dataType: 'jsonp',
    data: {
      action: 'opensearch',
      format: 'json',
      search: term
    }
  }).promise();
}
```

一旦创建，我们可以将对用户输入的限制和查询服务相结合，在这种情况下，我们调用flatMapLatest来获取该值，并确保我们不会引入任何乱序列调用。

```
const suggestions = distinct
  .flatMapLatest(searchWikipedia);
```

最后，我们在我们的可观察队列上调用subscribe方法，来提取数据:

```
suggestions.subscribe(
  data => {
    $results
      .empty()
      .append($.map(data[1], value =>  $('<li>').text(value)));
  },
  error => {
    $results
      .empty()
      .append($('<li>'))
        .text(`Error: ${error}`);
  });
```

于是我们就完成了整个过程。



>注1: 衍生自一些数组方法，使得我们可以把异步事件以集合的方式进行处理。