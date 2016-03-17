---
title: Java NIO学习笔记
date: 2016-03-17 20:11:32
tags:[NIO]
---
# Java NIO相关
NIO创建的目的是为了让程序员实现更快的I/O而无需编写自定义的本机代码。NIO将最耗时的I/O操作（即填充和提取缓冲区）转移回操作系统，因此可以极大的提高效率。
## NIO组成
Java NIO由下面几个核心部分组成：
+ Channels
+ Buffers
+ Selectors
通道是对原 I/O 包中的流的模拟。到任何目的地(或来自任何地方)的所有数据都必须通过一个 Channel对象。一个Buffer实质上是一个容器对象。发送给一个通道的所有对象都必须首先放到缓冲区中；同样地，从通道中读取的任何数据都要读到缓冲区中。

### Channel
+ FileChannel：从文件中读写数据
+ DatagramChannel：通过UDP读写数据
+ SocketChannel：通过TCP读写数据
+ ServerSocketChannel：可以监听新进来的TCP连接。对每个新进来的连接都会创建一个SocketChannel
### Buffer
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。
#### Buffer的capacity, position, limit
>![](http://ifeve.com/wp-content/uploads/2013/06/buffers-modes.png)
#####capacity
作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
#####position
当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
#####limit
在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。

当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

#### flip()方法
flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。

#### rewind()方法
Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

#### mark()和reset()方法
通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。

#### slice()方法
slice() 方法根据现有的缓冲区创建子缓冲区 。其中**新缓冲区与原来的缓冲区的一部分共享数据**。
在某种意义上，子缓冲区就像原来的缓冲区中的一个窗口 。窗口的起始和结束位置通过设置 position 和 limit 值来指定，然后调用 Buffer 的 slice() 方法：
```java
ByteBuffer buffer = ByteBuffer.allocate( 10 );
for (int i=0; i<buffer.capacity(); ++i) {
     buffer.put( (byte)i );
}
buffer.position( 3 );
buffer.limit( 7 );
ByteBuffer slice = buffer.slice();
```
上面的代码创建了一个包含槽3~槽6的子缓冲区
#### 直接和间接缓冲区
Sun的官方文档是这样描述直接缓冲区的。
> 给定一个直接字节缓冲区，Java 虚拟机将尽最大努力直接对它执行本机 I/O 操作。也就是说，它会在每一次调用底层操作系统的本机 I/O 操作之前(或之后)，尝试避免将缓冲区的内容拷贝到一个中间缓冲区中(或者从一个中间缓冲区中拷贝数据)。
```java
ByteBuffer buffer = ByteBuffer.allocateDirect( 1024 );
```
