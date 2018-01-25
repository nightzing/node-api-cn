
<!--type=misc-->

在v0.10之前的Node.js版本中，可读流接口更加简洁，但也没有那么强大。

* 与以往不同的是，[“data”][]事件现在可以立即发射，而不再需要等待调用[' stream.read()”][stream-read]方法。
  应用程序需要做一些工作来决定如何处理数据，将读取数据存储到缓冲区中，这样数据就不会丢失。
* [`stream.pause()`][stream-pause]方法是推荐使用的，但不是必须要使用的。这意味着即使当数据流处于暂停状态时，
  仍有必要接收[`'data'`][]事件。

在Node.js v0.10中，添加了[Readable] []类。 为了向后兼容旧版本的Node.js，
当[`'data'`] []事件处理程序被添加，或者当[`stream.resume（）`] [stream-resume]方法被调用时，可读流切换到“流动模式”。 
这样的作用是，即使不使用新的[`stream.read（）`] [stream-read]方法和[`'readable'`] []事件，
也不再需要担心丢失[`'data'`] []块。

虽然大多数时候应用程序将继续正常工作，除了以下几种情况：

* 没有添加['data'][]事件监听器。
* [' stream.resume()[stream-resume]方法从未被调用。
* 该流不被传送到任何可写的情况。

例如，请思考以下代码:

```js
// WARNING!  BROKEN!
net.createServer((socket) => {

  // we add an 'end' method, but never consume the data
  socket.on('end', () => {
    // It will never get here.
    socket.end('The message was received but was not processed.\n');
  });

}).listen(1337);
```

在v0.10之前的Node.js版本中，传入的消息数据将会被轻易的丢弃。 但是，在Node.js v0.10及更高版本中，套接字仍然存在。

这种情况下的解决方法是调用[' stream.resume()][stream-resume]方法开始数据流:

```js
// Workaround
net.createServer((socket) => {

  socket.on('end', () => {
    socket.end('The message was received but was not processed.\n');
  });

  // start the flow of data, discarding it.
  socket.resume();

}).listen(1337);
```

除了新的可读流切换到流动模式外，v0.10之前的Node.js版本的流可以用一个可读的类包装。
[' readable.wrap()'][' stream.wrap()']的方法。


