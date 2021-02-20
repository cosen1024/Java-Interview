我们都知道 HashMap 是线程不安全的，那 HashMap 为什么线程不安全？JDK1.8 还有这些问题吗？如何解决这些问题呢？本文将对该问题进行解密。

![](http://blog-img.coolsen.cn/img/HashMap为什么线程不安全.png)

## 多线程下扩容死循环

JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。

下面看看多线程情况下， JDK1.7 扩容死循环问题的分析。

![](http://blog-img.coolsen.cn/img/resize1.png)

新建一个更大尺寸的hash表，然后把数据从老的hash表中迁移到新的hash表中。重点看下transfer方法：

![](http://blog-img.coolsen.cn/img/resize2.png)

假设HashMap初始化大小为2，hash算法就是用key mod 表的长度，在mod 2以后都冲突在table[1]这里了。负载因子是 1.5 (默认为 0.75 )，由公式` threshold=负载因子 *  hash表长度`可得，`threshold=1.5 * 2 =3`，size=3，而 size>=threshold 就要扩容，所以 hash表要 resize 成 4。

未resize前的table如下图：

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第10页.png)

正常的ReHash，得到的结果如下图所示：

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第9页.png)

我们来看看多线程下的ReHash，假设现在有两个线程同时进行，线程1和线程2，两个线程都会新建新的数组，下面是resize 的过程。

**Step1:**

![](http://blog-img.coolsen.cn/img/carbon.png)

假设 **线程1** 在执行到`Entry<K,V> next = e.next;`之后，cpu 时间片用完了，被调度挂起，这时**线程1的 e** 指向 节点A，**线程1的 next** 指向节点B。

**线程2**继续执行，

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第1页.png)

**Step2:**

线程 1 被调度回来执行。

- 先是执行 newTalbe[i] = e;
- 然后是e = next，导致了e指向了节点B，
- 而下一次循环的next = e.next导致了next指向了节点A。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第2页.png)

**Step3:**

线程1 接着工作。**把节点B摘下来，放到newTable[i]的第一个，然后把e和next往下移**。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第3页.png)

**Step4:** 出现环形链表

e.next = newTable[i] 导致A.next 指向了 节点B，此时的B.next 已经指向了节点A，出现**环形链表**。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第4页.png)

如果get一个在此链表中不存在的key时，就会出现死循环了。如 get(11)时，就发生了死循环。

分析见get方法的源码：

![](http://blog-img.coolsen.cn/img/carbon1.png)

for循环中的`e = e.next`永远不会为空，那么，如果get一个在这个链表中不存在的key时，就会出现死循环了。

## 多线程的put可能导致元素的丢失

多线程同时执行 put 操作，如果计算出来的索引位置是相同的，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。此问题在JDK 1.7和 JDK 1.8 中都存在。

我们来看下JDK 1.8 中 put 方法的部分源码，重点看黄色部分：

![](http://blog-img.coolsen.cn/img/carbon4.png)



我们来演示个例子。

假设线程1和线程2同时执行put，线程1执行put(“1”, “A”)，线程2执行put(“5”, “B”)，hash算法就是用key mod 表的长度，表长度为4，在mod 4 以后都冲突在table[1]这里了。注：下面的例子，只演示了 `#1` 和`#2`代码的情况，其他代码也会出现类似情况。

正常情况下，put完成后，table的状态应该是下图中的任意一个。

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第6页.png)

下面来看看异常情况，两个线程都执行了`#1`处的`if ((p = tab[i = (n - 1) & hash]) == null)`这句代码。

此时假设线程1 先执行`#2`处的`tab[i] = newNode(hash, key, value, null);`

那么table会变成如下状态：

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第7页.png)

紧接着线程2 执行`tab[i] = newNode(hash, key, value, null);`

此时table会变成如下状态:

![](http://blog-img.coolsen.cn/img/map线程安全性问题-第8页.png)

这样一来，元素A就丢失了。

## put和get并发时，可能导致get为null

线程1执行put时，因为元素个数超出threshold而导致rehash，线程2此时执行get，有可能导致这个问题。此问题在JDK 1.7和 JDK 1.8 中都存在。

我们来看下JDK 1.8 中 resize 方法的部分源码，重点看黄色部分：

![](http://blog-img.coolsen.cn/img/carbon3.png)

在代码`#1`位置，用新计算的容量new了一个新的hash表，`#2`将新创建的空hash表赋值给实例变量table。

注意此时实例变量table是空的，如果此时另一个线程执行get，就会get出null。

## 巨人的肩膀

http://mailinator.blogspot.com/2009/06/beautiful-race-condition.html

https://coolshell.cn/articles/9606.html

https://juejin.cn/post/6844903554264596487

https://juejin.cn/post/6844903796225605640