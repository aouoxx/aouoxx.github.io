---
layout: post
title: java-io/nio
categories: java
description: java-io/nio
keywords: java
---

 <meta name="referrer" content="no-referrer"/>

### nio

```java
java nio的核心在于：通道(Channel)和缓冲区(Buffer)
通道表示打开到IO设备(例如:文件，套接字)的链接。
如果需要使用NIO，需要获取用于链接IO设备的通道，以及用于容纳数据的缓冲区。
然后操作缓冲区，对数据进行处理。简而言之，channel 负责传输，Buffer负责存储。

"nio的特性"
 NIO是New I/O简称,与旧式的基于流的I/O的方法相对,从名字看,它表示新的一套Java I/O标准。其特性如下
 —— nio 是基于块(Block)的,它以块为基本单元处理数据。(硬盘上存储数据的方式)
 —— 为所有的原始类型提供(Buffer)缓存支持
 —— 增加通道(Channel)对象,作为新的原始I/O抽象
 —— 支持锁和内存映射文件的文件访问接口
 —— 提供了基于Selector的异步网络IO

> 标准的io是面向字节流和字符流的,而nio则是面向通道和缓冲区的,数据总是从通道中读到buffer缓存区内,或者从buffer写入到通道中
> java nio的优势可以使我们进行非阻塞io操作。比如单线程中从通道中读取数据到buffer,同时可以继续做别的事情,当数据读取到
    buffer中后,线程再继续处理数据。写数据也一样
> java nio中有一个'selectors'的概念,selector可以检测多个通道的事件状态,

```

```java
'通道和缓冲区'
    通常来说NIO的所有IO都是从channel开始的。channel和流有点类似,通过Channel,我们即可以从channel把数据写到buffer中,也可以把数据从buffer中读到channel
 'channel主要的类'
   FileChannel //从文件读写数据
   DatagramChannel //通过UDP读写网络中的数据
   SocketChannel //通过TCP读写网络数据
   ServerSocketChannel //可以监听新进来的TCP连接,并

 缓冲区Buffer
   buffer是一个对象,它包含一些要写入或者读取的数据。在NIO类库中加入Buffer对象,体现了新库与原IO的一个重要的区别。
   在面向流的IO中,可以将数据直接写入或读取到Stream对象中,在NIO库中,所有数据都是用缓冲区处理(读写)。
   缓冲区的本质是一个数组,通常它是一个字节数组(ByteBuffer),也可以使用其他类型的数组。这个数组为缓冲区提供了数据访问读写等操作属性,如位置,容量,上限等
 'Buffer主要的类'
   ByteBuffer
   CharBuffer
   DoubelBuffer
   FloatBuffer
   IntBuffer
   LongBuffer
   ShortBuffer

 '选择器'
   选择器允许单线程操作多个通道。如果你的程序中有大量的连接,同时每个链接的IO带宽不高的话,这个特性非常好用。
   要使用Selector的话,我们必须把Channel注册到Selector上,然后就可以调用Selector的selectr()方法。
   这个方法会进入阻塞,指到有一个channel的状态符合条件。
```

### Buffer

```java
缓冲区本质上是一块可以写入数据,然后从中读取数据的内存块, 可以理解成是一个容器对象(含数组)。
这块内存被包装成NIO Buffer对象,并提供一组方法,用来方便的访问该快内存。
缓冲区就是在内存中预留指定大小的存储空间用来对输入/输出(I/O)的数据作临时存储,这部分预留的内存空间就叫做缓冲区。
使用缓冲区的好处
    > 减少实际的物理读写次数
    > 缓冲区在创建时就被分配内存,这块内存区域一直被重用,可以减少动态分配和内存回收的次数


"buffer的基本用法"
    > 写入数据到buffer
    > 调用flip()方法 //将写模式转换为读模式
    > 从Buffer中读取数据
    > 调用clear()或者compact()方法
当向buffer写入数据时,buffer会记录下写了多少数据。一旦要读取数据,需要通过flip()方法将buffer从写模式切换到读模式。
在读模式下,可以读取之前写入到buffer的所有数据。

一旦读写了所有的数据,就需要清空缓存区,让它可以再次被写入。有两种方式清空缓存区,调用clear()或compact()方法
> clear()方法会清空整个缓存区
> compact()只会清除已经读过的数据,任何未读的数据都被移动到缓存区的起始位置,新写入的数据将放到缓冲区未读数据的后面。

RandomAccessFile aFile = new RandomAccessFile("d:/ay.txt","rw");
FileChannel fileChannel = aFile.geyChannel();
//分配缓冲区的大小
ByteBuffer buf = ByteBuffer.allcate(48);
int byteRead = fileChannle.read(buf);
while(byteRead!=-1){
    System.out.println("Read:"+bytesRead);
    //buf.flip()的调用,首先读取数据到Buffer,然后反转Buffer,接着在从Buffer中去读数据
    buf.flip();
    //判是否有剩余
    while(buf.hasRemaing()){
      System.out.print((char)buf.get());
    }
    buf.clear();
    bytesRead=fileChannel.read(buf);
}
```

```java
public class RandomFile {

    public static void main(String[] args) throws Exception {
        RandomAccessFile rf = new RandomAccessFile("/Users/ssgao/Downloads/test-ss.txt","rw");
        FileChannel channel = rf.getChannel();
        /** mmap 内核系统调用！ */
        MappedByteBuffer byteBuffer = channel.map(FileChannel.MapMode.READ_WRITE,0,2048);
        /** 堆外空间 直接映射 内核可以访问*/
        // ByteBuffer bf = ByteBuffer.allocate(1024); // 分配在堆上, heap空间
        // ByteBuffer dirBf = ByteBuffer.allocateDirect(1024) ; // 分配在堆外, offheap空间
        byteBuffer.put("hello world~~".getBytes()); // 调用put后内容直接被放入文件
        TimeUnit.SECONDS.sleep(2);
    }
}
```

#### buffer 的三个重要属性

```java
"capacity"
    作为一个内存块,buffer有一个固定的大小值,也叫capacity。
    我们只能向buffer中写入capacity个byte,long,char等类型,一旦buffer满了,需要将其清空(通过读数据或清除数据)才能继续写数据。
"position"
    当你写数据到Buffer中时,position表示当前位置。初始的position值为0,当一个byte,long等数据写到buffer后。
    position会向前移动到写一个可以插入数据的Buffer单元。
    position最大可谓capacity-1。当读取数据的时候可以从从某个特定位置读取。
    当将Buffer从写模式切换到读模式时,position会被重置为0,当从buffer的position处读取数据时,position向前移动到下一个可读位置。
"limit"
    在写模式下,Buffer的limit表示我们能最多向buffer里面写入多少数据。
    写模式,limit等于Buffer的capacity。当切换Buffer到读模式时,limit表示我们最多能读到多少数据。
    因此,当切换buffer到读模式下,limit会被设置成写模式下的position值。也就是说,我们能读取到之前写入的所有数据(limit被设置成已写数据的数量,这个值在写模式下就是position)


     参数      |                   写模式                    |           读模式
-----------------------------------------------------------------------------------------------
位置(position)  |  缓冲区的位置,将从position的下一个位置写数据  | 缓冲区读取的位置,将从此位置后,读取数据
-------------------------------------------------------------------------------------------------------
容量(capacity)  | 缓冲区的总容量上线                           | 缓冲区的总容量上限
-----------------------------------------------------------------------------------------------------
上限(limit)     | 缓冲区的实际上限,小于等于容量.通常等于容量     | 代表可读取的总容量,和上次写入的数据量相等
-------------------------------------------------------------------------------------------------------

```

![image.png](https://cdn.nlark.com/yuque/0/2021/png/659846/1610803958539-7dd07f45-8759-470a-af5f-e7de00c1103a.png#height=260&id=FBsFD&margin=%5Bobject%20Object%5D&name=image.png&originHeight=208&originWidth=432&originalType=binary&ratio=1&size=19210&status=done&style=none&width=541)

#### **buffer 的相关类**

```java
"java nio有以下Buffer类型"
 ByteBuffer
 MappedByteBuffer  --> mmap
 CharBuffer
 DoubleBuffer
 FloatBuffer
 IntBuffer
 LongBuffer
 ShortBuffer



```

#### mappedbytebuffer

```java
MappedByteBuffer 使用直接内存的方式
    MappedByteBuffer是一种特殊的buffer,即直接缓冲区,其内容是文件的内存映射区域.
    ByteBuffer所使用的allocateDirect方式创建的buffer也是MappedByteBuffer类型。
    MappedByteBuffer类为抽象类,其子类为DirectByteBuffer.MappedByteBuffer也可以通过FileChannel.map方式获取。
    MappedByteBuffer和它所表示的文件映射关系在该缓冲区本身成为垃圾回收缓冲区之前一直保持有效。

MappedByteBuffer较之ByteBuffer新增的三个方法
    force() 缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文件
    load() 将缓冲区的内容载入内存，并返回该缓冲区的引用
    isLoaded() 如果缓冲区的内容在物理内存中，则返回真，否则返回假



"MappedByteBuffer map(MapMode mode,long position, long size)"
    FileChannel提供了map方法来把文件映射为MappedByteBuffer,可以把文件从position开始的size大小的区域映射为MappedByteBuffer。
    mode指出了可访问该内存映像文件的方式,有三种,分别为
    MapMode.READ_ONLY(只读):  试图修改得到的缓冲区将导致抛出ReadOnlyBufferException
    MapMode.READ_WRITE(读/写): 对得到的缓冲区的更改最终将写入文件,但该更改对映射到同一文件的其他程序不一定是可见的。
    MapMode.PRIVATE(专用): 可读可写,但是修改的内容不会写入文件,只是buffer自身的改变,这种能力成为"copy on write"
```

```java
RandomAccessFile raf = new RandomAccessFile("c:\\mapfile.txt","rw");
FileChannel fc = raf.getChannel();
//将文件映射到内存(堆外内存)中 mmap
/**
 * 参数1: FileChannel.MapMode.READ_WRITE  使用读写模式
 * 参数2: 0 可以直接修改的起始位置
 * 参数3: raf.length() 映射到内存的大小
 */
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE,0,raf.length());
while(mbb.hasRemaing()){
    sout((char)mbb.get());
}
mbb.put(0,(byte)98); //对buffer中的内容进行修改, 相当于修改了文件
raf.close();

```

#### buffer 相关的方法

```java
> buffer的分配
 要想获得一个Buffer对象首先要进行分配。每一个Buffer类都有一个allocate方法,allocateDirect方法。
 ByteBuffer buf = ByteBuffer.allocate(48); //分配48字节capacity的ByteBuffer,在堆空间分配
 CharBuffer buf = CharBuffer.allocate(1024); //分配一个可存储1024个字符的CharBuffer

> Buffer写数据
 写数据到buffer有两种方式:从channel写到Buffer/通过Buffer的put()方法写到Buffer里面
 从channel写到Buffer
  int bytesRead = inChannel.read(buf);// read into Buffer
 通过put方法写Buffer
  buffer.put(127); //put方法有很多版本,允许我们以不同的方式把数据写入到buffer中。
 例如,写到一个指定的位置或者把一个字节数组写入到Buffer
 put(int index,byte b)
     绝对写,向byteBuffer中下标为index的位置插入byte b,不改变position
 put(ByteBuffer src)
    把src中可读的部分(也就是position到limit)写入byteBuffer。 //注意buffer的反转
 put(byte[]src, int offset,int length)
    从数组中的offset到offset+length区域读取数据并使用相对写写入此byteBuffer

> Buffer中读取数据
  从Buffer中读取数据有两种方式
  >从Buffer读取数据到Channel
  >使用get()方法从Buffer中读取数据
  int byteWriter = fileChannel.write(buf);//从buffer中读取数据到channel
  byte aBye=buf.get();//使用get()方法读取数据
  get方法有很多版本,允许你以不同的方式从buffer中读取数据,例如从指定position读取或者从Buffer中读取数据到字节数组。
  get()
      从position位置读取一个byte,并将position+1,为下次读写做准备
  get(int index)
      读取byteBuffer中下标为index的byte,不改变position
  get(byte[]dst,int offset,int length);
      从position位置开始相对读，读length个byte，并写入dst下标从offset到offset+length的区域
"flip()方法"
  flip方法将Buffer模式反转
  调用flip()方法会将position设置为0,并将limit设置成之前position的值。limit=position;position=0;
  换句话说,position现在用于标记读的位置,limit表示之前写进了多少byte,char等--现在能读取多少个byte,char等。
"remaining()"
      return limit-position;返回limit和position之间相对位置差。
"hasRemaining()"
      return position<limit;返回是否还有未读数据内容
"rewind()方法"
  Buffer.rewind()将position设置为0，所以你可以重读Buffer中所有数据。
  limit保持不变,仍然表示能从Buffer中读取多少元素(byte,char等）

"clear()"
  一旦读完Buffer中的数据,需要让Buffer准备好再次被写入,可以通过clear()或compact()方法来完成
  clear()方法,position设置回0,limited被设置成capacity的值。换句话说buffer被清空了。
  如果buffer中有一些未读数据,调用clear()方法,数据将被"遗忘"

"compact()方法"
  compact()方法,如果buffer中有数据未读取,且后续要需要这些数据,但此时想先写入一些数据。
  compact()方法将所有未读的数据拷贝到Buffer起始处,然后将position设置到最后一个未读元正后面。
  limit属性依然像clear()方法一样,设置成capacity,现在Buffer准备好写数据了,并且不会覆盖未读数据

"mark()与reset()"
   mark()方法,可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。
   buffer.mark();
   buffer.reset();

"equals()方法"
  当满足下面条件是,表示两个buffer相等
  > 有相同的类型(byte,char,int等）
  > Buffer中剩余的byte,char等的个数相等
  > Buffer中所有剩余的byte,char等都相同
  如我们所见,equals只是比较Buffer的以部分,不是每一个在它里面的元素都比较。实际上它只比较Buffer中的剩余元素
"reset()方法"
  把position设置成mark的值,相当于之前做过一个标记,现在要退回到之前标记的地方
"position()方法"
  获取buffer的position的属性信息
"position(int position)"
  重置position参数.参数值需要符合边界限制.
```

### channel

#### channel 的简介

```java
"channel 简介"
  channel 是对数据的源头和数据目标点流途径的抽象,在这个意义上和InputStream和OutputStream类似。
  Channel是可以翻译为"通道,管道",而传输中的数据访问就像是其中流淌的水。
  Buffer和Channel的相互配合使用,才是java的NIO

"java nio的通道与流的区别"
> 即可以从通道中读取数据,又可以写数据到通道。但流的读写通常是单向的
> 通道可以异步读写
> 通道中的数据总是先读到一个buffer，或总是从一个buffer中写入
> channel用于在字节缓存区和位于Channel另一侧(通常是一个文件或套接字)之间有效的传输数据

"注意,通道必须结合buffer使用,不能直接向通道中读/写数据"
```

#### filechannel 的使用和实例

```java
FileChannel类代表与文件相连的通道。

使用FileChannel必须先打开它,但是我们无法直接打开一个FileChannel,需要通过一个InputStream,OutputStream或RandomAccessFile来获取一个FileChannel实例(通过对应的getChannel()方法来获取一个FileChannel对象)。
> 该实例实现了ByteChannel, ScatteringByteChannel和GatheringByteChannel接口，支持读写操作，分散读操作和集中写操作。

"FileChannel读取数据"
    RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
    FileChannel inChannel = aFile.getChannel();  //获取channel
    ByteBuffer buf = ByteBuffer.allocate(48);  //开辟48byte缓存
    //read()方法返回的int值表示了有多少字节被读到了Buffer中。如果返回-1,表示到了文件末尾
    int byteRead = inChannel.read(buf);  //从channle读取数据
    while (byteRead != -1){
       System.out.println("Read" + bytesRead);
        buf.flip();  //反转读写操作
        while(buf.hasRemaining()){
            System.out.println((char)buf.get());
        }
        buf.clear();  //清空缓存
        bytesRead = inChannel.read(buf);
    }

"向FileChannel中写数据"
    使用FileChannel.write()方法向FileChannel写数据.该方法的参数是一个Buffer
        String newData="New String to Writer to File ...";
        ByteBuffer buf = ByteBuffer.allocate(48);
        buf.clear();
        buf.put(newData.getBytes());
        buf.flip();
        // 因为无法保证write()方法一次能向FileChannel写入多少字节,
        // 因此重复调用write()方法,直到Buffer中没有尚未写入通道的字节
        while(buf.hasRemaining()){
            channel.write(buf);
        }
"关闭FileChannel"
        channel.close();

```

##### filechannel 的写

```java
public class FileChannelDemo {
    public static void main(String[] args) throws Exception {
        /**  创建一个输出流 -> Channel*/
        FileOutputStream fileOutputStream = new FileOutputStream("/Users/ssgao/Downloads/test-aa.txt");
        /** 获取fileChannel */
        FileChannel fileChannel =  fileOutputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        byteBuffer.put("xiaoxiao is a good girl".getBytes());
        /** 因为channel要从buffer中取数据, 数据需要将channel的模式进行切换 */
        byteBuffer.flip();
        /** 将byteBuffer的数据写入到channel*/
        fileChannel.write(byteBuffer);
        /** 关闭底层的数据流*/
        fileOutputStream.close();
    }
}
```

```java
public class FileChannelReadDemo {
    public static void main(String[] args) throws Exception {
        File file = new File("/Users/ssgao/Downloads/test-aa.txt");
        FileInputStream fileInputStream = new FileInputStream(file);
        /** 通过fileInputStream获取对应的FileChannel -> 实际类型 FileChannelImpl */
        FileChannel fileChannel = fileInputStream.getChannel();
        /** 创建缓冲区*/
        ByteBuffer byteBuffer = ByteBuffer.allocate((int) file.length());

        /** 将通过的数据读入buffer*/
        fileChannel.read(byteBuffer);
        /** 将字节转换成字符串*/
        System.out.println(new String(byteBuffer.array()));
    }
}

```

#### filechannel 的方法

```java

"open(Path path,OpenOption ...options)"

"read(ByteBuffer dst)"
   从FileChannel读取数据到Buffer.read()方法返回值为int类型,表示多少字节被插入buffer。如果返回-1表示到达文件结尾

"int write(ByteBuffer src)  方法"
  String newData="ssgao";
  ByteBuffer buf = ByteBuffer.allocate(48);
  buf.clear();
  buf.put(newData.getBytes());
  buf.flip();
  while(buf.hasRemaining()){
    channel.write(buf);
  }
 注意,write()不能保证写入的字节数,因此重复write()调用直到Buffer没有字节写入。


"position() 读取当前位置"
      position(long pos) 设置当前位置
     有时可能需要在FileChannle的某个特定位置进行数据的读/写操作.可以通过调用position()方法获取FileChannel的当前位置。
     也可以通过调用position(long pos)方法设置FileChannel的当前位置。
         long pos = fileChannel.position();
         channel.position(123+pos);
     如果将文位置设置在文件结束符之后,然后试图从文件通道中读取数据,读方法将返回-1,表示文件结束标志。
     如果将文件位设置在文件结束符之后,然后向通道中写数据,文件将撑大到当前位置并写入数据。
     这可能导致"文件空洞",磁盘上物理文件中写入的数据间有空隙。
"size()方法"
     fileChannel的size()方法将返回实例所关联的文件大小。
     long fileSize = channel.size();
"truncate方法"
     可以使用fileChannel的truncate()方法截取一个文件。
     截取文件时,文件中指定长度后面的部分将被删除。
     channel.truncate(1024);//截取文件前1024个字节
"force方法"
     fileChannel.force()方法将通道里尚未写入磁盘的数据强制写到磁盘上。
     出于性能方法考虑,操作系统会将数据缓存在内存中,所以无法保证写入到FileChannel的数据一定会及时写到磁盘上。要保证这一点,调用force()
     force()//该方法有一个布尔类型的参数,指明知否同时将文件元数据写到磁盘上。
"transferForm方法"
     FileChannel无法设置为非阻塞模式,它总是运行在阻塞模式下。
     FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中(将字节从给定的可读取字节通道传输到此通道的文件中)
     RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt","rw");
     FileChannel fromChannel = fromFile.getChannel;
     RandomAccessFile toFile = new RandomAccessFile("toFile.txt","rw");
     FileChannel toChannel = toFile.getChannel();
     long position=0;
     long count = fromChannel.size();//获取文件大小
     /**
      * postion 表示从position处开始向目标文件写入数据
      * count 表示最多传输的字节数(如果源通道的剩余空间小于count字节,则所传输的字节数要小于请求的字节数)
      *
      * 在SocketChannel的实现中,SocketChannel只会传输此刻准备好的数据(可能不足count字节)。
      * 因此,SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。
      */
     toChannel.transferFrom(position,count,fromChannel);
"transferTo方法"
  transferTo方法将数据从FileChannel传输到其他的Channel中
  RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt","rw");
  FileChannel fromChannel = fromFile.getChannel;
  RandomAccessFile toFile = new RandomAccessFile("toFile.txt","rw");
  FileChannel toChannel = toFile.getChannel();
  long position=0;
  long count = fromChannel.size();
   fromChannel.transferTo(position,count,toChannel);


```

### selector

```java
Java nio引入了选择器的概念,选择器用于监听多个通道的事件(比如,连接打开,数据到达)。并能够知晓通道是否为诸如读写时间做好准备。这样通过选择器一个
单独的线程就可以多个channel,从而管理多个网络连接。

selector提供选择已经"就绪任务"的能力,selector会不断轮询注册在其上的channel,
"如果某个Channel上面发生读或者写事件,这个channel就处于就绪状态",会被selector轮询出来,然后通过selectionKey可以获取就绪Channel的集合,进行后续的I/0操作。一个selector可以同时轮询多个Channel,因为JDK使用了epoll()代替传统的select实现,所以没有最大连接句柄1024/2048的限制,所以我们只需要一个线程负责Selector轮询,就可以接入成千上万的客户端

"使用selector"
  使用selector,需要向selector注册channel,然后调用它的select()方法。
  这个方法会一直阻塞到某个注册的通道有事件就绪。
  一旦这个方法返回,线程就可以处理这些事件了,事件的例子比如新连接进来,数据接收等。

selector允许单线程处理多个channel。仅用单个线程来处理多个Channels的好处是,只需要很少的线程来处理通道。
事实上,可以只用一个线程处理所有的通道,这样会大量的减少线程之间的上下文切换的开销。
如果正在处理事件时,有新的连接要接入,那么新的连接还是需要等待的,也即是某个事件处理事件非常长,新连接是要一直等待的。
所以NIO的多路复用适合大量短连接的处理情况。
```

#### selectorprovider 介绍

```java
SelectorProvider定义了创建selector, serverSocketChannel, SocketChannel等方法, 采用Java的SPI方式实现


   private static final Object lock = new Object();
   private static SelectorProvider provider = null;

   /**
    *  SelectorProvider中定义私有成员变量provider, 提供了provider方法进行创建。创建过程如下:
    *     如果provider已经创建,直接返回;
    *     如果定义了java.nio.channels.spi.SelectorProvider属性,则采用该属性定义的类创建SelectorProvider并返回
    *     通过SPI方式创建SelectorProvider并返回.
    *     以上都不满足,通过DefaultSelectorProvider创建
    */
   public static SelectorProvider provider() {
        synchronized (lock) {
            if (provider != null)
                return provider;
            return AccessController.doPrivileged(
                new PrivilegedAction<SelectorProvider>() {
                    public SelectorProvider run() {
                            if (loadProviderFromProperty())
                                return provider;
                            if (loadProviderAsService())
                                return provider;
                            provider = sun.nio.ch.DefaultSelectorProvider.create();
                            return provider;
                        }
                    });
        }
    }

    public abstract DatagramChannel openDatagramChannel()
        throws IOException;

    public abstract DatagramChannel openDatagramChannel(ProtocolFamily family)
        throws IOException;

    public abstract Pipe openPipe() throws IOException;

    public abstract AbstractSelector openSelector()
        throws IOException;

    public abstract ServerSocketChannel openServerSocketChannel()
        throws IOException;

    public abstract SocketChannel openSocketChannel()
        throws IOException;
```

```java
关于Selector类的主要涉及的两个重要的方法
    Selector.open();
    select()

 Selector selector = Selector.open()背后主要做了什么,发生了什么
 public static Selector open() throw IOException{
     return SelectorProvider.provider().openSelector();
 }

 函数功能: 打开一个选择器。

 这个新的选择器是通过调用系统默认SelectorProvider对象的openSelector方法来创建的。
 SelectorProvider.provider().openSelector()

```

#### selector 的方法

```java
"selector的创建"
  selector的创建可以通过调用此类的open方法,创建选择器,该方法使用系统默认的选择器提供者创建新的选择器。
  可可以通过选择器提供者的openSelector方法来创建选择器。
  Selector selector = Selector.open();

  调用Selector.open()时,选择器通过专门的工厂SelectorProvider来创建Selector的实现.
    SelectorProvider屏蔽了不同操作系统以及版本实现的差异性。
  SelectorProvider本身为一个抽象类,通过调用provider()提供对应的Provider实现,如PollSelectorProvider,EPollSelectorProvider


"selector注册通道"
  为了将Channel和Selector配合使用,必须将Channel注册到selector上。
  通过SelectableChannel.register()方法来实现,如下
  channel.configureBlocking(flase);//设置为非阻塞通道
  SelectionKey key = channel.register(selector,SelectionKey.OP_READ); //channel注册读取时间

"register()注册方法解释"
 与Selector一起使用时,channel必须处于非阻塞模式。(这意味着不能将FileChannel与Selector一起使用,因为FileChannel不能切换到非阻塞模式)
 套接字通道都可以与selector一起使用。
  register()方法的第二参数,这是一个"interest集合",意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型事件
        > connect  连接事件
        > accept   接受事件
        > read     读事件
        > write    写事件
  通道触发了一个事件意思就是该事件已经就绪。
        > 某个channel成功连接到另一个服务器称为"连接就绪"
        > 一个server socket channel 准备好接收新进入的连接称为"接受就绪"
        > 一个有数据可读的通道可以说是"读就绪"
        > 等待写数据的通道可以说是"写就绪"
  这四种事件用selectionKey的四个常量来表示
        > SelectionKey.OP_CONNECT
        > SelectionKey.OP_ACCEPT
        > SelectionKey.OP_READ
        > SelectionKey.OP_WRITE
 /** write事件很少用,我们写数据的时候也不需要注册WRITE事件,write事件主要用于描述底层socket缓冲区是否可用,一般情况都是可用的
* SelectKey 注册了写事件,不在合适的时间去除掉,会一直触发写时间
*/
  如果我们不止对其中一种事件感兴趣，那么用"位或"操作符将常量连接起来: int interestSet = SelectionKey.OP_READ|SelectKey.OP_WRITE
  SocketChannel可以注册监听 OP_READ|OP_WRITE|OP_CONNECT 事件
  ServerSocketChannel 只可以注册监听 OP_ACCEPT

```

#### selector-select&selectKeys

```java
"select()和selectKeys()的联合使用"
 一旦调用了select()方法,并且返回值表明了有一个或更多个通道就绪了
 然后可以通过调用selector的selectedKeys()方法,访问"已选择键集(selected Key Set)"中就绪通道。
 Set selectedKeys = selector.selectedKeys();

我们前面看到,当向Selector注册channel的时候,channel.register()方法会返回一个SelectionKey对象。
这个对象代表了注册该Selector的通道,可以通过SelectionKey的selectedKeySet()方法访问这些对象。
 Set selectedKeys = selector.selectedKeys();
 Iterator interator = selectedKeys.interator();
 while(interator.hasNext()){
    SelectionKey key = interator.next();
    if(key.isAcceptable()){
        //处理accept
    }else if(key.isReadable()){
        //处理read
    }
    //selector不会自己从已选择键集,中移除selectionKey实例。必须在处理完通道时自己移除
    //下次改通道变成就绪时,selector就会再次将其放入已选择键中。
    iterator.remove();
  }
 SelectionKey.channel()方法可以返回的通道需要转型成我们要处理的类型,如ServerSocketChannel或SocketChannel等.


```

#### selector-wake&close

```java
wakeup()
 > 某个线程调用select()方法阻塞后,即使没有通道(channel)就绪,也有办法让其从select()方法返回。
   只要让其他线程在第一个线程调用select()方法的那对象上调用selector.wakeup()方法即可,阻塞在selet()方法上的线程会立马返回。
 > 如果有其他线程调用了wakeup()方法,但当前没有线程阻塞在select()方法上,下个调用select()方法的线程会立即醒来(wake up).

close()
   用完selector后调用其close()方法关闭该Selector,且使注册到该Selector上的所有SelectionKey实例失效。
   通道本身不关闭。
```

### SelectKey 的介绍

```java
当向Selector注册Channel的时候,register()方法会返回一个SelectionKey对象。这个对象包含了一些我们感兴趣的属性
 > interest集合
 > ready集合
 > channel
 > selector
 > 附加的对象(可选) 对应的buffer(缓存数据)
 selectionKey对象是用来跟踪注册事件的句柄。
 在SelectionKey对象的有效期间,Selector会一直监控与SelectionKey对象相关的事件,如果事件发生,就会把selectionKey对象加入selected_key集合中。
 一个selector对象会包含三种类型的SelectionKey集合
     > all-keys集合
       —— 当前所有向Selector注册的SelectionKey的集合,Selector的keys()方法返回该集合
       —— "当register()方法执行时,新建一个SelectionKey,并把它加入Selector的all-keys集合中"
     > selected-keys集合
        —— 相关时间已经被Selector捕获的SelectionKey的集合,Selector的selectedKeys()方法返回该集合
        —— "执行selector的select()方法时,如果与SelectionKey相关的时间发生,这个SelectionKey就被加入到Selected-keys集合中"
        —— "程序直接调用selected-keys集合的remove()方法,或者调用它的iterator的remove()方法,都可以从selected-keys集合中删除一个SelectionKey"
     > cancelled-keys集合 —— 已经被取消的SelectionKeys的集合,selector没有提供访问这种集合的方法。
        —— "如果关闭了与SelectionKey对象关联的Channel对象/selector对象"
        —— "或调用了SelectionKey对象的cancel方法,该SelectKey就被加入cancelled-keys集合"

 "interest集合"
     interest集合是我们所选择的感兴趣的事件集合。可以通过SelectionKey读写interest集合。
     int interestSet = selectionKey.interestOps();
     boolean isinterestInAccept  = (interestSet & SelectionKey.OP_ACCEPT)==SelectionKey.OP_ACCEPT
     boolean isinterestInConnect = interestSet & SelectionKey.OP_CONNECT;
     boolean isinterestInRead = interestSet & SelectionKey.OP_READ;
     boolean isinterestInWrite = interestSet & SelectionKey.OP_WRITE;
    用"位与"操作interest集合给定的SelectionKey常量,可以确定某个确定的时间是否在interest集合中

  "ready集合"
     ready集合是通道已经准备就绪的操作集合。
     在一次选择Selection之后，我们首先会访问这个readySet。
     int readSet = selectionKey.readyOps(); //我们可以像检测interest集合那样,来检测channel中什么事件或操作已经就绪,也可以如下
     selectionKey.isAcceptable();
     selectionKey.isConnectable();
     selectionKey.isReadable();
     selectionKey.isWritable();

  "从selectKey中获取channel和selector"
    从SelectionKey访问Channel和Selector很简单,如下
    Channel channel = selectionKey.channel();
    //这里的channel可以强制转型为socketChannle或ServerSocketChannel
    Selector selector = selectionKey.selector();

   "附加对象"
    我们可以将一个对象或者更多的信息附加到SelectionKey上,这样就能更方便的识别某个给定的通道。
    例如,可以附加一个与通道一起使用的Buffer,或是包含聚集数据的某个对象。
    selectionKey.attach(object);
    Object attachObject = selectionKey.attachment();
    还可以在用register()方法向Selector注册Channel的时候附加对象。例如
    SelectionKey key = channel.register(selector,SelectionKey.OP_READ,object);
```

### nio 实例

```java
 我们如果需要一个即时消息服务器,可能有成千上万个客户端,同时连接到服务器,但是在任何时刻只有非常少量的消息需要读取和分发(如果采用传统socket的线程池或一线程一客户端方式,则会非常浪费资源),这就需要一种方法能阻塞等待,直到有一个信道可以进行IO操作。

"Nio的selector的功能"
 NIO的selector选择器就实现了这样的功能,一个Selector实例可以同时检查一组信道的I/O状态,它就类似一个观察者,只要我们把需要探知的SocketChannel告诉Selector,我们可以接着做别的事情,当有事件(比如,连接打开,数据到达等)发生时,它会通知我们,传回一组SelectionKey,我们读取这些Key,就会获得我们刚刚注册过的SocketChannel,然后我们从这个Channel中读取数据,接着我们可以处理这些数据。

"selector的内部原理"
 Selector内部实现原理实际是在做一个对所注册的Channel的轮询访问,不断的轮询(目前就这一个算法)一旦轮询到了一个Channel有所注册的事情发生,比如数据来了,它就会读取Channel中的数据,并对其进行处理。如果要使用选择器,需要创建一个Selector实例,并将其注册到想要的监控信道上 (通过channel的register()方法实现).然后调用Selector的select()方法,该方法会阻塞等待,直到有一个或多个信道准备好了I/O操作或等待超时,或另一个线程调用了该选择器的wakeup()方法。这样在一个单独的线程中,通过调用select()方法,就能检查多个信道是否准备好进行I/O操作,由于非阻塞I/O的异步特性,在检查的同时,我们也可以执行其它任
务。
```

```java
"服务器端"
    > 创建一个Selector实例
    > 将其注册到各种信道,并指定每个信道上感兴趣的IO操作
    > 重复执行
        1) 调用一种select() 方法
        2）获取选取的键列表
        3）对于已选键集中的每个键
            > 获取信道,并从中获取附件(如果为信道及其相关的key添加了附件的话)
            > 确定准备就绪的操作并执行,如果是accept操作,将接收的信道设置为非阻塞模式,并注册到选择器
            > 如果需要,修改键的兴趣操作集
            > 从已选中键集中移除键
"客户端"
   与基于多线程的TCP客户端大致相同，只是这里是通过信道建立的连接，但在等待连接建立及读写时，我们可以异步地执行其他任务。
```

#### server 端

```java
public class TcpSocketServer {
    //缓冲区长度
    private static final int BUFSIZE=256;
    //selector的select的等待超时时间
    private static final int TIMEOUT=3000;
    public static void main(String[] args) {
        //创建一个选择器
        try {
            Selector selector = Selector.open();
            //实例化一个信道
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            //将该信道绑定到指定端口
            serverSocketChannel.bind(new InetSocketAddress(6677));
            //配置信道为非阻塞模式
            serverSocketChannel.configureBlocking(false);
            //将信道注册到selector上
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            //创建一个实现了协议接口的对象
            TCPProtocol protocol = new EchoTcpProtocol(BUFSIZE);
            //不断轮询select方法,获取准备好的信道所关联的key集合
            while(true){
                //一直等待,直到有信道准备好了IO操作,如果超时没有返回0
                if(selector.select(TIMEOUT)==0){
                    continue;
                };
                //如果select返回值>0,说明有信道准备好了,获取信道所关联的KEY集合的Iterator实例
                Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                //循环取得集合中的每个键值
                while (keyIterator.hasNext()){
                    SelectionKey key = keyIterator.next();
                    //手动从键集中删除当前的key
                    keyIterator.remove();
                    //如果服务器端感兴趣的I/O操作是accept
                    if(key.isAcceptable()){
                        protocol.handlerAccept(key);
                    }
                    if(key.isReadable()){
                        System.out.println("---server-read-action---");
                        protocol.handlerRead(key);
                    }
                    if(key.isWritable()){
                        System.out.println("---server-write-action---");
                        protocol.handlerWrite(key);
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
/**该接口定义了对selector的兴趣事件的处理*/
public interface TCPProtocol {
    //accept I/O形式
    void handlerAccept(SelectionKey key);
    //read I/O形式
    void handlerRead(SelectionKey key);
    //write I/O形式
    void handlerWrite(SelectionKey key);
}

public class EchoTcpProtocol implements TCPProtocol {
    //缓冲区长度
    private int bufSize;
    EchoTcpProtocol(int bufSize){
        this.bufSize=bufSize;
    }
    //服务端信息已经准备好了接收新客户端连接
    public void handlerAccept(SelectionKey key) {
        try {
            SocketChannel socketChannel = ((ServerSocketChannel)key.channel()).accept();
            socketChannel.configureBlocking(false);
            //将选择器注册到连接到客户端信道,并指定该信道key值的属性为OP_READ,同时为该信道指定关联的附件
            socketChannel.register(key.selector(),SelectionKey.OP_READ, ByteBuffer.allocate(bufSize));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    //客户端信道已经准备好了从信道中读取数据到缓冲区
    public void handlerRead(SelectionKey key) {
        SocketChannel socketChannel = (SocketChannel) key.channel();
        //获取该信道锁关联的附件,这个为缓冲区
        ByteBuffer byteBuffer = (ByteBuffer) key.attachment();
        try {
            int bytesRead = socketChannel.read(byteBuffer);
            System.out.println("Receive from Client:"+bytesRead);

            //如果read()方法返回-1,说明客户端关闭了连接,那么客户端已经接受到了自己发送字节数相等数据,可以安全的关闭
            if(bytesRead==-1){
                socketChannel.close();
            }else if(bytesRead>0){
                /**
                 * 经测试发现byteBuffer.get(bytes,0,bytesRead) 并没有将0~bytesRead的数据写入到bytes数组中
                 * 而是将bytes的数据写入到了byteBuffer中,和put(bytes,0,bytes)的效果类似
                 */
                // byte[] bytes = new byte[bytesRead];
                // byteBuffer.get(bytes,0,bytesRead);
                //System.out.println(new String(bytes));
                System.out.println("Receive form Client:"+new String(byteBuffer.array(),0,bytesRead));
                //如果缓存区中读入了数据,则将该信道感兴趣的操作设置为可读可写
                key.interestOps(SelectionKey.OP_READ|SelectionKey.OP_WRITE);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    //客户端信道已经准备好了将数据从缓冲区写入信道
    public void handlerWrite(SelectionKey key) {
        //获取与该信道关联的缓冲区,里面有之前读取到的数据
        ByteBuffer byteBuffer = (ByteBuffer) key.attachment();
        //重置缓冲区,准备将数据写入信道
        byteBuffer.clear();
        byteBuffer.put("ssgao ai chenlin!".getBytes());
        SocketChannel socketChannel = (SocketChannel) key.channel();
        byteBuffer.flip();
        //将数据写入信道中
        try {
            //如果缓冲区中的数据已经全部写入了信道，则将该信道感兴趣的操作设置为可读
            //byte[] argument = "ssgao ai xiaoxiao!".getBytes();
            socketChannel.write(byteBuffer);
            key.interestOps(SelectionKey.OP_READ|SelectionKey.OP_WRITE);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

#### client 端

```java
public class TcpASocketClient {
    public static void main(String[] args) {
        //创建一个信道,并设置为非阻塞模式
        try {
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);
            //向服务端发起连接
            if(!socketChannel.connect(new InetSocketAddress("127.0.0.1",6677))){
                //不断轮询连接状态,知道完成连接
                while(!socketChannel.finishConnect()){
                    continue;
                }
            }
            byte[] argument = "ssgao ai xiaoxiao!".getBytes();
            //实例化读写的缓存区
            ByteBuffer writeBuf = ByteBuffer.wrap(argument);
            ByteBuffer readBuf = ByteBuffer.allocate(argument.length);
            //接收到的总字节数
            int totalBytesRcvd=0;
            //每次调用read()方法接收到的字节数
            int bytesRcvd;
            //如果用来向通道中写数据的缓存区还有剩余的字节,则继续将数据写入信道
            while (writeBuf.hasRemaining()){
                socketChannel.write(writeBuf);
            }
            //循环执行,直到接收到的字节数,和发送的字符串的字节数相等
            while(true){
                //如果read()接收到-1,表示服务端关闭抛出异常
                if((bytesRcvd=socketChannel.read(readBuf))==-1){
                    break ;
                }else if(bytesRcvd>0){
                    System.out.println("获取服务器端发送信息:-->");
                    System.out.println(new String(readBuf.array(),0,bytesRcvd));
                    readBuf.clear();
                }else{
                   // System.out.println(bytesRcvd);
                }
            }
            //打印出接收到数据
            System.out.println("Received:"+new String(readBuf.array()));
            //关闭信道
            socketChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

```java
"程序说明"
   上面服务端程序,select()方法第一次能选择出来的准备好的都是服务端信道,其关联键值的属性都为OP_ACCEPT,亦及有效操作都为accept,在执行handleAccept方法时,为取得连接的客户端信道也进行了注册,属性为OP_READ这样下次轮询调用select()方法时,便会检查到对read操作感兴趣的客户端信道(当然也有可能有关联accept操作兴趣集的信道)从而调用handleRead方法,在该方法中又注册了OP_WRITE属性,那么第三地调用select()方法时,便会检测对write操作感兴趣的客户端信道(当然也有可能有关联read操作兴集的信道),从而调用handlerWrite方法。

"NIOsocket通信需要注意事项"
  对于非阻塞SocketChannel来说,一旦


"几个需要注意的地方"
  对于非阻塞SocketChannel来说,一旦已经调用connect()方法发起连接,底层套接字可能既不是已经连接,也不是没有连接,而是正在连接。由于底层协议的工作机制,套接字可能会在这个状态一直保持下去,这时候就需要循环地调用finishConnect()方法来检查是否完成连接,在等待连接的同时线程也可以做其他事情,这便实现了线程的异步操作。

  write()方法的非阻塞调用只会写出其能够发送的数据,而不会阻塞等到所有数据,而后一起发送,因此在调用write()方法将数据写入信道时,一般要用到while循环,如
  while(buf.hasRemaining())
      channel.write(buf);

 任何对key(信道)所关联的兴趣操作集合的改变,都只在下次调用select()方法才会生效。

 selectKey()方法返回的键集是可修改的,实际上在两次调用select()方法之间,都必须手动将其清空,否则,它就会在下次调用select()方法时仍然保留在集合中,而且可能会有无用的操作来调用它换句话说,select()方法只会在已有的所选键集上添加键,它们不会创建新的键集。

对于ServerSocketChannel来说,accept是唯一的有效操作,而对于socketChannel来说,有效操作包括读,写和连接,另外,对于DatagramChannel,只有读写操作是有效的。
```

#### serversocketchannel

```java
java nio中ServerSocketChannel用于监听TCP链接请求的通道,正如Java网络编程中ServerSocket一样。
serverSocketChannel实例如下:
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
}

"打开一个ServerSocketChannel我们需要调用它的open()方法"
    ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
"关闭一个ServerSocketChannel我们需要调用它的close()方法"
    serverSocketChannel.close();


"阻塞模式监听新进来的连接"
 通过ServerSocketChannel.accept()方法监听新进来的连接,当accept()方法返回的时候,它返回一个包含新进来的连接SocketChannel。accept()方法会一直阻塞到有新连接到达。通常不会仅仅只监听一个连接,在while循环中调用accept()方法
 while(true){
     SocketChannel socketChannel = serverSocketChannel.accept();
 }

"非阻塞模式监听新进来的连接"
 ServerSocketChannel可以设置成非阻塞模式
 在非阻塞模式下,accept()方法会立刻返回,如果还没有新进来的连接,返回的将是null.
 因此需要检查返回的socketChannel是否是null
 ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()
 serverSocketChannel.socket().bind(new InetSocketAddress(9999));
 serverSocketChannel.configureBlocking(false);
 while(true){
     SocketChannel socketChannel = serverSocketChannel.accept();
     if(socketChannel!=null){
         //do something with socketChannel
     }
 }
```

#### socketchannel

```java
java nio中的socketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel。
 > 打开一个SocketChannel并连接到互联网上的某台服务器
 > 一个新连接到达ServerSocketChannel时,会创建一个SocketChannel

"打开SocketChannel"
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://aouo.com",80));

"从socketChannel读取数据到buffer"
 要从SocketChannel中读取数据,调用一个read()的方法之一。
  ByteBuffer buf = ByteBuffer.allocate(48);
  int byteRead = socketChannel.read(buf);
 首先,分配一个Buffer,从SocketChannel读取到的数据将会放到这个Buffer中。
 然后,调用SocketChannel.read()
     //该方法将数据从SocketChannel读到Buffer中。read()方法返回的int值表示读了多少字节到buffer中。如果返回-1,表示已经读到流的末尾了。

"写入SocketChannel"
 写数据到SocketChannel,使用write()方法,该方法以一个Buffer作为参数。
 String newData = "New String to write to file ...."+System.currentTimeMillis();
 ByteBuffer buffer = ByteBuffer.allocate(48);
 buffer.clear();
 buffer.put(newData.getBytes());
 buf.flip();
 while(buffer.hasRemaining()){
   channel.write(buffer);
 }
注意,SocketChannel.write()方法调用的是一个while循环中,writer方法无法保证能写多少字节到SocketChannel.所以我们重读调用write()直到Buffer没有要写的字节为止。


```

```java
"非阻塞连接connection()/finishConnect"
   如果SocketChannel在非阻塞模式,此时调用connect(),该方法可能在建立连接之前就返回了。为了确定连接是否建立,可以调用finishConnection()的方法。
   socketChannel.configureBlocking(false);
   socketChannel.connect(new InetSocketAddress("http://127.0.0.1",80));
   while(!socketChannel.finishConnect()){
     //wait or dosometing else ...
   }

"write()"
    非阻塞模式下,write()方法在尚未写出任何内容时可能就返回了,所以需要在循环中调用write().
    因为,write()方法无法保证能写多少字节到SocketChannel。所以，我们重复调用write()直到Buffer没有要写的字节为止。
    String newData = "新String写入file"+System.currentTimeMillis();
    ByteBuffer buf = ByteBuffer.allocate(48);
    buf.clear();
    buf.put(newData.getBytes());
    buf.flip();
    while(buf.hasRemaining()){
       channel.write(buf);  //将buf中的信息读出到socket,对应buf的读操作
    }
"read()"
    非阻塞模式下,read()方法尚未读取到任何数据时可能就返回了,所以需要关注它的int返回值,它会告诉我们读取了多少字节。
    channel.read(ByteBuffer xx);//表示将socket的信息写入到buf,对应buf的写操作
```

#### zerocopy

```java
public class ZeroClient {
    public static void main(String[] args) throws Exception {
        InetSocketAddress inetSocketAddress = new InetSocketAddress(9003);
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.bind(inetSocketAddress);
        /** 得到一个文件channel*/
        FileChannel fileChannel = new FileInputStream("/User/ssgao/Downloads/test-aa.txt").getChannel();
        Long startTime = System.currentTimeMillis();
        /**
         * 在Linux下一个transferTo 方法就可以完成传输
         * 在windows下一次调用transferTo只能发送8m, 就需要分段传输文件, 注意传输位置
         * transferTo 底层就是使用的零copy
         */
        Long transferCount = fileChannel.transferTo(0,fileChannel.size(),socketChannel);
        System.out.println("发送的总的字节数="+transferCount+";"+" 耗时:"+(System.currentTimeMillis()-startTime));
        fileChannel.close();
    }
}
```

```java
public class ZeroServer {
    public static void main(String[] args) throws Exception {
        InetSocketAddress socketAddress = new InetSocketAddress(7045);
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.bind(socketAddress);
        /** 创建buffer */
        ByteBuffer byteBuffer =  ByteBuffer.allocate(4096);
        while (true){
           SocketChannel socketChannel =  serverSocketChannel.accept();
           int readCount =0;
           while (-1!=readCount){
               try{
                  readCount = socketChannel.read(byteBuffer);
               }catch (Exception e){
                   e.printStackTrace();
               }
           }
           /** 将buffer 倒带, 将buffer的position=0 mark作废*/
           byteBuffer.rewind();
        }
    }
}

```

### 其他 IO 使用的类

#### InetAddress

```java
java.net.InetAddress类是Java对IP地址(包括IPV4和IPV6)的高层表示。大多数其他网络类都要用到这个类，包括Socket，ServerSocket，URL，DatagramSocket，DatagramPacket等。一般来讲，它包括一个主机名和一个IP地址。

InetAddress类没有公共构造函数，实际上，InetAddress有一些静态工厂方法，可以连接到DNS服务器，来解析主机名。最常用的是InetAddress.getByName();
    InetAddress inetAddress2 = InetAddress.getByName("192.168.56.101");

这个方法不只是设置InetAddress类中的一个私有String字段。实际上它会与本地DNS服务器建立一个连接来查找名字和数字地址(如果之前查找过这个主机，这个信息可能会保存在本地缓存，如果是这样，就不需要再建立一个网络连接) 如果DNS服务器找不到这个地址，这个方法就会抛出一个UnkownHostException异常，这是一个IOException的子类。
@Test
    public  void testBaiDu() {
        try {
            //获取其他机器的InetAddress的实例
            InetAddress inetAddress2 = InetAddress.getByName("www.ssgao1.com");
            System.out.println(inetAddress2.getHostName());
        }catch (Exception e){
            e.printStackTrace();
        }
    }
----------
在chrome浏览器中输入www.ssgao1.com 找不到ssgao1.com的服务器DNS 地址，但是输入ssgao的时候提示有(呵呵)程序中输出信息
-----------------
java.net.UnknownHostException: www.ssgao1.com: nodename nor servname provided, or not known
	at java.net.Inet6AddressImpl.lookupAllHostAddr(Native Method)
	at java.net.InetAddress$2.lookupAllHostAddr(InetAddress.java:928)
	at java.net.InetAddress.getAddressesFromNameService(InetAddress.java:1323)

调用getByName()并提供一个IP地址串作为参数，会为所请求的IP地址创建一个InetAddress对象，而不检查DNS。这说明，可能会为实际上不存也无法连接的主机创建InetAddress对象。由包含IP地址的字符串来创建InetAddress对象时，这个对象的主机名初始设置为这个字符串。只有当请求主机名(显示地通过getHostName()请求)，才会真正完成主机名的DNS查找。

```

```java
InetAddress inetAddress = InetAddress.getLocalHost();
此方法返回一个InetAddress对象，用来返回本地主机
InetAddress inetAddress = InetAddress.getByName("Lc")；
此方法返回一个InetAddress对象，用来在给定主机名的情况下确定主机的Ip地址
InetAddress inetAddress = InerAddress.getByAddress(byte[] addr)
此方法返回一个InetAddress对象，用来在给定原始IP地址的情况下，返回一个InetAddress对象
InetAddress[] inetAddress = InetAddress.getAllByName("Lc");
在给定主机名的情况下，根据系统上配置的名称服务返回其IP地址所组成的数组

------一些主要的方法----
byte[] getAddress()
返回此InetAddress对象的原始IP地址，
String getCanonicalName()
获取此IP地址的完全限定域名
String getHostAddress()
获取IP地址字符串
String getHostName()
获取此IP地址的主机名
boolean isReachable(int timeout)
测试是够可以达到该地址


@Test
    public  void testBaiDu() {
        try {
            //获取其他机器的InetAddress的实例
            InetAddress[] inetAddresses = InetAddress.getAllByName("www.baidu.com");
            for(InetAddress inetAddress:inetAddresses){
                System.out.println(inetAddress.getHostName());
                System.out.println(inetAddress.getHostAddress());
                System.out.println(inetAddress.isReachable(10));
                System.out.println(inetAddress.getCanonicalHostName());
            }
            //获取其他机器的InetAddress的实例
            InetAddress inetAddress = InetAddress.getByName("192.168.56.101");
            System.out.println(inetAddress.getHostName());
            System.out.println(inetAddress.isReachable(10));
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```
