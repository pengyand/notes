#### ArrayList 与 LinkedList 异同：

- 1.是否保证线程安全：ArrayList 和 LinkedList 都是不同步 ，也就是不保证线程安全
- 2.底层数据结构：ArrayList 底层使用的是 Object数组；LinkedList底层使用的是双向链表数据结构（JDK1.6之前为循环链表；JDK1.7取消了循环。注意 [双向链表和循环列表的区别](https://www.cnblogs.com/xingele0917/p/3696593.html)）
- 3.插入和删除是否受元素位置的影响：
  - ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。比如，执行 add（E e）方法的时候，ArrayList 会默认将指定的元素追加到此列表末尾，这种情况时间复杂度就是O(1)。但是如果要指定位置 i 插入或者删除元素的话（add(int index ,E element)）时间复杂度就为 O(n-i)。因为在进行上述操作的时候，集合中第 i 和第 i 个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。
  - LinkedList 采用链表存储，所以插入、删除元素时间复杂度不受元素位置的影响，都是近似 O(1)而数组为近似 O(n).
- 4.是否支持快速随机访问：LinkedList 不支持高效的随机访问，而 ArrayList 支持。快速随机访问就是通过元素的序号快速获取元素对象（对应 get(int index) 方法）。
- 5.内存空间占用：ArrayList 的空间浪费主要提现在 list 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则提现在它的每一个元素都需要消耗比 ArrayList 更多的空间（因为 存放直接后继和直接前驱以及数据）。

**补充内容：RandomAccess 接口**

~~~java
public interface RandomAccess{}
~~~

查看源码我们发现实际上 RandomAccess 接口中什么都没有定义。所以，在我们看来 RandomAccess 接口不过是一个标识符罢了。标识什么？标识这个接口的类具有随机访问能力。

在 binarySearch（）方法中，它要判断传入的 list 是否 是 RandomAccess 实例，如果是，就调用 indexedBinarySearch()方法，如果不是，就调用 iteratorBinarySearch()方法

~~~java
    public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
~~~

