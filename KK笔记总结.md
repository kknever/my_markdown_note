# 网络相关

### 1、TCP三次握手

第一次：客户端发送**SYN**包到服务器，客户端进入**SYN_SENT**状态，并等待服务器确认。
第二次：服务器收到客户端SYN包后，会响应**ACK**，同时服务器自己也需要发送一个SYN包给客户端，即最终回复给客户端**SYN+ACK**包，服务器进入**SYN_RECV**状态。
第三次：客户端收到服务器的SYN+ACK包，并向服务器回复**ACK**包，此包发送完毕后，客服端和服务器均进入**ESTABLISHED**状态，三次握手完成。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/ip_connect.gif?raw=true" alt="ip_connect.gif" style="zoom:40%;" />

> **扩展1：为什么是三次握手，而不是一次，两次或者是四次五次？
>
> 例如A和B通信：
> 如果是一次（A发送SYN到B），A根本就不知道B是否收到SYN包，肯定不行；
> 如果是两次（B收到SYN，并且回馈了SYN+ACK）：此时B同样不知道A是否收到包，不行；
> 如果是四次五次甚至多次：因为三次握手后AB双方已经确认互通了，此时还若还要再发送确认包就会消耗更多资源，没有意义。
>
> **扩展2：TCP建立连接三次握手时可能出现何种安全隐患？**
> 例如：当服务器收到客户端的SYN包后，立即回复了SYN+ACK，但是此时可能客户端突然断开了连接，那么服务器在一定时间内没有收到客户端的ACK就会不断尝试发送重发SYN+ACK直至超时（例如Linux默认63秒才会断开此无效连接）。因此：攻击者即可以模拟此种情况，*不停的向服务器发起连接并在发送SYN后立即下线，由此就会造成服务器连接资源耗尽（SYN Flood攻击）*。

### 2、TCP四次挥手

第一次：客户端发送**FIN**包到服务器，关闭客户端到服务器的数据传输，客户端进入**FIN_WAIT_1**状态。
第二次：服务器收到FIN后，回馈给客户端**ACK**包，服务器进入**CLOSE_WAIT**（被动关闭）状态。
第三次：服务器向客户端发送**FIN**包，关闭服务器到客户端的数据传输，服务器进入**LAST_ACK**状态。
第四次：客户端收到FIN后，回馈给服务器**ACK**包，服务器进入**CLOSED**状态，客户端进入**TIME_WAIT**状态，等待一段时间后，客户端进入**CLOSED**状态。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/ip_disconnect.gif?raw=true" alt="ip_disconnect.gif" style="zoom:77%;" />

> **扩展1：为什么建立连接是三次握手，而关闭连接确实四次挥手呢？**
> 为什么连接需要三次握手可参考上述TCP三次握手中扩展2。
> 为什么断开连接需要四次挥手：因为TCP连接是全双工的（即数据可在两个方向上同时传递），所以进行关闭时每个方向上都要单独进行关闭（这个单方向的关闭就叫半关闭），那么关闭时，通信双方都需要发送FIN报文来告诉对方自己即将关闭此通道上的数据传输，由此：双方发送FIN，并各自收到对方响应，即总共四次。
>
> **扩展2：服务器TCP出现大量CLOSE_WAIT问题**
> 原因：客户端关闭了SOCKET链接，但是服务器忙于读或者写，从而没有及时关闭对应连接。
> 解决办法：检查服务器代码，及时主动关闭旧的无效的SOCKET连接或其他资源的连接。
>
> ```shell
> shell> netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a,S[a]}'
> ```

### 3、TCP流量控制

所谓流量控制就是让发送方的发送速率不要太快，要让接收方来得及接收。

### 4、HTTP相关

#### 4.1、Cookie和Session

在WEB开发中，服务器可以为每个用户浏览器创建一个会话对象（session对象），在需要保存用户数据时，服务器程序可以把用户数据写到用户浏览器独占的session中，当用户使用浏览器访问其它程序时，其它程序可以从用户的session中取出该用户的数据，为用户服务。
cookie是一种在客户端保存状态的方案，session是一种在服务器端保存状态的方案。

#### 4.2、HTTP缓存机制

HTTP的缓存属于客户端缓存，所以我们认为浏览器存在一个缓存数据库，用于储存一些不经常变化的静态文件（图片、css、js等）。我们将缓存分为`强制缓存`和`协商缓存`。

**强制缓存**：当缓存数据库中已有所请求的数据时。客户端直接从缓存数据库中获取数据。当缓存数据库中没有所请求的数据时，客户端的才会从服务端获取数据。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/http_force_cache.png?raw=true" alt="http_force_cache.png" style="zoom:60%;" />

**协商缓存**：又称对比缓存，客户端会先从缓存数据库中获取到一个缓存数据的标识，得到标识后请求服务端验证是否失效，如果没有失效服务端会返回304，此时客户端直接从缓存中获取所请求的数据，如果标识失效，服务端会返回更新后的数据。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/http_exchange_cache.png?raw=true" alt="http_exchange_cache.png" style="zoom:70%;" />

两类缓存机制可以同时存在，强制缓存的优先级高于协商缓存，当执行强制缓存时，如若缓存命中，则直接使用缓存数据库数据，不在进行缓存协商。

**扩展：服务器是如何判断缓存是否失效的呢？**

**强制缓存：服务器响应的header中会用两个字段来表明Expires和Cache-Control。**
***Expires***：Expires的值为服务端返回的到期时间，即**下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据**。不过Expires 是HTTP 1.0的东西，现在默认浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。另一个问题是，到期时间是由服务端生成的，但是客户端时间可能跟服务端时间有误差，这就会导致缓存命中的误差，所以HTTP 1.1 的版本，使用Cache-Control替代。
***Cache-Control***：常见的取值有private、public、no-cache、max-age，no-store，默认为private。说明如下：
*private*：客户端可以缓存。
*public*：客户端和代理服务器都可缓存（前端可以认为public和private是一样的）。
*max-age=xxx*：缓存的内容将在 xxx 秒后失效。
*no-cache*：需要使用对比缓存来验证缓存数据。
no-store：所有内容都不会缓存，强制缓存和对比缓存都不会触发。

**协商（对比）缓存：使用Last-Modified / If-Modified-Since、Etag / If-None-Match来标识缓存状态。**
***Last-Modified*：**服务器在响应请求时，告诉浏览器资源的最后修改时间。
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/http_last_modify.png?raw=true" alt="http_last_modify.png" style="zoom:55%;" />

***If-Modified-Since*：**再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改时间。服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若资源的最后修改时间大于If-Modified-Since，说明资源又被改动过，则响应整片资源内容，返回状态码200；若资源的最后修改时间小于或等于If-Modified-Since，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/http_if_modify_since.png?raw=true" alt="http_if_modify_since.png" style="zoom:55%;" />

**Etag：**服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。*Etag的计算会占用服务端计算的资源，所有服务端的资源都是宝贵的，所以就很少使用Etag了。*
**If-None-Match：**再次请求服务器时，通过此字段通知服务器客户段缓存数据的唯一标识。服务器收到请求后发现有头If-None-Match 则与被请求资源的唯一标识进行比对。若不同，说明资源又被改动过，则响应整片资源内容，返回状态码200；若相同，说明资源无新修改，则响应HTTP 304，告知浏览器继续使用所保存的cache。
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/http_if_none_match.png?raw=true" alt="http_if_none_match.png" style="zoom:55%;" />

### 5、HTTPS相关

*SSL（Secure Sockets Layer）安全套接层和TLS（Transport Layer Security）传输层安全协议其实是一套东西。*

#### 5.1、SSL连接建立过程

- 客户端给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法，发送至服务器。

- 服务器确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random），反馈给客户端。

- 客户端确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使用数字证书中的公钥，加密这个随机数，发给服务器。

- 服务器使用自己的私钥，获取客户端发来的随机数（即Premaster secret）。

- 客户端和服务器根据约定的加密方法，使用前面的三个随机数（Client random、Server Random和Premaster secret），生成"对话密钥"（session key），用来加密接下来的整个对话过程。

  <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/ssl_example.png?raw=true" alt="ssl_example.png" style="zoom:67%;" />

#### 5.2、理解CA

理解CA需首先了解中间人攻击，如下图所示：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/ca_middle_attack.png?raw=true" alt="ca_middle_attack.png" style="zoom:67%;" />
当服务端向客户端返回公钥A1的时候，中间人将其替换成自己的公钥B1传送给浏览器。而浏览器此时一无所知，傻乎乎地使用公钥B1加密了密钥K发送出去，又被中间人截获，中间人利用自己的私钥B2解密，得到密钥K，再使用服务端的公钥A1加密传送给服务端，完成了通信链路，而服务端和客户端毫无感知。

出现这一问题的核心原因是**客户端无法确认收到的公钥是不是真的是服务端发来的**。为了解决这个问题，互联网引入了一个公信机构，这就是CA（Certificate Authority）。

### 6、IP和子网掩码

**子网掩码**：就是将某个IP地址划分成网络地址和主机地址两部分，不能单独存在，它必须结合IP地址一起使用。
子网掩码的设定必须遵循一定的规则。与IP地址相同，子网掩码的长度也是32位，左边是网络位，用二进制数字“1”表示；右边是主机位，用二进制数字“0”表示。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/ip_format_example.jpg?raw=true" alt="ip_format_example.jpg" style="zoom:96%;" />
**ip段/数字-如192.168.0.1/24是什么意思?**

后面这个数字标示了我们的网络号的位数，也就是子网掩码中前多少号为1
129.168.1.1/24 这个24就是告诉我们网络号是24位，即子网掩码是：11111111 11111111 11111111 00000000
即：255.255.255.0

**广播地址：**网络中主机位全为1即为广播地址，如网段为192.168(子网掩码255.255.0.0)，则广播地址为192.168.255.255，网段为192.168.0(子网掩码255.255.255.0)，则广播地址为192.168.0.255（也即：一个网段中最后一个地址是广波地址）。
*广播地址带表这个网络所有的主机，ping广播地址就是把ping包发给网络中所有主机，只要主机在线，那么就会回包，用来可以用来查看有哪些主机在线。*

# JAVA相关

## 1、Java基础

### 1.1、Java面向对象的特征

**抽象**：抽象是将一类对象的共同特征总结出来构造类的过程，包括数据抽象和行为抽象两方面。抽象只关注对象有哪些属性和行为，并不关注这些行为的细节是什么。

**封装**：隐藏对象的实现细节，仅对外公开接口，是针对一个对象来说的。

**多态**：多态性是指允许不同子类型的对象对同一消息作出不同的响应。简单的说就是用同样的对象引用调用同样的方法但是做了不同的事情。

**继承**：继承是从已有类得到继承信息创建新类的过程。提供继承信息的类被称为父类（超类、基类）；得到继承信息的类被称为子类（派生类）。

### 1.2、抽象类abstract和接口interface有什么区别及注意事项

- 抽象类和接口都不能被实例化。
- 每个类只能继承一个抽象类，但是可以实现多个接口。
- 抽象类表示的是，这个对象是什么。接口表示的是，这个对象能做什么。
- 抽象类是把相同的东西提取出来，即重用；接口是把功能模块进行归纳整理，降低耦合；

*接口可以继承(extends)接口，而且支持多重继承；抽象类可以实现(implements)接口；抽象类可继承具体类,也可以继承抽象类。*

abstract抽象方法应注意以下细节：

- 抽象方法不能是静态的：静态方法不能被子类重写，抽象方法必须被子类重写，冲突。
- 抽象方法不能是native的：本地方法是由本地代码（如C代码）实现的方法，而抽象方法是没有实现的，也是矛盾的。
- 抽象方法不能用synchronized关键字，synchronized和方法的实现细节有关，抽象方法不涉及实现细节，因此也是相互矛盾的。

### 1.3、反射机制

反射机制：Java的反射机制是在程序运行时，能够完全知道任何一个类，及其它的属性和方法，并且能够任意调用一个对象的属性和方法。这种运行时的动态获取就是Java的反射机制。其实这也是Java是动态语言的一个象征。

```java
Class clazz = Class.forName("com.test.Main");
Main m = (Main) clazz.newInstance();
//getDeclaredMethod()获取的是类自身声明的所有方法，包含public、protected和private方法。不包括基类。
//getMethod()获取的是类的所有共有方法，这就包括自身的所有public方法，和从基类继承的、从接口实现的所有public方法。
Method mFunctionA = clazz.getDeclaredMethod("functionA", String.class);
mFunctionA.setAccessible(true);
//invoke()第一个参数为类的实例，第二个参数为相应函数中的参数
Object ret = mFunctionA.invoke(m, "aaa");
```

### 1.4、Java四种引用方式

**强引用(StrongReference)**：new一个Object存放在堆内存，然后用一个引用指向它，这就是强引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。
**软引用(SoftReference)**：如果一个对象只具有软引用，则内存空间足够时，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。
**弱引用(WeakReference)**：弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。每次执行GC的时候，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
**虚引用(PhantomReference)**：如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。

### 1.5、泛型

泛型：即参数化类型。是将类型由原来的具体的类型参数化，然后在使用/调用时传入具体的类型。

> 例如：GenericClass<T>{}，一般使用以下字符进行泛型类型变量的描述：
> E：元素（Element），K：关键字（Key），N：数字（Number），T：类型（Type），V：值（Value）。

#### 1.5.1、为什么要使用泛型？

使用泛型的意义：

> 适用于多种数据类型执行相同的代码（代码复用）。
> 泛型中的类型在使用时指定，不需要强制类型转换（类型安全，编译器会检查类型）。

泛型包括：泛型类，泛型方法和泛型接口。

```java
//泛型类
public class GenericClass<T>{}
//泛型接口
public interface GenericInterface<T>{}
//泛型方法
public T genericAdd(T a, T b) {} 或 public <T extends Number> genericAdd(T a, T b) {}
```

#### 1.5.2、通配符及上下边界

> **通配符的作用是为了代替泛型类的类型实参。**
> 无边界通配符：<?>
> 使用无边界通配符可以让泛型接收任意类型的数据。
> 
> 上边界通配符 ：List<? extends T>
> 这个list放的是T或者T的子类型的对象，但是不能确定具体是什么类型，所以可以get，不能add（但可以add null值）。
> 
> 下边界通配符 ：List<? super T>
> list放的是至少是T类型的对象，所以可以add T或者T的子类型，但是get得到的类型不确定。

### 1.6、集合相关

#### 1.6.1、List和Set

List和Set都是collection的子类。

> List：有序（是指元素输出的顺序就是插入的顺序），元素可重复，允许null值。
> 常见List如：ArrayList、LinkedList和Vector。
>
> Set：无序（元素插入顺序和输出顺序可能不一致，默认通过 Comparator  或者 Comparable 维护了一个排序顺序），元素唯一。
> 常见Set如：HashSet、LinkedHashSet和TreeSet。

List扩容

> ArrayList的默认大小是10，扩容大小是新容量(newCapacity)=(oldCapacity + (oldCapacity >> 1))约是oldCapacity 的1.5倍。

#### 1.6.2、HashMap相关

> Java8以前HashMap基于数组+链表实现。
> 数组特点：查询速度快，增删速度慢。
> 链表特点：查询速度慢，增删速度快。
> *链表和数组速度快慢原因如下：*
> 在查询数据时，数组可以通过索引迅速确定位置，而链表只能依次寻找。
> 在增删数据时，数组每次插入元素都要将该元素后面的所有元素向后挪动一位，删除则将该元素之后所有元素向前移动一位。而链表只需要修改前后指针的指向即可。
>
> Java8即以后HashMap基于数组+链表+红黑树实现（*Java8及以后版本中，链表⻓度⼤于8的时候，链表会转成红⿊树*）。

**HashMap的存取规则**

> 简单说：HashMap在存储数据时，先根据key的hash找到数组下标，然后把数据放在对应下标的链表中。
>
> HashMap中数组存储位置计算规则，通过hash(key)&(len-1)获得（其中hash(key)表示key的hashCode，len表示当前数组的长度。
>
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_hashmap.png?raw=true" alt="java_hashmap.png" style="zoom:67%;" />
>
> 如上图所示的12&(16-1)=12,28&(16-1)=12,108&(16-1)=12,140&(16-1)=12。所以12、28、108以及140都存储在数组下标为12的位置。

**为什么说HashMap不保证有序，储存位置会随时间变化？**

> 通过hash值确定位置，与用户插入顺序不同；在达到阈值后，HashMap会调用resize方法扩容，扩容后位置发生变化。

**简述String中hashCode的实现？**

> String类中的hashCode计算方法是以31为权，每一位为字符的ASCII值进行运算，用自然溢出来等效取模。
>
> 扩展1：为什么选择31这个数字呢？
> 首先，31是一个素数，且31可以被JVM自动优化为移位操作，具有更高的性能。如31*3=(3<<5)-3。
> 扩展2：更大值的101也是一个素数，为什么不用呢？
> 因为值太大会导致计算的int（因为Java中的hashCode返回值都为int型）值较大，结果会溢出。如31^5=28629151，而101^5=2147483647，101的计算结果会比31大的多。

**Hashtable和HashMap区别？**

> 1. Hashtable是线程安全的，而HashMap不是。
> 2. HashMap中允许存在null键和null值（注意：ConcurrentHashMap不允许null键值），而Hashtable中不允许。
> 3. Hashtable已经弃用，我们可以用ConcurrentHashMap等替代。

**HashMap如何进行扩容的？扩容过程是什么？**

> 1. 在初次加载时，会调用resize()进行初始化。
> 2. 当put()时，会查看当前元素个数是否大于阈值（阈值=数组长度*负载因子），当大于时，调用resize方法扩容。
>    *<small>HashMap默认数组大小16(2的4次方），默认负载因子0.75，即默认阈值为：16x0.75=12</small>*
> 3. 新建一个数组，扩容后的容量是原来的两倍。
> 4. 将原来的数据重新计算位置，拷贝到新的table上。

**负载因子为什么是0.75，为什么不是0.5或者1？**

> 默认负载因子（0.75）在时间和空间成本上提供了很好的折衷。较高的值会降低空间开销，但提高查找成本（体现在大多数的HashMap类的操作增加处理时间，包括get和put）。较低的值会导致空间利用率不高。
>
> 扩展：为什么HashMap初始容量设置为16？
> 16=2的4次方，这个值一定要是2的幂指数，这样便于进行移位计算，具有更高的效率。其次设置为16，而不是8或者4或者32，实则没有特殊要求，而是作为一个经验值被采用了（太小了就有可能频繁发生扩容，影响效率。太大了又浪费空间，不划算）。

**ConcurrentHashMap有什么区别？**

> **ConcurrentHashMap通过分段锁实现了线程安全的HashMap，且基于分段锁的机制，效率比添加synchronized的Hashtable更高，且get()方法整个过程不会加锁，这也是提升效率的一种体现。**
> **分段锁**机制：将HashMap中的数组分成N的段(Segment)，然后分别对每个段的数据链表加锁，这样可以实现多个线程在访问HashMap中的数据时，因为不同区段的锁实现多线程访问，效率远比全盘加锁的synchronized模式（即串行模式：整个HashMap被锁住，每个线程访问时只能以此等待锁，所以理解为串行模式执行）高。
>
> **扩展1：为什么ConcurrentHashMap的get（读操作）不需要加锁？**
> 1）链表Node的成员val是用volatile修饰的，保证了多线程中的可见性。
> *Java内存模型的happens-before原则，对volatile字段的写入操作先于读操作，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值。*
> 2）读操作本身并不会造成数据的改变，读锁没有意义。
>
> **扩展2：happens-before原则**
> 在JMM中，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。
> 1）如果一个操作happens-before另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
> 2）两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）。

#### 1.6.3、CopyOnWrite容器

在此将CopyOnWriteArrayList和CopyOnWriteArraySet统称为CopyOnWrite容器。即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

CopyOnWrite容器读的时候不需要加锁，如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的。JDK中并没有提供CopyOnWriteMap，需自行实现。

缺点：由于写时复制机制，会耗费内存；无法保证数据实施一致性，只能保证最终一致性。

*<small>更多可参考：https://blog.csdn.net/hello_worldee/article/details/77880558</small>*

### 1.7、Java中的代理模式和AOP

静态代理和动态代理？

> 代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理模式是一种结构型设计模式，主要解决的问题是：在直接访问对象时带来的问题（解耦）。
>
> **静态代理**：在程序运行前代理类的.class文件就已经存在了。
> 示例如下：
>
> ```java
> public interface IUser1{//接口
>     void test1();
> }
> public class User1Impl implements IUser1 {//接口实现类
>     @Override
>     public void test1() {
>         System.out.println("IUser1 test1().");
>     }
> }
> public class User1ImplProxy implements IUser1 {//静态代理类
> 	// 目标对象
> 	private IUser1 user1Obj;
> 	// 通过构造方法传入目标对象
> 	public User1ImplProxy(IUser1 user1Obj){
> 		this.user1Obj=user1Obj;
> 	}
>     @Override
> 	public void test1() {
> 		System.out.println("User1ImplProxy.test1()");
> 	}
> }
> public static void main(String[] args) {
> 	Iuser1 user1Ojb = new User1ImplProxy(new User1Impl());//利用静态代理访问接口方法
> 	user1Obj.test1();
>     //输出：user1Obj.test1();
> }
> //优点：实现简单
> //缺点：代理对象只服务于一种类型的对象，如果要服务多类型的对象。势必要为每一种对象都进行代理，不能完全解耦。可维护性低，可重用性低。
> ```
>
> **动态代理**：在程序运行时运用反射机制动态创建而成。
> 示例如下：
>
> ```java
> public interface IUser2 {//接口
>     void test2();
> }
> public class User2Impl implements IUser2 {//接口实现
>     @Override
>     public void test2() {
>         System.out.println("IUser2 test2().");
>     }
> }
> public class TestLogHandler implements InvocationHandler {//动态代理实现
>     private Object object;
>     public Object newProxyInstance(Object obj) {
>         this.object = obj;
>         return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
>     }
>     @Override
>     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>         return method.invoke(this.object, args);
>     }
> }
> public static void main(String[] args) {
>  	TestLogHandler tl = new TestLogHandler();
>     IUser2 userObj2 = (IUser2) tl.newProxyInstance(new User2Impl());
>     userObj2.test2();
>     IUser1 userObj1 = (IUser1) tl.newProxyInstance(new User1Impl());
>     userObj1.test1();
>     //输出：
>     //IUser2 test2().
> 	//IUser1 test1().
> }
> //优点：代理类与被代理类完全解耦。代码重用性强（一个Handler可以实现多个接口）。
> //缺点：不易理解。
> ```
>
> *动态代理与**AOP(面向切面的编程)**的关系？*
> *AOP是一种思想，而动态代理是一种AOP思想的实现。*
> AOP 利用一种称为横切的技术，剖开对象的封装，并将影响多个类的公共行为封装到一个可重用模块，组成一个切面，即 Aspect 。
> "切面"就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，利于可操作性和可维护性。

### 1.8、lambda表达式

Lambda表达式语法由`参数列表`、`->`和`函数体`组成。函数体既可以是一个表达式也可以是一个代码块。

```java
(int x, int y) -> x + y                      //接收x和y两个整形参数并返回他们的和
() -> 66                                     //不接收任何参数直接返回66
(String name) -> {System.out.println(name);} //接收一个字符串然后打印出来
(View view) -> {view.setText("lalala");}     //接收一个View对象并调用setText方法
```

### 1.9、Java Bean是什么

**JavaBean是一个遵循特定写法的Java类**，它通常具有如下特点：

- 这个Java类必须具有一个无参的构造函数
- 属性必须私有化。
- 私有化的属性必须通过public类型的方法暴露给其它程序，并且方法的命名也必须遵守一定的命名规范。

如下代码所示：

```java
/**
 * Person类就是一个最简单的JavaBean
 */
public class Person {
    //------------------Person类封装的私有属性---------------
    private String name;
    private int age;
    //-------------------------------------------------------
    
    //------------------Person类的无参数构造方法---------------
    public Person() {}
    //---------------------------------------------------------
    
    //------------------Person类对外提供的用于访问私有属性的public方法-------
    //每个属性通常都具有相应的set、get方法，set方法称为属性修改器，get方法称为属性访问器。
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    //-----------------------------------------------------------------------
}
```

### 1.10、Java Steam API

Java 8 API添加了一个新的抽象称为流Stream，可以实现以一种声明的方式处理数据。
Stream 作为 Java 8 的一大亮点，它与 java.io 包里的 InputStream 和 OutputStream 是完全不同的概念。它也不同于 StAX 对 XML 解析的 Stream，也不是 Amazon Kinesis 对大数据实时处理的 Stream。Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作（aggregate operation），或者大批量数据操作 (bulk data operation)。Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性。同时它提供串行和并行两种模式进行汇聚操作，并发模式能够充分利用多核处理器的优势，使用 fork/join 并行方式来拆分任务和加速处理过程。通常编写并行代码很难而且容易出错, 但使用 Stream API 无需编写一行多线程的代码，就可以很方便地写出高性能的并发程序。

**什么是 Stream？**

Stream（流）是一个来自数据源的元素队列并支持聚合操作。Stream 是对集合（Collection）对象功能的增强，它不是集合元素，它不是数据结构并不保存数据，它是有关算法和计算的，它更像一个高级版本的 Iterator。原始版本的 Iterator，用户只能显式地一个一个遍历元素并对其执行某些操作；高级版本的 Stream，用户只要给出需要对其包含的元素执行什么操作，比如 “过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream 会隐式地在内部进行遍历，做出相应的数据转换。*Stream 就如同一个迭代器（Iterator），单向，不可往复，数据只能遍历一次，遍历过一次后即用尽了，就好比流水从面前流过，一去不复返。*

在 Java 8 中, 集合接口有两个方法来生成流：
**stream()** ： 为集合创建串行流。
**parallelStream()**：为集合创建并行流。

```java
//filter方法用于通过设置的条件过滤出元素
//示例1：利用filter过滤List中的空字符串
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
//使用filter方法过滤统计空字符串个数
long count = strings.stream().filter(string -> string.isEmpty()).count();
//parallelStream是流并行处理程序的代替方法。以下实例我们使用parallelStream来输出空字符串的数量
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();

//示例2：Stream 提供了新的方法'forEach'来迭代流中的每个数据。以下代码片段使用forEach输出了10个随机数
Random random = new Random();
//limit 方法用于获取指定数量的流，如下限制打印输出10个随机数
random.ints().limit(10).forEach(System.out::println);
//sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序
random.ints().limit(10).sorted().forEach(System.out::println);//双冒号运算就是Java中的[方法引用],[方法引用]的格式是 类名::方法名。一般用于lambda表达式

//示例3：map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
List<Integer> squaresList = numbers.stream().map( i -> i*i).collect(Collectors.toList());
```

### 1.11、装箱和拆箱

**什么是装箱和拆箱？**
Java中基础数据类型与它们的包装类进行运算时，编译器会自动帮我们进行转换，转换过程对程序员是透明的，这就是装箱和拆箱，装箱和拆箱可以让我们的代码更简洁易懂。
Java中8种基本数据类型和其对应的包装类如下所示：
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_boxing.png?raw=true" alt="java_boxing.png" style="zoom:50%;" />

*由原始类型转换为包装类型的过程称为装箱，反之则称为拆箱。*

**装箱和拆箱的事项原理？**
装箱是通过调用包装器类的 valueOf 方法实现的
拆箱是通过调用包装器类的 xxxValue 方法实现的，xxx代表对应的基本数据类型。
如int装箱的时候自动调用Integer的valueOf(int)方法；Integer拆箱的时候自动调用Integer的intValue方法。

```java
int a = 100;
Integer b = 100;
System.out.println(a == b);
//输出true，包含算术运算会触发自动拆箱（将b转换为基本类型int），因而比较a和b的值，故为true

Integer c = 100;
Integer d = 100;
System.out.println(c == d);
//输出false，比较c和d的内存，但是因为IntegerCache的策略，100对应的值已被预先缓存，所以c和d实际上执行的是同一个内存对象，故地址相等。

c = 200;
d = 200;
System.out.println(c == d);
//输出false

//IntegerCache类把-128到127（可调）的整数都提前实例化了，这样可以减少new Integer次数，提升效率。
//因此Integer中超过-128到127范围外的值，在比较时因为超出范围的Integer会被重新new，所以地址不一样。
//其他例如：ByteCahe、ShortCache和LongCache原理一致。
//注意：除了Integer可以通过参数（java.lang.Integer.IntegerCache.high）改变范围外，其它的都不行。
```

**装箱、拆箱的触发逻辑（何时触发装箱或拆箱）？**
1）进行 = 赋值操作（装箱或拆箱）
2）进行+，-，*，/混合运算 （拆箱）
3）进行>,<,==比较运算（拆箱）
4）调用equals进行比较（装箱）
5）ArrayList,HashMap等集合类 添加基础类型数据时（装箱）

### 1.12、理解Java注解（Annotation）

**注解**就相当于对源代码打的**标签**，给代码打上**标签**和删除**标签**对源代码没有任何影响。这些标签自己本身不起什么作用，而且外部工具通过访问这些标签，然后根据不同的标签做出了相应的处理。这是**注解**的精髓。例如IDE（IntelliJ Idea等），它检查发现某一个方法上面有`@Deprecated`这个注解，它就会在所有调用这个方法的地方将这个方法标记为删除。
访问和处理注解（Annotation）的工具统称为**APT(Annotation Processing Tool)**。

#### 1.12.1、基本类型的注解

1）**@Override**：让编译器检查被标记的方法，保证其重写了父类的某一个方法。此注解只能标记方法。
2）**@Deprecated**：标记某些程序元素已经过时，不再推荐使用了。
3）**@SuppressWarning**：告诉编译器不要显示警告。
4）**@SafeVarargs**：Java7新增，`@SuppressWarnings`可以用在各种需要取消警告的地方，而 `@SafeVarargs`主要用在取消参数的警告。这个注解是专为取消**堆污染**警告设置的，因为Java7会对可能产生堆污染的代码提出警告。此注解对非static或者非final类型的方法无效。

```java
//堆污染示例
//堆污染：即当一个泛型类型变量赋值给不是泛型类型变量，这种错误在编译期间能被编译器警告，但是可以忽略，直到运行时报错。
public static void test(List<String>... strLists) {
    List[] array = strLists;
    List<Integer> tmpList = Arrays.asList(42);
    array[0] = tmpList; //非法操作，但是没有警告
    String s = strLists[0].get(0); //ClassCastException at runtime!
}
```

5）**@FunctionalInterface**：Java8新增，标记型注解，告诉编译器检查被标注的接口是否是一个函数接口，即检查这个接口是否只包含一个抽象方法，只有函数接口才可以使用`Lambda`表达式创建实例。

#### 1.12.2、元注解

元注解是用来给其他注解打标签的注解，即用来注解其他注解的注解。（注解的注解）
1）**@Retention**：注解的生命周期，用于指定被此元注解标注的注解的保留时长。Retention策略如下：
      RetentionPolicy.SOURCE：注解信息只保留在源代码中，编译器编译源码时会将其直接丢弃。
	  RetentionPolicy.CLASS：注解信息会被编译器记录在类文件.class中，但在运行时会被JVM移除，这也是默认的策略。（仅在编译时起作用）
      RetentionPolicy.RUNTIME：注解信息会被编译器记录在类文件.class中并且在运行时会被JVM保留，可以通过反射读取。（仅在运行时起作用）
2）**@Target**：注解的作用目标，用于指定被此元注解标注的注解可以标注的程序元素。*@Target对应类型如下：*

```java
//Target对应类型
public enum ElementType {
    TYPE,//该注解可以用于类、接口（包括注解类型）或enum声明
    FIELD,//该注解可以用于字段(域)声明，包括enum实例
    METHOD,//该注解可以用于方法声明
    PARAMETER,//该注解可以用于参数声明
    CONSTRUCTOR,//该注解可以用于构造函数声明
    LOCAL_VARIABLE,//该注解可以用于局部变量声明
    ANNOTATION_TYPE,//该注解可以用于注解声明(应用于另一个注解上)
    PACKAGE,//该注解可以用于包声明
    TYPE_PARAMETER,//该注解可以用于类型参数声明（1.8新加入）
    TYPE_USE//类型使用声明（1.8新加入)
}
```

3）**@Documented**：将被标注的注解生成到javadoc中。
4）**@Inherited**：其让被修饰的注解拥有被继承的能力（即：是否允许子类集成该注解）。默认为false。只能应用于类声明。
5）**@Repeatable** ：Java8新增，使被修饰的注解可以重复的注解某一个程序元素。

#### 1.12.3、实现自定义注解

**1）自定义注解的语法要求**

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/custom_annotation_grammer.png?raw=true" alt="custom_annotation_grammer.png" style="zoom:40%;" />

**2）如何使用自定义注解**

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/custom_annotation_use.png?raw=true" alt="custom_annotation_use.png" style="zoom:40%;" />

**3）如何解析注解**

解析注解需要用到Java反射技术，完整示例如下：

```java
//示例：自定义一个注解，名为MyAnnotation
@Retention(RetentionPolicy.RUNTIME)//运行时生效，如果选择CLASS或SOURCE，那么在运行时无法通过反射取得相关值
@Inherited//继承，子类自动继承父类的注解
@Documented//生成JavaDoc时，会将自定义注解的信息生成到JavaDoc
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.TYPE})//注解所作用的目标对象
public @interface MyAnnotation {//注解是以关键字public @interface来定义的
    //注解中的方法不可以有参数，也不可以抛出异常。
    //同时方法只能返回java 8种基本类型，以及String、Class、Enumertation和Annotation类型，以及上述类型的数组。
    //方法的默认值不可以是null。
    public String name() default "hello";//default关键字指定默认值
    //此处请注意：若类中有且仅有一个成员变量，那么此处方法定义可为value()
    //如此定义，可以在使用注解时写“@MyAnnotation("anno string")”，而无需写成“@MyAnnotation(name = "anno string")”
}

//示例：如何使用自定义的注解MyAnnotation
@MyAnnotation(name = "Class Annotation")//类注解
public class User {
    @MyAnnotation(name = "field annotation")//成员变量注解
    private String name;
    @MyAnnotation(name = "method annotation")//方法注解
    public String sayHello(){
        return "";
    }
    public static void main(String[] args) throws ClassNotFoundException {
        //通过反射取得注解
        //1、获取类注解
        if (User.class.isAnnotationPresent(MyAnnotation.class)) {
            MyAnnotation annotation = User.class.getAnnotation(MyAnnotation.class);
            System.out.println(annotation.name());
        }
        //2、获取方法的注解
        Field[] fields = User.class.getDeclaredFields();
        for (Field field : fields) {
            MyAnnotation annotation = field.getAnnotation(MyAnnotation.class);
            if (annotation != null) {
                System.out.println(annotation.name());
            }
        }
        //3、获取成员变量的注解
        Method[] methods = User.class.getMethods();
        for (Method method : methods) {
            MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
            if (annotation != null) {
                System.out.println(annotation.name());
            }
        }
    }
}
//输出：
Class Annotation//类注解
field annotation//成员变量注解
method annotation//方法注解

//示例：注解继承效果，即使用了@Inherited元注解而实现的继承效果
public class User2 extends User {
    public static void main(String[] args) {
        //1、获取类注解
        if (User2.class.isAnnotationPresent(MyAnnotation.class)) {
            MyAnnotation annotation = User2.class.getAnnotation(MyAnnotation.class);
            System.out.println(annotation.name());
        }
        //2、获取方法的注解
        Field[] fields = User2.class.getDeclaredFields();
        for (Field field : fields) {
            MyAnnotation annotation = field.getAnnotation(MyAnnotation.class);
            if (annotation != null) {
                System.out.println(annotation.name());
            }
        }
        //3、获取成员变量的注解
        Method[] methods = User2.class.getMethods();
        for (Method method : methods) {
            MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
            if (annotation != null) {
                System.out.println(annotation.name());
            }
        }
    }
}
//输出：
Class Annotation//自动继承了父类的类注解
method annotation//自动继承了父类的方法注解
//因为父类成员变量私有，故子类无成员变量继承输出
```



### 1.13、可变参数（3个点）

类型后面三个点(String…)，是从Java 5开始，Java语言对方法参数支持一种新写法，叫可变长度参数列表，其语法就是类型后跟…，表示此处接受的参数为0到多个Object类型的对象，或者是一个Object[]。示例如下：

```java
public void test(int... args) {//可变参数接收
    //可变参数args可以理解为数组，即int[] args
    System.out.println(args.length);
    for (int arg : args) {
        System.out.println(arg);
    }
}
public static void main(String[] args) {
    test(1, 2, 3);//传递多个值
    test(100);//传递一个值
    test(new int[]{3,2,1});//传递数组
}
```

## 2、JVM/GC相关

### 2.1、JVM内存结构

JVM内存区域分为：`线程私有`和区域和`线程共有`的区域。
**线程私有的区域**：程序计数器、虚拟机栈、本地方法栈。*<small>（执行运算的区域。）</small>*
**线程共有的区域**：堆、方法区、运行时常量池。*<small>（存储数据的区域。）</small>*

*非线程共享（线程私有）部分*：
**程序计数器**：每个线程有有一个私有的程序计数器，任何时间一个线程都只会有一个方法正在执行，也就是所谓的当前方法。程序计数器存放的就是这个当前方法的执行的指令地址。
*<small>程序计数器只对Java方法进行计数，Native方法时计数器值为Undefined。</small>*
**虚拟机栈**：创建线程的时候会创建线程内的虚拟机栈，栈中存放着一个个的**栈帧（对应着一个个方法的调用）**。JVM 虚拟机栈有两种操作，分别是压栈和出站。栈帧中存放着局部变量表、方法返回值
和方法的正常或异常退出的定义等等。

> 栈由一系列栈帧组成，即一个方法对应一个栈帧，每个栈帧包含其方法对应的局部变量表、操作数栈、常量指针和动态链接。

**本地方法栈**： 跟 JVM 虚拟机栈比较类似，只不过它支持的是 Native 方法。
*<small>HotSpot虚拟机将虚拟机栈和本地方法栈合并为一个栈。</small>*

*线程共享部分：*
**堆**：堆是内存管理的核心区域，用来存放对象实例。几乎所有创建的对象实例都会直接分配到堆上。所以堆也是垃圾回收GC的主要工作区域，垃圾收集器会对堆有着更细的划分，最常见的就是把堆划分为新生代和老年代。
**方法区**：方法区主要存放类的结构信息，常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，为了与堆进行区分，通常又叫“非堆”。

> 注意：除开程序计数器不会发生内存泄漏，其他内存区域都有可能发生内存泄漏。
> 程序计数器、虚拟机栈和本地方法栈区域内存都会随着线程产生和消失，因此不需要过多考虑GC问题（但并不意味着栈区就不会发生内存溢出，比如递归循环层次太多就会导致栈帧过多，从而产生StackOverFlowError），因为方法结束或线程结束时，内存就自动跟着回收了。
> 而GC的主要工作区域为堆和方法区，JVM调优也主要针对这一部分。

**元空间和永久代**：元空间的本质和永久代类似，都是对JVM规范中方法区的实现。区别在于：**元空间并不在虚拟机中，而是使用本地内存。**

> Java7及之前的版本中，HostSpot虚拟机中又将方法区称作**永久代**，Java8及以后的版本，取消了永久代，取而代之的是元空间metaspace。永久代的参数-XX:PermSize和-XX：MaxPermSize也随之失效。

*举例扩展：因为元空间和永久代都是对方法区的实现，而方法区逻辑上又属于堆，所以类似于String常量池（常量池位于方法区）的内存溢出也即发生在堆上。*

> **扩展1：Java堆内存一定是线程共享的吗？**
> 此问题可以通过TLAB分配来解答，描述如下：
> (1) 堆是JVM中所有线程共享的，因此在其上进行对象内存的分配均需要进行加锁，这也导致了new对象的开销是比较大的。
> (2) Hotspot JVM为了提升对象内存分配的效率，对于所创建的线程都会分配一块独立的空间TLAB（Thread Local Allocation Buffer），其大小由JVM根据运行的情况计算而得，在TLAB上分配对象时不需要加锁，这部分Buffer是从堆中划分出来的，但是是`本地线程独享`的。
> (3) TLAB仅作用于新生代的Eden Space，因此在编写Java程序时，通常多个小的对象比大的对象分配起来更加高效。
>
> 注意：TLAB是线程独享的，但是**只是在“分配”这个动作上是线程独享的**，至于在读取、垃圾回收等动作上都是线程共享的。而且对于程序来讲，通过此方式申请的内存在使用上也没有什么区别。
>
> **扩展2：对象的内存分配流程**
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_mem_allocate_example.png?raw=true" alt="java_mem_allocate_example.png" style="zoom:80%;" />

### 2.2、JAVA对象在内存中的状态

**可达/可触及的**：Java对象被创建后，如果被一个或多个变量引用，那就是可达的。即从根节点可以触及到这个对象。即从根节点扫描，只要这个对象在引用链中，那就是可触及的。

**可恢复的**：对象不再被任何变量引用就进入了可恢复状态。在回收该对象之前，该对象的finalize()方法进行资源清理。如果在finalize()方法中重新让变量引用该对象，则该对象再次变为可达状态，否则该对象进入不可达状态。

**不可达的**：Java对象不被任何变量引用，且系统在调用对象的finalize()方法后依然没有使该对象变成可达状态（该对象依然没有被变量引用），那么该对象将变成不可达状态。只有当对象处于不可达状态时，系统才会真正回收该对象所占有的资源。

### 2.3、垃圾回收算法

垃圾回收需要完成两件事：找到垃圾，回收垃圾。

> 找到垃圾一般的话有两种方法：

**引用计数法*（因循环引用问题，已被废弃）*：** 当一个对象被引用时，它的引用计数器会加一，垃圾回收时会清理掉引用计数为0的对象。
*但这种方法有一个问题，比方说有两个对象 A 和 B，A 引用了 B，B 又引用了 A，除此之外没有别的对象引用 A 和 B，那么 A 和 B 在我们看来已经是垃圾对象，需要被回收，但它们的引用计数不为 0，没有达到回收的条件。正因为这个`循环引用`的问题，Java 并没有采用引用计数法。*
循环引用问题示例如下：

```java
public class TestRC {
    TestRC instance;
    public TestRC(String name) {
    }
    public static  void main(String[] args) {
        // 第一步
        A a = new TestRC("a");//a引用计数器refA = 1;
        B b = new TestRC("b");//b引用计数器refB = 1;
        // 第二步
        a.instance = b;//a引用计数器+1，refA = 2;
        b.instance = a;//b引用计数器+1，refB = 2;
        // 第三步
        a = null;//a引用计数器-1，refA = 1;
        b = null;//b引用计数器-1，refB = 1;
        //虽然 a，b都被置为null了，但是由于之前它们指向的对象互相指向了对方（引用计数最终无法减为0），所以造成无法回收a，b对象（故引用计数法无法解决循环引用问题）
    }
}
```

**可达性分析法（或者叫根搜索算法）：** 我们把 Java 中对象引用的关系看做一张图，从根级对象不可达的对象会被垃圾收集器清除。
*根级对象（GC roots）一般包括：*
*1) 虚拟机栈中（栈帧中的本地变量表）的引用的对象*
*2) 本地方法栈中的对象（JNI Native引用的对象）*
*3) 方法区中的静态对象*
*4) 方法区中的常量池的常量（例如主要指的是声明为final的常量）*
代码示例如下：

```java
//示例1：虚拟机栈中引用的对象
public class Test {
    public static  void main(String[] args) {
        Test a = new Test();
        a = null;
        //a 是栈帧中的本地变量，当a = null时，由于此时a充当了GC Root的作用，a与原来指向的实例 new Test() 断开了连接，所以对象会被回收。
    }
}
//示例2：方法区中的静态对象
public class Test {
    public static Test s;
    public static  void main(String[] args) {
        Test a = new Test();
        a.s = new Test();
        a = null;
		//当栈帧中的本地变量a = null时，由于a原来指向的对象与 GC Root (变量 a) 断开了连接，所以 a 原来指向的对象会被回收，而由于我们给s赋值了变量的引用，s在此时是类静态属性引用，充当了GC Root的作用，它指向的对象依然存活!
    }
}
//示例3：方法区中的常量池的常量
public class Test {
	public static final Test s = new Test();
    public static void main(String[] args) {
        Test a = new Test();
        a = null;
        //常量s指向的对象并不会因为a指向的对象被回收而回收
    }
}
```

> 回收垃圾一般有四种方法：

**标记-清除算法**：分为标记和清除两个阶段。
*标记阶段：*通过根节点，标记所有从根节点开始的可达对象（使用可达性分析算法，未被标记的对象就被视为垃圾对象）。
*清除阶段：*清除所有未被标记的对象。
*缺点：*效率不高（标记和清除都需要从头遍历到尾）；清除垃圾对象后会造成内存的碎片化。

**复制算法（即新生代GC，MinorGC）**：把堆等分成两块区域A和B，区域A负责分配对象，区域B不负责分配对象，对区域A使用上述标记法把存活的对象标记出来，然后把区域A中存活的对象都复制到区域B（存活对象都依次紧邻排列，即将存活对象复制到另一块未使用的内存区域B中），最后把区对象全部清理掉释放出空间，这样就解决了内存碎片的问题了。
*优点：*可以避免内存碎片化。
*缺点：*显而易见，它需要两倍的内存（空间浪费），且频繁复制存活对象也可能会存在效率低下问题。

**标记-整理算法（即老年代GC，FullGC）**：类似标记-清除算法，不同的是它在标记清除法的基础上添加了一个整理的过程。
*标记-清除阶段：*同标记-清除算法一致。
*整理阶段：*清除掉垃圾对象后会将存活的对象压缩。
*优点：*避免了内存的碎片化。
*缺点：*在标记的基础上还需要进行对象的移动（内存整理），效率也不算高。

**分代收集算法**：分代收集算法整合了以上算法优点，最大程度避免了它们的缺点，是虚拟机采用的首选算法。
1）在程序运行过程中，大部分的对象都很短命，朝生夕死，都会在很短的时间内就被回收了，所以分代收集算法根据对象存活周期的不同将堆分成新生代和老生代（Java8以前还有个永久代），新生代老年代默认比例1:2。
2）其中，新生代又分为Eden区，from Survivor区（简称S0）和to Survivor区（简称S1），三者比例默认为8:1:1。
3）新生代发生的 GC 称为 Young GC（也叫 Minor GC），老年代发生的 GC 称为 Old GC（也称为 Full GC）。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_yong_old.png?raw=true" alt="jvm_yong_old.png" style="zoom:77%;" />

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_yong_old2.png?raw=true" alt="jvm_yong_old2.png" style="zoom:88%;" />

***扩展：分代收集算法原理***
1）大部分对象在很短的时间内都会被回收，所以对象一般分配在Eden区，当 Eden 区将满时，触发 Minor GC。
2）经过 Minor GC 后只有少部分对象会存活，它们会被移到 S0 区*（这就是为啥空间大小Eden: S0: S1 = 8:1:1, Eden 区远大于 S0、S1的原因，因为在Eden区触发的 Minor GC把大部对象（接近98%）都回收了,只留下少量存活的对象，此时把它们移到S0或S1绰绰有余）*，同时对象年龄加一*（对象的年龄即发生 Minor GC 的次数）*，最后把 Eden 区对象全部清理以释放出空间。如下图所示：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_fendai1.gif?raw=true" alt="jvm_fendai1.gif" style="zoom:67%;" />

3）当触发下一次Minor GC时，会把Eden区的存活对象和S0（或S1）中的存活对象*（S0或S1中的存活对象经过每次Minor GC都可能被回收）*一起移到S1（Eden和S0的存活对象年龄+1），同时清空Eden和S0的空间。如下图所示：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_fendai2.gif?raw=true" alt="jvm_fendai2.gif" style="zoom:67%;" />

4）若再触发下一次Minor GC，则重复上一步，只不过此时变成了从 Eden，S1 区将存活对象复制到S0区，每次垃圾回收，S0、S1 角色互换，都是从 Eden，S0(或S1) 将存活对象移动到 S1(或S0)。也就是说**在Eden区的垃圾回收我们采用的是复制算法**，因为在 Eden 区分配的对象大部分在 Minor GC 后都消亡了，只剩下极少部分存活对象（这也是为啥 Eden:S0:S1 默认为 8:1:1 的原因），S0、S1区域也比较小，所以最大限度地降低了复制算法造成的对象频繁拷贝带来的开销。
5）当对象的年龄达到了我们设定的阈值，则会从S0（或S1）晋升到老年代。如年龄阈值设置为15。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_fendai3.gif?raw=true" alt="jvm_fendai3.gif" style="zoom:67%;" />


当某个对象分配需要大量的连续内存时（大对象），此时对象的创建不会分配在Eden区，会直接分配在老年代*（因为如果把大对象分配在Eden 区，Minor GC 后再移动到 S0、S1会有很大的开销（对象比较大，复制会比较慢，也占空间），也很快会占满 S0、S1 区，所以干脆就直接移到老年代）*。
***<small>新生代一般采用复制算法，而老年代一般采用标记-清理算法（因为在老生代由于对象比较多，占用的空间较大，使用复制算法会有较大开销（复制算法在对象存活率较高时要进行多次复制操作，同时浪费一半空间）所以根据老生代特点，在老年代进行的 GC一般采用的是标记整理法来进行回收）。</small>***

### 2.4、垃圾回收器分类

**Serial收集器（串行收集器）**：单线程收集器，进行垃圾回收时，必须暂停所有工作线程，直到收集过程结束。

> 配置示例：-XX:+UseSerialGC  (对应老年代回收-XX:+UseSerialOldGC)  

**ParNew收集器**：是Serial收集器的多线程版本。相比Serial收集器，Parallel最主要的优势在于使用多线程去完成垃圾清理工作，这样可以充分利用多核的特性，大幅降低gc时间。

> 配置示例：-XX:+UseParNewGC

**Parallel Scavenge收集器**：类似ParNew，但更加关注吞吐量。是JVM Server模式下默认的年轻代收集器。

> **停顿时间和吞吐量不可能同时调优。**
> **什么是吞吐量**：CPU用于用户代码的时间/CPU总消耗时间的比值，即=运行用户代码的时间/(运行用户代码时间+垃圾收集时间)。比如，虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。
> 配置示例：-XX:+UseParallelGC  (对应老年代回收-XX:+UseParallelOldGC) 

**CMS(Concurrent Mark Sweep并发标记清除)收集器**：CMS收集器在Minor GC时会暂停所有的应用线程，并以多线程的方式进行垃圾回收。在Full GC时不再暂停应用线程，而是使用若干个后台线程定期的对老年代空间进行扫描，及时回收其中不再使用的对象。

> 因为使用标记-清除算法，所以会产生内存碎片。
>
> CMS收集器在Minor GC时会暂停所有的应用线程，并以多线程的方式进行垃圾回收。在Full GC时不再暂停应用线程，而是使用若干个后台线程定期的对老年代空间进行扫描，及时回收其中不再使用的对象。
>
> 配置示例：-XX:+UseConcMarkSweepGC

**G1(Garbage First)收集器**：对垃圾回收进行了划分优先级的操作。G1收集器（或称为垃圾优先收集器）的设计初衷是为了尽量缩短处理超大堆（大于4GB）时产生的停顿。相对于CMS的优势而言是内存碎片的产生率大大降低。

> G1垃圾收集算法主要应用在多CPU大内存的服务中，在满足高吞吐量的同时，尽可能的满足垃圾回收时的暂停时间。
>
> G1老生代的收集不会为了释放老生代的空间对整个老生代做回收。相反，在任意时刻只有一部分老生代分区会被回收（由此减少了FullGC即FullGC的暂停时间），并且，这部分老生代分区将在下一次增量回收时与所有的新生代分区一起被收集。这就是我们所说的混合回收（Mixed GC）。
>
> 配置示例：-XX:+UseG1GC

***收集器在执行GC时，都会有STW(Stop-The-World)问题，即在 GC（minor GC 或 Full GC）期间，只有垃圾回收器线程在工作，其他工作线程则被挂起。***如下图所示：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_stw.png?raw=true" alt="jvm_stw.png" style="zoom:50%;" />


> **扩展1：为什么在垃圾收集期间其他工作线程会被挂起？**
> 想象一下，你一边在收垃圾，另外一群人一边丢垃圾，垃圾能收拾干净吗。
>
> **扩展2：如何理解STW中的安全点（safe point）？**
> 由于GC（Full GC或Minor GC）会影响性能，所以我们要在一个合适的时间点发起GC，这个时间点就被称为 safe point，这个时间点的选定既不能间隔太久以至于让 GC 时间太长导致程序过长时间卡顿，也不能过于频繁以至于过分增大运行时的负荷。safe point一般在以下特定位置触发：
> (1) 循环的末尾；(2) 方法返回前；(3) 调用方法的 call 之后；(4) 抛出异常的位置；
>
> **扩展3：为什么把新生代设置成 Eden、S0、S1区？为什么给对象设置年龄阈值？为什么默认把新生代与老年代的空间大小设置成1:2?**
> 都是为了***尽可能地避免对象过早地进入老年代，尽可能晚地触发 Full GC***。想想新生代如果只设置Eden会发生什么，后果就是每经过一次Minor GC，存活对象会过早地进入老年代，那么老年代很快就会装满，很快会触发Full GC，而对象其实在经过两三次的Minor GC后大部分都会消亡，所以有了S0、S1的缓冲，只有少数的对象会进入老年代，老年代大小也就不会这么快地增长，也就避免了过早地触发Full GC。

<small>*垃圾回收算法/收集器更多描述可参考：https://mp.weixin.qq.com/s/_AKQs-xXDHlk84HbwKUzOw*</small>

### 2.5、类加载机制

虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的Java类型，这就是虚拟机的类加载机制。

类加载过程包括：**加载、链接（含验证、准备、解析）、初始化（初始化完成后，其实类的加载过程已经结束了）。如下图所示：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_class_load.png?raw=true" alt="jvm_class_load.png" style="zoom:80%;" />

**加载**：是指，在Java文件编译成class文件后，将类的.class文件中的二进制数据读入内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个 java.lang.Class对象，用来封装类在方法区内的数据结构。

**链接/验证**：主要作用就是确保被加载的类的正确性。主要包括：文件格式的验证，元数据验证，字节码验证和符号引用验证。

**链接/准备**：主要为类变量分配内存并设置初始值。

**链接/解析**：虚拟机将常量池中的符号引用转化为直接引用的过程。

> **符号引用**：以一组符号来描述所引用的目标，可以是任何形式的字面量，只要是能无歧义的定位到目标就好，就好比在班级中，老师可以用张三来代表你，也可以用你的学号来代表你，但无论任何方式这些都只是一个代号（符号），这个代号指向你（符号引用）。
> **直接引用**：直接引用是可以指向目标的指针、相对偏移量或者是一个能直接或间接定位到目标的句柄。和虚拟机实现的内存有关，不同的虚拟机直接引用一般不同。

**初始化**：类加载机制的最后一步，程序代码才开始真正执行。

> 类的初始化过程：
> 1) 静态成员和static块 ---> 普通成员和非static块 ---> 构造函数。
> 2) 父类静态成员和static块 ---> 子类静态成员和static块 ---> 父类普通成员和非static块 ---> 父类构造函数 ---> 子类普通成员和非static块 ---> 子类构造函数。
> 3) 注意：final static修饰的静态常量在编译时就已初始化，所以调用时无需显式执行类初始化。

***类加载器类别***：Java自带以下3个类加载器：

**BootstrapClassLoader（启动类加载器）**：最顶层的加载类，主要加载核心类库，加载 jre/lib下的jar和class文件。

> 可以通过启动jvm时指定-Xbootclasspath和路径来改变Bootstrap ClassLoader的加载目录。例如：java -Xbootclasspath/a:path

**ExtClassLoader（扩展类加载器）**：扩展的类加载器，加载jre\lib\ext目录下的jar和class文件。还可以加载-D java.ext.dirs选项指定的目录。

**AppClassLoader（应用程序类加载器）**：也称为SystemAppClass，加载当前应用的classpath的所有类。

**双亲委派机制**：就是当加载一个类时，会优先使用父类加载器加载，当父类加载器无法加载时才会使用子类加载器去加载。这么做的目的是为了***避免类的重复加载***。

> 如何实现一个自定义类加载器？？
>
> 1) 继承ClassLoader，重写findClass()方法。
> 2) 在findClass()方法中调用defineClass()。
> 示例如下：
>
> ```java
> public class TestL {
>     static {
>         System.out.println("load TestL static");
>     }
> 
>     public TestL() {
>         System.out.println("构造器");
>     }
> }
> //记得使用javac 编译class文件
> ```
>
> ```java
> public class MyClassLoader extends ClassLoader {
>     private String path;
> 
>     public MyClassLoader(String path) {
>         this.path = path;
>     }
> 
>     @Override
>     protected Class<?> findClass(String name) throws ClassNotFoundException {
>         byte[] b = loadClassData(name);
>         return defineClass(name, b, 0, b.length);
>     }
> 
>     private byte[] loadClassData(String name) {
>         name = path + name + ".class";
>         InputStream is = null;
>         ByteArrayOutputStream bos = null;
>         try {
>             is = new FileInputStream(new File(name));
>             bos = new ByteArrayOutputStream();
>             int i = 0;
>             while ((i = is.read()) != -1) {
>                 bos.write(i);
>             }
>             return bos.toByteArray();
>         } catch (IOException e) {
>             e.printStackTrace();
>         } finally {
>             if (bos != null) {
>                 try {
>                     bos.close();
>                 } catch (IOException e) {
>                     e.printStackTrace();
>                 }
>             }
>             if (is != null) {
>                 try {
>                     is.close();
>                 } catch (IOException e) {
>                     e.printStackTrace();
>                 }
>             }
>         }
>         return null;
>     }
> }
> ```
>
> ```java
> public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
>         String path = new File(" ").getAbsolutePath();
>         System.out.println(path);
>         MyClassLoader loader = new MyClassLoader(path.trim() + "src" + File.separator + "com" + File.separator + "testjava" + File.separator + "test2" + File.separator);
>         System.out.println(loader);
>         Class c = loader.loadClass("com.testjava.test2.TestL");
>         System.out.println(c.getClassLoader());
>         c.newInstance();
>     }
> //输出：
> com.testjava.test2.MyClassLoader@74a14482
> sun.misc.Launcher$AppClassLoader@18b4aac2
> load TestL static
> 构造器
> ```

*更多JVM垃圾回收/类加载内容可参考：https://www.cnblogs.com/qianguyihao/p/4810168.html*

### 2.6、JVM调优

因为Java GC主要工作区域位于堆中，所以JVM调优实则也主要针对堆内存。

JVM中堆的大小主要有3个方面的限制：

> 操作系统的数据模型（32-bt还是64-bit）。
> 系统的可用虚拟内存。
> 系统的可用物理内存（32位系统下，一般限制在1.5G~2G；64位操作系统对内存无限制）。

调优举例：

1、手动设置堆内存大小：java -Xmx3550m -Xms3550m -Xmn2g -Xss128k

> 设置初始内存-Xms与-Xmx相同（-Xms和-Xmx设置的内存大小即为JVM堆的内存大小），可避免垃圾收集器在最小、最大之间收缩堆而产生额外的时间。
>
> -Xmn为设置年轻代大小，**JVM堆内存大小=年轻代大小 + 年老代大小 + 持久代/永久代大小**（Java8之前永久代一般为64MB固定大小，Java8及以后取消了永久代，所以此处调优一般仅关心年轻代和老年代即可）。
> **年轻代小**（**即老年代大**，JVM默认模式：年轻代和老年代1:2）：此种模式下，**FullGC产生的频率较低**（因为FullGC一般发生于老年代），**而普通GC频率就会较高**（因为年轻代小，普通GC一般多发生于年轻代）。
> **年轻代大**（**即老年代小**），则与上述逻辑**相反**。
> *<small>普通GC频率高时，则意味着每次普通GC所耗费的时长也较短。</small>*
>
> 综上，所以年轻代和老年代优化准则一般为：**如果应用存在大量的临时对象，应该选择更大的年轻代；如果存在相对较多的持久对象，年老代应该适当增大。**（一般在不影响FullGC的频率前提下，可适当将年轻代和老年代比例由系统默认的1:2调整为1:1，即-Xmn为-Xmx的1/2。）
>
> -Xss：设置每个线程的堆栈大小。（在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，所以一般情况下无需优化此参数。）

2、合理打印输出GC日志，供分析调优。

> - **-XX:+PrintGC**
> - **-XX:+PrintGCDetails**
> - **-XX:+PrintGCTimeStamps**
> - **-Xloggc:filename**  //与上面几个配合使用，把相关日志信息记录到文件以便分析。

3、考虑应用场景情况下，合理使用CMS或G1垃圾回收器。因为这两种收集器都是基于并发模式的收集器，可以降低垃圾回收stop-the-world的时间。且这两种收集器在老年代垃圾回收时，都可以降低FullGC的频率。

> 例如G1调优：-XX:InitiatingHeapOccupancyPercent=30
>
> -XX:InitiatingHeapOccupancyPercent表示指定整个堆的使用率达到多少时，执行一次并发标记周期，默认45。过大会导致并发标记周期迟迟不能启动, 增加FullGC的可能, 过小会导致GC频繁, 会导致应用程序性能有所下降。



### 2.7、面试考点举例

String s = new String("xyz")；创建了几个字符串对象？

> 视情况而定，当常量池中没有“xyz”对象时是创建两个，一个存放在常量池中（即"xyz"字符串常量），另一个存放在堆（内存）中（即：new String()创建的堆对象）。

String a = "abc"；String b = new String("xyz")；a存放在哪里？b存放在哪里？"abc"和"xyz"又存放在哪里？

> a和b都存放于栈上，"abc"存放于字符串常量池，"xyz"存放于字符串常量池，new String()存放于堆，并指向常量池"xyz"引用。

如何理解intern方法？

> String str1 = “abc”;
> String str2 = new String(“abc”);
> String str3 = str2.intern();
> System.out.println( str1= =str2 ); //false
> System.out.println( str1= =str3 ); //true
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_stack_example1.jpg?raw=true" alt="jvm_stack_example1.jpg" style="zoom:67%;" />

finalize方法是否等同于C++中的析构函数？

> finalize()方法：执行时间不确定（依赖垃圾回收执行等待时间，不可预测），所以不等同于C++中的析构函数。
> JDK9即以后版本finalize()方法已被废弃，不再推荐使用。

## **3、并发/锁/线程相关**

### 3.1、线程Thread中的start和run有什么区别？

> start方法会创建一个新的子线程并启动，而run方法只是普通方法调用并不会启动新的子线程。二者其实没有可比性。

### 3.2、Thread和Runnable有什么区别？

> - Thread是类，Runnable是接口。
> - Thread不能实现线程之间变量资源共享，Runnable则可以。
> 
> 举例：3个窗口卖10张票，new Thread()时表示3个窗口都会卖10张票，new Thread(runnable)时表示3个窗口共卖10张票。即：Thread是多个线程完成多个任务，Runnable是多个线程完成一个任务。

### 3.3、理解Thread的join方法

join()方法作用是：“等待该线程终止”，即调用子线程的线程必须在等待子线程运行结束后才能继续执行。
举例如：**一个线程A执行了thread.join()语句，其含义是：当前线程A等待thread线程终止后才从thread方法返回（后续代码才会继续执行）。**
代码示例如下：

```java
public class MyThread extends Thread implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread1 run...");
        try {
            java.lang.Thread.currentThread().sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread1 run ok.");
    }
}
public class MyThread2 extends Thread implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread2 run...");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread2 run ok.");
    }
}
public static void main(String[] args) throws InterruptedException {
    MyThread mt =  new MyThread();
    MyThread2 mt2 = new MyThread2();
    mt.start();
    mt.join();
    mt2.start();
    System.out.println("main...ok");   
}
//输出：
Thread1 run...
Thread1 run ok.
main...ok
Thread2 run...
Thread2 run ok.
```

### 3.4、Runnable和Callable区别

Runnable不会返回结果，并且无法抛出返回结果的异常；
Callable被线程执行后，可以返回值，这个返回值可以被Future拿到，也就是说，Future可以拿到异步执行任务的返回值，且Callable中的call方法会抛出异常。

*应用示例可见下述3.5章节。*

### 3.5、如何获取线程的返回值

- 使用join方法阻塞当前线程，以等待子线程处理完毕。

- 通过Callable接口实现（Callable接口一般配合FutureTask或者线程池使用）。

```java
public class MyCallable implements Callable<Object> {
    @Override
    public Object call() throws Exception {
        System.out.println("callable call..." + Thread.currentThread().getName());
        Thread.sleep(3000);
        System.out.println("callable ok");
        return 100;//Callable可以返回结果
    }
}

//通过FutureTask实现获取子线程返回值
public static void main(String[] args) {
    FutureTask<Object> f = new FutureTask<>(new MyCallable());
    Thread t = new Thread(f);
    t.start();
    if (!f.isDone()) {//isDone会判断Callable中的call方法是否已执行完成
        System.out.println("task is not done, wait...");
    }
    try {
        //FutureTask中的get方法会阻塞当前调用Callable子线程的线程，直到Callable中的call方法执行完毕，并获得call方法执行完成后的返回值。
        System.out.println("return value: " + f.get() + ", " + Thread.currentThread().getName());
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
//输出：
task is not done, wait...
callable call...Thread-0
callable ok
return value: 100, main
    
//通过线程池Future获取子线程返回值
public static void main(String[] args) {
    ExecutorService threadPool = Executors.newCachedThreadPool();
    Future<Object> f = threadPool.submit(new MyCallable());
    if (!f.isDone()) {
        System.out.println("task is not done, wait...");
    }
    try {
        System.out.println("return value; " + f.get() + ", " + Thread.currentThread().getName());
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    } finally {
        threadPool.shutdown();
    }
}
//输出：
task is not done, wait...
callable call...pool-1-thread-1
callable ok
return value; 100, main
```

### 3.6、sleep和wait的区别

- sleep可以在任何地方使用，wait只能在synchronized方法或者块中的使用（因为wait方法会释放锁资源，所以必须在synchronized代码块中有锁的情况下才有效）。

- sleep只会让出CPU，不会释放持有的资源锁；wait会让出CPU并且释放持有的资源锁。

```java
public static void main(String[] args) {
    final Object lock = new Object();
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("Thread1 try to get lock.");
            synchronized (lock) {
                System.out.println("Thread1 get lock.");
                //try {
                //    Thread.sleep(50);
                //} catch (InterruptedException e) {
                //    e.printStackTrace();
                //}
                try {
                    lock.wait(50);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread1 done.");
            }
        }
    }).start();
    try {
        Thread.sleep(20);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("Thread2 try to get lock.");
            synchronized (lock) {
                System.out.println("Thread2 get lock.");
                System.out.println("Thread2 done.");
            }
        }
    }).start();
}
//输出：
Thread1 try to get lock.
Thread1 get lock.
Thread2 try to get lock.
Thread2 get lock.
Thread2 done.//表明wait会释放锁并让出CPU，让Thread2获得锁并执行
Thread1 done.
//如果打开sleep注释，则输出如下：
Thread1 try to get lock.
Thread1 get lock.
Thread2 try to get lock.
Thread1 done.//表明sleep不会释放锁
Thread2 get lock.//Thread2必须在Threa1执行完成后，获得锁才可执行
Thread2 done.
```

### 3.7、notify和notifyAll区别

- notifyAll会让所有针对该锁并处于等待状态的线程全部去竞争获取锁的机会（唤醒后争夺锁失败的线程又会重新处于等待状态(wait)，等待下一次再被唤醒）。
- notify只会随机选取一个针对该锁并处于等待状态的线程去竞争获取锁的机会。

```java
public static void main(String[] args) {
    final Object lock = new Object();
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("Thread1 try to get lock.");
            synchronized (lock) {
                System.out.println("Thread1 get lock.");
                try {
                    lock.wait();//Thread1会释放锁并进入无限等待状态，直到被主动唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread1 done.");
            }
        }
    }).start();
    try {
        Thread.sleep(20);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("Thread2 try to get lock.");
            synchronized (lock) {
                System.out.println("Thread2 get lock.");
                System.out.println("Thread2 done.");
                lock.notify();//主动唤醒持有该锁的一个线程，此处会唤醒Thread1
            }
        }
    }).start();
}
//输出：
Thread1 try to get lock.
Thread1 get lock.
Thread2 try to get lock.
Thread2 get lock.
Thread2 done.
Thread1 done.//因为Thread2调用了lock.notify()，会唤醒持有该lock的一个线程，因为此处只有Thread1和Thread2持有该锁，所以Thread1会被唤醒。
//若调用lock.notifyAll()则会唤醒持有该锁的所有线程，被唤醒的所有线程会继续争夺锁资源。
```

### 3.8、理解Thread中的yield()方法

- yield()方法会告知线程调度器当前线程愿意让出CPU，但是线程调度器有可能会忽略此告知。
  比如：ThreadA调用yield之后，会让其他线程争取执行权（如ThreadB、ThreadC等），但是ThreadA自身同样可以参与争取执行权（如ThreadB和ThreadC都没有执行，ThreadA仍继续执行）。
- 注意：**yield()不会释放当前当前线程所持有的锁（即yield不会对锁的逻辑产生任何影响）**。

```java
public static void main(String[] args) {
    Runnable task = new Runnable() {
        @Override
        public void run() {
            for (int i = 1; i <= 5; i++) {
                System.out.print(Thread.currentThread().getName() + i + "  ");
                if (i == 3) {
                    Thread.currentThread().yield();
                }
            }
        }
    };

    Thread t1 = new Thread(task, "A");
    Thread t2 = new Thread(task, "B");
    t1.start();
    t2.start();
}
//输出：
//注意：每次输出结果可能随机，证明线程调度器可能会忽略某线程让出的CPU资源
A1  A2  B1  A3  A4  A5  B2  B3  B4  B5
```

### 3.9、理解Thread中的interrupt()方法以及线程如何停止

- interrupt()会发送给对应Thread一个中断信号，但对应线程并不会因为中断信号而停止运行，而是应该在检测到interrupt信号之后再自行决定处理逻辑。
- 停止线程不应该使用stop方法（因为一个线程在未正常结束之前被强行stop可能会造成线程持有的锁永久不能释放或者资源得不到回收，所以stop方法被废弃了），而应**使用标识位**或**interrupt中断信号**来判断并终止线程。

```java
static volatile boolean flag = false;//标识位
public static void main(String[] args) {
    Runnable task = new Runnable() {
        @Override
        public void run() {
            try {
                int count = 0;
                while (true) {
                    //调用Thread.interrupt()方法后，应在线程逻辑中主动检测isInterrupted状态，从而处理是否中断线程，Thread.interrupt()方法本身不会自动中断线程，其仅为一个信号量而已。
                    if (Thread.currentThread().isInterrupted() || flag) {
                        System.out.println(Thread.currentThread().getName() + " isInterrupted: " + Thread.currentThread().isInterrupted() + ", flag : " + flag);
                        break;
                    }
                    System.out.println(Thread.currentThread().getName() +" task : " + count++ + ", ThreadState : " + Thread.currentThread().getState());
                    //如果此逻辑块中存在wait、join或sleep方法，一定要捕获InterruptedException异常
                }
            } catch(InterruptedException e) {
                //此处建议捕获InterruptedException异常
                //因为：如果线程在逻辑代码中调用了wait、join或sleep方法，那么发送interrupt中断信号的时候（即对应try语句中正在执行wait、join或者sleep操作时（即线程阻塞时），接收到interrupt中断信号），会产生InterruptedException异常，所以在此获取此异常更安全。
            }
            System.out.println("task run end.");
        }
    };

    Thread t = new Thread(task);
    t.start();
    try {
        Thread.sleep(10);
        System.out.println("try to interrupt thread...");
        //t.interrupt();//通过interrupt信号量中断线程
        flag = true;//通过标识位中断线程
        //t.join();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("main end.");
}
//输出：
...
Thread-0 task : 387, ThreadState : RUNNABLE
Thread-0 task : 388, ThreadState : RUNNABLE
try to interrupt thread...
main end.
Thread-0 task : 389, ThreadState : RUNNABLE
Thread-0 isInterrupted: false, flag : true
task run end.//线程终止
//若使用t.join()，则main会等待Thread通过interrupt信号量或标识位终止线程后再执行main，输出会如下：
...
Thread-0 task : 272, ThreadState : RUNNABLE
Thread-0 task : 273, ThreadState : RUNNABLE
try to interrupt thread...
Thread-0 task : 274, ThreadState : RUNNABLE
Thread-0 isInterrupted: false, flag : true
task run end.
main end.//join等待执行效果
```

### 3.10、线程池

利用Executors类创建线程池。

**为什么要使用线程池？线程池具有哪些优点？**
(1) **降低资源消耗**：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
(2) **提高响应速度**：当任务到达时，任务可以不需要的等到线程创建就能立即执行。
(3) **提高线程的可管理性**：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

**线程池大小设定建议**

> CPU密集型：线程数=CPU核数+1
> I/O密集型：线程数=CPU核数 * ( 1+平均等待时间/平均工作时间 )

**线程池已满之后会发生什么？**

> ThreadPoolExecutor会丢弃任务，并抛出RejectedExecutionException异常。

**执行execute()方法和submit()方法的区别是什么呢？**

(1) execute()方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否。
(2) submit()方法用于提交需要返回值的任务。线程池会返回一个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执行成功。

> 通过 `Future` 的 `get()`方法来获取返回值，`get()`方法会阻塞当前线程直到任务完成，而使用 `get（long timeout，TimeUnit unit）`方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

### 3.11、线程安全/锁

#### 3.11.1、什么是线程安全

当某个变量/对象被多个线程访问操作时，仍能保证其数据正确有效，那么它就是线程安全的。

#### 3.11.2、类锁和对象锁

**synchronized**：同一时刻只允许一个线程访问某个对象。**synchronized分为对象锁和类锁**。
**对象锁**：当synchronized锁住当前实例或非静态方法和变量时，即为对象锁。
对象锁3种类型示例如下：

```java
private Object lock = new Object;
public void testLock1() {
    synchronized(lock) {//锁住非静态变量
        //do something
    }
}
public void testLock2() {
    synchronized(this) {//锁住当前实例
        //do something
    }
}
public synchronized void testLock3() {//锁住非静态方法
    //do something
}
```

*对象锁对于不同对象没有锁的约束。*

**类锁**：当synchronized锁住类的静态变量或静态方法及xxx.class时，即为类锁。
类锁3种类型示例如下：

```java
private static Object lock = new Object();
public void testLock1() {
    sychronized(lock) {//锁住静态变量，同一个静态变量无论在何处调用都一样
        //do something
    }
}
public static synchronized void testLock2() {//锁住静态方法
    //do something
}
public void testLock3() {
    synchronized (XXX.class) {//锁住xxx.class
        //do something
    }
}
```

*类锁对于同一个类生成的不同对象都具有约束作用。*

*<u>注意：类锁和对象锁互不干扰，互不影响。</u>*

#### 3.11.3、JAVA内存模型（JMM）

> Java虚拟机规范试图定义一种Java内存模型（JMM）,来屏蔽掉各种硬件和操作系统的内存访问差异，让Java程序在各种平台上都能达到一致的内存访问效果。**目的是解决由于多线程通过共享内存进行通信时，存在的原子性、可见性以及有序性问题。**
> 
> **原子性**：即一个操作或一系列是不可中断的。即使是在多个线程的情况下，操作一旦开始，就不会被其他线程干扰。比如一个线程执行变量a++的操作，一个线程执行变量a--的操作，那么执行a--操作的线程不允许打断a++线程的执行。
> 
> **可见性**：就是当一个线程对一个共享变量做了修改后，其他线程可以立即感知到该变量的改变。因为Java内存模型划分了工作内存和主内存，所以一个共享变量要在多个线程中可见必须经过工作内存-->主内存-->其他线程的工作内存这样的复制过程。
> 
> **有序性**：编译器和处理器在执行程序指令时，为了提高性能，可能会对执行进行重排（即变换程序指令的顺序）。
> 
> 在Java内存模型中，**JVM将内存组织为主内存和工作内存两个部分。**
> **主内存**主要包括本地方法区和堆。
> 每个线程都有一个**工作内存**，工作内存存储在CPU存储在高速缓存或者寄存器中，是主内存数据的拷贝。
> 工作内存是私有的，主内存是数据共享的。

#### 3.11.4、volatile关键字

> volatile保证此变量对所有线程的可见性，且可防止指令重排，但是并不保证原子性。
> 因为volatile不保证原子性，所以在多线程读写变量时，仍需要使用synchronized等锁来保证原子性。所以仅靠volatile关键字不能保证变量的线程安全。

#### 3.11.5、synchronized和ReentrantLock区别

> - synchronized是关键字，ReentrantLock是类；且ReentrantLock可以实现公平锁（即等待最久的线程可以优先获得锁），而synchronized不能实现公平锁。
> - synchronized代码块/方法执行完毕后会自动释放锁，而ReentrantLock需调用unlock()方法主动释放锁。
> - synchronized无法中断一个正在等候获得锁的线程，而ReentrantLock支持中断（ReentrantLock.lockInterruptibly()方法就支持响应中断），且性能较好。
> - ReentrantLock可以跨函数（因为ReentrantLock是类，可以构造对象实现在多个函数方法访问），synchronized不行（仅能针对某个方法或者代码段）。
> - 使用ReentrantLock结合Condition，可以实现一个锁多个条件信号，便于复杂的线程间同步通信。而synchronized/object方式就不行。示例如下：
>
> ```java
> import java.util.LinkedList;
> import java.util.concurrent.locks.Condition;
> import java.util.concurrent.locks.ReentrantLock;
> public class BlockingQueueDemo<E> {
>     int size;//阻塞队列最大容量
>     ReentrantLock lock = new ReentrantLock(true);
>     LinkedList<E> list = new LinkedList<>();//队列底层实现
>     Condition conditionFull = lock.newCondition();//队列满时的等待条件
>     Condition conditionEmpty = lock.newCondition();//队列空时的等待条件
> 
>     public BlockingQueueDemo(int size) {//构造函数，初始化队列大小
>         this.size = size;
>     }
> 
>     public void enqueue(E e) throws InterruptedException {//入队
>         lock.lock();
>         try {
>             while (list.size() == size) {//队列已满,在conditionFull条件上等待
>                 conditionFull.await();
>             }
>             list.add(e);//入队:加入链表末尾
>             System.out.println("入队：" + e);
>             conditionEmpty.signal(); //通知在conditionEmpty条件上等待的线程
>         } finally {
>             lock.unlock();
>         }
>     }
> 
>     public E dequeue() throws InterruptedException {//出队
>         E e;
>         lock.lock();
>         try {
>             while (list.size() == 0) {//队列为空,在conditionEmpty条件上等待
>                 conditionEmpty.await();
>             }
>             e = list.removeFirst();//出队:移除链表首元素
>             System.out.println("出队：" + e);
>             conditionFull.signal();//通知在conditionFull条件上等待的线程
>             return e;
>         } finally {
>             lock.unlock();
>         }
>     }
> 
>     public static void main(String[] args) {
>         final BlockingQueueDemo<Integer> queue = new BlockingQueueDemo<>(2);//限定队列大小为2
>         for (int i = 0; i < 5; i++) {
>             final int data = i;
>             new Thread(new Runnable() {
>                 @Override
>                 public void run() {
>                     try {
>                         queue.enqueue(data);
>                     } catch (InterruptedException e) {
>                         e.printStackTrace();
>                     }
>                 }
>             }).start();
>         }
>         for (int i = 0; i < 10; i++) {
>             new Thread(new Runnable() {
>                 @Override
>                 public void run() {
>                     try {
>                         Integer data = queue.dequeue();
>                     } catch (InterruptedException e) {
>                         e.printStackTrace();
>                     }
>                 }
>             }).start();
>         }
>     }
> }
> //输出：
> 入队：4
> 入队：0
> 出队：4
> 入队：2
> 出队：0
> 出队：2
> 入队：1
> 入队：3
> 出队：1
> 出队：3
> //由结果看出队列满和空的条件信号会交替执行
> ```
>
> Condition.await()方法：对应Object.wait()方法，会释放锁，让当前线程等待，支持唤醒，支持线程中断。
> Condition.signal()方法：对应Object.notify()方法，会唤醒一个等待中的线程。
> Condition.sinnalAll()方法：对应Object.notifyAll()方法，会唤醒所有等待中的线程。
>
> *ReentrantLock.lock和lockInterruptibly区别：lock方法优先考虑获取锁，待锁获取成功后，才响应中断，而lockInterruptibly则相反。*
>
> 
>
> **扩展：如何理解synchronized实现原理？**
>
> *对象同步时（如：synchronized(this){}或synchronized(object){}），如下所述：*
> 1）每个对象有一个监视器锁（monitor）。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权。
> 2）指令执行时，monitor的进入数减1，如果减1后进入数为0，那线程退出monitor，不再是这个monitor的所有者。其他被这个monitor阻塞的线程可以尝试去获取这个 monitor 的所有权。 
>
> *方法同步时（如：synchronized void method(){}），如下所述：*
> 方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 
> *<small>详情可参考：https://www.cnblogs.com/paddix/p/5367116.html</small>*

#### 3.11.6、什么是锁的重入（可重入锁）

> 当一个线程再次请求自己持有的锁，这种情况就是锁的重入。如下所示：
>
> ```java
> private Object lock = new Object();
> synchronized (lock) {
> 	synchronized (lock) {//此处再次请求自己所持有的锁，即为重入
> 		System.out.println("重入");
> 			//do something
> 		}
> 	//do something
> }
> //synchronized、ReentrantLock等都是可重入锁
> ```
> ***可重入锁可以避免死锁***

#### 3.11.7、悲观锁和乐观锁

> **悲观锁**：始终认为会发生并发冲突，因此在一个线程处理数据时，会令其他线程阻塞以避免访问数据，如synchronized就是一种悲观锁。
> **乐观锁**：始终认为不会发生并发问题，因此在访问数据时，不会加锁，只是在执行更新的时候判断一下在此期间别人是否修改了数据。乐观锁一般通过**版本号version**和**CAS（Compare and Swap）方式**实现。例如java.util.concurrent (JUC) 包中的AtomicInteger就是一种以CAS方式实现的乐观锁。
>
> **扩展1：如何理解CAS？**
> CAS 是乐观锁的一种实现方式，是一种轻量级锁。
> CAS 操作的流程：线程在读取数据时不进行加锁，在准备写回数据时，比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程。如下图所示：
>
> **扩展2：什么是ABA问题？如何解决ABA问题？**
>有一个变量值为A，来了一个线程把值改成了B，又来了一个线程把值又改回了A，此时线程来判断变量的值发现还是A，所以就不知道这个值到底有没有被人改过。
> 解决办法：
> 1）使用版本号：在修改变量前去查询他原来的值的时候再带一个版本号，每次判断就连值和版本号一起判断，判断成功就给版本号加1。
> 2）使用时间戳：类似于上述的版本号，只不过附加的数据是时间戳。

#### 3.11.8、ThreadLocal是什么

ThreadLocal是线程局部变量，即实现**相同线程数据共享，不同的线程数据隔离**，和普通变量的不同在于：每个线程持有这个变量的一个副本，可以独立修改（set方法）和访问（get方法）这个变量，并且线程之间不会发生冲突。

```java
public static void main(String[] args) throws InterruptedException {
	final ThreadLocal<String> threadLocal = new ThreadLocal<>();
  	//此处是在主线程中set值，所以只能主线程访问到
    threadLocal.set("main set : " + Thread.currentThread().getName());
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
           		//此处是在Thread-0子线程中set值，所以只能此子线程访问到
                threadLocal.set("subThread get : " + Thread.currentThread().getName());
                System.out.println(threadLocal.get());
            } finally {
                //注意：ThreadLocal可能发生内存泄漏
                //所以在相关逻辑处理完成后，应主动释放内存
                threadLocal.remove();
            }
         }
     });
     Thread t2 = new Thread(new Runnable() {
         @Override
         public void run() {
             //此处子线程没有set值，所以获取不到任何值，为null
             System.out.println("t2 get : " + threadLocal.get());
         }
     });
     t.start();
     t.join();
     t2.start();
     System.out.println("main get : " + threadLocal.get());
}
//输出：
subThread get : Thread-0
main get : main set : main
t2 get : null//t2线程中没有set局部变量，所以null
//不同线程获取到不同的值
```

**扩展：ThreadLocal原理？**

每个Thread内部维护着一个ThreadLocalMap，它是一个Map。这个映射表的Key是一个弱引用，其实就是ThreadLocal本身，Value是真正存的线程变量Object。也就是说ThreadLocal本身并不真正存储线程的变量值，它只是一个工具，用来维护Thread内部的Map，帮助存和取。注意上图的虚线，它代表一个弱引用类型，而弱引用的生命周期只能存活到下次GC前。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_threadlocal.jpg?raw=true" alt="java_threadlocal.jpg" style="zoom:80%;" />

ThreadLocal在ThreadLocalMap中是以一个弱引用身份被Entry中的Key引用的，因此如果ThreadLocal没有外部强引用来引用它，那么ThreadLocal会在下次JVM垃圾收集时被回收。这个时候就会出现Entry中Key已经被回收，出现一个null Key的情况，外部读取ThreadLocalMap中的元素是无法通过null Key来找到Value的。因此如果当前线程的生命周期很长，一直存在，那么其内部的ThreadLocalMap对象也一直生存下来，这些null key就存在一条强引用链的关系一直存在：Thread --> ThreadLocalMap-->Entry-->Value，这条强引用链会导致Entry不会回收，Value也不会回收，但Entry中的Key却已经被回收的情况，造成内存泄漏。**如何避免内存泄漏：每次使用完ThreadLocal，都调用它的remove()方法，清除数据。**

作者：Misout
链接：https://www.jianshu.com/p/a1cd61fa22da
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

ThredLocal中维护了一个ThreadLocalMap，在get和set时，将当前线程的实例标识作为key，从而不同线程取得不同的值。

```java
//ThreadLocal源码get方法
//调用ThreadLocal.get方法时，实际上是从当前线程中获取ThreadLocalMap<ThreadLocal, Object>，然后根据当前ThreadLocal获取当前线程共享变量Object。
//ThreadLocal.set，ThreadLocal.remove实际上是同样的道理。
public T get() {
    Thread t = Thread.currentThread();//当前线程实例
    ThreadLocalMap map = getMap(t);//从map中区的Entry对象
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;//取值并返回
            return result;
        }
    }
    return setInitialValue();
}
//ThreadLocal源码set方法
public void set(T value) {
    Thread t = Thread.currentThread();//当前线程实例
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

*<small>更多ThreadLocal问题参考：https://www.jianshu.com/p/a1cd61fa22da</small>*

#### 3.11.9、实现一个线程安全的单例模式（Double-Check，双检查模式）

```java
public class Singleton {
    // 将自身实例化对象设置为一个属性，并用static修饰
    private static volatile Singleton instance;//volatile保证所有线程可见性
    // 构造方法私有化
    private Singleton() {}
    // 静态方法返回该实例
    public static Singleton getInstance() {
        // 第一次检查instance是否被实例化出来，如果没有进入if块
        if(instance == null) {
            synchronized (Singleton.class) {
                // 某个线程取得了类锁，实例化对象前第二次检查instance是否已经被实例化出来，如果没有，才最终实例出对象
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
//因为做了两次instance == null检查，所以又被称为双检查模式
//此种双检查模式实现的单利模式能实现线程安全，避免多线程时可能创建重复的实例对象
```

#### 3.11.10、如何判断一个对象当前是否持有某个锁？

> Thread.holdsLock(Object)方法可以判断**当前线程是否持有某个对象的锁**。

### 3.12、4种同步器

#### 3.12.1、CountDownLatch（闭锁）

CountDownLatch（闭锁）：让主线程等待一组事件发生后继续执行。例如一个任务A需要等待其他2个任务完成后才能执行，此时可用闭锁。

```java
public static void main(String[] args) {
    final CountDownLatch latch = new CountDownLatch(2);
    new Thread() {
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " run.");
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " end.");
                latch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }.start();
    new Thread() {
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " run.");
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + " end.");
                latch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }.start();

    try {
        //等待2个子线程执行完毕...
        latch.await();//也可使用超时判断机制，如：latch.await(1, TimeUnit.SECONDS);
        //2个子线程已经执行完毕"
        System.out.println("main end.");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
//输出：
Thread-0 run.
Thread-1 run.
Thread-0 end.
Thread-1 end.
main end.//此处main线程等待2个子线程结束后才执行
```

#### 3.12.2、CyclicBarrier（栅栏）

CyclicBarrier（栅栏）：阻塞当前线程，等他其他线程，直到最后一个线程执行结束才执行。

```java
public static void main(String[] args) {
    //累计3个线程执行完成后才执行CyclicBarrier中的方法
    final CyclicBarrier cyclicBarrier = new CyclicBarrier(3, new Runnable() {
        @Override
        public void run() {
            System.out.println("3 thread run end.");
        }
    });
    for(int i = 1; i <= 3; i++) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName() + " run.");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    System.out.println("main end.");
}
//输出：
main end.
Thread-0 run.
Thread-1 run.
Thread-2 run.
3 thread run end.//等待限定的3个其他线程完成后才执行
```

#### 3.12.3、Semaphore（信号量）

Semaphore（信号量）：控制某个资源可同时被多少个线程访问。

```java
public static void main(String[] args) {
    final Semaphore semaphore = new Semaphore(2);//限定同时两个线程访问

    for(int i = 1; i <= 3; i++){
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + " run.");
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " end.");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            }
        }).start();
    }
}
//输出：
Thread-1 run.
Thread-0 run.
Thread-0 end.
Thread-1 end.
Thread-2 run.//因为使用信号量限定了2个线程访问，所以线程2只能在线程0或1释放后才能执行
Thread-2 end.
```

#### 3.12.4、Exchanger（交换器）

Exchanger（交换器）：两个线程到达同步点后，相互交换数据。

```java
private static final Exchanger<String> exchanger = new Exchanger<String>();
public static void main(String[] args) {
    //设定ThreadA和ThreadB两个线程
    Thread t1 = new Thread(task("StringA"), "ThreadA");
    Thread t2 = new Thread(task("StringB"), "ThreadB");
    t1.start();
    t2.start();
}
private static Runnable task(final String str) {
    return new Runnable() {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " run. " + new Date().toLocaleString());
                if (Thread.currentThread().getName().equals("ThreadA")) {
                    TimeUnit.SECONDS.sleep(2);//设定A线程睡眠2秒
                } else {
                    TimeUnit.SECONDS.sleep(5);//设定B线程睡眠5秒
                }
                String str2 = exchanger.exchange(str);//exchange方法即同步点
                System.out.println(Thread.currentThread().getName() + " get " + str2 + " " + new Date().toLocaleString());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
}
//输出：
ThreadA run. 2020-4-16 16:46:01
ThreadB run. 2020-4-16 16:46:01
ThreadA get StringB 2020-4-16 16:46:06//ThreadA实际上睡眠了5秒，并交换了String
ThreadB get StringA 2020-4-16 16:46:06//ThreadB也交换了String
//根据结果可见，exchanger会在exchange方法时阻塞相关线程，等待两个交换数据完成后再继续执行线程。
```

*CountDownLatch和CyclicBarrier其实是相反的操作，CountDownLatch是递减，CyclicBarrier是递增。*

### 3.13、理解AQS

**什么是AQS？**

AQS，全程AbstractQueuedSynchronizer，位于java.util.concurrent.locks包中，是一个用来构建锁和同步器的框架，如ReentrantLock、Semaphore、FutureTask等都是基于AQS实现的。

**AQS的原理**

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。AQS使用一个voliate int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。AQS使用CAS对该同步状态进行原子操作实现对其值的修改。

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_aqs.png?raw=true" alt="java_aqs.png" style="zoom:67%;" />

AQS定义了两种资源获取方式：
独占：只有一个线程能访问执行，又根据是否按队列的顺序分为公平锁和非公平锁，如ReentrantLock） 
共享：多个线程可同时访问执行，如Semaphore/CountDownLatch，Semaphore、CountDownLatCh、 CyclicBarrier ）。*ReentrantReadWriteLock 可以看成是组合式，允许多个线程同时对某一资源进行读。*

## 4、I/O相关

### 4.1、IO的分类

根据数据的流向分为：**输入流**和**输出流**。
**输入流** ：把数据从`其他设备`上读取到`内存`中的流。
**输出流** ：把数据从`内存` 中写出到`其他设备`上的流。

根据数据的类型分为：**字节流**和**字符流**。
**字节流**：以字节为单位，读写数据的流，按照8位传输。字节流的类通常以Stream结尾。
**字符流**：以字符为单位，读写数据的流，按照16位传输。字符流的类通常以reader和writer结尾。

### 4.2、BIO、NIO和AIO区别

BIO：同步阻塞IO，服务器实现模式为**一个连接对应一个线程**，即客户端有连接请求时服务器端就需要启动一个线程进行处理，所以如果这个连接不做任何事情会造成不必要的线程开销。
NIO：非阻塞同步IO，核心由Channel、Buffer和Selector组成。服务器实现为**一个请求对应一个线程**，即客户端连接的请求都会注册到多路复用器上，多路复用器轮询到连接有IO请求时才启动一个线程进行处理。
AIO：非阻塞异步IO，基于事件和回调机制。服务器实现模式为**一个有效请求对应一个线程**，客户端的IO请求都是由OS（内核）先完成，再通知到服务器应用去启动线程进行处理。

### 4.3、理解NIO中的零拷贝

在NIO中，把数据从从一个Channel拷贝到另一个Channel时，避免了两次用户态和内核态的上下文切换，即”零拷贝“，效率较高。

> 扩展：什么是用户态和内核态？
> **内核态**：cpu可以访问内存的所有数据，包括外围设备，例如硬盘，网卡，cpu也可以将自己从一个程序切换到另一个程序。
> **用户态**：只能受限的访问内存，且不允许访问外围设备，占用cpu的能力被剥夺，cpu资源可以被其他程序获取。
>
> 为什么要有用户态和内核态？
> 由于需要限制不同的程序之间的访问能力, 防止他们获取别的程序的内存数据, 或者获取外围设备的数据, 并发送到网络, CPU划分出两个权限等级 -- 用户态和内核态。
>
> 用户态和内核态的切换？
> 所有用户程序都是运行在用户态的, 但是有时候程序确实需要做一些内核态的事情, 例如从硬盘读取数据, 或者从键盘获取输入等. 而唯一可以做这些事情的就是操作系统, 所以此时程序就需要先操作系统请求以程序的名义来执行这些操作。

### 4.4、理解NIO中的MappedByteBuffer

MappedByteBuffer是NIO引入的文件内存映射方案，读写性能极高（特别是大文件）。
MappedByteBuffer使用虚拟内存，因此不受JVM内存大小限制（所以读取大文件也不会产生内存溢出）。

```java
//MappedByteBuffer读取文件示例如下：

File file = new File(filePath);
FileInputStream in = new FileInputStream(file);
MappedByteBuffer byteBuffer = in.getChannel().map(FileChannel.MapMode.READ_ONLY, 0, file.length());
//使用byteBuffer do something
//....
//最后记得手动unmap，否则会导致文件被占用
try {
    Method m = FileChannelImpl.class.getDeclaredMethod("unmap",
                                                       MappedByteBuffer.class);
    m.setAccessible(true);
    m.invoke(FileChannelImpl.class, byteBuffer);
} catch (Exception e) {
}
byteBuffer.clear();//文件读取使用完成后，清理缓存
byteBuffer = null;//reset变量
```

> 扩展1：什么是内存映射？
> 内存映射mmap() 是操作系统中一种内存映射的方法。内存映射简单的讲就是将用户空间的一块内存区域映射到内核空间。映射关系建立后，用户对这块内存区域的修改可以直接反应到内核空间；反之内核空间对这段区域的修改也能直接反应到用户空间。内存映射能减少数据拷贝次数，实现用户空间和内核空间的高效互动。两个空间各自的修改能直接反映在映射的内存区域，从而被对方空间及时感知
>
> 扩展2：为什么MappedByteBuffer性能高？
> 传统读取文件方法：文件从硬盘拷贝到内核空间的一个缓冲区，再将这些数据拷贝到用户空间，发生了两次拷贝。
> MappedByteBuffer方法：直接将文件从硬盘拷贝到用户空间，只进行了一次数据拷贝。

**MappedByteBuffer缺陷？**

> 映射的字节缓冲区(MappedByteBuffer ) 不提供关闭或销毁方法。也就是说，创建完直接缓冲区，就一直有效，直到缓冲区本身被垃圾收集。示例如下：
>
> ```java
> //MappedByteBuffer无法自动释放内存，导致文件被占用
> //手动unmap
> try {
>    	Method m = FileChannelImpl.class.getDeclaredMethod("unmap", MappedByteBuffer.class);
>    	m.setAccessible(true);
>    	m.invoke(FileChannelImpl.class, byteBuffer);
> } catch (Exception e) {
>    	e.printStackTrace();
> }
> ```

## 5、数据结构

*此章节列举常见的几种数据结构。*
数据结构是一种存储和管理数据的方式，有两种类型的数据结构：线性和非线性数据结构。
**线性数据结构**：是一个有序数据元素的集合。各数据元素之间的关系是一对一的关系，即除了第一个和最后一个数据元素之外，其它数据元素都是首尾相接的。如：栈、队列和链表等。

> 线性数据结构特点：结构简单，易于使用，内存利用率较低。

**非线性数据结构**：各数据元素不在一个线性序列中。各数据元素可能与零个或者多个其他数据元素产生联系。如：树、多维数组和图等。

> 非线性数据结特点：结构复杂，内存利用率高。

### 5.1、数组

数组是可以在内存中**连续存储多个元素的结构**，在**内存中的分配是连续的**，通过下标访问数组元素。
优点：简单，元素查找速度快（通过下标查找）。
缺点：数组大小固定后无法扩容；只能存储一种类型数据；添加删除元素效率低（因为要移动其他元素）。
*<small>数组是使用最广泛的数据结构，栈、队列等其他数据结构均由数组演变而来。</small>*

### 5.2、栈

栈是一种线性数据结构，特点是：先进后出，或者说是后进先出（LIFO）。
*<small>程序中常用的撤销（Ctrl+Z）操作基本都是用栈实现的。</small>*

### 5.3、队列

队列也是一种线性数据结构，特点是：先进先出（FIFO）。

### 5.4、链表

链表也是一种线性数据结构，链表中包含N个节点，其中每个节点包含对应数据和指向后续节点的指针。但链表在物理存储上是可以是非连续的。
优点：不需要初始化容量，可以任意加减元素；添加或者删除元素时只需要改变前后两个元素结点的指针域指向地址即可，效率高。
缺点：含有大量的指针域，占用空间较大；查找元素时需要遍历链表来查找，耗时。

### 5.5、树

树是一种层次化的数据结构（非线性数据结构），由n(n>=0)个节点组成的层次关系集合。

**二叉树（Binary Tree）**：具有唯一根节点，每个节点最多支持2个子节点（左孩子、右孩子；左右孩子都为空的节点称为叶子节点；左孩子对应的树称为左子树，右孩子对应的树称为右子树）。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/btree.png?raw=true" alt="btree.png" style="zoom:50%;" />

**二分搜索树（Binary Search Tree）**：或称为二叉查找树，二分搜索树属于二叉树；每个节点大于其作子树所有节点的值，且小于其右子树所有节点的值（即二分搜索树的子节点都为二分搜索树）。*存储的元素必须具有可比性。*

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/bstree.png?raw=true" alt="bstree.png" style="zoom:50%;" />

二叉树的遍历分为**深度优先遍历**和**广度优先遍历**。

深度优先遍历（DFS）包含先序、中序和后序三种方式：


- **先序遍历** -> 先打印中间节点，再依次打印左节点和右节点。
- **中序遍历** -> 先打印左节点，再打印中间节点，最后打印右节点。
- **后序遍历** -> 先依次打印左节点和右节点，再打印中间节点。

广度优先遍历（BFS）：又可叫做层序遍历。
*树的深度优先算法非递归实现一般采用栈实现，而广度优先算法非递归实现一般采用队列实现。*
*<small>BFS占用内存较大，必须遍历所有分支；DFS占用内存较少，**DFS查找所需的最大次数等同于二分搜索树的最大高度**；一般情况下DFS是最优方案。</small>*

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Stack;

/**
 * 实现二分搜索树
 */
public class BinarySearchTree<E extends Comparable<E>> {
    private class Node {//定义一个二叉树节点
        public E e;
        public Node left, right;//左孩子和右孩子

        public Node(E e) {
            this.e = e;
            this.left = null;
            this.right = null;
        }
    }
    private Node root;//根节点
    private int size;//树的大小

    public BinarySearchTree() {
        this.root = null;
        this.size = 0;
    }

    //获取树的大小
    public int getSize() {
        return size;
    }

    //树是否为空
    public boolean isEmpty() {
        return size == 0;
    }

    //向树中添加一个新元素
    public void add(E e) {
        root = add(root, e);
    }

    //递归算法：向以node为根节点的二分搜索树中插入元素E
    //返回新插入节点后的二分搜索树的根节点
    private Node add(Node node, E e) {
        if (node == null) {
            size++;
            return new Node(e);
        }
        if (e.compareTo(node.e) < 0) {
            //比节点的左孩子小，那么将元素e插入到node节点的左子树
            node.left = add(node.left, e);
        } else if (e.compareTo(node.e) > 0) {
            //那么将元素e插入到node节点的右子树
            node.right = add(node.right, e);
        } else {
            //两个元素相等，重复，忽略
        }
        return node;
    }

    //删除树中最小的元素，位于树的左下角
    //返回：被删除的最小元素
    public E removeMin() {
        return removeMinAndReturnMin(root).e;
    }

    //循环查找树的最小值，并删除
    //返回：被删除的树的最小值
    private Node removeMinAndReturnMin(Node node) {
        if (node == null) {
            throw new IllegalArgumentException("树为空");
        }
        Node minNodeLeft = null;//中间变量，暂存最小节点的父节点
        while (node.left != null) {
            minNodeLeft = node;
            node = node.left;
        }
        System.out.println(minNodeLeft.e);
        Node minNode = node;//中间变量，树的最小节点
        System.out.println(minNode.e);
        node = null;
        if (minNode.right != null) {
            //最小节点的右孩子存在，那么其父节点的左孩子直接指向最小节点的右孩子
            minNodeLeft.left = minNode.right;
        } else {
            //最小节点右孩子不存在，直接删除最小节点（即最小节点的父节点左孩子被删除）
            minNodeLeft.left = null;
        }
        size--;
        return minNode;
    }

    //递归：删除最小元素
    //返回树的根节点
    private Node removeMin(Node node) {
        if (node == null) {
            throw new IllegalArgumentException("树为空");
        }
        if (node.left == null) {
            Node rightNode = node.right;
            node.right = null;
            size--;
            return rightNode;
        } else {
            node.left = removeMin(node.left);
            return node;
        }
    }

    //移除树中最大元素
    public E removeMax() {
        return removeMax(root).e;
    }

    //递归实现移除树中最大元素
    //返回根节点
    private Node removeMax(Node node) {
        //如果右节点为空，其子节点作为根节点
        if (node.right == null) {
            Node leftNode = node.left;
            node.left = null;
            size--;
            return leftNode;
        }
        node.right = removeMax(node.right);
        return node;
    }

    //查找树中最大元素
    public E maximum() {
        if (size == 0) {
            throw new IllegalArgumentException("BST is empty!");
        }
        return maximum(root).e;
    }

    //递归查找树中最大元素
    private Node maximum(Node node) {
        if (node.right == null) {
            return node;
        }
        return maximum(node.right);
    }

    //查找树中最小元素
    public E minimum() {
        if (size == 0) {
            throw new IllegalArgumentException("树为空");
        }
        return minimum(root).e;
    }

    //递归查找树中最小元素
    private Node minimum(Node node) {
        if (node.left == null) {
            return node;
        }
        return minimum(node.left);
    }

    //移除一个元素
    public void remove(E e) {
        remove(root, e);
    }

    //递归：移除一个元素
    //返回：被移除的元素
    private Node remove(Node node, E e) {
        if (node == null) {
            return null;
        }
        if (e.compareTo(node.e) < 0) {
            node.left = remove(node.left, e);
            return node;
        } else if (e.compareTo(node.e) > 0) {
            node.right = remove(node.right, e);
            return node;
        } else {  //e.equal(node.e)
            //待删除节点左节点均为空的情况，即待删除节点只有右孩子
            if (node.left == null) {
                Node rightNode = node.right;
                node.right = null;
                size--;
                return rightNode;
            }
            //待删除节点右节点均为空的情况，即待删除节点只有左孩子
            if (node.right == null) {
                Node leftNode = node.left;
                node.left = null;
                size--;
                return leftNode;
            }

            //待删除节点左右节点均不为空的情况
            //找到比待删除节点大的最小节点，即待删除节点右子树的最小节点
            //用这个节点顶替删除节点的位置
            Node successor = minimum(node.right);
            successor.right = removeMin(node.right);
            successor.left = node.left;
            node.left = node.right = null;
            return successor;
        }
    }

    //判断树中是否存在某个元素
    public boolean contains(E e) {
        return contains(root, e);
    }

    //递归实现：判断树中是否存在某个元素
    private boolean contains(Node node, E e) {
        if (node == null) {
            return false;
        }
        if (e.equals(node.e)) {
            return true;//存在
        } else if (e.compareTo(node.e) < 0) {
            return contains(node.left, e);//元素比节点小，那么和此节点的左孩子比较
        } else {
            return contains(node.right, e);
        }
    }

    //前序遍历二分搜索树
    public void preOrder() {
        preOrder(root);//从根节点开始遍历
    }

    //递归算法：前序遍历
    private void preOrder(Node node) {
        if (node == null) {
            return;//节点为空
        }
        //前序遍历：先打印中间节点，再打印左节点和右节点
        System.out.print(node.e + "  ");
        preOrder(node.left);
        preOrder(node.right);
    }

    //非递归实现前序遍历
    //利用栈实现
    //非递归实现树的遍历较为复杂
    public void preOrderNR() {
        Stack<Node> stack = new Stack<>();
        stack.push(root);
        while (!stack.isEmpty()) {
            Node cur = stack.pop();
            System.out.print(cur.e + "  ");
            if (cur.right != null) {
                stack.push(cur.right);
            }
            if (cur.left != null) {
                stack.push(cur.left);
            }
        }
    }

    //中序遍历二分搜索树
    public void inOrder() {
        inOrder(root);//从根节点开始遍历
    }

    //递归算法：中序遍历
    private void inOrder(Node node) {
        if (node == null) {
            return;//节点为空
        }
        //中序遍历：先打印左节点，再打印中间节点，最后打印右节点
        inOrder(node.left);
        System.out.print(node.e + "  ");
        inOrder(node.right);
    }

    //后序遍历二分搜索树
    public void postOrder() {
        postOrder(root);//从根节点开始遍历
    }

    //递归算法：后序遍历
    private void postOrder(Node node) {
        if (node == null) {
            return;//节点为空
        }
        //后序遍历：先打印左节点和右节点，再打印中间节点
        postOrder(node.left);
        postOrder(node.right);
        System.out.print(node.e + "  ");
    }

    // 二分搜索树的层序遍历（广度优先遍历）
    public void levelOrder() {
        if (root == null) {
            return;
        }
        Queue<Node> q = new LinkedList<>();//借助队列实现
        q.add(root);
        while (!q.isEmpty()) {
            Node currentNode = q.remove();
            System.out.print(currentNode.e + "  ");
            if (currentNode.left != null) {
                q.add(currentNode.left);//当前节点的左节点存在，借助队列先进先出，下次循环先读取对应左节点
            }
            if (currentNode.right != null) {
                q.add(currentNode.right);//当前节点的右节点
            }
        }
    }

    //获取树的层级（高度）
    public int getTreeLevel() {
        return getTreeLevel(root);
    }

    //递归实现计算树的层级
    private int getTreeLevel(Node root) {
        if (root == null) {
            return 0;
        } else {
            int l = getTreeLevel(root.left);
            int r = getTreeLevel(root.right);
            return l > r ? l + 1 : r + 1;
        }
    }

    public static void main(String[] args) {
        BinarySearchTree<Integer> tree = new BinarySearchTree<>();
        int[] nums = {28, 16, 30, 13, 22, 29, 42};
        for (int num : nums) {
            tree.add(num);
        }
        System.out.println("树的节点个数：" + tree.getSize());
        System.out.print("前序遍历：");
        tree.preOrder();
        System.out.println("\n非递归实现前序遍历：");
        tree.preOrderNR();
        System.out.print("\n中序遍历：");
        tree.inOrder();
        System.out.print("\n后序遍历：");
        tree.postOrder();
        System.out.print("\n层序遍历（广度优先遍历）：");
        tree.levelOrder();
        System.out.println("\n树中是否存在元素：" + 30 + " ，" + tree.contains(30));

        System.out.println("\n树中最大元素：" + tree.maximum() + "，树中最小元素：" + tree.minimum());
//        System.out.print("\n删除最小元素：\"" + tree.removeMin() + "\" 后，层序遍历（广度优先遍历）：");
//        tree.levelOrder();
//        System.out.print("\n删除最大元素：\"" + tree.removeMax() + "\" 后，层序遍历（广度优先遍历）：");
//        tree.levelOrder();

//        System.out.println("\n删除一个元素：\"13\" 后，层序遍历（广度优先遍历）：");
//        tree.remove(13);
//        tree.levelOrder();

        System.out.println("\n树的层级（高度）：" + tree.getTreeLevel());
    }
}
//输出：
树的节点个数：7
前序遍历：28  16  13  22  30  29  42  
非递归实现前序遍历：
28  16  13  22  30  29  42  
中序遍历：13  16  22  28  29  30  42  
后序遍历：13  22  16  29  42  30  28  
层序遍历（广度优先遍历）：28  16  30  13  22  29  42  
树中是否存在元素：30 ，true
树中最大元素：42，树中最小元素：13
树的层级（高度）：3
```

**满二叉树**：除了叶子节点，其他所有的节点都存在左孩子和右孩子两个节点。

**完全二叉树**：完全二叉树从根结点到倒数第二层满足完美二叉树，最后一层可以不完全填充，其叶子结点都靠左对齐。（即树的空缺部分一定在树的右下角部分）

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/full_btree.png?raw=true" alt="full_btree.png" style="zoom:80%;" />

**平衡二叉树**（AVL树）：具有二分搜索树的全部特性，且对于任意一个节点，其左子树和右子树的高度差不能超过1。因此又被称为**高度平衡树**。
*<small>**平衡二叉树是为了解决二叉查找树退化为链表的情况。**</small>*

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/balance_btree.png?raw=true" alt="balance_btree.png" style="zoom:50%;" />


> **平衡因子**：树中平衡因子大于1，即表示不满足平衡二叉树的特点了。
>
> **叶子节点：平衡因子为0，其他节点的平衡因子即为对应节点的左子树高度减去右子树高度差。**
> 如下图所示：黑色的数字表示节点的高度，蓝色数字表示节点的平衡因子。
>
> <center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/balance_factor.png?raw=true" alt="balance_factor.png" style="zoom:50%;" />

**2-3树**：2-3树由二节点和三节点构成绝对平衡的树。2-3树每次添加元素不会直接添加，而是进行节点融合，在融合之后，根据情况，进行分开融合等操作，将树转化为一个绝对平衡的2-3树。示例如下：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/a23tree.png?raw=true" alt="a23tree.png" style="zoom:50%;" />

**红黑树**（RB Tree）：是一种自平衡的二叉树（非绝对平衡的二叉树）。自平衡就是指在插入和删除的过程中，红黑树会采取一定的策略对树的组织形式进行调整，以尽可能的减少树的高度，从而节省查找的时间。红黑树特性如下：

- 具有二分搜索树的全部特性。
- 根节点是黑色的。
- 每个叶子节点（最后的空节点）都是黑色的。
- 如果一个节点是红色的，那么他的孩子节点都是黑色的。
- 从任意一个节点到叶子节点，其经过的黑色节点个数是一样的（即黑节点平衡）。

*<small>**红黑树是为了解决平衡树在插入、删除等操作需要频繁调整的情况。**</small>*
*<small>**红黑树的红色节点表示在2-3模式下理解时，可以与父节点融合。**</small>*

红黑树的平衡策略可以简单概括为三种： **左旋转** 、 **右旋转** ，以及 **变色** 。
左旋转：对于当前结点而言，如果右子结点为红色，左子结点为黑色，则执行左旋转。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rbtree_turn_left.png?raw=true" alt="rbtree_turn_left.png" style="zoom:50%;" />


右旋转：对于当前结点而言，如果左子、左孙子结点均为红色，则执行右旋转。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rbtree_turn_right.png?raw=true" alt="rbtree_turn_right.png" style="zoom:50%;" />


变色：对于当前结点而言，如果左、右子结点均为红色，则执行变色。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rbtree_change_color.png?raw=true" alt="rbtree_change_color.png" style="zoom:50%;" />


红黑树等价于2-3树。示例如下：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rb_23_tree1.png?raw=true" alt="rb_23_tree1.png" style="zoom:50%;" />

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rb_23_tree2.png?raw=true" alt="rb_23_tree2.png" style="zoom:50%;" />

> 平衡二叉树和红黑树比较？
>
> 在数据结构频繁变动的情况下，红黑树因为不用保持绝对的平衡，所以性能会高一些；
> 在数据不变的情况下，平衡二叉树的深度更小，所以查找效率实际更高一些。
>
> 平衡二叉树和红黑树的时间复杂度都是O(logN)

**B树**：B树也叫或B-树、B_树。B树不是二叉树，是多叉树。B树的高度远远小于AVL树和红黑树(B树是一颗“矮胖子”)，磁盘IO次数大大减少。
>B树（B-tree，B-树就是B树）是一种树状数据结构，它能够存储数据、对其进行排序并允许以O(log n)的时间复杂度运行进行查找、顺序读取、插入和删除的数据结构。B树，概括来说是一个节点可以拥有多于2个子节点的二叉查找树。与自平衡二叉查找树不同，B-树为系统最优化大块数据的读和写操作。B-tree算法减少定位记录时所经历的中间过程，从而加快存取速度。普遍运用在数据库和文件系统。

**B+树**：在B树的基础上，将非叶节点改造为不存储数据的纯索引节点，进一步降低了树的高度；此外将叶节点使用指针连接成链表，范围查询更加高效。

### 5.6、哈希表

哈希表又称散列表，是根据关键码(key)和值 (value) 直接进行访问的数据结构，通过key和value来映射到集合中的一个位置，这样就可以很快找到集合中的对应元素。这种存储空间可以充分利用数组的查找优势来查找元素，所以查找的速度很快（如Java中的HashMap）。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/java_hashtable.jpg?raw=true" alt="java_hashtable.jpg" style="zoom:67%;" />


### 5.7、图

图是由结点的有穷集合V和边的集合E组成。其中，为了与树形结构加以区别，在图结构中常常将结点称为顶点，边是顶点的有序偶对，若两个顶点之间存在一条边，就表示这两个顶点具有相邻关系。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/graph.png?raw=true" alt="graph.png" style="zoom:67%;" />


### 5.8、堆

堆是一种比较特殊的数据结构，可以被看做一棵树的数组对象，具有以下的性质：

- 堆中某个节点的值总是不大于或不小于其父节点的值。

- 堆总是一棵完全二叉树。

  <center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/heap.jpg?raw=true" alt="heap.jpg" style="zoom:67%;" />

## 6、算法

### 6.1、常用排序算法

#### 6.1.1、冒泡排序

冒泡排序是最简单的排序之一了，其大体思想就是通过与相邻元素的比较和交换来把小的数交换到最前面。这个过程类似于水泡向上升一样，因此而得名。

```java
//冒泡排序时间复杂度O(n^2)，比较次数多，交换次数多。因此是效率极低的算法。
//冒泡排序是一种稳定的算法。
public static void bubbleSort(int[] arr) {
    if(arr == null || arr.length == 0) {
        return;
    }
    int len = arr.length;
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

#### 6.1.2、快速排序

与冒泡排序类似，首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

```java
//选择排序时间复杂度O(n^2)。比较次数多，交换次数少。
public static void selectSort(int[] arr) {
    if (arr == null || arr.length == 0) {
        return;
    }
    int minIndex = 0;
    for (int i = 0; i < arr.length - 1; i++) { //只需要比较n-1次
        minIndex = i;
        for (int j = i + 1; j < arr.length; j++) { //从i+1开始比较，因为minIndex默认为i了，i就没必要比了。
            if (arr[j] < arr[minIndex]) {
                int temp = arr[j];
                arr[j] = arr[minIndex];
                arr[minIndex] = temp;
            }
        }
    }
}
```

### 6.2、搜索算法

### 6.3、LRU算法



## 7、RxJava响应式编程

RxJava是一个异步操作库，一个能让你用极其简洁的逻辑去处理繁琐复杂任务的异步事件库。

### 7.1、观察者模式

观察者模式4大要素：**被观察者(Observable)**、**观察者(Observer)**、**订阅(subscribe)**和**事件(Event)**。
简言之，即**观察者通过订阅某个事件来观察被观察者**。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rx_observer1.png?raw=true" alt="rx_observer1.png" style="zoom:70%;" />


RxJava中的观察者模式，与通用的观察者模式所不同的是RxJava把多个事件作为一个序列，有开始(onSubscribe)，结束(onComplete)，错误(onError)，多个事件(onNext)，也就是说在开始之后，结束或者出错之前可以发送多个事件给观察者。
*注意：RxJava中onError()和onComplete()只能调用一个。*

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rx_observer2.png?raw=true" alt="rx_observer2.png" style="zoom:70%;" />



RxJava响应式编程一般分为以下3步：

> 1、创建被观察者（即数据源）并产生数据。
> 2、创建观察者。
> 3、订阅（关系绑定）。

***RxJava代码示例：***

```java
 public static void main(String[] args) {
     //标准对象方法调用方式
     //被观察者
     Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
         public void subscribe(ObservableEmitter emitter) throws Throwable {
             //创建数据并发送，包括状态和错误信息
             //数据发送
             emitter.onNext(1);
             emitter.onNext(2);
             emitter.onNext(3);
             System.out.println("被观察者发送数据工作线程 " + Thread.currentThread().getName());
             //emitter.onError(new IllegalArgumentException("Test Exception!"));
             //完成事件发送
             emitter.onComplete();
         }
     });
     //观察者
     Observer observer = new Observer<Integer>() {
         //订阅时被调用
         public void onSubscribe(Disposable d) {
             System.out.println("onSubscribe1");
             //调用d.dispose();//可中断数据发送
         }
         //发送数据时被多次调用
         public void onNext(Integer o) {
             System.out.println("onNext1 : " + o + " 接收数据工作线程 " + Thread.currentThread().getName());
         }
         //出错时被调用
         public void onError(Throwable e) {
             System.out.println("onError1 : " + e.getMessage());
         }
         //完成时被调用
         public void onComplete() {
             System.out.println("onComplete1");
         }
     };
     //订阅事件
     observable
         //.subscribeOn(Schedulers.newThread())
         .subscribe(observer);

     //链式调用风格
     Observable.just("str1", "str2")
         .map(s -> s + " append str test")//lambda格式的map转换
         .subscribe(new Observer<String>() {
             @Override
             public void onSubscribe(Disposable d) {
                 System.out.println("onSubscribe2");
             }

             @Override
             public void onNext(String s) {
                 System.out.println("onNext2 : " + s);
             }

             @Override
             public void onError(Throwable e) {
                 System.out.println("onError2 : " + e.getMessage());
             }

             @Override
             public void onComplete() {
                 System.out.println("onComplete2");
             }
         });
 }
//输出：
onSubscribe1
onNext1 : 1 接收数据工作线程 main
onNext1 : 2 接收数据工作线程 main
onNext1 : 3 接收数据工作线程 main
被观察者发送数据工作线程 main
onComplete1
onSubscribe2
onNext2 : str1 append str test
onNext2 : str2 append str test
onComplete2
```

### 7.2、基类

io.reactivex.Flowable：发送0个N个的数据，支持Reactive-Streams和背压
io.reactivex.Observable：发送0个N个的数据，不支持背压，
io.reactivex.Single：只能发送单个数据或者一个错误
io.reactivex.Completable：没有发送任何数据，但只处理 onComplete 和 onError 事件。
io.reactivex.Maybe：能够发射0或者1个数据，要么成功，要么失败。

> 什么是背压？
> 被观察者发送消息太快以至于它的操作符或者订阅者不能及时处理相关的消息，从而操作消息的阻塞现象。

### 7.3、操作符

操作符作用于Observable数据源，可以对待发送的数据进行操作变换。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rx_operator.png?raw=true" alt="rx_operator.png" style="zoom:60%;" />


SubscribeOn和ObserverOn的区别

> **SubscribeOn**：**指定被观察者(Observable)的调度器（工作线程）**，即数据源产生的线程。
> **ObserverOn**：**指定观察者(Observer)的调度器（工作线程）**。
> *如：Android中，请求网络需要在子线程，而请求完成的数据展示又需要在主线程中。*
> 
> <center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/rx_android.png?raw=true" alt="rx_android.png" style="zoom:67%;" />

### 7.4、调度器（schedulers）

RxJava通过调度器来处理多线程/线程切换问题。
RxJava中包含的一些调度器示例如下：

> **Schedulers.io()**：用于IO密集型的操作，例如读写SD卡文件，查询数据库，访问网络等。具有线程缓存机制，在此调度器接收到任务后，先检查线程缓存池中，是否有空闲的线程，如果有，则复用，如果没有则创建新的线程，并加入到线程池中，如果每次都没有空闲线程使用，可以无上限的创建新线程。
> **Schedulers.newThread()**：在每执行一个任务时创建一个新的线程，不具有线程缓存机制，因为创建一个新的线程比复用一个线程的成本更高，虽然使用Schedulers.io( )的地方，都可以使用Schedulers.newThread( )，但是，Schedulers.newThread( )的效率没有Schedulers.io( )高。
> **Schedulers.computation()**：用于CPU 密集型计算任务，即不会被 I/O 等操作限制性能的耗时操作。例如xml,json文件的解析，Bitmap图片的压缩取样等，具有固定的线程池，大小为CPU的核数。不可以用于I/O操作，因为I/O操作的等待时间会浪费CPU。
> **Schedulers.single()**：拥有一个线程单例，所有的任务都在这一个线程中执行，当此线程中有任务执行时，其他任务将会按照先进先出的顺序依次执行。
> **Schedulers.trampoline()**：在当前线程立即执行任务，如果当前线程有任务在执行，则会将其暂停，等插入进来的任务执行完之后，再将未完成的任务接着执行。
> **AndroidSchedulers.mainThread()**：Android平台调度器。在Android UI线程中执行任务，为Android开发定制。



## 8、框架相关

### 8.1、Spring

#### 8.1.1、CAP理论

C(Consistency)：一致性（如多节点数据一致）
A(Availability)：可用性（保证服务可用，如多节点备用服务）
P(Partition tolerance)：分区容忍性（是否可以将数据存到多个地方）
***一个系统中不可能同时满足C,A,P,只能同时满足两项。***

例如：Eureka（保证AP），Zookeeper（保证CP）。

#### 8.1.2、AOP

AOP（Aspect Orient Programming），我们一般称为面向方面（切面）编程，作为面向对象的一种补充，用于处理系统中分布于各个模块的横切关注点，比如事务管理、日志、缓存等等。
*AOP实现主要分为 静态代理（AspectJ） 和 动态代理（Spring AOP） 。*
*<small>更多可参考：https://blog.csdn.net/xiaojin21cen/article/details/79487769</small>*

#### 8.1.3、控制反转IOC

控制反转IOC(Inversion of Control)：在Java开发中，IoC意 味着将你设计好的类交给系统去控制，而不是在你的类内部控制。

控制反转（IoC）
依赖注入（DI）
依赖注入三种表现形式：
a)构造方法注入

```java
class A {
    B b;
    C c;
    public A(B b, C c) {
        this.b = b;
        this.c = c;
    }
} 
```


b)Setter 注入

```java
class A {
    B b;
    C c;
    public void setB(B b) {
        this.b = b;
    }
    public void setC(C c) {
        this.c = c;
    }
}
```


c)接口注入

```java
interface Setter {
    void setB(B b);
    void setC(C c);
}
class A implements Setter{
    B b;
    C c;
    public void setB(B b) {
        this.b = b;
    }
    public void setC(C c) {
        this.c = c;
    }
}
```

SpringCloud基础功能：
1）服务治理：Eureka
2）客户端负载均衡：Ribbon
3）服务容错保护：Hystrix
4）声明式服务调用：Feign
5）API网关服务：Zuul
6）分布式配置中心：SpringCloud Config

SpringCloud高级功能：
1）消息总线：SpringCloud Bus
2）消息驱动的微服务：SpringCloud Stream
3）分布式服务跟踪：SpringCloud Sleuth

*<small>更多可参考：https://mp.weixin.qq.com/s/pGSx8eKFH3YnUos3SM2ITw</small>*



## 9、Java性能分析调优工具

大多数调优分析软件均是分析JVM heap dump文件（`heap dump`文件是一个二进制文件，它保存了某一时刻JVM堆中对象使用情况，是指定时刻的Java堆栈的快照）。*在未利用任何工具dump的情况下，使用JDK自带命令也能获取JVM heap dump信息，如下：*
**jmap -dump:live,format=b,file=C:\Users\Administrator\Desktop\heap.hprof <对应PID>**

*注意：JAVA_HOME当前环境变量，32位JDK只能分析32位Java程序，64位JDK只能分析64位Java程序（利用MAT，JProfiler等分析工具同样如此，需要区分JDK）。*

### 9.1、什么是JMX

所谓JMX，是Java Management Extensions (Java管理扩展)的缩写，是一个为应用程序植入管理功能的框架。用户可以在任何Java应用程序中使用这些代理和服务实现管理。

> Java飞行记录器（Java Flight Recorder，JFR），是JVM内置的记录引擎，能收集JVM和Java应用数据，帮助开发者诊断分析Java应用。JFR是商业版 JDK 的一项分析工具，
> ***注意：JFR自JDK11起才开源，故OpenJDK11之前的版本不支持JFR，若使用Oracle JDK（支持JFR），则需要使用-XX:+UnlockCommercialFeatures和-XX:+FlightRecorder来声明解除商业许可并开启JFR。***

**如何通过JMX连接监控远程的JVM进程？**

首先，远程Java程序实例需要开启JMX服务，在执行Java程序启动命令时可作如下配置（针对Java8版本）：
-Dcom.sun.management.jmxremote=true
-Dcom.sun.management.jmxremote.port=7091 #JMX连接端口，默认7091
-Dcom.sun.management.jmxremote.rmi.port=7091 #映射RMI连接端口，同上述端口配置保持一致
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=远程JVM对应服务器的IP
示例如下（启动某Java程序实例时开启 JMX，并允许远程访问）：

```shell
java -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=7091 -Dcom.sun.management.jmxremote.rmi.port=7091 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=47.97.221.150 -jar ct-rmfx-server-v2-2.0.0.jar
```

*只有开启远程JVM的JMX服务，下述JMC等调试工具才能连接监控。*

**什么是MBean？**

MBean即被管理的Java Bean。使用MBean能管理资源（包括属性、方法等），**MBean必须定义在接口中，然后MBean必须实现这个接口；它的命名也必须遵循一定的规范，例如MBean为Hello，则接口必须为HelloMBean**。使用MBean管理属性或方法示例如下：

```java
//MBean必须定义在接口中
//接口的命名规范为以具体的实现类为前缀，如此处对应的类为Hello，那么接口名即为HelloMBean
public interface HelloMBean {
    //对应某个属性，则必须实现set和get方法，如此处set和get对应的属性即为value
    void setValue(String value);
    String getValue();
    //调用某个方法
    String testCall();
}

//MBean必须实现对应的接口
//该类名称必须与实现的接口的前缀保持一致（即MBean前面的名称），此处类名Hello即对应HelloMBean的前缀Hello
public class Hello implements HelloMBean {
    String value;//属性value

    @Override
    public void setValue(String value) {
        System.out.println("setValue : "+value);
        this.value = value;
    }
    @Override
    public String getValue() {
        System.out.println("getValue : "+ this.value);
        return this.value;
    }
    @Override
    public String testCall() {
        System.out.println("testCall..." + new Date());
        return "返回值:" + System.currentTimeMillis();
    }
}

public static void main(String[] args) {
    //通过工厂类获取MBeanServer，用来做MBean的容器
    MBeanServer server = ManagementFactory.getPlatformMBeanServer();
    //ObjectName中的取名是有一定规范的，格式为：“域名：name=MBean名称”
    //其中“域名”和“MBean名称”可以任意取，用于唯一标识MBean的实现。如下所示
    ObjectName helloName = new ObjectName("MyJMXBean:name=TestJavaMBean");
    //注入到MBeanServer中
    server.registerMBean(new Hello(), helloName);
    
    //---------------------------------------------------------------------
    //上述实现MBean后，可通过JConsole或JMX等工具来访问JMX
    //同样通过下述方式，可以通过JMX提供的工具页访问（Web访问方式）
    //注意：此Web方式需要借助jdmk依赖包，启动成功后，在网页中访问对应端口即可，如8082
    ObjectName adapterName = new ObjectName("HelloAgent:name=htmladapter,port=8082");
    HtmlAdaptorServer adapter = new HtmlAdaptorServer();
    server.registerMBean(adapter, adapterName);
    adapter.start();
}
```

通过上述Java代码配置并启动运行后，就可以用过JMX管理端（如：JMC或JConsole等）来控制调用MBean了。如下所示：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_jmx.png?raw=true" alt="jvm_jmx.png" style="zoom:67%;" />

如上图操作所示：过属性操作设置value，此时Java程序会System.out输出“setValue : 123”或“getValue : 123”。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jvm_jmx2.png?raw=true" alt="jvm_jmx2.png" style="zoom:67%;" />

如上图操作所示：通过操作调用某方法，此时会返回方法调用返回值，并在Java程序中System.out输出：“testCall...Wed May 13 13:26:01 CST 2020”
*更多MBean使用方法可参考：https://www.cnblogs.com/dongguacai/p/5900507.html*

> **扩展：JMX和jstatd区别**
> JVM远程监控可采用两种方式进行连接，JMX与jstatd，它们有如下区别：
> JMX：为某个Java实例的监控，可以查看线程，对Cpu及内存进行抽样，可以强制回收垃圾。
> jstatd：为整个JVM环境的监控，可以查看GC状态。
> *一般保证每个实例独立监控，互不干扰，多采用JMX方式，jstatd在此不作叙述。*
>
> Tips：使用JMX时，可以在需要监控JVM或管理MBean时，通过开启防火墙对应端口来访问，无需监控或管理时关闭防火墙对应端口来屏蔽访问保证安全；由此通过管控防火墙端口方式，就不用每次执行对应java命令来启用或者不启用JMX管理服务了。

### 9.2、Eclipse Memory Analyzer (MAT)

MAT是一款JVM内存分析工具，利用现有的heap dump文件或者通过“File” --> “Acquire Heap Dump”直接获取当前JVM heap dump信息，可用来分析Java程序是否产生内存泄漏。

### 9.2、Java Mission Control (JMC)

JDk7 7u40之后自带。主要有两种功能：

- 实时监控JVM运行时的状态
- Java Flight Recorder 取样分析

### 9.3、JProfiler

JProfiler是由ej-technologies GmbH公司开发的一款性能瓶颈分析工具，JProfiler不仅可以分析内存，还能对JVM CPU、Threads等展开分析。

**JProfiler的数据采集方式：**
JProfier采集方式分为两种：Sampling(样本采集)和Instrumentation。
Sampling： 每隔一定时间(5ms)将每个线程栈中方法栈中的信息统计出来，对应用影响小，但对一些数据/特性不支持（如：方法的调用次数）。
Instrumentation：在class加载之前，JProfier把相关功能代码写入到需要分析的class中，对正在运行的jvm有一定影响。

**JProfiler的启动方式：**
Attach Mode：直接对本机JVM正在运行的Java程序进行分析。
Profile at startup：在被分析的JVM启动时，将指定的JProfiler Agent手动加载到该jvm。JProfiler GUI 将收集信息类型和策略等配置信息通过socket发送给JProfiler Agent，收到这些信息后该jvm才会启动。
Prepare for profiling：和Profile at startup的主要区别：被分析的JVM不需要收到JProfiler GUI 的相关配置信息就可以启动。
Offline profiling：对现有的heap dump文件进行分析。

### 9.4、JMeter

JMeter是开源软件Apache基金会下的一个性能测试工具，用来测试部署在服务器端的应用程序的性能。
参考：https://jmeter.apache.org/

## 10、其他考点

### 10.1、Java是值传递还是引用传递？

是**值传递**。Java 语言的参数传递只有值传递。当一个实例对象作为参数被传递到方法中时，参数的值就是该对象的引用的一个副本。指向同一个对象，对象的内容可以在被调用的方法内改变，但对象的引用(不是引用的副本) 是永远不会改变的。

### 10.2、静态变量和成员变量区别？

静态变量属于类，成员变量属于对象。
静态变量存储在方法区，成员变量存储在堆栈区。
静态变量在类加载时即存在，伴随类的一生，而成员变量则是伴随对象产生和消失。
静态变量可以通过类和对象来调用，而成员变量只能通过对象来调用。
静态方法可以直接直接使用静态变量，而不能使用成员变量。

### 10.3、clone方法如何理解？

Java中可以使用new操作符创建对象，或者使用clone方法复制对象。clone方法最终将调用JVM中的原生方法完成对象复制（不会调用对象构造函数），所以一般使用clone方法复制对象要比新建一个对象然后逐一进行元素复制效率要高。

要使用clone方法，需要将类实现Cloneable接口，并根据需求实现clone接口的深拷贝和浅拷贝。这里要注意深拷贝和浅拷贝问题，如果该类内部变量是引用类型的，并且内部变量类没有实现Cloneable接口，那么克隆出来的该变量是浅拷贝的（只是复制了引用，两个引用指向统一实例）。

**浅拷贝**是指拷贝对象时仅仅拷贝对象本身（包括对象中的基本变量），而不拷贝对象包含的引用指向的对象。
**深拷贝**不仅拷贝对象本身，而且拷贝对象包含的引用指向的所有对象。
*通过clone方法复制对象时，若不对clone()方法进行改写，则调用此方法得到的对象为浅拷贝。*

### 10.4、两个对象equals相等，那么hashCode也相同吗？

对象equals相等，那么hashCode也一定相等，反之则不一定。且hashCode不等，equals必定不等。
*如果值相同，hashCode不同，就会造成Hashset、HashMap等借助hashCode实现的数据结构出现错乱，相同的值或者key可能出现多次。*

> **在每个改写equals方法的类中，你也必须改写hashCode()方法。**
> 
>***hashCode主要用于在hash表中（如Hashtable、HashSet和HashMap）进行对象查找或比较时提高效率（比较过程为先比较hashCode，hashCode不同的两个对象必定不等，若hashCode相同，则再通过equals比较对象是否相等）。***
> **<small>一般对于需要存放到Set或者Map中的键值对元素，建议按需重写equals和hashCode方法，以保证其唯一性。</small>**
>
> **复写equals方法一般套路：**
> 
>```java
> @Override
>public boolean equals(Object o) {
>  if(this == o) {//先比较两个对象内存地址是否一样
>      return true;
>  }
>     if(o == null) {//再判断待比较对象是否为空
>         return false;
>     }
>     if(getClass != o.getClass()) {//再判断两个对象是否为同一种类型的对象
>         return false;
>     }
>     //最后在比较两个对象是否完全一致。如下：
>     return this.intTypeValue == o.intTypeValue;
>    }
>    ```
>    
> **复写hashCode方法一般套路：**
> 
>```java
> @Override
>public int hashCode() {
>  int B = 31;//一个素数，此处用31是因为参考了String类的hashCode方法中推荐的素数31
>  int hash = 0;//返回值hashCode是一个int型
>  hash = hash * B + this.intTypeValue;//示例：int型的成员变量直接加上相应值
>     hash = hash * B + this.strTypeValue.toLowerCase().hashCode();//示例：String类型的成员变量调用String类实现的hashCode返回对应值
>     return hash;
>    }
>    ```

### 10.5、equals和==区别？

**==**：如果是基本数据类型（如int，double等）,比较的是它们的值是否相等。而对于引用类型（如Object），则比较的是对象的内存地址。

**equals**：在String类中，JDK默认复写了equals方法，所以在比较时会先判断对象内存地址是否一样，然后再判断字符串内容是否一样，而在非String类中，默认情况下，equals方法和==都是比较的对象内存地址，不会比较具体字段内容。

### 10.6、System.identityHashCode(Object) 和 Object.hashCode()两种hash值有什么区别？

Object.hashCode() 是根据`Object的内容` 来产生hash值的。
System.identityHashCode(Object) 是根据`Object的内存地址`来产生hash值的。

### 10.7、类中的方法调用顺序？

```java
class A {
    static {
        System.out.print("1");
    }
    public A() {
        System.out.print("2");
    }
}

class B extends A{
    static {
        System.out.print("a");
    }
    public B() {
        System.out.print("b");
    }
}

public class Main {
    public static void main(String[] args) {
    	A ab = new B();
        ab = new B();
    }
}

//输出为1a2b2b，表明：创建对象时构造器的调用顺序是： 父类静态初始化块 -> 子类静态初始化块 -> 父类初始化块 ->调用了父类构造器 -> 子类初始化块 -> 调用子类的构造器。
```

### 10.8、Error和Exception有什么区别？

**Error**：是系统抛出的，不能再运行时捕获，比如内存溢出。
**Exception**：是程序抛出进行捕获并处理的异常，比如类型转换错误、空指针等，通过捕获Exception可以使程序在发生异常时仍可正常运行。

### 10.9、finally语句一定会执行吗？

在某些极端特殊的情况下可能不会执行：如JVM崩溃，或者主动调用了System.exit()方法。

### 10.10、boolean占几个字节？

1）boolean类型被编译为int类型，即在JVM里占用字节和int完全一样，int是4个字节，于是boolean也是4字节。
2）boolean数组在Oracle的JVM中，编码为byte数组，每个boolean元素占用8位=1字节。

综上，所以它的“大小”并不是精确定义的，1个字节、4个字节都是有可能的。

> 扩展：为什么编译时将boolean编译为int呢？而不使用byte或short不是更节省空间吗？
> 因为32位CPU（这里指CPU硬件，而非操作系统的32位或64位）使用4个字节（32位）进行寻址，从性能来看，32位的机器，处理32位的int是最快的，比byte，short等都会快很多。所以最好的选择是int type。

### 10.11、String、StringBuffer和StringBuilder区别？

String中的字符char[]采用了final修饰符，所以String对象不可变，是线程安全的。
StringBuffer是线程安全的，StringBuilder不是线性安全的，因为StringBuffer添加了同步锁，所以在单线程情况下StringBuilder性能较StringBuffer高。

### 10.12、如何简单快速实现String字符串反转？

可以采用StringBuilder或者StringBuffer的reverse方法。

 ```java
String s = "abc";
StringBuilder sb = new StringBuilder(s);
System.out.println(sb.reverse());
 ```

### 10.13、对于超大文件的复制有什么好的办法？

利用NIO的FileChanel（因为使用了内存映射技术）。示例如下：

```java
try {
  	FileInputStream in = new FileInputStream(source);
  	FileOutputStream out = new FileOutputStream(target);
    FileChannel inChannel = in.getChannel();
    WritableByteChannel outChannel = out.getChannel();
    inChannel.transferTo(0, inChannel.size(), outChannel);
    inChannel.close();
    outChannel.close();
    in.close();
    out.close();
} catch (FileNotFoundException e) {
    e.printStackTrace();
} catch (IOException e) {
    e.printStackTrace();
}
```

### 10.14、 为什么要谨慎使用ArrayList中的subList方法？

List的subList方法并没有创建一个新的List，而是使用了原List的视图，这个视图使用内部类SubList表示，通过subList获得的数据会和元List数据相互影响，所以要谨慎使用。如下所示：

```java
public static void main(String[] args) {
    List<String> sourceList = new ArrayList<String>() {{
        add("H");
        add("E");
        add("L");
        add("L");
        add("O");
    }};

    List subList = sourceList.subList(2, 4);
    System.out.println("sourceList ： " + sourceList);
    System.out.println("sourceList.subList(2, 4) 得到List(即subList) ："+subList);
    subList.set(1, "666");
    System.out.println("subList.set(3,666) 得到List(即subList) ："+subList);
    System.out.println("sourceList ： " + sourceList);
    
    subList = Lists.newArrayList(subList);
    System.out.println("通过newArrayList拷贝List数据" + subList);
    subList.set(0, "lala");
    System.out.println("subList.set(0,lala) 得到List(即subList) ："+subList);
    System.out.println("sourceList ： " + sourceList);
}
//输出
sourceList ： [H, E, L, L, O]
sourceList.subList(2, 4) 得到List(即subList) ：[L, L]
subList.set(3,666) 得到List(即subList) ：[L, 666]
sourceList ： [H, E, L, 666, O]
//此处由结果看出，subList生成的新List该表数据时影响了原List的数据
通过newArrayList拷贝List数据[L, 666]
subList.set(0,lala) 得到List(即subList) ：[lala, 666]
sourceList ： [H, E, L, 666, O]
//此处结果看出，newArrayList拷贝的List修改数据时不会影响原List数据
```

### 10.15、循环体中高效拼接字符串

循环体内，字符串的连接方式，使用 `StringBuilder` 的 `append` 方法进行扩展。而不要使用`+`。
因为，字符串在拼接过程中，是将String转成了StringBuilder后，使用其append方法进行处理的。如果在循环体重使用`+`来拼接字符串，就会循环创建多个StringBuilder对象，因而频繁的创建对象，不仅会耗时，还会造成内存资源的浪费。

```java
//利用此示例比对，可发现利用StringBuilder与+拼接字符串耗时差异非常明显
public static void main(String[] args) {
    long t1 = System.currentTimeMillis();
    //String str = null;
    StringBuilder sbStr = new StringBuilder();
    //这里是初始字符串定义
    for (int i = 0; i < 50000; i++) {
        //这里是字符串拼接代码
        //str += "str " + i;
        sbStr.append("str ").append(i);
    }
    //System.out.println(str);
    //System.out.println(sbStr.toString());
    long t2 = System.currentTimeMillis();
    System.out.println("cost:" + (t2 - t1));
}
```

### 10.16、如何判断Java整型数相加溢出？

int类型一般占4个字节，故取值范围 -2^31 ~ 2^31-1。若使用最大值+1或最小值减1时都会发生溢出。示例如下：

```java
//int最大值
//Integer.MAX_VALUE或2^31-1计算可得：2147483647
//int最小值
//Integer.MIN_VALUE或-2^31计算可得：-2147483648
//使用2147483647+1，或使用-2147483648-1，都会产生溢出

//补充：负数的二进制等于正数二进制的反码+1（反码+1又称作为补码）
//最高位为符号位，0位正数，1位负数

//判断数据是否溢出
//方法1：如果两个数都是正数，那么判断结果是否为正数，且小于Integer.MAX_VALUE；减法反之。
//方法2：使用Math中的方法，利用位操作
Math.addExact(x,y)//加法
Math.subtractExact(x,y)//减法
Math.multiplyExact(x,y)//乘法
```



# Android相关

## 1、基础篇

### 1.1、Android四大组件

**Activity**：Activity是Android程序与用户交互的窗口,对用户来说是可见的。
**service**：后台服务于Activity,是一个服务，不可见。
**Content Provider**：应用程序间通用的共享数据的一种方式，对外提提供数据。
**BroadCast Receiver**：接受一种或者多种Intent作触发事件，接受相关消息，做一些简单处理。

### 1.2、Activity相关
#### 1.2.1、Activity/Fragment生命周期

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_activity_lifecycle.png?raw=true" alt="android_activity_lifecycle.png" style="zoom:67%;" /><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_fragment_lifecycle.png?raw=true" alt="android_fragment_lifecycle.png" style="zoom:67%;" />

* onStart 和 onStop是针对于Activity是否可见而调用；而onResume和onStop是针对于Activity是否置于前端而调用。
* ActivityA跳转ActivityB时：A.onPause()→B.onCreate()→B.onStart()→B.onResume()→A.onStop
    ActivityB返回ActivityA时：
B.onPause()→A.onRestart()/A.onCreate()→A.onStart()→A.onResume()→B.onStop()
 * onPause和onStop中不能执行耗时任务，否则ANR。
 * onSaveInstanceState和onRestoreInstanceState调用以及屏幕旋转规则：
      Activity.onCreate() → onCreate → onRestoreInstanceState。
      Activity(意外情况/配置变化) → onSaveInstanceState → onDestory。
      onRestoreInstanceState中的Bundle对象不可能为空，而onCreate中的Bundle可以为空。当在Mainifest.xml文件中配置configChanges参数时，屏幕旋转会调用onConfigChanges方法。

#### 1.2.2、Activity的四种状态

**running** 状态：表示activity正在运行，处于一个可见可交互的状态，与之对应的生命周期方法是onResume()方法。
**paused**状态：表示activity处于一个失去焦点的状态，此时activity处于一个可见或者部分可见的状态，同时activity也无法与用户进行交互（如：启动一个Dialog），与之对应的生命周期方法是onPause()方法。
**stopped **状态：表示activity已经退居后台，处于一个完全不可见的状态，如果此时手机的内存不足，activity有可能会被系统回收。与之对应生命周期方法是onStop()方法。
**killed **状态：表示activity已经被杀死，已经被系统回收。与之对应的生命周期方法onDestory()方法。

#### 1.2.3、Activity启动模式/任务栈

**standard**：每一次启动，都会生成一个新的实例，放入栈顶中。
**singleTop**：通过singelTop启动Activity时，如果发现有需要启动的实例正在栈顶，责直接重用，否则生成新的实例。
*<small>singleTop模式下，若Activity已处于栈顶，那么不会直接调用onCreate、onStart方法，会调用onNewIntent方法。</small>*
**singleTask**：通过singleTask启动Activity时，如果发现有需要启动的实例正在栈中，责直接移除它上边的实例，并重用该实例，否则生成新的实例。
*<small>singleTask调用OnNewIntent方法与singleTop一致，singleTask自带clearTop的效果。</small>*
**singleInstance**：通过singleTask启动Activity时，会启用一个新的栈结构，并将新生成的实例放入栈中。
*Activity在启动时，根据taskAffinity指定任务栈（默认为程序包名），如果指定的任务栈存在，则就算使用FLAG_ACTIVITY_NEW_TASK等参数启动Activity，也不会创建新的栈。*

### 1.3、Service相关

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_service_lifecycle.png?raw=true" alt="android_service_lifecycle.png" style="zoom:72%;" />

#### 1.3.1、startService和bindService区别

1、startService开启服务以后，与activity就没有关联，不受影响，独立运行。
2、bindService开启服务以后，与activity存在关联，退出activity时必须调用unbindService方法，否则会报ServiceConnection泄漏的错误。
3、可以同时使用startService和bindService启动服务（没有先后顺序要求），启动时onCreate仅执行一次。但是，关闭服务时，必须确保stopService和unbindService均被调用（没有先后顺序要求，两者均调用后，服务关闭，onDestory仍执行一次），否则服务不会被关闭，onDestory不会被执行。
4、bindService一般多用于进程间通信。bindService方式启动服务不会调用onStartCommand方法。

**注意：Android O（8.0）后台startService异常问题**
**Android O及以上，后台Service中不能再使用startService启动服务（例如：在Service的onDestroy方法发中，调用startService重新启动自身），否则可能出现异常，但使用startForegroundService启动服务后，需要在5s内在运行的Service中调用setForeground方法。**


#### 1.3.2、理解IntentService

1、IntentService 是继承自 Service，内部通过HandlerThread启动一个新线程处理耗时操作么，可以看做是Service和HandlerThread的结合体，在完成了使命之后会自动停止，适合需要在工作线程处理UI无关任务的场景。
2、如果启动 IntentService 多次，那么每一个耗时操作会以工作队列的方式在 IntentService 的 onHandleIntent 回调方法中执行，依次去执行，使用串行的方式，执行完自动结束。
3、IntentService生命周期：在所有任务执行完毕后，自动结束生命。

```java
public class MyIntentService extends IntentService {
    public MyIntentService() {
        super("MyIntentService");
    }
    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        //在此可执行耗时的任务
        //此方法运行位于子线程中，不会ANR
        //此处执行的任务，即使停止IntentService也不会被打断，仍会自动执行结束
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 1.3.3、Activity与Service通信

1）实现Service中的Binder接口，示例如下：

```java
//定义自己的Binder类
public class MyBinder extends Binder {
    public void callBanZheng(int money){
        testMethod();
    }
}
//在Service中实现Binder接口
@Override
public IBinder onBind(Intent intent) {
    return new MyBinder();
}

//通过ServiceConnection关联
private class MyConn implements ServiceConnection {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        //当服务连接成功调用
        //获取Binder对象
        myBinder = (MyBinder) service;
    }
    @Override
    public void onServiceDisconnected(ComponentName name) {
        //失去连接
    }
}

Intent intent = new Intent(this,BanZhengService.class);
MyConn conn = new MyConn();
MyBinder myBinder;
//在Activity中绑定或移除绑定服务，从而使用myBinder对象调用Service相关方法。
bindService(intent, conn, BIND_AUTO_CREATE);
unbindService(conn);
```

2）通过广播
3）回调等

#### 1.3.4、理解JobService

Job Scheduler运行在设备有可用资源或者合适的条件时触发任务。在创建任务时可以自定义各种条件。当满足生命的条件时，系统将在app的 JobService中执行定义的任务。Job Scheduler还可以根据Doze模式和应用程序的待机限制执行必要的操作。用这种方式执行任务，可以让设备长时间处于休眠状态，从而延长电池使用时长。一般来说Job Scheduler可以用来执行对时间要求不严格的所有任务。

```java
//JobService示例
//JobService在Doze模式下会失效
public class MyJob extends JobService {
    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("JobService onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        System.out.println("JobService onStartCommand");
        return super.onStartCommand(intent, flags, startId);
    }

    //JobService运行在主线程，需要另外开启线程做耗时工作
    @Override
    public boolean onStartJob(JobParameters params) {
        System.out.println("[Job Start], in subThread : " + !(Looper.getMainLooper().getThread() == Thread.currentThread()));
        try {
            System.out.println("do job..." + new Date().toLocaleString());
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //当onStartJob返回true的时候，必须在合适时机手动调用jobFinished方法以结束job
            //第二个参数true时，表明job这次做不完了，计划在下次某个时间继续吧
            jobFinished(params, false);
        }

        //返回false说明job已经完成，不是耗时的任务
        //返回true说明job在异步执行（耗时任务），需要手动调用jobFinished方法告诉系统job完成
        return true;
    }

    //当系统收到一个cancel job的请求时，并且这个job仍然在执行(且onStartJob返回true，调用JobFinished()之前)，系统就会调用onStopJob方法。
    @Override
    public boolean onStopJob(JobParameters params) {
        System.out.println("[Job Stop]");
        //执行此方法以为job服务马上就要停止了，应停止一切工作，不再继续执行了

        // true 需要重试
        // false 不再重试 丢弃job
        return false;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        System.out.println("JobService onDestroy");

        //JobService执行完成会自动关闭服务
        //在此可以重复调用执行任务
        //callJob();
    }
}

//调用此方法即可启动JobService
public void callJob() {
    JobInfo jobInfo = new JobInfo.Builder(1, new ComponentName(getPackageName(), MyJob.class.getName()))
        .setMinimumLatency(5000)//任务最少延迟时间
        .setOverrideDeadline(60000)//任务deadline，当到期没达到指定条件也会开始执行
        //.setPeriodic(15 * 60 * 1000)//设置任务重复执行的周期，不同设备要求的最小值可能不一样，一般15分钟
        //setPeriodic和setMinimumLatency、setOverrideDeadline相互排斥，不能共用
        .setRequiredNetworkType(JobInfo.NETWORK_TYPE_ANY)//网络条件，默认值NETWORK_TYPE_NONE
        .setRequiresCharging(false)//是否充电
        .setRequiresDeviceIdle(false)//设备是否空闲
        .setPersisted(true) //设备重启后是否继续执行，只有配置了监听开机广播的应用才有效。
        .setBackoffCriteria(3000, JobInfo.BACKOFF_POLICY_LINEAR) //设置退避/重试策略
        .build();
    JobScheduler jobScheduler = (JobScheduler) getSystemService(Context.JOB_SCHEDULER_SERVICE);
    int r =  jobScheduler.schedule(jobInfo);
    System.out.println("call job result : " + r);
    //print exist jobs
    //List<JobInfo> allJobs = jobScheduler.getAllPendingJobs();
    //for (JobInfo j : allJobs) {
    //    System.out.println("pending job : " + j.toString());
    //}
}

//使用JobService需要在Manifest中添加对应权限
<service android:name=".MyJob"
         android:permission="android.permission.BIND_JOB_SERVICE">
</service>
```

### 1.4、ContentProvider

ContentProvider是应用程序间通用的共享数据的一种方式。底层采用Android中的Binder机制实现。

**多个进程同时调用一个 ContentProvider 的 query 获取数据，ContentPrvoider 是如何反应的呢？**
ContentProvider可以接受来自其他进程的数据请求。尽管 ContentResolver 与 ContentProvider 类隐藏了实现细节，但是 ContentProvider 所提供的 *query()*，*insert()*，*delete()*，*update()* 都是***在 ContentProvider 进程的线程池中被调用执行的，而不是进程的主线程中***。这个线程池是由 Binder 创建和维护的，其实使用的就是每个应用进程中的Binder线程池。

**可以直接连接数据查询数据，为什么还要设计ContentProvider？**
1）封装接口，隐藏数据的实现方式，对外提供统一的数据访问接口。
2）提供一种跨进程的数据访问方式，只需要 Uri 即可访问数据。由系统来管理 ContentProvider 的创建、生命周期及访问的线程分配，简化我们在应用间共享数据（进程间通信）的方式。我们只管通过 ContentResolver 访问 ContentProvider 所提示的数据接口，而不需要担心它所在进程是启动还是未启动。
3）更好的数据访问权限管理。可以对数据进行权限设置，不同URI对应不同的权限，只有符合权限要求的组件才能访问到ContentProvider的数据。

**运行在主线程的 ContentProvider 为什么不会影响主线程的UI操作？**
ContentProvider 的 *onCreate()* 是运行在 UI 线程的，而 *query()*，*insert()*，*delete()*，*update()* 是运行在线程池中的工作线程的，所以调用这四个方法并不会阻塞 ContentProvider 所在进程的主线程，但可能会阻塞调用者所在的进程的UI线程。

### 1.5、Broadcast相关

1、BroadcastReceiver的生命周期只有一个回调方法onReceive(Context context, Intent intent)；无法进行耗时操作，即使启动线程处理，也是处于非活动状态，有可能被系统杀掉。如果需要进行耗时操作，可以启动一个service处理。
2、广播是通过Intent携带需要传递的数据的，Intent是通过Binder机制实现的，Binder对数据大小有限制，不同系统ROM不一样，一般为1M。

#### 1.5.1、普通广播

**sendBroadcast()**发送普通广播，普通广播会被所有注册的BroadcastReceiver接收到，且顺序是无序的。

#### 1.5.2、有序广播

**sendOrderedBroadcast()**发送有序广播，有序广播中的“有序”是针对广播接收者而言的，指的是发送出去的广播被BroadcastReceiver按照先后循序接收。具体按照priority属性值从大到小排序，对于具有相同的priority的动态广播和静态广播，动态广播会排在前面。先接收的BroadcastReceiver可以对此有序广播进行截断，使后面的BroadcastReceiver不再接收到此广播，也可以对广播进行修改。

#### 1.5.3、粘性广播（已废弃）

**sendStickyBroadcast()**发送粘性广播，正常情况下如果发送者发送了某个广播，而接收者在这个广播发送后才注册自己的Receiver，这时接收者便无法接收到 刚才的广播，为此Android引入了StickyBroadcast，在广播发送结束后会保存刚刚发送的广播（Intent），这样当接收者注册完 Receiver后就可以接收到刚才已经发布的广播。***粘性广播因安全性在API21之后已被废弃。***

#### 1.5.4、前台广播和后台广播

设置Flag FLAG_RECEIVER_FOREGROUND表明其为前台广播，如： intent.setFlags(Intent.FLAG_RECEIVER_FOREGROUND);
**前台广播为什么比后台广播快？**
前台广播拥有更高的优先级，更短的超时间隔（前台广播10s，后天广播60s），前台广播对应消息队列也较为空闲（因为系统默认广播类型为后台广播）。且后台广播的设计思想就是当前应用优先，尽可能多让收到广播的应用有充足的时间把事件做完。而前台广播的目的是紧急通知，设计上就倾向于当前应用赶快处理完，尽快传给下一个。


### 1.6、进程优先级

前台进程 > 可见进程 > 服务进程 > 后台进程 > 空进程

### 1.7、动态权限

Android 6.0(M)及以上支持运行时动态获取权限，示例如下：

```java
//原生模式示例
int checkP = getPackageManager().checkPermission(Manifest.permission.READ_PHONE_STATE, BuildConfig.APPLICATION_ID);
//写法2
//int checkP = ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.READ_PHONE_STATE);
//写法3
//int checkP = checkPermission(Manifest.permission.READ_PHONE_STATE, android.os.Process.myPid(), android.os.Process.myUid());
if (checkP == PackageManager.PERMISSION_GRANTED) {//已授权
    String id = tm.getDeviceId();
} else {//无权限，申请授权
    requestPermissions(new String[]{Manifest.permission.READ_PHONE_STATE}, P_CODE);
}
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    //在此回调中判断申请的授权是否已被授予
    ...
}


//EasyPermissions模式示例，EasyPermissions是google提供的开源库，使得动态权限申请更为便捷
implementation 'pub.devrel:easypermissions:1.1.2'//gradle配置获取依赖库
//java代码
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    // 将结果转发到EasyPermissions
    EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
}
String[] PERMISSIONS = new String[]{Manifest.permission.READ_PHONE_STATE};
if (!EasyPermissions.hasPermissions(this, PERMISSIONS)) {
    EasyPermissions.requestPermissions(
        new PermissionRequest.Builder(this, 0, PERMISSIONS)
        .setRationale("APP需要一些权限，是否授权？")
        .setPositiveButtonText("授权")
        .setNegativeButtonText("取消")
        .setTheme(R.style.Theme_AppCompat_Light_Dialog)
        .build());
} else {
    //已有权限，do something
}
@Override
public void onPermissionsGranted(int requestCode, List<String> perms) {
    //申请的权限被授予的情况
}
@Override
public void onPermissionsDenied(int requestCode, List<String> perms) {
	//权限被拒绝的情况
}


//RxPermissons模式示例，RxPermissions也是一个三方开源框架
//构造参数可以是Activity或者Fragment，为Fragment时，应为Fragment.instance，而非Fragment.getActivity()所获取的Context
RxPermissions rxPermissions = new RxPermissions(this);
rxPermissions.request(Manifest.permission.READ_PHONE_STATE)
    .subscribe(granted -> {
        if (granted) { // Always true pre-M
            //取得授权
        } else {
            //权限被拒
        }
});
```

### 1.8、Doze和Standby模式

#### 1.8.1、基本概念

从Android6.0开始，Android提供了两种省电延长电池寿命的功能：Doze和App Standby模式。两种策略都用于节能，以延长设备使用时间。

##### 1.8.1.1、Doze模式

**Doze**：用户的Android设备处于**未充电状态**，静止且**屏幕关闭一段时间**（即用户不操作设备）之后，Android系统就会自动进入Doze模式。

Doze模式下，系统会限制应用对CPU和网络的访问，反应为以下特点：
1）网络会被挂起，阻止应用程序访问网络。
2）wakelock锁被忽略，应用持有的wakelock暂时失效。
3）AlarmManager定时任务会被推迟（除非使用setAndAllowWhileIdle和setExactAndAllowWhileIdle）。
4）系统WiFi停止扫描刷新。
5）各种同步将会被停止（Sync Adapter）。
6）JobScheduler（JobService）不会工作。

Doze模式包含以下几种状态：
1）**ACTIVE**：手机设备处于激活活动状态。
2）**INACTIVE**：屏幕关闭进入非活动状态。
3）**IDLE_PENDING**：每隔30分钟让App进入等待空闲预备状态。
4）**IDLE**：空闲状态。
5）**IDLE_MAINTENANCE**：处理挂起任务（此维护窗口一般为30s到5min不等）。

Doze模式通过下述方式可重新唤醒应用：
1）打开屏幕。
2）连接电源。

##### 1.8.1.2、Standby模式

**Standby**：当用户长时间为与应用交互（或理解为未使用应用），该应用就会处于App Standby状态，系统将把对应App标识为空闲状态（空闲状态下APP所有功能都将停止，包括Doze模式下AlarmManager可用的setExactAndAllowWhileIdle方法也将失效，APP几乎处于完全被杀死状态）。除非使用下述方式重新唤醒应用：
1）用户主动启动并使用应用。
2）应用存在一个前台进程（前台活动或前台服务，或有组件被另一前台活动及服务调用使用）。
3）应用生成一个用户能在锁屏或通知中心看到的Notification。
4）用户给设备充电时，系统会将所有处于Standby状态的应用释放，允许它们自由的访问网络并执行所有Standby期间暂停的Jobs和Sync。如果应用长时间处于空闲状态，Android系统将会允许处于空闲状态的应用以大约一天一次的频率访问网络。

#### 1.8.2、Doze和Standby区别

Doze需要屏幕关闭（通常晚上睡觉或长时间屏幕关闭才会进入），Doze模式下JobService不会工作，但AlarmManager使用setAndAllowWhileIdle和setExactAndAllowWhileIdle时有效。
App Standby不需要屏幕关闭，就算App进入后台一段时间也会受到连接网络等限制。

#### 1.8.3、适配Doze和Standby

Doze：
1）Doze限制了网络应用，对应及时消息应用，无特殊方案（国外可使用GCM）
2）将对应应用加入系统白名单，避开Doze（标准Android ROM有效，国产手机ROM无效，厂商强制会杀进程）

```java
//1 在android manifest文件里面加入权限
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS"/>
//2 请求加入白名单代码
String packageName = mContext.getPackageName();
mContext.getSystemService(Context.POWER_SERVICE);
Intent intent = new Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS);
intent.setData(Uri.parse("package:" + packageName));
startActivity(intent);
```

3）使用AlarmManager的setAndAllowWhileIdle和setExactAndAllowWhileIdle方法来适配周期性任务。

```java
//AlarmManager示例
public class AliveService extends Service {
    private static AliveService instance;
    public static AliveService getInstance() {
        return instance;
    }
    PendingIntent pendingIntent;
    AlarmManager alarmManager;

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        System.out.println("AliveService onCreate");
        instance = this;

        setAlarm();
    }

    public void setAlarm() {
        if (alarmManager == null || pendingIntent == null) {
            alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
            Intent i = new Intent(this, AliveReceiver.class);
            pendingIntent = PendingIntent.getBroadcast(this, 0,
                    i, PendingIntent.FLAG_UPDATE_CURRENT);
        }

        //部分设备有最小值限制，例如不能低于5s或者9分钟，此处应根据业务场景合理考量
        long triggerAtTime = SystemClock.elapsedRealtime() + 10000;//间隔10s
        //使用下述方法触发的事件，在Doze模式下不会失效，仍能正常运行
//        alarmManager.setAndAllowWhileIdle(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pendingIntent);
        alarmManager.setExactAndAllowWhileIdle(AlarmManager.ELAPSED_REALTIME_WAKEUP, triggerAtTime, pendingIntent);
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return START_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        instance = null;
    }
}

public class AliveReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        System.out.println("AliveReceiver onReceive : " + new Date().toLocaleString());
        if (AliveService.getInstance() != null) {
            AliveService.getInstance().setAlarm();//重新计划下次触发
        }
    }
}
```

Standby：

App Standby模式无法适配后台运行，只要用户经常使用APP或者间隔不太长的时间使用APP，一般不会进入Standby模式，若进入只能主动打开并使用已唤醒。

#### 1.8.4、测试Doze和Standby模式

测试进入Doze模式步骤如下：

```shell
#1、锁屏
#2、取消设备充电状态
adb shell dumpsys battery unplug
#3、查看当前设备电池状态，确保无充电等状态
adb shell dumpsys battery
#4、设置设备能启用idle模式，不执行此步骤可能导致后续无法设置IDLE状态
adb shell dumpsys deviceidle enable
#5、设置idle状态，多次输入此命令，直到状态变为IDLE
#使用adb shell dumpsys deviceidle也可查看状态mState=IDLE
#或使用adb shell dumpsys deviceidle force-idle强制进入IDLE状态
adb shell dumpsys deviceidle step
#6、恢复状态，即取消Doze状态
adb shell dumpsys deviceidle disable
```

测试进入Standby模式步骤如下：

```shell
#1、取消设备充电状态
adb shell dumpsys battery unplug
#2、查看当前设备电池状态，确保无充电等状态
adb shell dumpsys battery
#3、设置设备能启用idle模式，不执行此步骤可能导致后续无法设置IDLE状态
adb shell dumpsys deviceidle enable
#4、设置进入Standby模式
#使用命令adb shell am get-inactive <packageName>可查看对应APP是否处于Standby模式
adb shell am set-inactive <packageName> true
#5、恢复状态，即取消Standby模式
adb shell am set-inactive <packageName> false
```



### 1.9、adb调试相关

#### 1.9.1、基本用法

```shell
#查看连接设备
adb devices
#多设备下指定连接某一个设备
adb -s <设备序列号> shell
#关闭adb服务
adb kill-server
#开启adb服务
adb start-server
```

#### 1.9.2、dumpsys

dumpsys是一种在 Android 设备上运行的工具，可提供有关系统服务的信息。

```shell
#示例：查看可与dumpsys配合使用的系统服务的完整列表
adb shell dumpsys -l
#示例：查看内存信息
adb shell dumpsys meminfo [package-name]
#可以使用findStr过滤匹配文本，若使用grep，则需要先使用adb shell进入shell环境，再使用dumpsys ... | grep ...
adb shell dumpsys meminfo [package-name] | findStr test
#示例：查看正在运行的 Services
adb shell dumpsys activity services [package-name]
```

#### 1.9.3、pm

pm即PackageManager，应用管理（包管理）。

```shell
#示例：查看第三方应用
adb shell pm list packages -3
#示例：清除应用数据与缓存
adb shell pm clear <packagename>
```

#### 1.9.4、am

am即ActivityManager，交互命令。

```shell
#示例：启动指定的 Activity
adb shell am start com.test.mytestandroidapplication/.MainActivity
#示例：强制停止<packagename>对应的相关进程
adb shell am force-stop com.test.mytestandroidapplication
```

#### 1.9.5、上传下载文件

```shell
#从android存储介质中拉取文件
adb pull <remote> <local>
#从本地上传文件到android存储介质中
adb push <local> <remote>
```

#### 1.9.6、logcat日志

```shell
#默认日志输出
adb logcat
#将日志输出保存到文件
adb logcat -v time > D:\log.txt
#按级别过滤显示日志
adb logcat *:W
#按特定标签输出日志，如：指定输出System.out的日志
adb logcat -s "System.out"
```

#### 1.9.7、bugreport

```shell
#生成bugreport文件
#bugreport文件可以用于Battery-Historian分析
adb bugreport
#battery-historian在线网站可访问：https://bathist.ef.lc/
```

## 2、高级篇

### 2.1、Android启动流程

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_system_init.png?raw=true" alt="android_system_init.png" style="zoom:50%;" />

#### 2.1.1、理解Zygote

Zygote作用主要体现在两点：1）启动system_server进程，以便为系统提供各种Service资源；2）孵化应用进程。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_init_zygote.png?raw=true" alt="android_init_zygote.png" style="zoom:67%;" />

init进程（init进程是Linux内核启动的第一个用户级进程）是Linux程序的初始化起点，init进程孵化Zygote进程，Zygote孵化（fork）Java进程（即所有Android应用进程）。所有的Java进程启动都由Zygote完成，system_server也是一个Java进程。


#### 2.1.2、理解ServiceManager



#### 2.1.3、理解ServiceFlinger



#### 2.1.4、Activity启动流程

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_activity_init.png?raw=true" alt="android_activity_init.png" style="zoom:67%;" />



### 2.2、View相关

#### 2.2.1、View的绘制流程

View的绘制：从ViewRootImpl类的performTraversals开始，经过measure，layout，draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。
1）**measure**：用于计算大小。对于一个view来说需要通过measure来确定自己的宽高，而对于viewgroup来说需要循环遍历子view，分别完成子view的measure，最终确定viewgroup的大小，measure阶段有个类比较重要MeasureSpec，这个类封装了size和mode，在进行measure的时候需要根据父容器的MeasureSpec和自己的layoutparams来确定自己的MeasureSpec。
2）**layout**：用于放置视图位置布局。viewgroup用来确定子元素的位置，当viewgroup的位置被确定后，它在onlayout方法中会遍历所有的子元素并调用其layout方法，然后在子view的layout方法中调用子view的onlayout方法确定子view的位置。
3）**draw**：用于绘制视图。绘制一般分以下几步：
a.绘制背景background
b.绘制自己，就是调用onDraw
c.绘制children，就是调用dispatchDraw
d.绘制装饰，比如说滚动条(scrollbars)

#### 2.2.2、自定义View

自定义View主要包含以下步骤（或方法复写）：
1）自定义属性的声明与获取（attrs.xml定义属性声明，TypeArray读取并设置，即实现app:textColor="red"类配置）。
2）复写onMeasure测量方法
3）复写onLayout布局方法（onLayout方法是父控件决定子View的位置，如果是自定义View则可以忽略此方法，如果是自定义ViewGroup则需要复写，以便决定其子View的显示位置）。
4）复写onDraw视图绘制方法。
5）处理当前View触摸事件onTouchEvent（多点触控，滑动等操作）。
6）按需处理onInterceptTouchEvent事件（作用于ViewGroup，这个事件是从父控件开始往子控件传的）。

总结如下：
1）事件的传递顺序是从Activity→Window→View。
2）默认情况下View中的dispatchTouchEvent方法会调用onTouchEvent方法（如果OnTouchListener拦截就不会再调用onTouchEvent方法）。
3）View中事件的顺序为dispatchTouchEvent→OnTouchListener→onTouchEvent→OnLongClickListener（在DOWN中触发）→OnClickListener（在UP中触发）。
4）在ViewGroup中事件的触发顺序为dispatchTouchEvent→onInterceptTouchEvent→onTouchEvent。
5）默认情况下Button，ImageButton等Button的子控件都是会消耗事件的，即onTouchEvent默认返回true，而ImageView，TextView，LinearLayout等一些控件默认是不消耗事件的，即onTouchEvent默认返回为false。

#### 2.2.3、View性能优化

1）线性布局(LinearLayout)和相对布局(RelativeLayout)性能有差异：相同层次下，因为相对布局会进行两次measure，所以线性布局性能更高（注意：若LinearLayout使用weight属性时，也会执行两次measure），但是当布局层次较多时，建议使用相对布局。
2）减少Layout嵌套，并结合使用<include/>（include可实现重用布局文件）、<merge/>（merge一般和include结合使用，主要是为了防止在引用布局文件时产生多余的布局嵌套）可以复用视图组件，优化View性能。
3）对于隐藏的布局文件结构的情况，可以使用ViewStub实现仅在需要时才加载视图布局。

#### 2.2.4、View动画

Android动画包含视图动画（也叫补间动画）、属性动画和帧动画、物理动画。
*补间动画如Scale、Rotate、transparent；属性动画如计算alpha，反转xy坐标等。*

#### 2.3.5、其他

**1）理解invalidate、postInvalidate和requestLayout的区别**

invalidate：会调用onDraw进行重绘，且只能在主线程。
postInvalidate：可以在其他线程调用。
requestLayout：会调用onLayout和onMeasure，不一定会调用onDraw，同样为主线程调用。

> 当动态移动一个View的位置，或者View的大小、形状发生了变化的时候调用requestLayout方法。
> invalidate方法常用于内部调用(比如 setVisiblity())或者需要刷新界面的时候。

**2）如何处理View滑动冲突？**

外部拦截：重写onInterceptTouchEvent方法，内部拦截：重写dispatchTouchEvent方法，同时配合requestDisAllowInterceptTouchEvent方法。

**3）View的gravity和layout_gravity有什么不同？**

gravity是控制当前View内布局的位置，layout_gravity是控制View在父布局中的位置。

**4）理解View的postDelayed方法**


不管是view的postDelayed方法，还是Handler的post方法，通过包装后最终都会走Handler的sendMessageAtTime方法，通过MessageQueue的enqueueMessage方法将message加入队列，加入时按时间排序（可以理解成Message是一个有序队列，时间是其排序依据）。当Looper从MessageQueue中调用next方法取出message时，如果还没有到时间，就会阻塞等待；同时，此时如果再添加message到队列，会自动关联检查当前有没有被阻塞的message，如果需要唤醒，则会在此时进行唤醒。

**5）RecyclerView和ListView的缓存有何差异？**

RecyclerView采用四级缓存（RecycledViewPool缓存ViewHolder，ViewCacheExtension自定义缓存，屏幕内缓存+屏幕外缓存）。
ListView采用两级缓存（屏幕内缓存+屏幕外缓存）。缓存机制通过RecycleBin来实现，ActiveViews 和 ScrapViews分别表示在屏幕上出现的View和视图滑出屏幕的View（ActiveView变为ScrapView）。

**6）dp和px转换**

```java
px = (int) (getResources().getDisplayMetrics().density * dpValue + 0.5f);
dp = (int) (pxValue / getResources().getDisplayMetrics().density + 0.5f);
//加0.5f的目的是为了保证精度
```

**7）如何理解ConstraintLayout约束布局？**

`ConstraintLayout` 相对于 `RelativeLayout`来说性能更好（能减少多层级嵌套），布局上也更加灵活。使用类似于RelativeLayout，可用于*可视化编辑*。详见：https://developer.android.google.cn/training/constraint-layout?hl=zh-cn

### 2.3、Handler/Looper相关

#### 2.3.1、Handler机制

Handler，Message，looper和MessageQueue构成了安卓的消息机制，handler创建后可以通过sendMessage将消息加入消息队列，然后looper不断的将消息从MessageQueue中取出来，回调到Hander的handleMessage方法，从而实现线程的通信。

**扩展：Android中使用Handler注意内存泄漏问题？**

```java
//错误示例：此方法实现的简单Handler对象容易内存泄漏
private final Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {}
};

//利用弱引用实现的Handler可以避免Activity等destory后Handler造成的内存泄漏
public abstract class WeakReferenceHandler<T> extends Handler {
    private WeakReference<T> mReference;

    public WeakReferenceHandler(T reference) {
        mReference = new WeakReference<T>(reference);
    }

    @Override
    public void handleMessage(Message msg) {
        if (mReference.get() == null) {
            return;
        }
        handleMessage(mReference.get(), msg);
    }

    protected abstract void handleMessage(T reference, Message msg);
}
```

#### 2.3.2、HandlerThread

HandlerThread本质上就是一个普通Thread，只不过内部建立了Looper。

**使用HandlerThread几大优点：**
1）创造维护一个后台异步线程，需要的时候就可以丢一个任务给它，使用比较灵活。
2）Android系统提供的，使用简单方便，内部自己封装了Looper+Handler机制，可以代替Thread + Looper + Handler的写法。
3）可以避免项目中随处可见的 new Thread().start()，增加系统开销。

```java
//HandlerThread的使用示例
public class MainActivity extends AppCompatActivity {
    TextView tv;
    Button btn;

    Handler workHandler, mainHandler;
    HandlerThread handlerThread;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv = findViewById(R.id.tv);
        btn = findViewById(R.id.btn);


        handlerThread = new HandlerThread("MyHandlerThread-1");
        handlerThread.start();

        mainHandler = new Handler();
        workHandler = new Handler(handlerThread.getLooper()) {
            @Override
            public void handleMessage(@NonNull Message msg) {
                //借助HandlerThread，在此可以实现一个线程处理多个任务，避免频繁new Thread耗费资源
                //注意：在HandlerThread，就算是多个任务，也是串行执行处理，和线程池稍有差异。
                //所以不要再HandlerThread中执行耗时任务，否则会引起阻塞。
                System.out.println("是否主线程1：" + (Thread.currentThread() == Looper.getMainLooper().getThread()));
                mainHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        System.out.println("是否主线程2：" + (Thread.currentThread() == Looper.getMainLooper().getThread()));
                        tv.setText(new Date().toString());
                    }
                });
            }
        };

        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                System.out.println("是否主线程3：" + (Thread.currentThread() == Looper.getMainLooper().getThread()));
                workHandler.sendEmptyMessage(0);
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (handlerThread != null) {
            handlerThread.quit();//退出HandlerThread
        }
    }
}
```

#### 2.3.3、IdleHandler

使用当前线程的MessageQueue.addIdleHandler方法可以在消息队列中添加一个IdelHandler。

```java
//示例：在主线程中添加一个IdleHandler
MessageQueue messageQueue = Looper.myQueue();
messageQueue.addIdleHandler(new MessageQueue.IdleHandler() {
    @Override
    public boolean queueIdle() {
        System.out.println("queueIdle");
        //返回false时，表示该IdelHandler只执行一次，触发queueIdle方法后，添加的此IdleHandler即会被删除。
        //若为true，则会一直响应阻塞事件，类似于Swing中的addListener
        return false;
    }
});
//当MessageQueue阻塞（为空）时，即当前线程空闲时，会回调IdleHandler中的方法。
//添加IdelHandler时，若添加时消息队列已经为空，则当时不会触发回调，会等到下次队列阻塞（清空）时调用。
```

IdleHandler适用场景

1）当启动Activity时，需要延时执行一些操作，以免启动过慢，我们常常使用以下方式延迟执行任务，但是在延迟时间上却不好控制。

```java
handler.postDelayed(new Runnable() {
    @Override
    public void run() {
        //do something
    }
}, 1000);//有时，此处延迟时间值不好确定，例如有可能是1000，也有可能是5000才合适。
//示例：改用IdleHandler处理
Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
    @Override
    public boolean queueIdle() {
        //do something
        //等待主线程中的事务处理完成后，即空闲后，自动触发
        return false;
    }
});
```

2）例如IM类型应用，通常情况下，每次收到一个IM消息时，都会刷新一次界面，但是当短时间内， 收到多条消息时，就会刷新多次界面，容易造成卡顿，影响性能，此时就可以使用一个工作线程监听IM消息，在通过添加IdelHandler的方式通知界面刷新，避免短时间内多次刷新界面情况的发生。

#### 2.3.4、消息屏障

在Android的消息机制中，其实有三种消息: 普通消息、异步消息及消息屏障。

**消息屏障**也是一种消息，但是它的target为 null（即message的target为null，就是对应sendMessage的Handler为空）。可以通过MessageQueue中的postSyncBarrier方法发送一个消息屏障（该方法为私有，需要反射调用）。
**设置了同步屏障之后，Handler只会处理异步消息。换句话说，同步屏障为Handler消息机制增加了一种简单的优先级机制，异步消息的优先级要高于同步消息。**

**异步消息**和普通消息一样，只不过它被设置setAsynchronous 为true。

**扩展：为什么Looper中一直死循环却不会造成ANR？**

```java
public static final void main(String[] args) {
    ...
    //创建Looper和MessageQueue
    Looper.prepareMainLooper();
    ...
    //轮询器开始轮询
    Looper.loop();
    ...
}
//由上述main方法可知：如果main方法中没有looper进行循环，那么主线程一运行完毕就会退出。APP还怎么运行？？
```

Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。即：**APP的代码其实就是在这个循环里面去执行的，当然不会阻塞了。**

### 2.4、Binder机制

**Binder是一种进程间通信机制**。Binder 基于 C/S 架构。

在操作系统中，使用“进程隔离”技术，防止进程A可以操作进程B的数据，进程A和进程B的虚拟地址空间也不相同，此时进程A和B之间如果需要进行通信则需要采取某种通信机制才能完成，*在Android系统中这种通信机制被称作Binder机制。*

为什么要使用Binder机制来替换Linux自带的管道、Socket等跨进程通信机制呢？
**性能**：Binder通信机制更加高效，Binder数据拷贝只需要一次。而类似管道、Socket之类的IPC每次数据拷贝都需要2次（即数据先从发送方缓存区拷贝到内核开辟的一块缓存区中，然后从内核缓存区拷贝到接收方缓存区，其过程至少有两次拷贝）。共享内存方式虽说一次内存拷贝都不需要，但实现方式又比较复杂。
**安全**：由于传统进程通信方式没有对通信的双方和身方做出严格的验证，没有任何安全措施，只有上层协议才会去架构（比如Socket通信ip地址是客户端手动填入，很容易进行伪造），而Binder机制协议本身就支持通信双方进行身份验证，因而提升了安全性。

**Binder框架定义了四个角色**：Server，Client，ServiceManager以及Binder驱动。其中Server，Client，ServiceManager运行于用户空间，驱动运行于内核空间。这四个角色的关系和互联网类似：Server是服务器，Client是客户终端，ServiceManager是域名服务器（DNS），驱动是路由器。

在Binder中一次完整的IPC通信流程是怎么样的？

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_binder_ipc.png?raw=true" alt="android_binder_ipc.png" style="zoom:55%;" />

1）首先Client端向Binder驱动写了一个TRANSACTION的指令。
2）Binder驱动收到以后，给Client一个回执，就是TRANSACTION_COMPLETE(Client端进入休眠等待回复)。
3）回执发完之后，Binder驱动把这个指令提供TRANSACTION转发给Server端。
4）Server端收到后，退出休眠，处理这个请求，处理完之后，REPLY返回给Binder驱动。
5）Binder驱动收到Server端的回复之后，也会给Server端一个回执，就是TRANSACTION_COMPLETE(Server端收到后进入休眠)。
6）最后Binder驱动把这个返回结果通过BR_REPLY返回给Client端
 *Client端等待回复的时候处于休眠状态，Server端在处理请求外处于休眠状态。*

> **扩展1：如何理解Binder驱动？**
> 跨进程通信是需要内核空间做支持的。传统的 IPC 机制如管道、Socket 都是内核的一部分，因此通过内核支持来实现进程间通信自然是没问题的。但是 Binder 并不是 Linux 系统内核的一部分，那怎么办呢？这就得益于 Linux 的**动态内核可加载模块**（Loadable Kernel Module，LKM）的机制；模块是具有独立功能的程序，它可以被单独编译，但是不能独立运行。它在运行时被链接到内核作为内核的一部分运行。这样，Android 系统就可以通过动态添加一个内核模块运行在内核空间，用户进程之间通过这个内核模块作为桥梁来实现通信。**在 Android 系统中，这个运行在内核空间，负责各个用户进程通过 Binder 实现通信的内核模块就叫 Binder 驱动（Binder Driver）**。
>
> **扩展2：Android中有哪些进程间通信方式？**
>
> 1. Bundle : 只支持四大组件
> 2. 文件共享：不适合并发
> 3. Messenger：封装了AIDL
> 4. AIDL：通过binder实现
> 5. ContentProvider：共享数据
> 6. Socket：适用于网络等

### 2.5、UI刷新机制（vsync机制）

屏幕显示图像的刷新分为从左到右的水平刷新，及从上到下垂直刷新。在绘制图像时每画完一行发出一个水平同步信号，画完所有行发出一个垂直同步信号。即当整个屏幕刷新完毕，一个垂直刷新周期完成，两个相邻的垂直信号之前会有短暂的空白期（如刷新频率60帧，1000/16=16ms，那么每个vsync信号间隔16ms），此时发出 vsync（垂直同步）信号。

**扩展1：Android的刷新频率是60帧/秒，这是不是意味着每隔16ms就会调用一次onDraw方法？**

否，60帧是指屏幕图像刷新频率，是否会调用onDraw需要看具体应用是否调用了invalidate方法或requestLayout方法。

**扩展2：调用invalidate()之后会马上刷新屏幕图像吗？**

不会，需要到等到下一个vsync信号到来（即下一个刷新周期）。

**扩展3：为什么说主线程做耗时操作会丢帧？**

因为在主线程做耗时操作，就会影响下一帧的绘制，导致屏幕图像无法在接下来的刷新信号来临时刷新显示对应图像，就导致丢帧了。

**扩展4：什么是GPU过度绘制？**

屏幕上某一像素点在一帧中被重复绘制多次，就是过度绘制。
如下图，多个卡片跌在一起，但是只有第一个卡片是完全可见的。背后的卡片只有部分可见。但是android系统在绘制时会将下层的卡片进行绘制，接着再将上层的卡片进行绘制。但其实，下层卡片不可见的部分是不需要进行绘制的，只有可见部分才需要进行绘制。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/androi_overdraw.png?raw=true" alt="androi_overdraw.png" style="zoom:33%;" />


> 通过开发者选项中打开“调试GPU过度绘制”，选择“显示过度绘制区域”，可查看应用过度绘制情况，其中颜色说明如下：
> 原色：没有过度绘制
> 蓝色：1 次过度绘制
> 绿色：2 次过度绘制
> 粉色：3 次过度绘制（理论上需优化）
> 红色：4 次及以上过度绘制（理论上需优化）

### 2.6、AMS、PMS和WMS

AMS（ActivityManagerService）：主要用于管理所有应用程序的Activity。

WMS（WindowManagerService）：管理各个窗口，隐藏，显示等。

PMS（PackageManagerService）：用来管理跟踪所有应用APK，安装，解析，控制权限等。

### 2.7、JNI/NDK

#### 2.7.1、概述

**JNI**是`Java Native Interface`的缩写，即 Java 的本地接口。目的是使得 Java 与本地其他语言（如 C/C++）进行交互。JNI 是属于Java的一部分，与Android无直接关系。

> **扩展：JNA是什么？**
> JNA(Java Native Access)是一个开源的Java框架，是Sun公司推出的一种调用本地方法的技术，建立在JNI基上的一个框架。JNA简化了调用本地方法的过程，可以直接调用本地共享库中的函数，不需要写Java代码之外的程序。

**NDK**是`Native Development Kit`的缩写，是 Android 的工具开发包。作用是更方便和快速开发 C/C++ 的动态库，并自动将动态库与应用一起打包到 apk。NDK 是属于Android的一部分，与Java无直接关系。

Android NDK工程，一般比普通Java工程多如下几部分内容（即在普通Java工程中添加如下内容即可支持NDK开发）：

```groovy
//1、app.gradle文件中添加如下两部分externalNativeBuild内容
android {
    ...
    defaultConfig {
        ...
        externalNativeBuild {
            cmake {
                cppFlags "-std=c++11"
            }
        }
    }
    ...
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
            version "3.10.2"
        }
    }
}
//2、src/main/目录下添加cpp目录，并在cpp目录中添加CMakeLists.txt文件及其他c/cpp文件
```

#### 2.7.2、静态注册和动态注册

JNI中函数的注册方法分为静态注册和动态注册两种。

**静态注册**：通过`JNIEXPORT`和`JNICALL`两个宏定义声明，`Java + 包名 + 类名 + 方法名` 形式的函数名。缺点是方法名较长。示例如下：

```c++
//java中的包名在JNI中体现为下划线
//extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码。加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。
extern "C" JNIEXPORT jstring JNICALL Java_com_test_app_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

**动态注册**：通常在`JNI_OnLoad`方法（系统在初始加载动态链接库的时候，JVM会调用JNI_OnLoad(JavaVM* jvm, void* reserved)方法）中通过`RegisterNatives`方法注册，可以不再遵从固定的命名写法。示例如下：

```c++
#include <jni.h>
#include <string>
#include <android/log.h>
#define LOG_TAG "LALALA"
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__)

//native方法
jint get_random_num()
{
    LOGD("call native method...");
    return rand();
}

/*需要注册的函数列表，放在JNINativeMethod 类型的数组中，
以后如果需要增加函数，只需在这里添加就行了
参数：
1.Java代码中用native关键字声明的函数名字符串
2.签名（传进来参数类型和返回值类型的说明）
3.C/C++中对应函数的函数名（地址）
*/
static JNINativeMethod getMethods[] = {
        {"getRandomNum", "()I", (void *) get_random_num},
};

//指定类的路径，通过FindClass方法来找到对应的类
const char *className = "com/test/mytestandroidapplication/TestDataBindingActivity";
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void* reserved)
{
    JNIEnv* env = NULL;
    //判断虚拟机状态是否有问题
    if (vm->GetEnv((void **) &env, JNI_VERSION_1_6) != JNI_OK) {
        return JNI_ERR;
    }
    if (env == NULL) {
        return JNI_ERR;
    }
    //找到声明native方法的类
    jclass clazz = env->FindClass(className);
    if(clazz == NULL){
        return JNI_ERR;
    }
    //注册函数 参数：java类 所要注册的函数数组 注册函数的个数
    if (env->RegisterNatives(clazz, getMethods, sizeof(getMethods)/ sizeof(getMethods[0])) < 0) {
        return JNI_ERR;
    }
    //返回jni 的版本
    return JNI_VERSION_1_6;
}
```

> **扩展：JNI类型对应表**
> "()" 中的字符表示参数，后面的则代表返回值，如下示例：
> 1）"()V"，可表示void Func();
> 2）"(II)V"，可表示void Func(int, int);
> 3）"(Ljava/lang/String;Ljava/lang/String;)V"，可表示void Func(java.lang.String, java.lang.String);
>
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/jni_type.png?raw=true" alt="jni_type.png" style="zoom:55%;" />

#### 2.7.3、JNIEnv 和 JavaVM

**JNIEnv**：表示Java调用native语言的环境，封装了几乎全部JNI方法的指针。**JNIEnv只在创建它的线程生效，不能跨线程传递，不同线程的JNIEnv彼此独立。**
在native环境下创建的线程，要想和Java通信，即需要获取一个JNIEnv对象。通过`AttachCurrentThread`和`DetachCurrentThread`方法将native的线程与JavaVM关联和解除关联。

**JavaVM**：JavaVM是虚拟机在 JNI 层的代表。一个进程只有一个 JavaVM，所有的线程共用一个 JavaVM。

#### 2.7.4、JNI引用类型

**全局引用**：通过`NewGlobalRef`和`DeleteGlobalRef`方法创建和释放一个全局引用。全局引用能在多个线程中被使用，且不会被 GC 回收，只能手动释放。

```c++
env->NewGlobalRef();
env->DeleteGlobalRef();
```

**局部引用**：通过`NewLocalRef`和`DeleteLocalRef`方法创建和释放一个局部引用。局部引用只在创建它的 native 方法中有效，包括其调用的其它函数中有效。理论上局部应用无需手动释放，它们会在 native 方法返回时全部自动释放，但为了避免内存消耗保险起见，还是建议不使用变量时手动释放。

```c++
env->NewLocalRef();//一般在一个方法中，默认都是局部引用（即局部变量）
env->DeleteLocalRef();
```

**弱引用**：当内存不足时 , 会被系统自动回收（但仍建议对无用的资源手动进行回收）。

```c++
env->NewWeakGlobalRef();
env->DeleteWeakGlobalRef();
```

#### 2.7.5、如何进行so动态库崩溃定位

使用addr2line命令，示例如下：

```shell
#dmesg输出段错误信息（segfault）
dmesg
#如输出：[40549.344322] main[11123]: segfault at 400580 ip 00000000004004dd sp 00007ffd7d3e6ad0 error 7 in main[400000+1000]
#其中第3个值00007ffd7d3e6ad0表示：程序出错时的堆栈指针，使用addr2line解析此地址可定位到错误代码
#使用addr2line工具定位
addr2line -C -f -e main 00007ffd7d3e6ad0
```

更多so崩溃定位技巧可参考：https://blog.csdn.net/afei__/article/details/81181827

#### 2.7.6、cmake

CMake是一种跨平台编译工具，比make更为高级，使用起来要方便得多。CMake主要是编写CMakeLists.txt文件，然后用cmake命令将CMakeLists.txt文件转化为make所需要的makefile文件，最后用make命令编译源码生成可执行程序或共享库so。

CMakeLists.txt语法示例如下：

```cmake
#cmake命令不区分大小写，但变量区分大小写，一般均使用小写
#cmake中使用#进行注释

#cmake verson，指定cmake版本 
cmake_minimum_required(VERSION 3.4.1)

#设置一个变量
set(var "正在编译ndk")
message(${var})#打印输出值，${变量名}表示引用一个变量
#cmake常量信息
message(${CMAKE_CURRENT_LIST_FILE})#输出当前CMakeLists.txt文件路径
message(${CMAKE_CURRENT_LIST_DIR})#输出当前CMakeLists.txt文件所在文件夹的路径
#逻辑操作IF
if(true)
...
message("true")
...
endif()

#把c/cpp文件编译成动态库
add_library( #库的名字
             native-lib
             #动态库还是静态库SHARED/STATIC
             SHARED
             #源码文件
             native-lib.cpp
        )
#从系统查找依赖库
find_library( #库名称
              log-lib
			  #系统库
              log )        
#动态库的关联
target_link_libraries( #目标库
                       native-lib
                       #依赖库
                       ${log-lib} )
```

更多JNI详解可参考：https://blog.csdn.net/afei__/article/details/81016413 或 https://www.imooc.com/video/20907

## 3、Gradle/打包/签名相关

**Gradle**是个构建系统，能够简化你的编译、打包、测试过程。熟悉Java的同学，可以把Gradle类比成Maven。

**Gradle Wrapper**的作用是简化Gradle本身的安装、部署。不同版本的项目可能需要不同版本的Gradle，手工部署的话比较麻烦，而且可能产生冲突，所以需要Gradle Wrapper帮你搞定这些事情。Gradle Wrapper是Gradle项目的一部分。

**Android Plugin for Gradle**是一堆适合Android开发的Gradle插件的集合，主要由Google的Android团队开发，Gradle不是Android的专属构建系统，但是有了Android Plugin for Gradle的话，你会发现使用Gradle构建Android项目尤其的简单。

### 3.1、gradle几种依赖的区别

**api**：编译时提供并打包进apk
**provided**：编译时提供但不打包进apk
**implemention**：将该依赖隐藏在内部，而不对外部公开
总结：如果api依赖，一个module发生变化，这条依赖链上所有的module都需要重新编译；而implemention，只有直接依赖这个module需要重新编译。
在全部远程（即module存在于maven仓库，而非本地工程module）依赖模式下，无论是api还是implemention都起不到依赖隔离的作用。

### 3.2、gradle build和assemble

根据gradle tasks描述，build是assemble+check的集合，assemble会构建apk，概括如下：
assemble：这个task用于组合项目中的所有输出。
check：这个task用于执行所有检查。
connectedCheck：这个task将会在一个指定的设备或者模拟器上执行检查，它们可以同时在所有连接的设备上执行。
deviceCheck：通过APIs连接远程设备来执行检查，这是在CL服务器上使用的。
build：这个task执行assemble和check的所有工作。
clean：这个task清空项目的所有输出。

### 3.3、多渠道打包

**1）Android原生gradle打包方式**
Android 原生多渠道打包主要利用gradle productFlavors实现，当渠道数较多时，可能会存在打包效率低下问题，示例如下：

```groovy
//在app对应gradle脚本中配置
android {
    ...
    //可定义多个维度
    flavorDimensions "mark"
    //多个渠道
    productFlavors {
        wandoujia{ dimension "mark" }
        xiaomi{ dimension "mark" }
    }
    productFlavors.all {
        //注入值到Manifest中的meta-data
        //多个值用逗号分隔，如[UMENG_CHANNEL: name, JPUSH_CHANNEL: name]
        flavor -> flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]
    }
    //自定义打包时apk名字
    android.applicationVariants.all { variant ->
        variant.outputs.all {
            // abc_渠道名_版本名.apk  还可以拼接其他app内容如：variant.versionCode  variant.buildType.name
            outputFileName = "app_${variant.name}_${variant.versionName}_${new Date().format("yyyy-MM-dd")}.apk"
        }
    }
}
```


对应AndroidManifest.xml meta-data声明示例如下：

```xml
<!--变量采用${变量名}这样来替换-->
<meta-data
	android:name="UMENG_CHANNEL"
	android:value="${UMENG_CHANNEL_VALUE}" />
```

```java
try {
    //android 原生gradle获取渠道信息，可根据获取的渠道信息处理不同的业务逻辑
    ApplicationInfo appInfo = getPackageManager().getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA);
    //获取值，如app.gradle文件中productFlavors部分声明的：wandoujia、xiaomi等
    String value = appInfo.metaData.getString("UMENG_CHANNEL");
    System.out.println("gradle channel manifest value : " + value);
} catch (Exception e) {
    e.printStackTrace();
}
```

**2）使用三方框架Walle打包**

尽管原生gradle可以实现多渠道打包，但是当渠道数较多时，打包效率低，耗时较长。此时借助三方框架Walle可以提示打包效率。
Walle多渠道打包示例如下：

```groovy
//在Project对应gradle脚本中配置
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.meituan.android.walle:plugin:1.1.6'
    }
}

//在app对应gradle脚本中配置
apply plugin: 'walle'
dependencies {
    ...
    implementation 'com.meituan.android.walle:library:1.1.6'
    ...
}
...
walle {
    // 指定渠道包的输出路径
    apkOutputFolder = new File("${project.buildDir}/outputs/channels");
    // 定制渠道包的APK的文件名称
    apkFileNameFormat = '${appName}-${packageName}-${channel}-${buildType}-v${versionName}-${versionCode}-${buildTime}.apk';
    // 渠道配置文件
    channelFile = new File("${project.getProjectDir()}/channel")
}
```

在app对应同级目录下（即app根目录下）创建channel文件（注意无后缀，同上述渠道配置文件处声明的一致即可），并在文件中逐行添加渠道信息。如下图所示：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_walle_example1.png?raw=true" alt="android_walle_example1.png" style="zoom:60%;" />

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_walle_example2.png?raw=true" alt="android_walle_example2.png" style="zoom:77%;" />

```java
//Walle获取渠道信息，可根据获取的渠道信息处理不同的业务逻辑
//获取值，如channel文件中声明的：Qh360、Yyb等
String channel = WalleChannelReader.getChannel(this.getApplicationContext());
System.out.println("walle channel: " + channel);
```

### 3.4、v1和v2签名

**v1签名**(jarsigner)：META-INF目录用来存放签名信息文件，因此不对此目录进行签名校验。
**v2签名**(APK Signature Scheme v2)：验证压缩文件的所有字节。
Android7以上先验证v2签名，v2不存在则验证v1签名。
**zipalign**：zip对齐，因为APK包的本质是一个zip压缩文档，经过边界对齐方式优化能使包内未压缩的数据有序的排列，从而减少应用程序运行时的内存消耗 ，通过空间换时间的方式提高执行效率（zipalign后的apk包体积增大了90KB左右）。
**proguard**：此工具是用于压缩，优化，混淆我们的代码，主要作用是可以移除代码中的无用类，字段，方法和属性。缩小apk的体积，增加项目被反编译的难度。

## 4、Android逆向

Android逆向一般过程如下：

apktool反编译 --> 修改smaili汇编 --> 重新打包 --> 签名

### 4.1、对抗反编译工具

对抗反编译工具：即让apk或dex文件无法正常被相关工具进行反编译。常见的反编译工具如：apktool、dex2jar等。

### 4.2、对抗Android模拟器

一些情况下对Android进行逆向时，可能会将APP跑在Android模拟器上而非真机上，此时检测如果Android APP运行在模拟器上，则退出。一般可通过如下方式检测当前运行环境是为模拟器：
1）检测是否包含模拟器特定文件。
2）检测默认电话号码。
3）检测设备ID或IMSI。
4）检测手机硬件信息。
5）检测运营商信息。

### 4.3、对抗APK重打包

APK重打包是指：利用反编译工具得到smali代码，修改相关逻辑后，又由smali重新打包为APK文件。
一般情况下重打包的APK签名会变化，可以根据应用程序签名信息校验当前APP的签名是否变化，从而来对抗重打包。

### 4.4、IDA动态调试

Android APP编译执行过程：
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_ida_example1.png?raw=true" alt="android_ida_example1.png" style="zoom:67%;" />

### 4.5、对抗IDA动态调试

### 4.6、加壳

## 5、架构相关

### 5.1、设计模式

**MVC/MVP模式**

**MVC**：实体层Model（ 应用程序的主体部分，所有业务逻辑都应写在该层），视图层View（程序的 UI 界面，用于向用户展示数据以及接收用户的输入，如对应布局xml文件），以及控制层Controller（控制器用于更新 UI 界面和数据实例，如对应Activity）。

> View 层接受用户的输入，然后通过 Controller 修改对应的 Model 实例；同时，当 Model 实例的数据发生变化的时候，需要修改 UI 界面，可以通过 Controller 更新界面。（View 层也可以直接更新 Model 实例的数据，而不用每次都通过 Controller，这样对于一些简单的数据更新工作会变得方便许多。）

**MVP**：按照 MVC 的分层，Activity 和 Fragment（后面只说 Activity）应该属于 View 层，用于展示 UI 界面，以及接收用户的输入，此外还要承担一些生命周期的工作。Activity 是在 Android 开发中充当非常重要的角色，特别是 它的生命周期的功能，所以开发的时候我们经常把一些业务逻辑直接写在 Activity 里面，这非常直观方便，代价就是 Activity 会越来越臃肿，超过 1000 行代码是常有的事，而且如果是一些可以通用的业务逻辑（比如用户登录），写在具体的 Activity 里就意味着这个逻辑不能复用了。如果有进行代码重构经验的人，看到 1000 + 行的类肯定会有所顾虑。因此，Activity 不仅承担了 View 的角色，还承担了一部分的 Controller 角色，这样一来 V 和 C 就耦合在一起了，虽然这样写方便，但是如果业务调整的话，要维护起来就难了，而且在一个臃肿的 Activity 类查找业务逻辑的代码也会非常麻烦，所以看起来有必要在 Activity 中，把 View 和 Controller 抽离开来，而这就是 MVP 模式的工作了。

> MVP 把 Activity 中的 UI 逻辑抽象成 View 接口，把业务逻辑抽象成 Presenter 接口，Model 类还是原来的 Model。

> MVC和MVP最大区别：MVC中，View 层也可以直接更新 Model 实例的数据，而不用每次都通过 Controller；但是MVP中View 并不能直接对Model进行操作。

*其他：Android 常规的开发模式经常被称为 MV 模式（Model-View），引入数据绑定后的 MVVM 模式（Model-View-ViewModel），与MVP模式稍有区别。*

MVP示例：
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_mvp_example.png?raw=true" alt="android_mvp_example.png" style="zoom:67%;" />

```java
public interface BasePresenter {}

public interface BaseView<T> {}

public interface IMainContract {
    interface View extends BaseView<MainPresenterImpl> {
        void showToast(String str);
    }

    interface Presenter extends BasePresenter {
        void testMethod1();
        int retMethod2();
    }
}

//契约类，关联view和model业务逻辑
//每个契约类对应一个Activity模块，逻辑清晰
public class MainPresenterImpl implements IMainContract.Presenter {
    private IMainContract.View view;
    public MainPresenterImpl(IMainContract.View view) {
        this.view = view;
    }

    @Override
    public void testMethod1() {
        view.showToast("llall");
    }

    @Override
    public int retMethod2() {
        return 0;
    }
}

//Activity在MVC中作为Control，实际上也是View的一部分（MVP中也是如此）
//在此实现View接口方法，然后绑定Presenter
public class MainActivity extends BaseActivity implements IMainContract.View {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout_test);
        MainPresenterImpl presenter = new MainPresenterImpl(this);

        findViewById(R.id.btnTest).setOnClickListener(v -> {
            presenter.testMethod1();
        });
    }

    @Override
    public void showToast(String str) {
        Toast.makeText(this, str, Toast.LENGTH_LONG).show();
    }
}
```

### 5.2、国内镜像加速

使用国内公共代理镜像仓库，可以加速依赖库的下载编译。



### 5.3、私有maven仓库(nexus oss)

创建私有的maven仓库，可以便于进行项目包管理。



## 6、开源框架相关

### 6.1、Glide



### 6.2、EventBus

是一款在 Android 开发中使用的**发布/订阅事件总线框架，基于观察者模式**，将事件的接收者和发送者分开，简化了组件之间的通信，使用简单、效率高、体积小！

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_eventbus.png?raw=true" alt="android_eventbus.png" style="zoom:44%;" />


```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	EventBus.getDefault().register(this);//注册EventBus
    ...
    EventBus.getDefault().post(new Object());//发送消息事件
}
@Override
protected void onDestroy() {
    EventBus.getDefault().unregister(this);//反注册
    ...
}
//注解接收事件
@Subscribe(threadMode = ThreadMode.MAIN)
public void test(Object object) {//必须有参数
    Toast.makeText(this, "receive : "+ object, Toast.LENGTH_LONG).show();
}
//ThreadMode.POSTING，默认的线程模式，在那个线程发送事件就在对应线程处理事件，避免了线程切换，效率高。
//ThreadMode.MAIN，如在主线程（UI线程）发送事件，则直接在主线程处理事件；如果在子线程发送事件，则先将事件入队列，然后通过 Handler 切换到主线程，依次处理事件。
//ThreadMode.MAIN_ORDERED，无论在那个线程发送事件，都先将事件入队列，然后通过 Handler 切换到主线程，依次处理事件。
//ThreadMode.BACKGROUND，如果在主线程发送事件，则先将事件入队列，然后通过线程池依次处理事件；如果在子线程发送事件，则直接在发送事件的线程处理事件。
//ThreadMode.ASYNC，无论在那个线程发送事件，都将事件入队列，然后通过线程池处理。
```


**post和postSticky区别**

**post**：发送普通消息事件，接收者需要在register注册后才会收到消息事件。
**postSticky**：发送粘性消息事件，接收者在register之后仍可以收到之前发送的消息事件。类似于Android中的粘性广播。

```java
//通过下述方法可以移除Map中缓存的Sticky粘性消息。
EventBus.getDefault().removeAllStickyEvents();
EventBus.getDefault().removeStickyEvent(Object);
```

### 6.3、LeakCanary

LeakCanary可用于Android内存泄露检测。

LeakCanary原理

>- 当一个Activity Destory之后，将它放在一个WeakReference弱引用中
>- 把这个WeakReference关联到一个ReferenceQueue
>- 查看ReferenceQueue中是否存在Activity的引用
>- 如果Activity泄露了，就Dump出heap信息，然后去分析内存泄露的路径

### 6.4、ARouter

一个用于帮助Android App进行组件化改造的框架（支持模块间的路由、通信、解耦）。ARouter直接翻译过来就是路由，可以用来映射页面关系，实现跳转相关的功能。在Android中，常被用来进行组件化通讯。

```java
//部分示例
//普通跳转
ARouter.getInstance().build("/activity/test/turn").navigation();
//跳转并携带参数
ARouter.getInstance().build("/activity/test/turn")
					.withString("key2", "String888")
					.navigation();
//通过URL跳转
Uri testUriMix = Uri.parse("arouter://myapphost/activity/test/turn");
ARouter.getInstance().build(testUriMix)
    .withString("key2", "value1")
    .navigation();
```

**Android默认提供了跳转（如startActivity），为什么还要使用ARouter？**
1）在一些复杂的业务场景下（如电商），可提前做好页面映射（URL跳转），方便配置管理。
2）业务逻辑存在大量应用跳转时（如多个页面需要身份认证后才可跳转），可统一配置跳转逻辑（如拦截器）。

```java
//拦截器示例
//priority表示拦截器优先级，数值越大优先级越低，如1>2>3
@Interceptor(name = "login", priority = 3)
public class LoginInterceptorImpl implements IInterceptor {
    public void process(Postcard postcard, InterceptorCallback callback) {
        String path = postcard.getPath();
        System.out.println("Interceptor process : " + path);
        if (isNotLogin()) {
            // 如果没有登录
            if (path.equals("/activity/test/turn2")) {//例如/activity/test/turn2此路径表示登录界面
                callback.onContinue(postcard);//跳到其他界面（如登录界面）
            } else {// 需要登录的直接拦截下来
                callback.onInterrupt(null);//中断跳转
            }
        } else {
            // 如果已经登录不拦截
            callback.onContinue(postcard);
        }
    }

    public void init(Context context) {
        //此方法只会走一次
    }
    private boolean isNotLogin() {//例如判断是否已登录
        return true;
    }
}

//访问目标/activity/test/turn
ARouter.getInstance().build("/activity/test/turn")
    .withString("param", "传递参数")
    .navigation(TestDataBindingActivity.this, new NavCallback() {
        @Override
        public void onArrival(Postcard postcard) {
            System.out.println("onArrival : " + postcard.toString());
        }

        @Override
        public void onInterrupt(Postcard postcard) {//被拦截器拦截了
            String path = postcard.getPath();
            System.out.println("onInterrupt : " + path);
            Bundle bundle = postcard.getExtras();
            //处理拦截逻辑，如此处跳转另外的界面（如：可以是登录界面/activity/test/turn2）
            ARouter.getInstance().build("/activity/test/turn2")
                .with(bundle)
                .withString("path", path)
                .navigation();
        }
    });
```

更多ARouter使用参考：https://github.com/alibaba/ARouter/blob/master/README_CN.md

### 6.5、增林更新/热修复Tinker

Tinker是腾讯的开源的Android平台热修复（hot-fix）方案，支持在无需升级APK的前提下更新 dex, library and resources 文件。

#### 6.5.1、增量更新

> **注意：Android增量更新与热修复是两种不同的概念。**
>
> **热修复**一般是用于当已经发布的app有Bug需要修复的时候，开发者修改代码并发布补丁，让应用能够在**不需要重新安装APK**的情况下实现更新，主流方案有[Tinker](https://github.com/Tencent/tinker)、[AndFix](https://github.com/alibaba/AndFix)等。
> **增量更新**的目的是为了减少更新app所需要下载的包体积大小，常见如手机端游戏，apk包体积为几百M，但有时更新只需下载十几M的安装包即可完成更新，**需要重新安装APK**。
> **增量更新一般采用bsdiff在Linux服务器端生成查分补丁包，移动端下载patch差分文件后，通过bspatch来合成新的安装包即可（架构示意图如下图所示）。bsdiff工具见 http://www.daemonology.net/bsdiff/**
>
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_bsdiff.png?raw=true" alt="android_bsdiff.png" style="zoom:60%;" />
>
> **补充1：bsdiff编译注意事项**
> bsdiff编译依赖于bzip2组件，需要安装相关依赖包（如CentOS下：yum install bzip2 bzip2-devel，或下载bzip2-1.0.6.tar.gz独立包进行编译安装），将bsdiff解压后，编辑其对应Makefile文件（此文件中原有排版格式有误，需要对文件中.ifndef和.endif两行进行缩进，否则可能无法正常编译）并编译即可（编译后生成bsdiff和bspatch工具）。
>
> **补充2：Android NDK集成bspatch功能步骤及注意事项**
> 1）将bzip2库中的相关头文件和源码文件放入Android工程cpp目录下的bzip2文件夹，将解压后的bsdiff源码bspatch.c同样放置于cpp目录下（工程结构示意图如下图所示），且对bspatch.c文件进行如下修改：
> <img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_bspatch.png?raw=true" alt="android_bspatch.png" style="zoom:55%;" />
>
> ```c++
> //正确引入相关文件
> #include "bzip2/bzlib.h"
> #include "bzip2/bzlib.c"
> #include "bzip2/compress.c"
> #include "bzip2/decompress.c"
> #include "bzip2/crctable.c"
> #include "bzip2/randtable.c"
> #include "bzip2/huffman.c"
> #include "bzip2/blocksort.c"
> 
> //将main函数修改为其他名称，如下
> //int main(int argc,char * argv[])
> int bs_patch(int argc,char * argv[] {
>  ...
>  //注意bspatch.c中fopen方法，在android设备上打开文件可能会出错，返回NULL
>  //此问题多半是因为Android文件读写权限问题，特别是Android 10及以上版本，SD卡存储区域权限产生了变更，参考android scoped storage策略
>  ...
> }
> ```
>
> 2）编写相关JNI cpp代码
>
> ```c++
> extern "C" {
> 	int bs_patch(int argc, char *argv[]);//声明bs_patch方法，即调用了bspatch.c中的bs_patch方法
> }
> //JNI方法
> extern "C"
> JNIEXPORT jint JNICALL
> Java_com_test_MainActivity_patch(JNIEnv *env, jclass clazz,
>                               jstring oldAPKPath_,
>                               jstring newAPKPath_,
>                               jstring patchPath_) {
>      LOGD("Call patch native...");
>      //获取相关文件路径信息
>      const char *oldAPKPath = env->GetStringUTFChars(oldAPKPath_, 0);
>      const char *newAPKPath = env->GetStringUTFChars(newAPKPath_, 0);
>      const char *patchPath = env->GetStringUTFChars(patchPath_, 0);
>      int argc = 4;
>      char* argv[4];
>      argv[0] = "bspatch";
>      argv[1] = const_cast<char *>(oldAPKPath);
>      argv[2] = const_cast<char *>(newAPKPath);
>      argv[3] = const_cast<char *>(patchPath);
>      LOGD("1 : %s", argv[1]);
>      LOGD("2 : %s", argv[2]);
>      LOGD("3 : %s", argv[3]);
>      int ret = bs_patch(argc, argv);
>      LOGD("ret %d", ret);
>      //释放内存
>      env->ReleaseStringUTFChars(oldAPKPath_, oldAPKPath);
>      env->ReleaseStringUTFChars(newAPKPath_, newAPKPath);
>      env->ReleaseStringUTFChars(patchPath_, patchPath);
>      return ret;
> }
> ```
>
> 3）对CMakeLists.txt进行如下修改，以便编译动态库。
>
> ```cmake
> add_library(
>           native-lib
>           SHARED
>           bspatch.c #编译bspatch
>           native-lib.cpp)#编译JNI cpp代码
> ```
>
> 根据上述两步编译生成相关so，并在Java中调用相关ndk代码合成新的安装包并安装即可，示例如下：
>
> ```java
> //声明native方法
> public static native int patch(String oldApk, String newApk, String patch);
> //新旧安装包与不定文件的路径
> //此处文件路径注意权限问题，特别是Android 10以上android scoped storage策略
> final File destApk = new File("/data/data/com.test.myapp/target_new.apk");
> final File patch = new File("/data/data/com.test.myapp/my.patch");
> //调用native代码，合成新的APK，合成完成后安装新的APK即可
> patch(getApplicationInfo().sourceDir,//通过此方法可以获取当前APP的apk路径（即可视为旧的APK，与patch文件合成后可以产生新的APK）
>     destApk.getAbsolutePath(), patch.getAbsolutePath());
> //安装合成后的新的APK
> ...
> ```

#### 6.5.2、热更新Tinker

Tinker是腾讯开源的Android APP热修复方案。要正确配置使用Tinker一般可按如下步骤配置：

1）配置gradle

```groovy
//1、配置Project对应build.gradle
buildscript {
    ...
    dependencies {
        ...
        classpath "com.tencent.tinker:tinker-patch-gradle-plugin:1.9.14" //添加Tinker gradle插件
    }
}

//2、配置app对应build.gradle
android {
    ...
    defaultConfig {
        ...
        multiDexEnabled false
        multiDexKeepProguard file("tinker_multidexkeep.pro") //参考tinker混淆文件
        ...
        buildConfigField "String", "MESSAGE", "\"I am the base apk\""
        buildConfigField "String", "TINKER_ID", "\"${getTinkerIdValue()}\""
        buildConfigField "String", "PLATFORM", "\"all\""
        ...
    }
    signingConfigs {
        release {
            try {
                storeFile file("./keystore/release.keystore")
                storePassword "testres"
                keyAlias "testres"
                keyPassword "testres"
            } catch (ex) {
                throw new InvalidUserDataException(ex.toString())
            }
        }

        debug {
            storeFile file("./keystore/debug.keystore")
        }
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
        debug {
            debuggable true
            minifyEnabled false
            signingConfig signingConfigs.debug
        }
    }
    dexOptions {
        //支持大型项目
        jumboMode = true
    }
    packagingOptions {
        exclude "/META-INF/**"
    }
}
dependencies {
    ...
    implementation 'androidx.multidex:multidex:2.0.1'
    ...
    //Tinker
    def tinker_version = "1.9.14"
    api("com.tencent.tinker:tinker-android-lib:$tinker_version") { changing = true }
    implementation("com.tencent.tinker:tinker-android-loader:$tinker_version") { changing = true }
    annotationProcessor("com.tencent.tinker:tinker-android-anno:$tinker_version") { changing = true }
    compileOnly("com.tencent.tinker:tinker-android-anno:$tinker_version") { changing = true }
    ...
}

//3、Tinker详细配置
//Tinker相关
def bakPath = file("${buildDir}/bakApk/")
/**
 * 使用assembleRelease构建基准APK
 * 使用tinkerPatchRelease构建补丁包
 */
ext {
    //是否使用Tinker(当你的项目处于开发调试阶段时，可以改为false)
    tinkerEnabled = true
    //基础包文件路径（名字这里写死为old-app.apk。用于比较新旧app以生成补丁包，不管是debug还是release编译）
    tinkerOldApkPath = "${bakPath}/old-app.apk"
    //基础包的mapping.txt文件路径
    //用于辅助混淆补丁包的生成，一般在生成release版app时会使用到混淆，所以这个mapping.txt文件一般只是用于release安装包补丁的生成
    tinkerApplyMappingPath = "${bakPath}/old-app-mapping.txt"
    //基础包的R.txt文件路径
    //如果你的安装包中资源文件有改动，则需要使用该R.txt文件来辅助生成补丁包
    tinkerApplyResourcePath = "${bakPath}/old-app-R.txt"
    //only use for build all flavor, if not, just ignore this field
    tinkerBuildFlavorDirectory = "${bakPath}/flavor"
}
def getOldApkPath() {
    return hasProperty("OLD_APK") ? OLD_APK : ext.tinkerOldApkPath
}
def getApplyMappingPath() {
    return hasProperty("APPLY_MAPPING") ? APPLY_MAPPING : ext.tinkerApplyMappingPath
}
def getApplyResourceMappingPath() {
    return hasProperty("APPLY_RESOURCE") ? APPLY_RESOURCE : ext.tinkerApplyResourcePath
}
def getTinkerIdValue() {
    return hasProperty("TINKER_ID") ? TINKER_ID : android.defaultConfig.versionName
}
def buildWithTinker() {
    return hasProperty("TINKER_ENABLE") ? Boolean.parseBoolean(TINKER_ENABLE) : ext.tinkerEnabled
}
def getTinkerBuildFlavorDirectory() {
    return ext.tinkerBuildFlavorDirectory
}

if (buildWithTinker()) {
    apply plugin: 'com.tencent.tinker.patch'
    tinkerPatch {
        //必要配置，默认null，指定旧版本APK路径
        oldApk = getOldApkPath()
        /**
         * 可选配置，默认 'false'
         * 如果出现以下的情况，并且ignoreWarning为false，编译将被中断。因为这些情况可能会导致编译出来的patch包带来风险：
         * 1. minSdkVersion小于14，但是dexMode的值为"raw";
         * 2. 新编译的安装包出现新增的四大组件(Activity, BroadcastReceiver...)；
         * 3. 定义在dex.loader用于加载补丁的类不在main dex中;
         * 4. 定义在dex.loader用于加载补丁的类出现修改；
         * 5. resources.arsc改变，但没有使用applyResourceMapping编译。
         */
        ignoreWarning = false
        /**
         * 可选配置，默认 'true'
         * 在运行过程中，需要验证基准apk包与补丁包的签名是否一致，是否需要为你签名。
         */
        useSign = true
        /**
         * 可选配置，默认 'true'
         * 是否打开tinker的功能
         */
        tinkerEnable = buildWithTinker()
        //Warning, applyMapping will affect the normal android build!
        buildConfig {//编译相关的配置项
            /**
             * 可选配置，默认 'null'
             * 在编译新的apk时候，我们希望通过保持旧apk的proguard混淆方式，
             * 从而减少补丁包的大小。这个只是推荐设置，
             * 不设置applyMapping也不会影响任何的assemble编译。
             */
            applyMapping = getApplyMappingPath()
            /**
             * 可选配置，默认 'null'
             * 在编译新的apk时候，我们希望通过旧apk的R.txt文件保持ResId的分配，
             * 这样不仅可以减少补丁包的大小，同时也避免由于ResId改变导致remote view异常。
             */
            applyResourceMapping = getApplyResourceMappingPath()
            /**
             * 必要配置，默认 'null'
             * 在运行过程中，我们需要验证基准apk包的tinkerId是否等于补丁包的tinkerId。
             * 这个是决定补丁包能运行在哪些基准包上面，一般来说我们可以使用git版本号、versionName等等。
             */
            tinkerId = getTinkerIdValue()
            /**
             * 必要配置，默认值 false
             * 如果我们有多个dex,编译补丁时可能会由于类的移动导致变更增多。
             * 若打开keepDexApply模式，补丁包将根据基准包的类分布来编译。
             */
            keepDexApply = false
            /**
             * 可选配置, 默认 'false'
             * 是否使用加固模式，仅仅将变更的类合成补丁。注意，这种模式仅仅可以用于加固应用中。
             */
            isProtectedApp = false
            /**
             * 可选配置, 默认 'false'
             * 是否支持新增非export的Activity
             */
            supportHotplugComponent = false
        }

        dex {//	dex相关的配置项
            /**
             * 可选配置，默认 'jar'
             * 参数值只能是'raw'或者'jar'。
             * 对于'raw'模式，我们将会保持输入dex的格式。
             * 对于'jar'模式，我们将会把输入dex重新压缩封装到jar。
             *      如果你的minSdkVersion小于14，你必须选择‘jar’模式，
             *      而且它更省存储空间，但是验证md5时比'raw'模式耗时。
             *      默认我们并不会去校验md5,一般情况下选择jar模式即可。
             */
            dexMode = "jar"
            /**
             * 必要配置，默认 '[]'
             * 需要处理dex路径，支持*、?通配符，必须使用'/'分割。路径是相对安装包的，例如assets/...
             */
            pattern = ["classes*.dex",
                       "assets/secondary-dex-?.jar"]
            /**
             * 必要配置，默认 '[]'
             * 这一项非常重要，它定义了哪些类在加载补丁包的时候会用到。这些类是通过Tinker无法修改的类，也是一定要放在main dex的类。
             * 这里需要定义的类有：
             * 1. 你自己定义的Application类；
             * 2. Tinker库中用于加载补丁包的部分类，即com.tencent.tinker.loader.*；
             * 3. 如果你自定义了TinkerLoader，需要将它以及它引用的所有类也加入loader中；
             * 4. 其他一些你不希望被更改的类，例如Sample中的BaseBuildInfo类。这里需要注意的是，这些类的直接引用类也需要加入到loader中。或者你需要将这个类变成非preverify。
             * 5. 使用1.7.6版本之后的gradle版本，参数1、2会自动填写。若使用newApk或者命令行版本编译，1、2依然需要手动填写
             */
            loader = [
                    "com.test.mytestandroidapplication.MyTinkerApplication"
            ]
        }

        lib {//lib相关的配置项
            /**
             * 可选配置，默认 '[]'
             * 需要处理lib路径，支持*、?通配符，必须使用'/'分割。
             * 与dex.pattern一致, 路径是相对安装包的，例如assets/...
             */
            pattern = ["lib/*/*.so"]
        }

        res {//res相关的配置项
            /**
             * 可选配置，默认 '[]'
             * 需要处理res路径，支持*、?通配符，必须使用'/'分割。
             * 与dex.pattern一致, 路径是相对安装包的，例如assets/...，
             * 务必注意的是，只有满足pattern的资源才会放到合成后的资源包。
             */
            pattern = ["res/*", "assets/*", "resources.arsc", "AndroidManifest.xml"]
            /**
             * 可选配置，默认 '[]'
             * 支持*、?通配符，必须使用'/'分割。
             * 若满足ignoreChange的pattern，在编译时会忽略该文件的新增、删除与修改。
             * 最极端的情况，ignoreChange与上面的pattern一致，即会完全忽略所有资源的修改。
             */
            ignoreChange = ["assets/sample_meta.txt"]
            /**
             * default 100kb
             * 对于修改的资源，如果大于largeModSize，我们将使用bsdiff算法。
             * 这可以降低补丁包的大小，但是会增加合成时的复杂度。默认大小为100kb
             */
            largeModSize = 100
        }

        packageConfig {//用于生成补丁包中的'package_meta.txt'文件
            /**
             * 可选配置，如 'TINKER_ID, TINKER_ID_VALUE' 'NEW_TINKER_ID, NEW_TINKER_ID_VALUE'
             * configField("key", "value"),
             * 默认我们自动从基准安装包与新安装包的Manifest中读取tinkerId,
             * 并自动写入configField。在这里，你可以定义其他的信息，
             * 在运行时可以通过TinkerLoadResult.getPackageConfigByName得到相应的数值。
             * 但是建议直接通过修改代码来实现，例如BuildConfig。
             */
            configField("patchMessage", "tinker is sample to use")
            configField("platform", "all")
            configField("patchVersion", "1.0")
        }

        sevenZip {//7zip路径配置项，执行前提是useSign为true
            /**
             * 可选配置，default '7za'
             * 例如"com.tencent.mm:SevenZip:1.1.10"，将自动根据机器属性获得对应的7za运行文件，推荐使用。
             */
            zipArtifact = "com.tencent.mm:SevenZip:1.1.10"
            /**
             * 可选位置，default '7za'
             * 系统中的7za路径，例如"/usr/local/bin/7za"。path设置会覆盖zipArtifact，若都不设置，将直接使用7za去尝试。
             */
            //path = "/usr/local/bin/7za"
        }
    }

    List<String> flavors = new ArrayList<>();
    project.android.productFlavors.each { flavor ->
        flavors.add(flavor.name)
    }
    boolean hasFlavors = flavors.size() > 0
    def date = new Date().format("MMdd-HH-mm-ss")
    /**
     * bak apk and mapping
     */
    android.applicationVariants.all { variant ->
        /**
         * task type, you want to bak
         */
        def taskName = variant.name
        tasks.all {
            if ("assemble${taskName.capitalize()}".equalsIgnoreCase(it.name)) {

                it.doLast {
                    copy {
                        def fileNamePrefix = "${project.name}-${variant.baseName}"
                        def newFileNamePrefix = hasFlavors ? "${fileNamePrefix}" : "${fileNamePrefix}-${date}"

                        def destPath = hasFlavors ? file("${bakPath}/${project.name}-${date}/${variant.flavorName}") : bakPath

                        if (variant.metaClass.hasProperty(variant, 'packageApplicationProvider')) {
                            def packageAndroidArtifact = variant.packageApplicationProvider.get()
                            if (packageAndroidArtifact != null) {
                                try {
                                    from new File(packageAndroidArtifact.outputDirectory.getAsFile().get(), variant.outputs.first().apkData.outputFileName)
                                } catch (Exception e) {
                                    from new File(packageAndroidArtifact.outputDirectory, variant.outputs.first().apkData.outputFileName)
                                }
                            } else {
                                from variant.outputs.first().mainOutputFile.outputFile
                            }
                        } else {
                            from variant.outputs.first().outputFile
                        }

                        into destPath
                        rename { String fileName ->
                            fileName.replace("${fileNamePrefix}.apk", "${newFileNamePrefix}.apk")
                        }

                        from "${buildDir}/outputs/mapping/${variant.dirName}/mapping.txt"
                        into destPath
                        rename { String fileName ->
                            fileName.replace("mapping.txt", "${newFileNamePrefix}-mapping.txt")
                        }

                        from "${buildDir}/intermediates/symbols/${variant.dirName}/R.txt"
                        from "${buildDir}/intermediates/symbol_list/${variant.dirName}/R.txt"
                        from "${buildDir}/intermediates/runtime_symbol_list/${variant.dirName}/R.txt"
                        into destPath
                        rename { String fileName ->
                            fileName.replace("R.txt", "${newFileNamePrefix}-R.txt")
                        }
                    }
                }
            }
        }
    }
    project.afterEvaluate {
        //sample use for build all flavor for one time
        if (hasFlavors) {
            task(tinkerPatchAllFlavorRelease) {
                group = 'tinker'
                def originOldPath = getTinkerBuildFlavorDirectory()
                for (String flavor : flavors) {
                    def tinkerTask = tasks.getByName("tinkerPatch${flavor.capitalize()}Release")
                    dependsOn tinkerTask
                    def preAssembleTask = tasks.getByName("process${flavor.capitalize()}ReleaseManifest")
                    preAssembleTask.doFirst {
                        String flavorName = preAssembleTask.name.substring(7, 8).toLowerCase() + preAssembleTask.name.substring(8, preAssembleTask.name.length() - 15)
                        project.tinkerPatch.oldApk = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release.apk"
                        project.tinkerPatch.buildConfig.applyMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release-mapping.txt"
                        project.tinkerPatch.buildConfig.applyResourceMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-release-R.txt"
                    }
                }
            }

            task(tinkerPatchAllFlavorDebug) {
                group = 'tinker'
                def originOldPath = getTinkerBuildFlavorDirectory()
                for (String flavor : flavors) {
                    def tinkerTask = tasks.getByName("tinkerPatch${flavor.capitalize()}Debug")
                    dependsOn tinkerTask
                    def preAssembleTask = tasks.getByName("process${flavor.capitalize()}DebugManifest")
                    preAssembleTask.doFirst {
                        String flavorName = preAssembleTask.name.substring(7, 8).toLowerCase() + preAssembleTask.name.substring(8, preAssembleTask.name.length() - 13)
                        project.tinkerPatch.oldApk = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug.apk"
                        project.tinkerPatch.buildConfig.applyMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug-mapping.txt"
                        project.tinkerPatch.buildConfig.applyResourceMapping = "${originOldPath}/${flavorName}/${project.name}-${flavorName}-debug-R.txt"
                    }
                }
            }
        }
    }
}
task sortPublicTxt() {
    doLast {
        File originalFile = project.file("public.txt")
        File sortedFile = project.file("public_sort.txt")
        List<String> sortedLines = new ArrayList<>()
        originalFile.eachLine {
            sortedLines.add(it)
        }
        Collections.sort(sortedLines)
        sortedFile.delete()
        sortedLines.each {
            sortedFile.append("${it}\n")
        }
    }
}
```

2）添加Java代码

参考官方示例https://github.com/Tencent/tinker/tree/dev/tinker-sample-android，添加如下Java文件：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_tinker_example.png?raw=true" alt="android_tinker_example.png" style="zoom:55%;" />

```java
@DefaultLifeCycle(application = "com.test.mytestandroidapplication.MyTinkerApplication",//application类名。只能用字符串（需填写完整包名路径），这个MyApplication文件是不存在的，但可以在AndroidManifest.xml的application标签上使用（name）
        flags = ShareConstants.TINKER_ENABLE_ALL,// tinkerFlags
        loaderClass = "com.tencent.tinker.loader.TinkerLoader",//loaderClassName, 我们这里使用默认即可!（可不写）
        loadVerifyFlag = false)//tinkerLoadVerifyFlag
public class TinkerApplicationLike extends DefaultApplicationLike {
    private static final String TAG = "Tinker.SampleApplicationLike";
    public TinkerApplicationLike(Application application, int tinkerFlags, boolean tinkerLoadVerifyFlag,
                                 long applicationStartElapsedTime, long applicationStartMillisTime, Intent tinkerResultIntent) {
        super(application, tinkerFlags, tinkerLoadVerifyFlag, 
              applicationStartElapsedTime, applicationStartMillisTime, tinkerResultIntent);
    }

    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    @Override
    public void onBaseContextAttached(Context base) {
        super.onBaseContextAttached(base);
        //you must install multiDex whatever tinker is installed!
        MultiDex.install(base);

        SampleApplicationContext.application = getApplication();
        SampleApplicationContext.context = getApplication();
        TinkerManager.setTinkerApplicationLike(this);

        TinkerManager.initFastCrashProtect();
        //should set before tinker is installed
        TinkerManager.setUpgradeRetryEnable(true);

        //optional set logIml, or you can use default debug log
        TinkerInstaller.setLogIml(new MyLogImp());

        //installTinker after load multiDex
        //or you can put com.tencent.tinker.** to main dex
        TinkerManager.installTinker(this);
        Tinker tinker = Tinker.with(getApplication());
        // 可以将之前自定义的Application中onCreate()方法所执行的操作搬到这里...
    }

    @TargetApi(Build.VERSION_CODES.ICE_CREAM_SANDWICH)
    public void registerActivityLifecycleCallbacks(Application.ActivityLifecycleCallbacks callback) {
        getApplication().registerActivityLifecycleCallbacks(callback);
    }
}

//在工程Manifest中引用application name=".MyTinkerApplication"
```

3）生成补丁并应用补丁

确保旧的apk处于app/build/bakApk/目录下，执行tinkerPatchDebug操作生成补丁包（位于app/build/outputs/apk/tinkerPatch/debug/目录下），将生成的补丁包拷贝至手机端指定位置执行补丁即可。

```java
//补丁文件地址，实际应用中可将.apk改为.zip等文件格式，避免网络运营商劫持
String path = "/data/data/com.test.mytestandroidapplication/patch.apk";
File file = new File(path);
if (file.exists()) {
    if (file.length() > 0) {
        System.out.println("Tinker onReceiveUpgradePatch...");
        TinkerInstaller.onReceiveUpgradePatch(TestDataBindingActivity.this, file.getAbsolutePath());
    }
}
```

Bugly+Tinker解决方案参考：https://bugly.qq.com/docs/user-guide/instruction-manual-android-hotfix-demo/#top

更多Tinker内容可参考：https://blog.csdn.net/leol_2/article/details/100747377

### 6.6、组件化/插件化

Atlas、VirtualAPK、RePlugin在Android Q版本（及AndroidX模式下）兼容性有待验证。

## 7、Android JetPack

Jetpack是一套由Google推出的库（开发组件）、工具和指南，可帮助开发者更轻松地编写优质应用。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_jetpack.png?raw=true" alt="android_jetpack.png" style="zoom:60%;" />

#### 7.1、Room数据库框架

```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();
    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);
    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);
    @Insert
    void insertAll(User... users);
    @Delete
    void delete(User user);
}

@Entity
public class User {
    @PrimaryKey
    private int uid;
    @ColumnInfo(name = "first_name")
    private String firstName;
    @ColumnInfo(name = "last_name")
    private String lastName;

    //getter setter
    public int getUid() {
        return uid;
    }
    public void setUid(int uid) {
        this.uid = uid;
    }
    public String getFirstName() {
        return firstName;
    }
    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }
    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}

@Database(entities = {User.class}, version = 1,exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

#### 7.2、WorkManager

WorkManager类似于JobService，主要是为了完成计划任务，具有两个特点：
1）**针对不需要及时完成的任务**（异步执行，会根据约束条件选择在合适的时机执行）
2）**保证任务一定会被执行**

WorkManager可以自动维护后台任务，同时可适应不同的条件，同时满足后台Service和静态广播，内部维护着JobScheduler，而在6.0以下系统版本则可自动切换为AlarmManager。

```java
//WorkManager示例
//WorkManager能保证任务会被执行
public class MyWorker extends Worker {
    public MyWorker(@NonNull Context context, @NonNull WorkerParameters workerParams) {
        super(context, workerParams);
    }

    //此方法运行在子线程，在doWork()方法中可执行耗时任务
    @Override
    public Result doWork() {
        System.out.println("doWork...in subThread : " + !(Looper.getMainLooper().getThread() == Thread.currentThread()));
        //接收外面传递进来的数据
        String inputData = getInputData().getString("input_data");
        // 任务执行完成后返回数据
        Data outputData = new Data.Builder().putString("output_data", "Task Success!").build();
        System.out.println("doWork receive data : " + inputData);
        //执行成功返回Result.success()
        //执行失败返回Result.failure()
        //需要重新执行返回Result.retry()
        return Result.success(outputData);
    }
}

 Constraints constraints = new Constraints.Builder()
     .setRequiresCharging(false)
     .setRequiredNetworkType(NetworkType.CONNECTED)
     .setRequiresBatteryNotLow(false)
     .build();
//WorkManager和Worker之间的参数传递。数据的传递通过Data对象来完成
Data inputData = new Data.Builder().putString("input_data", "Hello World!").build();
//一次性任务
//周期性任务使用PeriodicWorkRequest
OneTimeWorkRequest uploadWorkRequest = new OneTimeWorkRequest.Builder(MyWorker.class)
    .setConstraints(constraints)//设置触发条件
    .setInputData(inputData)//通过setInputData()方法向Worker传递数据
    .build();
WorkManager.getInstance(MainActivity.this).enqueue(uploadWorkRequest);
//通过LiveData，我们便可以在任务状态发生变化的时候，收到通知。
WorkManager.getInstance(MainActivity.this).getWorkInfoByIdLiveData(uploadWorkRequest.getId())
    .observe(MainActivity.this, new Observer<WorkInfo>() {
        @Override
        public void onChanged(WorkInfo workInfo) {
            System.out.println("workInfo:" + workInfo);
        }
    });
//WorkManager通过LiveData的WorkInfo.getOutputData()，得到从Worker传递过来的数据。
WorkManager.getInstance(MainActivity.this).getWorkInfoByIdLiveData(uploadWorkRequest.getId())
    .observe(MainActivity.this, new Observer<WorkInfo>() {
        @Override
        public void onChanged(WorkInfo workInfo) {
            if (workInfo != null && workInfo.getState() == WorkInfo.State.SUCCEEDED) {
                String outputData = workInfo.getOutputData().getString("output_data");
                System.out.println("liveData getWorkData : " + outputData);
            }
        }
    });
//取消任务
//WorkManager.getInstance(MainActivity.this).cancelAllWork();
```

#### 7.3、DataBinding

DataBinding 是 Google 在 Jetpack 中推出的一款数据绑定的支持库，利用该库可以实现在页面组件中直接绑定应用程序的数据源。简单示例如下：

```groovy
//gradle中添加databinding支持
android {
    ...
    dataBinding {
        enabled true//添加这个
    }
    ...
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--layout 为跟布局-->
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <!--data 节点中存放数据-->
    <data>
        <!--data 中的 variable 标签为变量，类似于定义了一个变量，
        name 为变量名，type 为变量全限定类型名，包括包名。-->
        <variable name="title" type="java.lang.String" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <!--布局中通过 @{} 来引用这个变量的值-->
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{title}" />

        <Button
            android:id="@+id/btn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Test" />
    </LinearLayout>
</layout>
```

```java
public class TestDataBindingActivity extends Activity {
    //编译文件的命名一般都是通过xml的文件名生成如activity_main.xml，则会生成ActivityMainBinding;
    //item_list_name则会生成ItemListNameBinding。
    ActivityTestDatabindingBinding dataBinding;
    Button btn;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        dataBinding = DataBindingUtil.setContentView(this, R.layout.activity_test_databinding);
        btn = findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dataBinding.setTitle("lalala" + System.currentTimeMillis());
            }
        });

        dataBinding.setTitle("NewTitle" + new Date().toString());
    }
}
```



## 8、kotlin相关

协程

## 9、flutter相关

flutter是跨平台的移动UI框架。使用Dart语言开发，在flutter中，一切皆是组件。

### 9.1、Dart



## 10、理解小程序开发

微信小程序开发
https://developers.weixin.qq.com/miniprogram/dev/framework/

## 11、面试考点举例

### 11.1、ViewHolder，Handler等类为什么要被声明为静态内部类static？

非静态内部类会持有外部类的引用，在一些情况下很可能造成内存泄漏，所以一般声明为静态内部类，但并不是说一定要生命成静态内部类。

### 11.2、Parcelable和Serializable序列化区别？

1）Serializable只需实现Serializable接口即可，而Parcelable需要实现Parcelable接口。
2）Serializable是利用Java的反射机制实现序列化，需要大量IO操作并创建许多的临时对象，容易触发垃圾回收机制，效率不高；而Parcelable不需要反射，且数据也存在内存中，故而效率更高。
3）在内存中传递数据（如Activity跳转）时，建议使用Parcelable，持久化到本地文件存储时，建议使用Serializable，因为Parcelable为了效率完全没有照顾版本间的兼容性，不同版本的系统可能Parcelable兼容性不一样。
4）Serializable中的serialVersionUID用于标记对象属性信息的版本号，只有当serialVersionUID一致的对象才能正常进行序列化和反序列化操作，否则序列化后无法正常被反序列化，会提示对象序列化和反序列化的版本号不同。

```java
//示例：Parcelable用法
public class User implements Parcelable {
    private String name;
    private int age;

    protected User(Parcel in) {
        name = in.readString();
        age = in.readInt();
    }

    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {//从Parcel读取对象
            return new User(in);
        }
        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    //把值写入Parcel中
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
    }
}
```

### 11.3、举例说明常见的内存泄漏？

1）不恰当的使用static变量。
2）忘记关闭各种连接，如IO流、Cursor等。
3）不恰当的内部类，因为内部类持有外部类的引用，当内部类存活时间较长时，导致外部类也不能正确的回收（常发生在使用Handler的时候）。
4）不恰当的单例模式，例如错误的将某个Activity给单例持有，或者在不该使用单例的地方使用了单例。
5）使用错误的Context：Application 和 Activity的context生命周期不一样，不要让生命周期长的对象引用activity context，即保证引用activity的对象要与activity本身生命周期是一样的。
6）webview造成的内存泄漏。WebView只要使用一次，内存就不会被释放，所以WebView都存在内存泄漏的问题，通常的解决办法是为WebView单开一个进程，使用AIDL进行通信，根据业务需求在合适的时机释放掉。
7）Bitmap导致内存泄漏。Bitmap资源是比较占内存的，所以一定要在不使用的时候及时进行清理，避免静态变量持有大的bitmap对象。
8）监听器未关闭，很多需要register和unregister的系统服务要在合适的时候进行unregister,手动添加的listener也需要及时移除。

扩展：如何尽力避免内存溢出（OOM）？
1）使用更加轻量的数据结构，如：使用SparseArray替代HashMap。Android系统为移动系统设计的容器ArrayMap更加高效，占用内存更少，因为HashMap需要一个额外的实例对象来记录Mapping的操作。而SparesArray高效的避免了key和value的自动装箱，而且避免了装箱后的解箱。
2）避免在Android中使用Enum。在android中使用枚举类不仅会增加apk体积，同时也会增加运行时内存，所以在架构设计上还是要慎用枚举类。如果希望进行编译期类型检查可以使用`@IntDef`和`@StringDef`类保证类型安全。

```java
//枚举样式
public enum Color {
    RED, GREEN, BLACK, YELLOW
}
//使用IntDef实现改写枚举
@IntDef({Color2.RED, Color2.GREEN, Color2.BLACK, Color2.YELLOW})//限定为MAN,WOMEN
@Retention(RetentionPolicy.SOURCE)//表示注解所存活的时间,在运行时,而不会存在. class 文件.
public @interface Color2 {//接口，定义新的注解类型
    int RED = 1;
    int GREEN = 2;
    int BLACK = 3;
    int YELLOW = 4;
    int f = 5;
}
```

3）减少Bitmap对象的内存占用，Bitmap特别耗内存，Bitmap一般可做如下优化，以减少内存消耗：
	  通过InSampleSize设置合适的缩放比例，避免不必要的大图载入。
      选择合适的解码格式（如：ARGB_8888/RBG_565/ARGB_4444/ALPHA_8）。
4）循环拼接字符串时，采用StringBuilder.append()方法，避免字符串直接相加（因为会循环创建多个StringBuilder对象，耗费内存）。
5）避免在类似onDraw这样的方法中创建对象，因为它会迅速占用大量内存，引起频繁的GC甚至内存抖动。

### 11.4、理解ANR

ANR是Application Not Responding 的缩写,当应用程序无响应时，会弹出ANR窗口，让用户选择继续等待还是关闭应用。
处理超过时间会造成ANR的地方：触摸操作等（5s），BroadCast（前台广播10s，后台广播60s），Service（前台服务20s，后天服务200s）、ContentProvider（10s）。

为什么Looper.loop不会造成ANR？

> ANR是应用程序无响应，原因是有事件在主线程运行时间过长造成新的事件无法处理，或者当前事件运行时间太长。
> Looper.loop会循环处理到来的Message，当MessageQueue为空是，线程处于阻塞状态，释放cpu资源，所以不会造成ANR。

### 11.5、描述APK的安装流程

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/android_install_apk_process.png?raw=true" alt="android_install_apk_process.png" style="zoom:50%;" />

> 解压文件到data/app目录下。
> 资源管理器加载资源文件。
> 解析解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。
> 然后对dex文件进行优化，并保存在dalvik-cache目录下。
> 将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。
> 安装完成后，发送广播。

### 11.6、如何在不压缩的情况下加载超大图片？

> 使用BitmapRegionDecoder来实现显示指定区域的图像，比如拖拽显示全球地图等场景。

计算一张100px*100px的图片在内存中占用多大内存？

> 内存大小 = 100\*100\*单位像素所占字节数。
> 单位像素所占字节数和编码方式有关：ARGB_8888占8+8+8+8=32bit=4byte；ARGB_4444占4+4+4+4 =16bit=2byte；ALPHA_8占1byte；RGB_565占5+6+5=16bit=2byte；
> 故而如选择ARGB_8888编码时：100px\*100px大小图片所占内存为100\*100*4=40000bytes。

layout-sw600dp、layout-w600dp和layout-h600dp的区别？

> layout-sw600dp：当你的屏幕的`绝对宽度`大于600dp时，屏幕就会自动调用layout-sw600dp文件夹里面的布局。
> layout-w600dp：当你的屏幕的`相对宽度`大于600dp时，屏幕就会自动调用layout-w600dp文件夹里面的布局。
> layout-h600dp：与layout-w600dp类似。
> *`绝对宽度`是指手机的实际宽度，即与手机是否横屏没关系，也就是手机较小的边的长度。*
> *`相对宽度`是指手机相对放置的宽度；即当手机竖屏时，为较小边的长度；当手机横屏时，为较长边的长度。*

### 11.7、Android中的严格模式

严苛模式是一个开发工具，能够检测程序中的违例，从而修复。最常用的地方就是主线程中disk的读写和network。目前能有两大策略，线程策略（ThreadPolicy，线程策略）和Vm策略（VmPolicy，虚拟机策略）。在Application或者Activity中的onCreate方法中可进行配置，示例如下：

> ```java
> public void onCreate() {
>   if (BuildConfig.DEBUG) {//调试摸下用于测试调优性能，Release模式下不建议开启
>       StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
>               .detectDiskReads()//侦测磁盘读
>               .detectDiskWrites()//侦测磁盘写
>               .detectNetwork()//侦测网络操作
>               //Penalty是英语“处罚”的意思，所以凡是以penalty开头的方法都表示违规时要做出什么反应。
>               .penaltyLog()//侦测出错误违规时，将违规信息写入系统日志
>               //.penaltyDialog()//违规时，向开发者显示一个恼人的Dialog对话框。
>               .build());
>       StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
>               .detectActivityLeaks()//侦测Activity（活动）泄露
>               .detectLeakedSqlLiteObjects()//检测sqlite对象，如cursors
>               .detectLeakedClosableObjects()//检查为管理的Closable对象
>               //.penaltyLog()//违规时，将违规信息写入系统日志。
>               .penaltyDeath()//违规时，直接使应用崩溃。
>               .build());
>   }
>   super.onCreate();
> }
> ```

如何捕Android获全局异常？

> 利用Thread.UncaughtExceptionHandler中的uncaughtException方法进行捕获。

URI和URL区别？

> URI 在于I(Identifier)是统一资源标示符，可以唯一标识一个资源。
> URL在于Locater，一般来说（URL）统一资源定位符，可以提供找到该资源的路径，比如http://www.zhihu.com/question/21950864，但URL又是URI，因为它可以标识一个资源，所以URL又是URI的子集。
> 举个是个URI但不是URL的例子：urn:isbn:0-486-27557-4，这个是一本书的isbn，可以唯一标识这本书，更确切说这个是URN。

如果最大限度避免OOM

1）使用更加轻量的数据结构：如使用ArrayMap/SparseArray替代HashMap（HashMap更耗内存），因为它需要额外的实例对象来记录Mapping操作，SparseArray更加高效，因为它避免了Key Value的自动装箱，和装箱后的解箱操作
2.便面枚举的使用，可以用静态常量或者注解@IntDef替代
3.Bitmap优化:
a.尺寸压缩：通过InSampleSize设置合适的缩放
b.颜色质量：设置合适的format，ARGB_6666/RBG_545/ARGB_4444/ALPHA_6，存在很大差异
c.inBitmap:使用inBitmap属性可以告知Bitmap解码器去尝试使用已经存在的内存区域，新解码的Bitmap会尝试去使用之前那张Bitmap在Heap中所占据的pixel data内存区域，而不是去问内存重新申请一块区域来存放Bitmap。利用这种特性，即使是上千张的图片，也只会仅仅只需要占用屏幕所能够显示的图片数量的内存大小，但复用存在一些限制，具体体现在：在Android 4.4之前只能重用相同大小的Bitmap的内存，而Android 4.4及以后版本则只要后来的Bitmap比之前的小即可。使用inBitmap参数前，每创建一个Bitmap对象都会分配一块内存供其使用，而使用了inBitmap参数后，多个Bitmap可以复用一块内存，这样可以提高性能
4.StringBuilder替代String: 在有些时候，代码中会需要使用到大量的字符串拼接的操作，这种时候有必要考虑使用StringBuilder来替代频繁的“+”
5.避免在类似onDraw这样的方法中创建对象，因为它会迅速占用大量内存，引起频繁的GC甚至内存抖动
6.减少内存泄漏也是一种避免OOM的方法



# 数据库相关

## 1、索引相关知识

### 1.1、为什么要使用索引？

避免全表扫描，可快速查询数据。

### 1.2、什么信息可以成为索引？

主键、唯一键及普通键等只要能让数据具备一定区分性的字段都可以设置为索引。

### 1.3、索引的数据结构

1）二分搜索树（二叉查找树）
2）B树
3）B+树（最优索引算法）
4）HASH
5）BitMap（Oracle）

> 扩展：为什么不适用Hash索引，有什么缺点？
> 因为Hash值和树结果的区别，决定了Hash仅能满足“=”和“IN”的条件检索，不能使用范围查询；无法避免表扫描和排序操作；大量Hash值相等的数据效率可能比树索引低。

### 1.4、什么是联合索引？什么又是联合索引的最左匹配原则？

在一张表中，对多个字段同时建立索引，则称之为联合索引。
示例如下：

```sql
CREATE TABLE `fx_event_model` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `model_type` int(10) NOT NULL COMMENT '类型',
  `model_value` varchar(255) NOT NULL COMMENT '值',
  PRIMARY KEY (`id`),
  UNIQUE KEY (`model_type`, `model_value`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='状态表';
-- 如上`model_type`, `model_value`组成一个联合索引
-- 注意：联合索引有顺序区别，如(`model_type`, `model_value`)和(`model_value`, `model_type`)是完全不同的两种联合索引
```

**最左匹配原则**：即以最左边的为起点任何连续的索引都能匹配上。同时遇到范围查询(>、<、between、like)就会停止匹配。例如：
1）UNIQUE KEY (A,B,C) 这样一个联合索引，SQL会首先匹配A（A位于一个，即最左侧），然后再匹配B和C。
2）其次，如果使用仅使用(B,C)这样的组合来检索数据的话，因为找不到A，所以会使得联合索引失效；同样，如果使用(A,C)来检索数据，那么会首先匹配A，然后再匹配C，此时联合索引同样是失效的。
3）最后，如检索条件A=1 and B>2 and C=3，此时C=3的检索条件会失效，因为B>2是范围查询，范围查询条件之后的检索条件会被忽略，相当于执行了A=1 and B>2之后的数据集再做了C=3的条件判断（即联合索引失效），但=和in关键字可以乱序（如A=1 and B=2 and C=3与B=2 and C=3 and A=1同效，联合索引并不会失效，因为SQL引擎会自动优化）。
4）*若联合索引失效，也就意味着本身创建联合索引用来提升检索效率的目的没有达到，性能并未实质性提升。*

### 1.5、索引是建立的越多越好吗？

否，可作如下解释：
1）数据量小的表不需要建立索引（因为建立索引会增加额外的索引开销）。
2）在数据变更时，需要对响应的索引进行维护，那么索引越多维护成本越高。
3）索引建立的越多，则需要越多的存储空间来存储索引（如100页的书却存在50页的目录）。

## 2、锁/事务相关知识

### 2.1、共享锁和排它锁

共享锁：即读锁，排他锁：即写锁。
当数据进行查询（SELECT）的时候会自动添加读锁，进行增（INSERT）、删（DELETE）、改（UPDATE）时会自动添加写锁。

```sql
-- 如何测试给表添加一个读锁或写锁
lock tables 相关表名 read; #反之写锁为write
unlock tables; #释放被当前会话持有的任何锁
```

1）sessionA对一张表中的数据先上读锁（即查询数据），sessionB也对其上读锁（即查询数据），此时A和B都能正常读取到响应数据，其中一方并不会被另一方阻塞。
2）sessionA对一张表中的数据先上读锁（即查询数据），sessionB对其上写锁（即增删改），此时sessionB会被阻塞，需等待读锁释放完成。
3）反之，先上写锁，再上读锁或写锁，则会被阻塞（即写锁具有排他性）。
*总结：先上共享锁后，可以再上共享锁，但是不能上排他锁；先上排他锁后，不能上共享锁或排它锁。*

### 2.2、数据库事务的四大特性ACID

原子性（Atomic）：所有操作要么全部执行，要么全部不执行。
一致性（Consistency）：数据库中的数据应满足完整性约束。
隔离性（Isolation）：多个事务中，一个事务的执行，不应该干扰其他事务的执行。
持久性（Durability）：任意一个事务对数据库的修改，都应该永久保存在数据库中，当系统发生故障时，其事务对数据库的修改不能丢失，如InnoDB中日志文件记录事务操作的过程。

### 2.3、事务的隔离级别及各级别下的并发访问问题

数据库事务的隔离级别有4个，由低到高依次为READ-UNCOMMITTED、READ-COMMITTED、REPEATABLE-READ、SERIALIZABLE，这四个级别可以逐个解决脏读、不可重复读、幻读这几类问题。

*下面对上述4种隔离级别，结合并发可能出现的问题进行分别描述：*
*并发可能出现的问题如下：*
**更新丢失（lost update）**：即一个事务的更新覆盖了另一个事务的更新。如下所示：

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/sql_transaction_lost_update.png?raw=true" alt="sql_transaction_lost_update.png" style="zoom:50%;" />

**脏读**：一个事务读到另一个事务未提交的更新数据。即：当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据， 那么另外一个事务读到的这个数据是脏数据，依据脏数据所做的操作可能是不正确的。

**不可重复读**：指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。

**幻读**：指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。
*<small>不可重复读侧重于对统一数据的修改，幻读侧重于新增或删除数据。</small>*

*4种隔离级别描述如下：*
**未读提交（READ-UNCOMMITTED，RU）**：不能避免脏读、不可重复读和幻读。

**已提交读（READ-COMMITTED，RC）**：可避免脏读，不可避免不可重复读和幻读。

**可重复读（REPEATABLE-READ，RR）**：可避免脏读和不可重复读，不可避免幻读。MySQL InnoDB默认事务隔离级别。

**串行化（SERIALIZABLE）**：可同时避免脏读、不可重复读和幻读，但效率较低。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/sql_transaction_ioslation.png?raw=true" alt="sql_transaction_ioslation.png" style="zoom:55%;" />


> **扩展：MySQL InnoDB默认使用RR事务隔离级别，为什么也能避免幻读？**
> 基于next-key锁，即行锁+gap锁的方式来避免幻读。
> 行锁：即对单个记录行上的锁。
> gap锁：间隙锁，即锁定一个范围，但不包括记录本身。GAP锁的目的，是为了防止同一事务的两次当前读，出现幻读的情况。


```sql
-- 通过次SQL语句可以查询当前数据库的事务隔离级别
select @@tx_isolation;
-- 通过下述SQL可以修改当前数据库事务的隔离级别
set session transaction isolation level read uncommitted;
# 其他级别如：read committed, repeatable read, serializable

-- 事务操作测试SQL
start transaction;
# 相关操作 select update等
commit; #提交事务
rollback; #回滚事务
```

### 2.4、当前读和快照读

**快照读：**读取的是快照版本，也就是**历史版本。**普通的SELECT查询对应快照读。快照度是不加锁的非阻塞读操作。

**当前读**：读取的是**最新版本**。UPDATE、DELETE、INSERT、SELECT … LOCK IN SHARE MODE、SELECT … FOR UPDATE即对应当前读。

## 3、MySQL相关

### 3.1、MyISAM和InnoDB两个引擎有什么区别？

InnoDB支持事务，支持行级锁和表级锁；MyISAM不支持事务，仅支持表级锁。
InnoDB支持外键，而MyISAM不支持。（附：外键即可以将数据与另一张表关联起来的那个字段。目的是为了一张表记录的数据不要太过冗余。）
InnoDB数据内容和索引存一起存放在一个文件中，MyISAM数据内容和索引分两个文件存放。

### 3.2、MyISAM和InnoDB适用场景

MyISAM：频繁执行全表count操作；对数据增删改频率不高，但查询频率高。没有事务的场景。

InnoDB：增删改查都比较频繁；对稳定性要求高，有事务要求。

### 3.3、如何优化慢查询SQL？

1）根据慢日志查询慢SQL 

```sql
-- 查询当前sql配置，关注下述配置项：
-- long_query_time 即超过多少秒的查询被视为慢查询，默认10秒，可改为1秒
-- slow_query_log 是否开启慢查询日志记录功能，默认关闭
-- slow_query_log_file 记录慢查询日志的文件路径，即最终通过查看此文件内容来分析慢查询
show variables like '%query%'
-- 查询慢查询统计（即记录产生了多少次慢查询）
show status like '%slow_queries%' 

-- 下述设置和配置也可以直接写入配置文件my.cnf
-- 设置开启慢查询日志
set global slow_query_log = on;
-- 配置查询超过多少秒即被视为慢查询
set global long_query_time = 1;
```

2）利用explain关键字分析慢查询

```sql
-- 例如分析某select慢查询如下
-- 此explain sql语句并不会被真正运行，仅用于分析sql语句
explain select * from ct_user order by name;
```

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/sql_explain_example.png?raw=true" alt="sql_explain_example.png" style="zoom:60%;" />

此处分析慢查询注意关注字段如下：
type字段：如果是index或者all，则表示执行了全表扫描，效率较低，需要进行优化。

> type对应值的权级如下：
> system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > all

extra字段：如果是Using filesort或者Using temporary，效率较低，则需要进行优化。

> Using filesort：表示MySQL使用“文件排序”，此操作肯能发生在内存或者磁盘上，表原有的索引会失效（即不会根据表原配置的索引进行查找排序）。
> Using temporary：表示MySQL会对查询结果进行临时表（创建使用临时表可能带来耗时）排序，一般发生于Order by 和 Group by操作时。

3）根据上述1或2的方式查询分析对应的慢SQL语句，然后优化相关SQL语句（例如查询时借助索引来提升效率）。

### 3.4、读写分离和集群

ProxySQL  MyCAT  Atlas

## 4、Redis相关

**Redis 5种主要数据类型**

字符串类型（string），散列类型（hash），列表类型（list），集合类型（set），有序集合类型（zset）。

*Redis和MemeCached区别：1）redis支持更为丰富的数据类型；2）redis可以持久化其数据。*

### 4.1、如何从海量数据中匹配查询某一固定前缀的key？

不能直接使用keys [pattern]命令来匹配，海量数据可能会阻塞redis，造成卡顿；应当使用scan命令。

```shell
#SCAN命令每次被调用之后，都会向用户返回一个新的游标，用户在下次迭代时需要使用这个新游标作为 SCAN 命令的游标参数，以此来延续之前的迭代过程。
scan 0 match k1* count 10 #此语句查找匹配前缀满足k1开头的keys，并且每次返回不超过10个
#运行结果示例如下：
1) "11534336"		#此行即新的游标
2) 1) "k179302810"  #返回的数据结果集
   2) "k182082002"
   3) "k194802810"
#继续迭代返回时，可以使用上次的游标作为开始，如下：
scan 11534336 match k1* count 10
1) "30932992"
2) 1) "k100824802"
   2) "k180280180"
```

### 4.2、如何通过Redis实现分布式锁？

**什么是分布式锁？**

分布式锁：是控制分布式系统或不同系统之间共同访问共享资源的一种锁的实现。分布式锁具有以下特点（即分布式锁需要解决的问题）：
1）互斥性：任意时刻只能有一个客户端对象获取锁。
2）安全性：锁只能由持有该锁的客户端对象删除，不能由其他客户端对象删除。
3）死锁：持有锁的客户端对象由于异常等原因未能释放掉锁，导致其他客户端对象永远也不能获取到锁资源。
4）容错：在分布式系统中，当部分节点宕机时，客户端仍然能正常获取和释放锁。

**利用Redis实现分布式锁**

1）可以使用setnx结合expire指令来实现，*但是可能引起原子性问题（如setnx指令执行后，程序崩溃了，导致无法执行expire指令，因此会出现key永久不过期不会释放锁）*。

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/redis_setnx.png?raw=true" alt="redis_setnx.png" style="zoom:50%;" />

<center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/redis_expire.png?raw=true" alt="redis_expire.png" style="zoom:50%;" />

2）新版本的Redis，直接只用SET命令也可以实现变量的过期和原子性。SET命令说明如下：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/redis_set_example.png?raw=true" alt="redis_set_example.png" style="zoom:50%;" />

> **扩展1：大量的key同时过期会有什么影响？**
> 大量key如果再同一时期触发过期，那么Redis就会在同一时期清除大量的key，因此可能会出现短暂的卡顿现象。严重的话可能会出现缓存雪崩。
> 解决方案：在设置key过期时间的时候，附加一个随机值，使得过期时间尽量分散开来。
>
> 
>
> **扩展2：如何理解并解决Redis中的缓存雪崩、缓存穿透和缓存并发问题？**
> **缓存雪崩**：数据未加载到缓存中，或者缓存同一时间大面积的失效，从而导致所有请求都去查数据库，导致数据库CPU和内存负载过高，甚至宕机。
> 解决办法：1）配置Redis高可用；2）缓存降级（即保证核心服务可用，其他服务可有损）；3）项目上线前，提前演练测试；
>
> **缓存穿透**：是指查询一个不存在的数据。*例如：从缓存redis没有命中，需要从mysql数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，造成缓存穿透。*
> 解决办法：如果查询数据库也为空，直接设置一个默认值存放到缓存，这样第二次到缓冲中获取就有值了，而不会继续访问数据库。设置一个过期时间或者当有值的时候将缓存中的值替换掉即可。或使用BloomFilter（布隆过滤器，***详见扩展3***）。
>
> **缓存并发**：多个Redis的client同时set key引起的并发问题。Redis本身是单线程操作，多个client并发操作，按照先到先执行的原则，先到的先执行，其余的会被阻塞。
> 解决办法：缓存预热（即系统上线后，默认将相关的缓存数据直接加载到缓存系统），避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题。
>
> 
>
> **扩展3：布隆过滤器（BloomFilter）**
> 是一个很长的二进制向量和一系列随机映射函数。***布隆过滤器用于检索一个元素是否在一个集合中***。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。
>
> ```java
> import com.google.common.hash.BloomFilter;
> import com.google.common.hash.Funnels;
> //布隆过滤器示例（采用guava库实现）
> public class Test1 {
> 	private static int size = 1000000;
>     private static BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), size);
>     //或通过下述方法创建指定误报率为0.01%的布隆过滤器
>     //private static BloomFilter<Integer> bloomFilter = BloomFilter.create(Funnels.integerFunnel(), size, 0.0001);
>     public static void main(String[] args) {
>         for (int i = 0; i < size; i++) {
>             bloomFilter.put(i);
>         }
> 
>         System.out.println("0~1000000中判断是否存在29999这个数字：");
>         long startTime = System.nanoTime(); // 获取开始时间
>         //判断这一百万个数中是否包含29999这个数
>         if (bloomFilter.mightContain(29999)) {
>             //System.out.println("命中了");
>         }
>         long endTime = System.nanoTime();   // 获取结束时间
>         System.out.println("布隆过滤器判断是否存在耗时： " + (endTime - startTime) + "ns" + "，误报率：" + bloomFilter.expectedFpp() * 100 + "%");
> 
>         long s = System.nanoTime();
>         for (int i = 0; i < 1000000; i++) {
>             if (i == 29999) {
>                 break;
>             }
>         }
>         System.out.println("普通循环比较判断是否存在耗时：" + (System.nanoTime() - s) + "ns");
>     }
> }
> //输出：
> 0~1000000中判断是否存在29999这个数字：
> 布隆过滤器判断是否存在耗时： 38000ns，误报率：3.003410604421397%
> 普通循环比较判断是否存在耗时：1453100ns
> //由上述输出结果可以看出，布隆过滤器在某集合中判断某一元素是否存在时，效率远比循环比对高
> ```

### 4.3、如何使用Redis实现异步队列？

使用Redis发布/订阅者模式，subscribe和pubscribe命令。注意：Redis中发布订阅者模式是无状态的，无法保证可达（如：pub时某客户端还未上线，而后再次上线时客户端也无法接收到消息，需要使用消息队列Kafka/RocketMQ等）。

### 4.4、Redis持久化

**RDB方式：即快照方式，保持某个时间点的全量快照数据。**
可通过配置redis.conf配置文件来操作，或使用save命令（阻塞）或bgsave命令（非阻塞）。RDB快照可在以下场景触发：
1）根据redis.conf配置文件中的save m n定时触发（此时实际调用bgsave，不会阻塞）。
2）主从复制时，主节点自动触发。
3）执行Debug Reload时。
4）执行shutdown且没有开启AOF持久化时。
*RDB持久化缺点：*
1）RDB方式持久化每次dump时，都是将所有的内存数据全量备份到磁盘，可能会因为磁盘I/O影响性能。
2）由于RDB是间隔触发，因此可能会因为Redis意外终止而导致某个时间段的数据未被dump，造成数据丢失。

**AOF(Append-Only-File)方式：保存写状态，记录除查询意外的所有数据变更状态指令，并增加保存到aof文件中。**
*AOF持久化缺点：*
由于持续增量写aof文件，aof文件体积大，且记录的操作命令越多，数据恢复时间越长。

Redis中数据恢复流程如下：
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/redis_aof_process.png?raw=true" alt="redis_aof_process.png" style="zoom:67%;" />

> **扩展：Redis突然掉电数据丢失情况会怎样？**
> 取决于AOF日志sync属性的配置，如果不要求性能，在每条写指令时都sync一下磁盘，就不会丢失数据。但是在高性能的要求下每次都sync是不现实的，一般都使用定时sync，比如1s1次，这个时候最多就会丢失1s的数据。

### 4.5、如何理解Redis中的Pipeline？

**为什么要使用Pipeline？**

Redis 的工作过程是基于 请求/响应 模式的。正常情况下，客户端发送一个命令，等待 Redis 应答；Redis 接收到命令，处理后应答。请求发出到响应的时间叫做往返时间，即 RTT（Round Time Trip）。在这种情况下，如果需要执行大量的命令，就需要等待上一条命令应答后再执行。这中间不仅仅多了许多次 RTT，而且还频繁的调用系统 IO，发送网络请求。为了提升效率，pipeline 出现了，它允许客户端可以一次发送多条命令，而不等待上一条命令执行的结果。

Pipeline可以将多次IO往返的时间缩短为一次，但是Pipeline执行的指令之间不应该有依赖性（如第二条指令需要依赖第一条指令的结果），且Pipeline组装的指令个数不宜太多，否则可能增加客户端的等待时间，还可能会造成网络阻塞，应尝试将多个指令拆分成多个Pipeline执行。

```java
//示例：使用pipeline执行多条命令
@Test
public void testPipelineSyncAll() {
    // 工具类初始化
    Jedis jedis = new Jedis("192.168.1.111", 6379);
    jedis.auth("12345678");
    // 获取pipeline对象
    Pipeline pipe = jedis.pipelined();
    pipe.multi();
    pipe.set("name", "james"); // 调值
    pipe.incr("age");// 自增
    pipe.get("name");
    pipe.discard();
    // 将不同类型的操作命令合并提交，并将操作操作以list返回
    List<Object> list = pipe.syncAndReturnAll();
    for (Object obj : list) {
        // 将操作结果打印出来
        System.out.println(obj);
    }
    // 断开连接，释放资源
    jedis.disconnect();
}

```

### 4.6、Redis同步机制

Redis中的同步机制可分为全量同步和增量同步两部分。全量同步即slave启动时进行的初始化同步，增量同步即初始化同步完成后Redis运行过程中的指令同步。过程可作如下描述：
1）第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将RDB文件全量同步到复制节点。
2）复制节点接受完成后将RDB镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。后续的增量数据通过AOF日志同步即可。

### **4.7、Redis Sentinel（哨兵）和Redis Cluster（集群）**

Redis Sentinal着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。

Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

> **扩展：流言协议Gossip？**
> Gossip 过程是由种子节点发起，当一个种子节点有状态需要更新到网络中的其他节点时，它会随机的选择周围几个节点散播消息，收到消息的节点也会重复该过程，直至最终网络中所有的节点都收到了消息。这个过程可能需要一定的时间，由于不能保证某个时刻所有节点都收到消息，但是理论上最终所有节点都会收到消息，因此它是一个最终一致性协议。
> 
> <center><img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/gossip_protocal.webp?raw=true" alt="gossip_protocal.webp" style="zoom:67%;" />



## 5、MongoDB


MongoDB是一个**文档数据库**，提供好的性能，领先的非关系型数据库。采用BSON存储文档数据。BSON是一种类json的一种二进制形式的存储格式，简称Binary JSON。相对于json多了date类型和二进制数组。

### 5.1、如何理解MongoDB中的集合和文档？

**集合（collections）**：就是一组 MongoDB 文档。它相当于关系型数据库（RDBMS）中的表这种概念。
*MongoDB中每个数据实例（即数据库，如db1，db2等）可包含一个或多个集合。*
**文档（document）**：多个键值对的有序存放在一起就是文档。文档是MongoDB中的基本单元，相当于关系型数据库中表对应的一条记录。
*一个集合内的多个文档可以有多个不同的字段。*
***<small>MongDB与RDBMS对比，及MongoDB相关示例如下：</small>***
<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/mongo_example.png?raw=true" alt="mongo_example.png" style="zoom:62%;" />

```shell
#示例：查询显示集合
> show collections
runoob
system.indexes

#示例：查询显示文档
> db.mycol.find().pretty()
{
   "_id": 100,
   "title": "MongoDB Overview", 
   "description": "MongoDB is no sql database",
   "by": "yiibai tutorials",
   "url": "http://www.yiibai.com",
   "tags": ["mongodb", "database", "NoSQL"],
   "likes": "100"
}
```

### 5.2、如何理解MongoDB中的命名空间

MongoDB每个集合和每个索引都对应一个命名空间（命名空间是：数据库名称和集合名称的串联），这些命名空间的元数据集中在16M的*.ns文件中，平均每个命名占用约 628 字节，也即整个数据库的命名空间的上限约为24000。

如mydb数据库中mycollection集合对应的命名空间即为：mydb.mycollection

### 5.3、理解MongoDB中的GridFS

为了存储和检索大文件，例如图像，视频文件和音频文件，使用GridFS。默认情况下，它使用两个文件fs.files和fs.chunks来存储文件的元数据和数据块。

### 5.4、复制集/副本集（Replication）和切片（Sharding）

**复制集**即为主从复制，复制集指定为承载相同数据集的一组mongo实例。 在复制集中，一个节点是主节点，另一个节点是辅助节点。 所有数据都从主节点复制到辅助节点。

**切片**是把大型数据集进行分区成更小的可管理的片,这些数据片分散到不同的mongoDB节点,这些节点组成了分片集群。

### 5.5、MongDB索引优化、慢查询优化

**1）开启MongoDB内置查询分析器，记录慢查询。**

db.setProfilingLevel(n,{m}),n的取值可选0,1,2；（其中，0是默认值表示不记录；1表示记录慢速操作,如果值为1,m必须赋值单位为ms,用于定义慢速查询时间的阈值；2表示记录所有的读写操作；）

例如:db.setProfilingLevel(1,300)，即表示执行时间超过300ms的语句视为慢查询并记录。
*使用db.setProfilingLevel()配置记录器后，系统会自动将对应慢查询记录到system.profile集合（表）中。（通过show collections可查看到system.profile集合；通过db.system.find().pretty()可查看system.profile集合中的数据）*

**2）通过查询分析器找到慢查询之后，使用explain分析慢查询。**

例如：db.orders.find({'price':{'$lt':2000}}).explain('executionStats')，其中explain参数说明如下：
"queryPlanner"：是默认值,表示仅仅展示执行计划信息。
"executionStats" ：表示展示执行计划信息同时展示被选中的执行计划的执行情况信息。
"allPlansExecution" ：表示展示执行计划信息,并展示被选中的执行计划的执行情况信息,还展示备选的执行计划的执行情况信息。

分析解读explain结果，优化原则可描述如下：
(1) 根据需求建立索引，以满足每个查询都要使用索引以提高查询效率，对应winningPlan. stage值为IXSCAN（COLLSCAN 表示查询没有走索引，IXSCAN表示查询使用了索引）。
(2) 若能使得totalDocsExamined（检查的文档个数），nReturned（返回的文档个数）两个值相等，那么证明查询时扫描的数据记录和最终查询结果返回的数据集大小一致，获得高效率。

### 5.6、MongoDB如何优雅安全的关闭

在生产环境，若使用kill -9关掉MongoDB的进程，很可能造成MongoDB的数据丢失。可使用下述两种方式关闭：
1）mongo> use admin
	  mongo> db.shutdownServer()
2）shell> mongod --shutdown -f mongodb.conf

## 6、PostgreSQL

## 7、其他

（MySql,Oracle,SqlServer等关系型数据库）遵循的原则是：ACID原则（A：原子性。C：一致性。I：独立性。D：持久性。）。

（redis,Mogodb等非关系型数据库）遵循的原则是：CAP原则（C：强一致性。A:可用性。P：分区容错性）。

# 信息安全相关

重放攻击

中间人攻击

数字签名

PGP模型

PKI体系





# Python简述

**什么是解释器？**

当我们编写Python代码时，我们得到的是一个包含Python代码的以`.py`为扩展名的文本文件。要运行代码，就需要Python解释器去执行`.py`文件。由于整个Python语言从规范到解释器都是开源的，所以理论上，只要水平够高，任何人都可以编写Python解释器来执行Python代码。现已存在多种Python解释器，如：CPython（C语言实现的解释器，Python官方默认），Jython，PyPy等。

## 1、基础语法

### 1.1、基本输入和输出

```python
#tips：python中单行注释使用#号，多行注释使用三个单引号'''注释内容'''或者双引号括起来
name = input()#执行输入操作，并将输入的值赋值给name变量
print('aa','bb')#执行输出操作，print()函数也可以接受多个字符串，用逗号“,”隔开，就可以连成一串输出，遇到逗号“,”会替换输出一个空格。
#组合使用，例如输出字符串"1024*768=计算的值"
print("1024*768=", 1024*768)
```

### 1.2、基本数据类型

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/python_basic_data_types.png?raw=true" alt="python_basic_data_types.png" style="zoom:50%;" />

```python
#使用type函数可以查看数据类型，如:type(1)  type(1.1)

int#整数int
float#浮点数float，python不区分单精度和双精度，统一表示为浮点类型

bool#布尔型bool，布尔型实际上可以看做是int，可以通过type(int)来验证。

#字符串类型str，字符串可以使用单引号，双引号和三引号表示，如下所示：
'hello world' #单引号表示字符串
"hello world" #双引号表示字符串
#三个单引号或者双引号可以表示多行字符串
'''
	line1
	line2
'''
#示例：字符串截取
a = "abcdefg"
print(a[3:]) # 正序从字符串最左侧按0，1,2的顺序向右数
print(a[-4:]) # 倒叙从字符串最右侧按-1，-2的顺序向左数
# 故上述输出是一致的，均为：defg
# :冒号右侧为空，则表示从左侧规定位置偏移到字符串最末尾, 若冒号左侧为空，则相反
# :冒号右侧数字若大于字符长度了，则会按字符串长度截取，如a[0:100]，还是会输出：abcdefg
for i in a:
    print(i) # 遍历字符

#列表类型list
[1, 2, 3]
["str1", "str2", "str3"]
[1, 2, "str", True] #列表list中，可以是单独一种数据类型，也可以是多种类型的数据组合
[1, 2, "str", [11, 22]] #还可以嵌套list

#元组tuple
(1, 2, 3)
("str1", 1, True, 2)
(1,)#表示只有一个元素的元组
()#表示一个空的元组

#元组和列表区别
#元组和列表最大的区别就是，列表中的元素可以进行任意修改；而元组中的元素无法修改，除非将元组整体替换掉。可以理解为，tuple元组是一个只读版本的list列表。
#元组还具有以下特性：
#1、元组比列表的访问和处理速度更快，因此，当需要对指定元素进行访问，且不涉及修改元素的操作时，建议使用元组。

#集合set
{1, 2, 3}

#字典dict，类似Java Hash表
{"key1":"value1", "key2":"value2", "key3":100}
{}#表示空的字典
```

### 1.3、变量与基本运算

```python
#python中声明变量时，不需要定义变量类型，格式如：变量 = 某种数据，示例如下：
a = 100 #变量a赋值为int值100
b = "string100" #变量b赋值为字符串
c = [1, 2, 3] #变量c赋值为列表

3/2 #两数相除取结果浮点型
3//2 #两数相除取整型（注意，并非四舍五入）
2*2 #两数相乘
2**3 #2的3次方

a == b #比较的是a和b两个变量对应的值是否一样
a is b #比较的是a和b两个变量的内存地址是否一致
isinstance(a, str) #判断变量a是否为str类型变量
isinstance(a, (int, float, str)) #判断变量a的类型是否为int，float和str中的一种

#条件判断示例如下：
a = 1

if a > 0:#if条件中，无需像java一样使用花括号括起来，而是靠缩进来控制
    print("a>0")
    print("a>0 line2")#同一级别的缩进，即表示处于同一个代码段，即此处print和上一行print位于同一个if条件中
elif a == 0:#python中，冒号前默认建议不要空格
    print("a==0")
else:
    print("else")
    pass #pass实际表示空语句/占位语句

#python中建议代码最后空一行，否则会有警告
```

## 2、进阶语法

### 2.1、包和模块

一个.py文件即视为一个模块，包类似于Java中的包，即某个文件夹，但python中，每隔包对应的文件夹下都必须存在一个`__init__.py`文件（任何调用引用或执行此包中的任意.py文件时，此文件中的代码会自动执行，理解为包的初始化执行py文件，例如此包中重复import引用某个模块的代码可统一放置于此初始化文件中），否则无法正常视为一个python包。

```python
#在test1.py中定义
CA = "star111"

#在test2.py中引用
from mypkg.test1 import * #引用mypkg包下的test1模块
print(CA)
#from mypkg.test1 import CA as CCC #例：使用as作为别名引用
#print(CCC)

#import 和 from...import的区别
#import方式：是引入整个datetime包
import datetime
print(datetime.datetime.now()) # 使用import时，需要使用包名加模块名来调用类
#from...import方式：是只引入datetime包里的datetime类
from datetime import datetime
print(datetime.now()) # 使用from...import时，直接使用类

#更多python标准库，可参考：https://docs.python.org/zh-cn/3.8/library/index.html
```

### 2.2、函数

```python
# 利用def关键字定义一个函数
def test_func():
    print("这是一个没有参数的函数")
    pass

# 定义一个带参数的函数
def test_func_add(a, b):
    return a + b #return返回计算结果

#因为python是解释型语言，函数定义应该放置于调用之前，否则无法调用函数
print(test_func_add(1, 1))
print(test_func_add(a=2, b=1)) #关键字参数形式传值，可以指定某个值传递给某个形参

# 函数返回多个数据
def test_func3(a, b):
    result1 = a + "r1"
    result2 = b + "r2"
    return [result1, result2]
    #return result1, result2 #默认不加括号返回元组类型
    
# 通过定义变量来接收多个返回值（更直观易理解返回值表示的实际意义），避免使用[0]...[n]这样的方式来取值
get_result1, get_result2 = test_func3("aa", "bb")
print(get_result1, get_result2)

# 带默认参数的函数
def test_func4(a="defaultA", b=100):
    print("a : ", a)
    print("b : ", b)
    
# 带默认参数的函数，可以传值调用，也可以不传值调用
test_func4()
test_func4("ValueA", 200)

#lambda
def add(x, y):  # 定义普通函数
    return x+y

f = lambda x, y: x+y  # lambda表示方式
print(add(1, 2)) # 调用普通函数
print(f(1, 2))  # 调用lambda函数
```

### 2.3、类、面向对象

```python
# 在test包中，创建test3.py模块，内容如下：
class User:
    name = ''  # 定义一个属性
    age = 0
    height = 0
    __sex = 1   # 通过添加__前缀，定义私有变量
    addr = "Hangzhou"

    # 构造函数
    def __init__(self, name, age):
        self.name = name
        self.age = age
        print("call __init__")
        pass

    def get_user_info(self):  # 使用类的属性，需要调用self，如self.name
        print("name : ", self.name, ", age : ", self.age, ", height : ", self.height)
        print(self.__class__.name)  # 通过__class__可以调用类属性或类方法
        pass

    def __call_private_func(self):  # 通过添加__前缀, 定义私有方法, 私有方法外界无法访问
        print("call private func...", self.__sex)
        pass

    @staticmethod
    def test():  # 静态方法不需要self参数
        print("call class static method")

        
# 在test包中，创建test4.py模块，内容如下：
from test.test3 import User


class User2(User):  # 继承自User父类
    extend_weight = 0

    def __init__(self):
        print("call extend __init__")
        print(self.addr)  # 子类可访问父类的成员变量或方法
        pass

    
#创建测试模块，测试调用方法
from test.test3 import User
from test.test4 import User2

my_user = User("UserA", 20)  # 实例化对象
my_user.get_user_info()  # 调用对象中的方法
User.test()  # 通过类直接调用静态方法
my_user2 = User2() # 子类对象
```

## 扩展：正则表达式

### 元字符

元字符就是指那些在正则表达式中具有特殊意义的专用字符，可以用来写匹配的规则。

| 元字符 | 描述                                       |
| :----: | :----------------------------------------- |
|   ^    | 匹配以什么字符开始                         |
|   $    | 匹配以什么字符结束                         |
|   .    | 匹配任意除换行符"\n"外的字符               |
|   \d   | 匹配数字 0-9                               |
|   \D   | 匹配非数字                                 |
|   \s   | 匹配任何空白字符  空格  \t  \r  \n  \f  \v |
|   \S   | 匹配除了空白符以外的任一字符               |
|   \w   | 匹配包括下划线在内的任何字字符             |
|   \W   | 匹配非字母字符，即匹配特殊字符             |

```python
#元字符使用示例：匹配字符串中的数字
import re
a = "abcd3e4f1ga4bd2"
r = re.findall("\\d", a)
for i in r:
    print(i, end=" ")
```

### 限定符

| 限定符 | 描述                           |
| :----: | :----------------------------- |
|   ？   | 匹配前面的字符零次或一次       |
|   +    | 匹配前面的字符一次或者多次     |
|   *    | 匹配前面的字符零次或者多次     |
|  {n}   | 匹配前面的字符n次              |
|  {n,}  | 匹配前面的字符最少n次          |
| {n,m}  | 匹配前面的字符最少n次，最多m次 |

### 排除字符和选择字符

| 字符  | 描述                          |
| :---: | :---------------------------- |
|  [ ]  | 匹配 [ ] 中的任意一个字符     |
| [ ^ ] | 匹配除了 [ ] 中的任意一个字符 |
|  \|   | 匹配一个或者另一个字符        |

```python
a = "abcd3e4f1ga4bd2"
r = re.findall("[0-3]", a)  # 匹配字符串中的0到3的数字，又例如匹配非数字0到3的其他字符可表示为：[^0-3]
for i in r:
    print(i, end=" ")
```

### 转义字符和分组

|  字符  | 描述                                                       |
| :----: | :--------------------------------------------------------- |
|   \    | 将字符串中含有特殊字符进行转义，使其称为普通字符串的一部分 |
|  （）  | 改变限定符的作用范围，和进行分组                           |
| r或者R | 使 \ 的转义功能失效                                        |

更多Python内容参考：https://www.runoob.com/python3/python3-tutorial.html
									    https://www.liaoxuefeng.com/wiki/1016959663602400

# 分布式MQ相关

## 1、RocketMQ

## 2、Kafka

# WebRTC相关

## 1、基础

### 1.1、基本概念

**帧率**：摄像头一秒钟采集图像的次数称为帧率。帧率越高，视频就越平滑流畅，占的网络带宽就越多。

**采样率**：即一秒内采样的次数。每个采样用几个 bit 表示，称为采样位深或采样大小。

**轨（Track）**：“轨”在多媒体中表达的就是每条轨数据都是独立的，不会与其他轨相交，如 MP4 中的音频轨、视频轨，它们在 MP4 文件中是被分别存储的。

**流（Stream）**：可以理解为容器。在 WebRTC 中，“流”可以分为媒体流（MediaStream）和数据流（DataStream）。其中，媒体流可以存放 0 个或多个音频轨或视频轨；数据流可以存 0 个或多个数据轨。

自动增益：

Payload：

SDP（Session Description Protocal）：就是用文本描述的各端（PC 端、Mac 端、Android 端、iOS 端等）的能力。这里的能力指的是各端所支持的音频编解码器是什么，这些编解码器设定的参数是什么，使用的传输协议是什么，以及包括的音视频媒体是什么等等。

> 交换 SDP 的目的是为了让对方知道彼此具有哪些**能力**，然后根据双方各自的能力进行协商，协商出大家认可的音视频编解码器、编解码器相关的参数（如音频通道数，采样率等）、传输协议等信息。

1.2、音视频设备工作原理

**音频设备**：音频输入设备的主要工作是采集音频数据，而采集音频数据的本质就是模数转换（A/D），即将模似信号转换成数字信号。

> 模数转换使用的采集定理称为**奈奎斯特定理**，描述如下：
> *在进行模拟 / 数字信号的转换过程中，当采样率大于信号中最高频率的 2 倍时，采样之后的数字信号就完整地保留了原始信号中的信息。*
>
> 人类听觉范围的频率是 20Hz～20kHz 之间。对于日常语音交流（像电话），8kHz 采样率就可以满足人们的需求。但为了追求高品质、高保真，你需要将音频输入设备的采样率设置在 40kHz 以上，这样才能完整地将原始信号保留下来。例如我们平时听的数字音乐，一般其采样率都是 44.1k、48k 等，以确保其音质的无损。
> 采集到的数据再经过量化、编码，最终形成数字信号，这就是音频设备所要完成的工作。在量化和编码的过程中，采样大小（保存每个采样的二进制位个数）决定了每个采样最大可以表示的范围。如果采样大小是 8 位，则它表示的最大值是就是 28 -1，即 255；如果是 16 位，则其表示的最大数值是 65535。

**视频设备**：与音频输入设备很类似。当实物光通过镜头进行到摄像机后，它会通过视频设备的模数转换（A/D）模块，即光学传感器， 将光转换成数字信号，即 RGB（Red、Green、Blue）数据。

> 获得 RGB 数据后，还要通过 DSP（Digital Signal Processer）进行优化处理，如自动增强、白平衡、色彩饱和等都属于这一阶段要做的事情。
> 通过 DSP 优化处理后，就得到了 24 位的真彩色图片。因为每一种颜色由 8 位组成，而一个像素由 RGB 三种颜色构成，所以一个像素就需要用 24 位表示，故称之为**24 位真彩色**。
> 此时获得的 RGB 图像只是临时数据。因最终的图像数据还要进行压缩、传输，而编码器一般使用的输入格式为 YUV I420，所以在摄像头内部还有一个专门的模块用于将 RGB 图像转为 YUV 格式的图像。*YUV 也是一种色彩编码方法，主要用于电视系统以及模拟视频领域。它将亮度信息（Y）与色彩信息（UV）分离，即使没有 UV 信息一样可以显示完整的图像，只不过是黑白的，这样的设计很好地解决了彩色电视机与黑白电视的兼容问题。*

### 1.2、WebRTC架构类型

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/webrtc_mesh_sfu_mcu_demo.png?raw=true" alt="webrtc_mesh_sfu_mcu_demo.png" style="zoom:72%;" />

**Mesh**：每个端都与其它端互连，两两建立连接，如上图所示5个用户相互连接，共需要10个连接，如果每条连接占用带宽1mb（下述架构类似），则每个端上下行带宽都需要4mb，总共需要消耗带宽20mb。除了带宽问题，每个浏览器上还要有音视频“编码/解码”，cpu使用率也是问题，一般这种架构只能支持4-6人左右，不过优点也很明显，没有中心节点，实现很简单。

**MCU**（MultiPoint Control Unit）：一种传统的中心化架构，如上图所示每个用户浏览器仅与中心MCU服务器连接，MCU服务器负责所有的编解码和转码等操作，每个用户浏览器仅需1个连接，总共5个连接，上下行合计带宽10mb。用户浏览器端负载压力小，可以支持多人同时音视频通信，比较适合多人视频会议，但中心MCU服务器负载压力大，可能需要较高配置。

**SFU**（Selective Forwarding Unit）：与MCU类似，具有中心服务器，但是中心服务器只负责转发，不做太重的音视频处理，相比MCU服务器压力会较小。每个用户浏览器都需要建立1个连接用于上传自己的音视频，还需要有N-1个连接用户下载其他参与方的音视频数据，所以总的连接数为5*5=25，合计带宽为25mb，SFU典型应用场景为1对N的视频互动（如直播）。

### 1.3、NAT类型

NAT 基本上可以总结成 4 种类型：**完全锥型、IP 限制锥型、端口限制锥型和对称型**。

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/nat_full_cone.png?raw=true" alt="nat_full_cone.png" style="zoom:61%;" />





<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/nat_ip_restricted_cone.png?raw=true" alt="nat_ip_restricted_cone.png" style="zoom:61%;" />





<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/nat_port_restricted_cone.png?raw=true" alt="nat_port_restricted_cone.png" style="zoom:61%;" />





<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/nat_symmetric.png?raw=true" alt="nat_symmetric.png" style="zoom:61%;" />

## 2、示例

### 2.1、获取音视频数据

```javascript
//指定MediaStream中包含哪些类型的媒体轨（音频轨、视频轨），并且可为这些媒体轨设置一些限制。
const mediaStreamContrains = {
    video: {
        frameRate: {min: 20},//视频的帧率最小 20 帧每秒
  	    width: {min: 640, ideal: 1280},//宽度最小是 640，理想的宽度是 1280
  	    height: {min: 360, ideal: 720},//高度最小是 360，最理想高度是 720
  		aspectRatio: 16/9//宽高比是 16:9
    },
    audio: {
        echoCancellation: true,//开启回音消除
        noiseSuppression: true,//降噪
        autoGainControl: true//自动增益
    }
};
//通过getUserMedia方法访问音视频设备
var promise = navigator.mediaDevices.getUserMedia(mediaStreamContrains);
```

参数说明：

<img src="https://github.com/kknever/my_markdown_note/blob/master/image_resources/webrtc_getusermedia_api_param.png?raw=true" alt="webrtc_getusermedia_api_param.png" style="zoom:67%;" />