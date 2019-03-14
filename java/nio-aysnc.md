# [技术备忘录](../README.md) | [Java](README.md) | 利用NIO进行异步操作
## 目录
  1. [Selector](#selector)
  2. [Setting up an Asynchronous Network Reader](#set-up-async-network-reader)

## 问题
### 1. [Selector](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Selector.html)<a name="selector"></a>[↑](#top)
* The Selector is the entry object to set up an asynchronous system
  1. create a channel
  2. configure the channel to be non-blocking
  3. register this channel with this selector
     * A single selector can handle many channels.
  4. get the registration key
* Then the Channel will fire events to the selector.
* There are 4 events defined:
  * READ, WRITE: the channel is ready for reading or writing
* In case of socket channels:
  * CONNECT: a connection was established
  * ACCEPT: a connection was accepted
* The registration is configured to listen to certain events.
  * Those events are passed as parameters during the registration process.
  * But we can alos modify those listened events directly through the key.

* Once the system is set up
    1. we call select() on the selector.
       * This would be a blocking call until some other channels registered to
         this selector have events to be consumed.
    2. from the selector, we can get the list of the keys associated to those channels
       that have events to be processed.
### 2.Setting up an Asynchronous Network Reader<a name="set-up-async-network-reader"></a>[↑](#top)
```java
// 1. create a server socket channel and configure this channel to be non-blocking
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.configureBlocking(false);

// 2. then create a server socket from this channel and bind it to a port
ServerSocket serverSocket = serverSocketChannel.socket();
serverSocket.bind(new InetSocketAddress(12345));

// 3. Then we need to create a selector object.
Selector selector = Selector.open();

// 4. And register this server socket channel to this selector on the event OP_ACCEPT
//    This register call returns a selection key that should be stored for future use,
//    like unregistering this channel, checking for the validity, or changing the events
//    we are listening to.
// This step(registration) can be repeated with as many socket channels as we have, with
// as many ports we are listening to.
SelectionKey key = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

// 5. Then when the registrion process is done, we enter the event consuming loop. 
//    All we have to do is to call the select method on our selector object.(Remember that
//    this selector object can have many channels registered to it.) This select call is
//    blocking until there are events available in some of the channels. n is the number of
//    keys with available events, that is, with data ready to be consumed.
int n = selector.select();

// 6. Then after that, we need to call selector.selectedKeys(). It will return a set of
//    corresponding selection keys, that is, the registration keys that generated events.
Set<SelectionKey> selectedKeys = selector.selectedKeys();

// 7. Each key can be processed in a loop
for(SelectionKey key: selectedKeys){
    // 8. We make sure that the event is a connetion request as below:
    //    Checking if the selection key indeed received an accept event is done by
    //    invoking the readyOps method on the key object. This returns an int,
    //    which is in fact a filter of bits. And we need to check if the correspoding
    //    bit to the accept event has been set to 1 or not. 
    if ((key.readyOps() & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT) {
       // 9. Now that we have a connetion request, then we can get the corresponding
       //    channel using the key, because the key knows the channel it is linked to.
       ServerSocketChannel channel = (ServerSocketChannel)key.channel();
       // 10. And from this channel, by calling the accept method, we can open a second socket,
       //     which is a socket channel dedicated to the communication to the client that made
       //     the connection request.
       SocketChannel socketChannel = channel.accept();
       // 11. Of course, we are in a non-blocking world, we are not going to create a synchronous
       //     socket. We also configure this socket to be non-blocking. 
       socketChannel.configureBlocking(false);
       // 12. And we can also register this new channel using the same selector we already have on
       //     the read event to get the incoming data from this new client. 
       socketChannel.register(selector, SelectionKey.OP_READ);
       // At this point, our selector is registered with two channels, and will get the events
       // from both channels
       // 1) the first channel listens to incoming request.
       // 2) the second channel has been created to handle the communication with a given client
       // So we need to add the handling of the READ event for this second channel.
    } else if ((key.readOps() & SelectionKey.OP_READ) == SelectionKey.OP_READ) {
        // This is the READ handling part
        // First, get the corresponding channel and read it in a byte buffer
        SocketChannel channel = (SocketChannel)key.channel();
        // This accept method will return immediately, because if we are in this code, it means that
        // the data from this socket channel is available. If the data was not available, then we would
        // not be running this callback, that is only invoked only if the data is available.
        channel.accept(byteBuffer);
        // And once the data is in the byte buffer, then we can do something with it, whatever we need,
        // analyze the content, get the content, and act accordingly. This is where our business, our
        // application code will live.
        /// begin region - business code
        /// ...
        /// end region - business code

        byteBuffer.clear();
        // And there is some technical code to continue.
        // First, remove the key from the selected keys.
        selectedKeys.remove(key);
        // Cancel the key since we are done consuming the information
        key.cancel();
        // And probably closing this channel, since this channel was created just to handle this particular
        // communication with this client. Closing the channel with unregister this channel from the selector.
        channel.close();

    }
}
```

### 3. Setting up the selector to accept an incoming request.
* This is the first step: an incoming request has been received
    1. we need to make sure that the key corresponding to a connection request
    2. then we need to set up a connection with a socket and listen to the events on this socket