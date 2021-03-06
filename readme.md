Java 基础
1. JDK 和 JRE 有什么区别？

JDK是Java Development Kit 的简称，即java开发工具包，提供了java相关的开发环境和运行环境。
JRE是Java Runtime Environment的简称，即Java运行环境，提供Java运行所需要的环境。

实际上，JDK已经包含了JRE，还包含了编译Java源码的编译器 javac 和其他的Java程序调试和分析工具。
如果只是运行Java程序，安装JRE就够了，如果要开发Java程序，则需要安装JDK。


2.== 和 equals 的区别是什么？

==，比较的是基本类型的值，引用类型的地址；
equals，本质上是==，但是一般很多重写了equals方法，如String、Integer重写成了值比较，所以 equals 比较变成是值是否相等。

代码比较：
	String s1 = new String("老王"); 
	String s2 = new String("老王");
	System.out.println(s1.equals(s2)); // true
	Cat c1 = new Cat("王磊");
	Cat c2 = new Cat("王磊"); 
	System.out.println(c1.equals(c2)); // fals
 String （和Integer）重写了 Object 的 equals 方法，把引用比较改成了值比较。（Cat没有重写）	一般（不重写）是比较 引用（地址）

这里注意：如果Cat类是以下原始结构，则符合 c1.equals(c2) 为 fals，其hashCode也不同；
如果使用 @Data 的方式替换get和set方法，则输出为true，hashCode也相同，因为是@Data注解重写了equal和hashCode方法。
【https://projectlombok.org/features/Data】

class Cat {
    public Cat(String name) {
        this.name = name;
    }

    private String name;
 
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}


3.两个对象的 hashCode()相同，则 equals()也一定为 true，对吗？

不一定，hashCode相同，equals不一定为true。
【不然就不会出现哈希冲突问题了（即不同对象也可能存在相同hashCode）；
  如上面的Cat对象，都是“王磊”，hashCode可能相同，但是不同对象，equal比较的引用地址不一样】

代码解读：像 “通话” 和 “重地” 的 hashCode() 相同，然而 equals() 则为 false，因为在散列表中，hashCode() 相等即两个键值对的哈希值相等，然而哈希值相等，并不一定能得出键值对相等。

hashCode的作用原理和示例解析 见： https://blog.csdn.net/weixin_30604651/article/details/99526025
简要总结：
 1. 特性和作用：
 （1）HashCode的存在主要是用于查找的快捷性，而equals是用于比较两个对象的是否相等的。HashCode是用来在散列存储结构中（如List、Set）确定对象的存储地址的；
      哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。
      当集合要添加新的元素时，先调用这个元素的HashCode方法，就一下子能定位到它应该放置的物理位置上。
（2）如果两个对象相同， equals方法一定返回true，并且这两个对象的HashCode一定相同；
（4）两个对象的HashCode相同，并不一定表示两个对象就相同，也就是equals方法不一定返回true，只能够说明这两个对象在散列存储结构中，如Hashtable，他们存放在同一个篮子里。
（2）如果两个类有相同的HashCode，例如 9除以8 和 17除以8的余数都是1，也就是说，我们先通过 HashCode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过equals在这个桶里找到我们要的类。  

 2. HashCode作用
   Java中的集合有两类，一类是List，再有一类是Set。前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。 
   equals方法可用于保证元素不重复，但是，如果每增加一个元素就检查一次，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，就要调用1000次    equals方法。这显然会大大降低效率。 于是，Java采用了哈希表的原理。

哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。
这样一来，当集合要添加新的元素时，先调用这个元素的HashCode方法，就一下子能定位到它应该放置的物理位置上。

（1）如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；
（2）如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了；
（3）不相同的话，也就是发生了Hash key相同导致冲突的情况，那么就在这个Hash key的地方产生一个链表，将所有产生相同HashCode的对象放到这个单链表上去，串在一起（很少出现）。  这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。 

HashMap 采用一种所谓的 “Hash 算法” 来决定每个元素的存储位置。当程序执行 map.put(String,Obect)方法 时，系统将调用String（key值）的 hashCode() 方法得到其 hashCode 值 —— 每个 Java 对象都有 hashCode() 方法，都可通过该方法获得它的 hashCode 值。得到这个对象的 hashCode 值之后，系统会根据该 hashCode 值来决定该元素的存储位置。源码如下:略
  当系统决定存储 HashMap 中的 key-value 对时，完全没有考虑 Entry 中的 value，仅仅只是根据 key 来计算并决定每个 Entry 的存储位置。
  这也说明了前面的结论：我们完全可以把 Map 集合中的 value 当成 key 的附属，当系统决定了 key 的存储位置之后，value 随之保存在那里即可。
  
public class Main {
    public static void main(String[] args) {
	String str1 = "OK";
	StringBuffer str2 = new StringBuffer(str1);
	String str3 = new String(str1);
	StringBuilder str4 = new StringBuilder(str1);
	System.out.println(str1.hashCode());
	System.out.println(str2.hashCode());
	System.out.println(str3.hashCode());
	System.out.println(str4.hashCode());
    }
}

2524
2018699554
2524
1311053135

其中str1和str3拥有相同的散列码，这是因为字符串的散列码(即hsahCode值)是由内容导出的。而字符串缓冲str2和str4的却不想等，这是因为他俩没有hashCode方法，他们的散列码由Object的hashCode方法导出的对象的存储地址。【因为String重写了equals和hashCode方法，字符串缓冲流没有重写，用的还是Object的】

【***哈希表hashtable(key，value) 的原理如上，即Java创建对象存储对象也是用的哈希表hashtable(key，value)，不是只有hashMap才有到的。】

  链表法就是将相同hash值的对象组织成一个链表放在hash值对应的槽位；
  开放地址法是通过一个探测算法，当某个槽位已经被占据的情况下继续查找下一个可以使用的槽位。
  java.util.HashMap采用的链表法的方式，链表是单向链表。形成单链表的核心代码如下：
protected Entry(int hash, K key, V value, Entry<K,V> next) {
    this.hash = hash;
    this.key =  key;
    this.value = value;
    this.next = next;
}
【那每个节点结构是由数据域和指针域组成，数据域是存放数据的，而指针域存放下一结点的地址。https://cloud.tencent.com/developer/article/1525949】
  上面方法的代码很简单，但其中包含了一个设计：系统总是将新添加的 Entry 对象放入 table 数组的 bucketIndex 索引处——如果 bucketIndex 索引处已经有了一个 Entry 对象，那新添加的 Entry 对象指向原有的 Entry 对象(产生一个 Entry 链)，如果 bucketIndex 索引处没有 Entry 对象，也就是上面程序代码的 e 变量是 null，也就是新放入的 Entry 对象指向 null，也就是没有产生 Entry 链。


4.final 在 java 中有什么作用？
修饰 类：类不能被继承
修饰 方法：方法不能被重写/修改
修饰变量 被修饰后即为 常量，常量必须初始化，初始化后不能被修改。


5.java 中的 Math.round(-1.5) 等于多少？
答案：-1
因为在数轴上取值时，中间值（0.5）向右取整，所以正 0.5 是往上取整，负 0.5 是直接舍弃。
往大的方向舍弃，0.5 --> 1; 1.5 --> 2


6.String 属于基础的数据类型吗？

不是，是String包装类
基础类型有 8 种：byte、boolean、char、short、int、float、long、double，而 String 属于对象。


7.java 中操作字符串都有哪些类？它们之间有什么区别？

操作字符串的类有：String、StringBuffer、StringBuilder。

String 和 StringBuffer、StringBuilder 的区别在于 ：
	（1）String声明的是不可变的对象，每次操作都会产生新的String对象，然后将指针指向新的对象，而StringBuffer、StringBuilder 可以在原对象的基础上进行操作，所以经常改变字符串内容的情况下最好不要使用 String。
	（2）而StringBuffer 和 StringBuilder 之间的区别在于，StringBuffer是线程安全的，StringBuilder非线程安全，但是不安全的比安全的新能高，所以单线程使用builder，多线程用buffer。


8.String str="i"与 String str=new String("i")一样吗？

不一样，因为 内存分配的方式 不一样。
String str="i" ，Java虚拟机会将其分配到 常量池 中；而new的方式会分配到 堆内存中。
【栈:保存局部变量的值：
   包括 1.基本数据类型的值。
	2.保存类的实例，即堆区对象的引用（指针）。
	3.保存加载方法时的帧。
  堆：用来存放动态产生的数据，比如new出来的对象。】


9.如何将字符串反转？

使用 StringBuilder 或者 stringBuffer 的 reverse() 方法。
或者转数组反转输出。【toCharArray 转数组（将字符串分成单个字符的数组）】


10.String 类的常用方法都有那些？

	indexOf()：返回指定字符的索引。
	charAt()：返回指定索引处的字符。
	replace()：字符串替换。
	trim()：去除字符串两端空白。
	split()：分割字符串，返回一个分割后的字符串数组。
	getBytes()：返回字符串的 byte 类型数组。
	length()：返回字符串长度。
	toLowerCase()：将字符串转成小写字母。
	toUpperCase()：将字符串转成大写字符。
	substring()：截取字符串。
	equals()：字符串比较。
	toCharArray 转数组（将字符串分成单个字符的数组）


11.抽象类必须要有抽象方法吗？

抽象类不一定有抽象方法；有抽象方法一定是抽象类


12.普通类和抽象类有哪些区别？

普通类可以实例化，但是不能有抽象方法；
抽象类不能被实例化，只能被继承，有抽象方法


13.抽象类能使用 final 修饰吗？

不能，final修饰类不能被继承，而抽象类必须被继承才能使用


14.接口和抽象类有什么区别？

实现继承（使用）、构造函数、实现数量、方法访问修饰符。

接口：被实现 implements ；不能有构造函数；多实现；方法均为public。
抽象类：被继承 extends； 可以有构造函数； 单继承；方法可以是任意修饰符； 有抽象方法和普通方法。


15.java 中 IO 流分为几种？

输入流、输出流；字节流（ 8 位传输以字节为单位）、字符流（ 16 位传输以字符为单位）

 【 bit=1  二进制数据0或1 (简称 “位” 吧)  信息的最小单位
    byte = 8bit  1个字节等于8位 存储空间的基本计量单位
    一个英文字母 = 1byte = 8bit (1个英文字母是1个字节，也就是8位)
    一个汉字 = 2byte = 16bit (1个汉字是两个字节，也就是16位) 】


16.BIO、NIO、AIO 有什么区别？【后续1】


17.Files的常用方法都有哪些？
   copy、exists、createFile、delete、write



二、容器

18.java 容器都有哪些？

【Set、List、Map】

Collection
	List
		ArrayList
		LinkedList
		Vector (线程安全)
		Stack  (线程安全)
	Set
		HashSet
		LinkedHashSet
		TreeSet
Map
	HashMap
	LinkedHashMap
	TreeMap
	ConcurrentHashMap (线程安全)
	Hashtable (线程安全)
	
容器主要包括 Collection 和 Map 两种，其下又有很多子类，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。
Java容器类基本上都是在java.util包下，Java容器分为两大类，Collection类和Map类，结构如下图：


19.Collection 和 Collections 有什么区别？

【Collection是Java容器集合接口，有List、Set等；Collections是工具类】

Collection是一个集合接口，提供了对集合对象进行操作的所有通用接口方法；所有集合都是它的子类，如List、Set等。
Collections是一个包装类，包含了很多静态方法，不能被实例化，像工具类，如提供的排序方法，Collections.sort(list)。


20.List、Set、Map 之间的区别是什么？

List、Set 和 Map主要区别体现在 元素是否有序、元素是否允许重复；
List 元素有序，可重复；

Set 元素无序，不重复(去重且无序，无序是不按照插入的顺序输出)；其中TreeSet有序（用二叉树排序）
Map 元素无序，key值唯一，value可重复；其中TreeMap有序（用二叉树排序）

【其中，其中TreeSet有序，需要实现Comparable接口并自定义compareTo排序规则 --- 如class Teacher implements Comparable<Teacher> 】
【List分为ArrayList和LinkedList，其中ArrayList查询速度快，但增删速度较慢，LinkedList的增删速度快，查询速度较慢】

补充：
hashSet具有去重功能

例:  创建一个hashSet 保存 f f a a b b d d  
HashSet<String> set = new HashSet<>();
set.add("f"); 
set.add("f");
set.add("a");
set.add("a");
set.add("b");
set.add("b");
set.add("d");
set.add("d");
// 增强for循环
for (String string : set) {
	System.out.println(string);
}
打印结果 a b d f 


21.HashMap 和 Hashtable 有什么区别？

 (1) 存储：HashMap 允许 key 和 value 为 null，而 Hashtable 不允许。
 (2) 线程安全：Hashtable 是线程安全的，而 HashMap 是非线程安全的。

推荐：Hashtable 是保留类不建议使用，推荐在 单线程下使用 HashMap 替代，如需多线程用 ConcurrentHashMap 替代。


22.如何决定使用 HashMap 还是 TreeMap？

对于在 Map 中插入、删除、定位一个元素这类操作，HashMap 是最好的选择，因为相对而言 HashMap 的插入会更快，但如果你要对一个 key 集合进行有序的遍历，那 TreeMap 是更好的选择。

TreeMap<String, String> map = new TreeMap<String, String>();
public static void init(Map<String, String> map) {
     map.put("c", "1");
     map.put("a", "1");
     map.put("bb", "1");
     map.put("b", "1");
}
默认输出：
a : 1
b : 1
bb : 1
c : 1
参考：https://www.cnblogs.com/chenmo-xpw/p/4922641.html


23.说一下 HashMap 的实现原理？

HashMap 是基于 Hash 算法实现的，通过 put(key,value)存储，get(key)来获取。当传入 key 时，HashMap 会根据 key. hashCode() 计算出 hash 值，根据 hash 值将 value 保存在 bucket 里。
当计算出的 hash 值相同时，我们称之为 hash 冲突，HashMap 的做法是用 链表和红黑树 存储相同 hash 值的 value。当 hash 冲突的个数比较少时，使用链表；冲突个数多时则使用红黑树。
【链地址法：来一个元素加一个，让这个位置存储一个指针，指向一个链表，让所有相同位置的元素都放在这个链表中】
【链地址法，又有“头插”和“尾插”的方式，如 这里存放 1->6->7 ,插入链表的时采用‘头插’的方式，也就是插入链表的最前面（图中里数组最近的元素）（头插发，即从头部插入，后面的数据靠前，先插入的数据靠后） 】
【1.7之前是头插法，1.8之后是尾插法。头插法有时候会有死循环】
【为什么用“头插”？ 因为经常发生这样的事情：新加入的元素很可能被再次访问到，所以放到头的话，如果查找就不用再遍历链表了”】

相关参考链接：
https://www.jianshu.com/p/1b511b3889ae

https://cloud.tencent.com/developer/article/1361248

https://mp.weixin.qq.com/s?__biz=MzU1MDE4MzUxNA==&mid=2247483679&idx=1&sn=4a51847fb40fafe418bca0b906a03e0a&chksm=fba5362accd2bf3cba68324072863e93a354e65ecb1bdfb9f7dbd857af41f553b72d46b2cb36&scene=21#wechat_redirect

【 HashMap是由数组+链表/红黑树组成。 hash冲突多时会用红黑树，少时会用单链表。 Hash函数代表着一类函数。】
【** 什么时候用到红黑树：https://www.cnblogs.com/mzc-blogs/p/5800084.html
	3. put函数的实现
		put函数大致的思路为：

		对key的hashCode()做hash，然后再计算index;
		如果没碰撞直接放到bucket里；
		如果碰撞了，以链表的形式存在buckets后；
		如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
		如果节点已经存在就替换old value(保证key的唯一性)
		如果bucket满了(超过load factor*current capacity)，就要resize。（resize，我的理解就是rehash，在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中。）
		
	4. get函数的实现
		在理解了put之后，get就很简单了。大致思路如下：

		bucket里的第一个节点，直接命中；
		如果有冲突，则通过key.equals(k)去查找对应的entry
		若为树，则在树中通过key.equals(k)查找，O(logn)；
		若为链表，则在链表中通过key.equals(k)查找，O(n)。
】

参考：https://www.cnblogs.com/wangflower/p/12235595.html HashMap什么时候会触发链表转红黑树
结论是目前能触发转化的两个条件是：一个是链表的长度达到8个，一个是数组的长度达到64个

参考：https://blog.csdn.net/cenjia7278/article/details/100956198 HashMap底层数据结构之链表转红黑树的具体时机
HashMap中有关“链表转红黑树”阈值的声明：

   /**
    *  使用红黑树(而不是链表)来存放元素。
    *  当向至少具有这么多节点的链表再添加元素时，链表就将转换为红黑树。
    *  该值必须大于2，并且应该至少为8，以便于删除红黑树时转回链表。【当链表长度超过阈值（8）时，将链表转换为红黑树，】
    */
    static final int TREEIFY_THRESHOLD = 8;
 
    /**
     *  当 桶数组 容量小于该值时，优先进行扩容，而不是树化：
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
    ...
		//“binCount >= 7”：p从链表.index(0)开始，当binCount == 7时，p.index == 7,newNode.index == 8； 
		//也就是说，当链表已经有8个节点了，此时再新链上第9个节点，在成功添加了这个新节点之后，立马做链表转红黑树。
		if (binCount >= TREEIFY_THRESHOLD - 1)
			treeifyBin(tab, hash); //链表转红黑树 break;
	通过源码解析，我们已经很清楚HashMap是在“当链表已经有8个节点了，此时再新链上第9个节点，在成功添加了这个新节点之后，立马做链表转红黑树”。	
		
24.说一下 HashSet 的实现原理？

HashSet 是基于 HashMap 实现的，HashSet 底层使用 HashMap 来保存所有元素，因此 HashSet 的实现比较简单，相关 HashSet 的操作，基本上都是直接调用底层 HashMap 的相关方法来完成，HashSet 不允许重复的值（见20）。


25.ArrayList 和 LinkedList 的区别是什么？

 (1) 数据结构：ArrayList 是动态数组 的数据结构实现；LinkedList 是双向链表的数据结构实现 ；
 (2) 增删查速度：ArrayList 增删较慢，查询快；LinkedList 增删快，查询慢；因为  查询时 LinkedList 是线性的数据存储方式，所以需要移动指针从前往后依次查找； ArrayList 增删操作要影响数组内的其他数据的下标。

综合来说，在需要频繁读取集合中的元素时，更推荐使用 ArrayList，而在插入和删除操作较多时，更推荐使用 LinkedList。


26.如何实现数组和 List 之间的转换？

数组转List：Arrays.asList(array);
List转数组：使用 List 自带的 toArray() 方法，List.toArray()；


27.ArrayList 和 Vector 的区别是什么？

    (1) 线程安全：ArrayList 非线程安全；Vector是线程安全（Vector 使用了 Synchronized 来实现线程同步，是线程安全的）。
    (2) 性能：ArrayList性能方面 肯定优于 Vector。
    (3) 扩容：ArrayList 和 Vector 都会根据实际的需要动态的调整容量，只不过在 Vector 扩容每次会增加 1 倍，而 ArrayList 只会增加 50%。


28.Array 和 ArrayList 有何区别？

【Array是普通数组，初始化时必须声明大小； ArrayList是对象数组，长度可以变化；】

 (1) Array 可以存储基本数据类型和对象，List只能存储对象;
 (2) Array 是指定固定大小的，ArrayList大小可以自动扩充；
 (3) Array内置方法没有List多，如 addAll、removeAll、iteration等方法只有ArrayList有。


29.在 Queue 中 poll()和 remove()有什么区别？

相同点：都是返回第一个元素，并在队列中删除 返回的对象。
不同点：如果没有元素，poll会返回null； remove会抛出NoSuchElementException 异常

 Queue<String> queue = new LinkedList<String>();
  queue. offer("string"); // add
  System. out. println(queue. poll());
  System. out. println(queue. remove());
  System. out. println(queue. size());


30.哪些集合类是线程安全的？

参考：https://blog.csdn.net/laowang2915/article/details/73648208

【早在jdk的1.1版本中，所有的集合都是线程安全的。但是在1.2以及之后的版本中就出现了一些线程不安全的集合，为什么版本升级会出现一些线程不安全的集合呢？因为线程不安全的集合普遍比线程安全的集合效率高的多。随着业务的发展，特别是在web应用中，为了提高用户体验减少用户的等待时间，页面响应速度(也就是效率)是优先考虑的。而且对线程不安全的集合加锁以后也能达到安全的效果(但是效率会低，因为会有锁的获取以及等待)。其实在jdk源码中相同效果的集合线程安全的比线程不安全的就多了一个同步机制，但是效率上却低了不止一点点，因为效率低，所以已经不太建议使用了。下面举一些常用的功能相同却线程安全和不安全的集合】

【Collection 下，Vector、Stack 都是线程安全的；Map 下， HashTable 和 ConCurrentHashMap 是线程安全的】

Vector、Hashtable、Stack 都是线程安全的，而像 HashMap 则是非线程安全的，不过在 JDK 1.5 之后随着 Java. util. concurrent 并发包的出现，它们也有了自己对应的线程安全类，比如 HashMap 对应的线程安全类就是 ConcurrentHashMap。


31.迭代器 Iterator 是什么？

了解迭代器：https://blog.csdn.net/qq_33642117/article/details/52225247
【可迭代是Java集合框架下的所有集合类的一种共性，也就是把集合中的所有元素遍历一遍。
   迭代器（Iterator）模式，又叫做游标模式，它的含义是，提供一种方法访问一个容器对象中各个元素，而又不需暴露该对象的内部细节。】

public class Test3 {
	public static void main(String[] args) {
		List<String>list=new ArrayList<>();
		list.add("a");
		list.add("b");
		Iterator<String>it=list.iterator();//得到lits的迭代器
		//调用迭代器的hasNext方法，判断是否有下一个元素
		while (it.hasNext()) {
		    //将迭代器的下标移动一位，并得到当前位置的元素值
		    System.out.println(it.next());	
		}	
	}
}
注意：从Java5.0开始，迭代器可以被foreach循环所替代，但是foreach循环的本质也是使用Iterator进行遍历的。

总结： 迭代器，提供一种访问一个集合对象各个元素的途径，同时又不需要暴露该对象的内部细节。java通过提供Iterator和Iterable俩个接口来实现集合类的可迭代性，迭代器主要的用法是：首先用hasNext（）作为循环条件，再用next（）方法得到每一个元素，最后在进行相关的操作。


32.Iterator 怎么使用？有什么特点？

使用如上，Iterator 的特点是更加安全，因为它可以确保，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常。


33.Iterator 和 ListIterator 有什么区别？

 (1) Iterator 可以遍历所有的集合(List、Set集合)；ListIterator只能遍历List集合(ArrayList、LinkedList和Vector的时候可以使用)；
 (2) ListIterator 从 Iterator 接口继承（public interface ListIterator<E> extends Iterator<E> ），然后添加了一些额外的功能。如add方法，可以向List中添加对象，而Iterator不能。即 ListIterator方法多一些。 所以 ListIterator 可以双向遍历（有 向前/后遍历 的方法）。


34.怎么确保一个集合不能被修改？

 参考：https://blog.csdn.net/syilt/article/details/90548237
【使用 final 修饰，不行；如下，代码还是输出 three 。
  对于Map等集合类型，我们只能对于其引用不能被再次初始化，而其中的值则可以变化】

private static final Map<Integer, String> map = Maps.newHashMap();
static {
    map.put(1, "one");
    map.put(2, "two");
}
 
public static void main(String[] args) {
    map.put(1,"three");
    log.info("{}", map.get(1));
}

代码还是输出 three

可以使用 Collections. unmodifiableCollection(Collection c) 方法来创建一个只读集合，这样改变集合的任何操作都会抛出 Java.lang. UnsupportedOperationException 异常。

示例代码如下：
    List<String> list = new ArrayList<>();
    list.add("x");
    Collection<String> clist = Collections. unmodifiableCollection(list);
    clist.add("y"); // 运行时此行报错
    System.out.println(list. size());


三、多线程

35.并行和并发有什么区别？

【并行是多个进程同时运行（在同一个CPU上）；并发是一个进程多个线程同时运行（在同一个进程上）。】
并行：多个处理器或多核处理器 同时 处理多个任务。
并发：多个任务在同一个CPU核上。按细分的时间片轮流执行，从逻辑上看 那些任务是 同时执行的。

并发 = 两个队列和一台咖啡机。
并行 = 两个队列和两台咖啡机。


36.线程和进程的区别？

一个程序下至少有一个进程，一个进程下至少有一个线程，一个进程下也可以有多个线程来增加程序的执行速度。


37.守护线程是什么？

【  在Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) ；
    用个比较通俗的比如，任何一个守护线程都是整个JVM中所有非守护线程的保姆：

    只要当前JVM实例中尚存在任何一个非守护线程没有结束，守护线程就全部工作；只有当最后一个非守护线程结束时，守护线程随着JVM一同结束工作。
    Daemon的作用是为其他线程的运行提供便利服务，守护线程最典型的应用就是 GC (垃圾回收器)，它是一个称职的守护者。

    User和Daemon两者几乎没有区别，唯一的不同之处就在于虚拟机的离开：如果 User Thread已经全部退出运行了，只剩下Daemon Thread存在了，虚拟机也就退出了。 因为没有了被守护者，Daemon也就没有工作可做了，也就没有继续运行程序的必要了。    
    参考： https://blog.csdn.net/shimiso/article/details/8964414  】

用户线程：我们平常创建的普通线程。守护线程：用来服务于用户线程；不需要上层逻辑介入。

守护线程是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。在 Java 中垃圾回收线程就是特殊的守护线程。也就是说 守护线程 不依赖于终端，但是 依赖于系统，与系统“同生共死”。

那Java的守护线程是什么样子的呢。当JVM中所有的线程都是守护线程的时候，JVM就可以退出了；如果还有一个或以上的非守护线程则JVM不会退出。
JRE判断程序是否执行结束的标准是所有的前台执线程(用户线程)行完毕了，而不管后台线程(守护线程)的状态，因此，在使用后台线程时一定要注意这个问题。 
	
    example: 垃圾回收线程就是一个经典的守护线程，当我们的程序中不再有任何运行的Thread,程序就不会再产生垃圾，垃圾回收器也就无事可做，所以当垃圾回收线程是JVM上仅剩的线程时，垃圾回收线程会自动离开。它始终在低级别的状态中运行，用于实时监控和管理系统中的可回收资源。


38.创建线程有哪几种方式？
  【参考 ： https://blog.csdn.net/qq_38974634/article/details/81315900 创建多线程的4种方式】
   
  (1) 继承Thread类，重写run方法
  (2) 实现Runnable接口，实现run方法
  (3) 实现Callable接口，创建Callable接口的实现类，并实现call()方法，该call()方法将作为线程执行体，并且有返回值。
  (4) 线程池

/**
 * **通过继承Thread实现run方法
 */
public class ThreadDemo extends Thread{
    @Override
    public void run(){
        XXX
        System.out.println(getName()+"的演出结束了！");
    }
}
使用：ThreadDemo threadDemo = new ThreadDemo();
      threadDemo.start();
================================第二方式 (类似 实现runnable接口)==============================
【 https://blog.csdn.net/beidaol/article/details/89135277 什么是多线程？如何实现多线程？】

// 线程1
Thread t1 = new Thread(new Runnable() {
    @Override
    public void run() {
       // Thread.currentThread()  返回当前线程的引用
       lockTest.method(Thread.currentThread());
    }
 }, "t1Name");
   
// 线程2
Thread t2 = new Thread(new Runnable() {
     @Override
     public void run() {
        lockTest.method(Thread.currentThread());
     }
 }, "t2Name");
     
     
/**
 * **通过实现Runnable接口实现run方法
 */
 使用：  TestRunnable testRunnable = new TestRunnable();
        Thread thread = new Thread(testRunnable);
        //thread.setDaemon(true);
        thread.start();
	
class TestRunnable implements Runnable{
    @Override
    public void run() {
        xxx
        System.out.println(Thread.currentThread().getName()+"的演出结束了！");
    }
    public void sing(){
        System.out.println("haha");
    }
}
使用：
public static void main(String[] args) {
     //启动线程通过继承Thread
     Thread actor=new ThreadDemo();
     actor.setName("Mr. Thread");
     actor.start();

     //启动线程通过实现Runnable接口
//   Thread runnableThread = new Thread(new TestRunnable(),"Ms. Runnable");
//   runnableThread.start();
}


/** **Callable 方式 */
public class CallableThreadTest implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int i = 0;
        for (; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " " + i);
        }
        return i;
    }
}

    // 使用1：
    public static void main(String[] args) {
        CallableThreadTest ctt = new CallableThreadTest();
        FutureTask<Integer> ft = new FutureTask<>(ctt);
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName() + " 的循环变量i的值" + i);
            if (i == 20) {
                new Thread(ft, "有返回值的线程").start();
            }
        }
    }
    // 使用2：
	CallableDemo callableDemo = new CallableDemo();
    	FutureTask futureTask = new FutureTask<>(callableDemo); 
    	new Thread(futureTask).start();
    	List<Integer> lists = (List<Integer>)futureTask.get(); //获取返回值
    	for (Integer integer : lists) {
		System.out.print(integer + "  ");
	}

补充：
//implements创建对象：只能通过new Thread(Runnable target)方式创建对象，只要target对象是同一个，那么创建出来的线程访问的资源对象都相同 （即，第一个runnable执行后的属性变化影响第二个；第二个是第一个执行后的属性数据）
参考：[https://www.cnblogs.com/qq438649499/p/12190813.html] 并发编程@线程基础知识回顾
【如    TestRunnable runnable1 = new TestRunnable();
        Thread runnableThread1 = new Thread(runnable1);
        runnableThread1.start();

      // TestRunnable runnable2 = new TestRunnable(); //如果是 new新的TestRunnable而新开的Thread，则还是新的原始线程。
        Thread runnableThread2 = new Thread(runnable1); //这里的第二个线程，是以为第一个runnable1的数据为开始的了。
        runnableThread2.start();】

 两个线程，两个线程访问的 两个线程各自的资源 （两个线程资源）
 TestThread testThread1 = new TestThread();
 TestThread testThread2 = new TestThread();
 Thread t1 = new Thread(testThread1);
 Thread t2 = new Thread(testThread2);

 两个线程，同一个资源对象（一个线程资源执行两个）
 TestThread testThread1 = new TestThread();
 TestThread testThread2 = new TestThread();
 Thread t1 = new Thread(testThread1);
 Thread t2 = new Thread(testThread1);

//Implements创建对象：只能通过new Thread(Runnable target)方式创建对象，只要target对象是同一个，那么创建出来的线程访问的资源对象都相同

 t1.start();
 t2.start();

【 https://blog.csdn.net/beidaol/article/details/89135277 什么是多线程？如何实现多线程？
  非线程安全：非线程安全的类，有共享的变量。
  线程安全：当多个线程访问某个方法时，不管你通过怎样的调用方式、或者说这些线程如何交替地执行，我们在主程序中不需要去做任何的同步，
  这个类的结果行为都是我们设想的正确行为，那么我们就可以说这个类是线程安全的。】
【 synchronized 理解为只有一个线程可以使用被修饰的代码，如修饰方法，则只允许一个线程占有，另一个要等待】

【参考： https://blog.csdn.net/sinat_34814635/article/details/78959162 lock锁说明】


39.说一下 runnable 和 callable 有什么区别？

【实现Runable 接口的run方法没有返回值，Callable有返回值 （Callable<Integer>，也体现在run方法上）】
  runnable 没有返回值，callable 可以拿到有返回值，callable 可以看作是 runnable 的补充。


40.线程有哪些状态？

【线程状态有5种：初始、准备，运行、阻塞(等待阻塞、同步阻塞、其他阻塞)、死亡】 
【https://blog.csdn.net/xingjing1226/article/details/81977129 线程的5种状态详解（**有线程的状态图 和 与等待队列相关的步骤和图）
参考如下：https://blog.csdn.net/wuxing26jiayou/article/details/76647961 进程和线程区别详解

线程和进程一样分为五个阶段：创建 New 、就绪 Runnable (start())、运行 Running 、阻塞 Blocked 、终止 Terminated ；
线程有3个基本状态：执行、就绪、阻塞;
多线程中栈一般用于存放局部变量，为线程私有，堆一般用于存放程序共享数据。

线程不安全：访问时不提供数据访问保护，有可能出现多个线程先后更改数据造成所得到的数据是脏数据。
原因：线程安全问题都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。】

NEW 尚未启动
RUNNABLE 正在执行中
BLOCKED 阻塞的（被同步锁或者IO锁阻塞）
WAITING 永久等待状态
TIMED_WAITING 等待指定的时间重新被唤醒的状态
TERMINATED 执行完成


【sleep(Thread使用，让当前线程睡眠)、wait(对象锁使用，让当前拥有锁的线程放弃锁)；
  notify、notifyAll 都是对象锁来调用的，(随机/全部)唤醒等待它（对象锁）的线程】

41.sleep() 和 wait() 有什么区别？

  (1) 哪个类：sleep 来自Thread类， wait 是来自Object。
  (2) 释放锁机制：sleep()不会释放锁； wait()会释放锁。
  (3) 用法：sleep() 睡眠时间结束自动恢复； wait()需要 notify()/notifyAll() 唤醒
      Wait通常被用于线程间交互，sleep通常被用于暂停执行。
      

42.notify()和 notifyAll()有什么区别？

【notify 唤醒单个线程，notifyAll唤醒所有线程 --- 对象锁来调用的该方法，(随机)唤醒等待他的线程】
  notifyAll()会唤醒所有的线程（唤醒所有在等待该对象锁的线程），notify()之后唤醒一个线程（唤醒哪个是随机的）。

obj.wait()，当前线程调用对象的wait()方法，当前线程释放对象锁，进入等待队列。
依靠notify()/notifyAll()唤醒或者wait(long timeout)timeout时间到自动唤醒。
obj.notify()唤醒在此对象监视器上等待的单个线程，选择是任意性的。
notifyAll()唤醒 在此对象监视器上 等待的所有线程。


43.线程的 run()和 start()有什么区别？

   start()进入启动线程，进入准备状态，run()用于 执行线程的运行时代码 ，处于运行状态。
   run() 可以重复调用，而 start() 只能调用一次。

补充：11) 为什么我们调用start()方法时会执行run()方法，为什么我们不能直接调用run()方法？
	这是另一个非常经典的java多线程面试问题。这也是我刚开始写线程程序时候的困惑。现在这个问题通常在电话面试或者是在初中级Java面试的第一轮被问到。这个问题的回答应该是这样的，当你调用start()方法时你将创建新的线程，并且执行在run()方法里的代码。但是如果你直接调用run()方法，它不会创建新的线程也不会执行调用线程的代码。阅读我之前写的《start与run方法的区别》这篇文章来获得更多信息。

Example2：
1、用start方法 和 直接run方法 启动线程的区别：
示例：https://juejin.im/post/5b09274af265da0de25759d5

2、run方法又是一个什么样的方法？run方法与start方法有什么关联？
	run()方法当作普通方法的方式调用；
	run()其实是一个普通方法，只不过当线程调用了start( )方法后，一旦线程被CPU调度，处于运行状态，那么线程才会去调用这个run（）方法；

结论：start方法是用于启动线程的，可以实现并发（两个线程分别执行自己的程序），而run方法只是一个普通方法，是不能实现并发的，只是在并发执行的时候会调用（两个线程，先后执行完）。


44.创建线程池有哪几种方式？ 【可以理解为线程池就是一个初始化好的对象数组，需要对象时不需要new了，从池中获取会更快，也不会频繁的创建销毁】

【线程池的使用：
        我们有两种常见的创建线程的方法，一种是继承Thread类，一种是实现Runnable的接口，Thread类其实也是实现了Runnable接口。
	但是我们创建这两种线程在运行结束后都会被虚拟机销毁，如果线程数量多的话，频繁的创建和销毁线程会大大浪费时间和效率，更重要的是浪费内存，因为正常来说线程执行完毕后死亡，线程对象变成垃圾！那么有没有一种方法能让线程运行完后不立即销毁，而是让线程重复使用，继续执行其他的任务哪？我们使用线程池就能很好地解决这个问题。】

【 1. 线程池的概念：（即 要使用线程时不需要创建线程了，而是将任务传给线程池）
     线程池就是首先创建一些线程，它们的 集合 称为线程池。使用线程池可以很好地提高性能，线程池在系统启动时即创建大量空闲的线程，程序将一个任务传给线程池，线程池就会启动一条线程来执行这个任务，执行结束以后，该线程并不会死亡，而是再次返回线程池中成为空闲状态，等待执行下一个任务。

   2. 线程池的工作机制
         2.1 在线程池的编程模式下，任务是提交给整个线程池，而不是直接提交给某个线程，线程池在拿到任务后，就在内部寻找是否有空闲的线程，如果有，则将任务交给某个空闲的线程。
         2.1 一个线程同时只能执行一个任务，但可以同时向一个线程池提交多个任务。

   3. 使用线程池的原因：
        多线程运行时间，系统不断的启动和关闭新线程，成本非常高，会过渡消耗系统资源，以及过渡切换线程的危险，从而可能导致系统资源的崩溃。这时，线程池就是最好的选择了。】


线程池创建有七种方式，最核心的是最后一种：
    newSingleThreadExecutor()：它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目；
    ......
    newWorkStealingPool(int parallelism)：这是一个经常被人忽略的线程池，Java 8 才加入这个创建方法，其内部会构建ForkJoinPool，利用Work-Stealing算法，并行地处理任务，不保证处理顺序；
    ThreadPoolExecutor()：是最原始的线程池创建，上面1-3创建方式都是对ThreadPoolExecutor的封装。

使用代码如下。

45.线程池都有哪些状态？【待续深究2】
【可接受新任务 也可运行任务，不接受新任务但可运行已有任务，不接受任务也不运行任务】
【RUNNING 可接受新任务 并 运行等待队列的任务；
  SHUTDOWN 不接受新任务但会继续处理等待队列中的任务（运行已有任务）；
  STOP 不接受新任务，也不再处理等待队列中的任务，中断正在执行任务的线程；
  tidying 所有的任务都销毁了，workCount 为 0，线程池的状态在转换为 TIDYING 状态时，会执行钩子方法 terminated()；
  TERMINATED：terminated()方法结束后，线程池的状态就会变成这个。】


46.线程池中 submit() 和 execute()方法有什么区别？
【参考 ： https://blog.csdn.net/qq_38974634/article/details/81315900 创建多线程的4种方式】

【submit(Callnable/Runnable)的参数可以是Callnable和Runnable，而 execute(runnable)只有runnable】

execute()：只能执行 Runnable 类型的任务。
submit()：可以执行 Runnable 和 Callable 类型的任务。

Callable 类型的任务可以获取执行的返回值，而 Runnable 执行无返回值。

线程池使用：
public class TestThreadPool {
    public static void main(String[] args) throws ExecutionException, InterruptedException {

        //声明线程池大小 (类似于 声明数组)
        ExecutorService threadPoolExecutorService = Executors.newFixedThreadPool(5);

        //装载 多个线程，方便打印出来
        List<Future<List<Integer>>> submitLists = new LinkedList<>();

        // 调用线程池threadPoolExecutorService通过submit方法添加线程，这里循环5次添加5个线程任务
        // threadPoolExecutorService.submit返回的 为一条线程，这里装5条线程（submit）到list容器中
        for (int i = 0; i < 5; i++) {
            Future<List<Integer>> submit = threadPoolExecutorService.submit(new Callable<List<Integer>>() {
                @Override
                public List<Integer> call() throws Exception {
                    System.out.println(Thread.currentThread().getName() + "  ");
                    List<Integer> lists = new LinkedList<>();
                    for (int i = 0; i < 5; i++) {
                        lists.add(i);
                    }
                    return lists;
                }
            });
            submitLists.add(submit);
        }

        for (Future<List<Integer>> future : submitLists) {
            System.out.println(future.get());
        }    
    }
}


47.在 java 程序中怎么保证多线程的运行安全？

【线程安全问题 都是由全局变量及静态变量引起的。若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。】

 (1)  使用安全类，如 Java.util.concurrent 下的类。
 (2)  使用 自动锁 synchronized。
 (3)  使用手动锁 Lock。

手动锁 Java 示例代码如下：
    Lock lock = new ReentrantLock();
    lock. lock();  //还有tryLock() 区别是当所别其他线程占用，Lock会等待，tryLock不会等待，但tryLock可以设置等待时，过时就停止
    try {
       System. out. println("获得锁");
    } catch (Exception e) {
       // TODO: handle exception
    } finally {
       System. out. println("释放锁");
       lock. unlock();
    }
    
    
自动锁 synchronized示例代码：

    当两个并发线程(thread1和thread2)访问同一个对象(syncThread)中的synchronized代码块时，在同一时刻只能有一个线程得到执行，另一个线程受阻塞，必须等待当前线程执行完这个代码块以后才能执行该代码块。
    Thread1和thread2是互斥的，因为在执行synchronized代码块时会锁定当前的对象，只有执行完该代码块才能释放该对象锁，下一个线程才能执行并锁定该对象。为什么上面的例子中thread1和thread2同时在执行。这是因为synchronized只锁定对象，每个对象只有一个锁（lock）与之相关联。

总结：
    A. 无论synchronized关键字加在 方法上还是对象上，如果它作用的对象是 非静态 的，则它取得的锁是 对象（根据对象作为锁）；
       如果synchronized作用的对象是一个 静态方法或一个类，则它取得的锁 是对类 (类作为锁)，该类所有的对象同一把锁。 
    B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。 
    C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。


48.多线程锁的升级原理是什么？

【29_平时写代码如何对synchronized优化：
	1.减少synchronized的范围
		同步代码块中尽量短，减少同步代码块中代码的执行时间，减少锁的竞争。
	2.降低synchronized锁的粒度
		将一个锁拆分成多个锁提高并发度。
		
典例：Hashtable和ConcurrentHashMap
	Hashtable hs = new Hashtable();
	hs.put("aa","aa");
	hs.put("bb","bb");
**hashTable：锁住整个哈希表，一个操作正在进行时，其他操作也同时锁定，效率低下。（即，进行put操作时，get操作也在等待锁的释放）
ConcurrentHashMap：get方法不加锁，put方法是synchronized + casTabAt对节点进行加锁(不同的bucket桶加不同锁)，这样不同一个线程写时，另一个线程也可以读取。

写代码时，不要提高类来加锁；如两段无业务相关的代码方法：因为这样就是同一个锁，和hashTable的锁机制一样，效率低，并发行低。
	public void test01(){
		synchronized(Demo.class){
			......
		}
	}
	public void test02(){
		synchronized(Demo.class){
			......
		}
	}

	3.读写分离
	如 ConcurrentHashMap，读取时不加锁，写入和删除时加锁；还有如其他集合CopyOnwriteArrayList和CopyOnwriteSet
】

synchronized 锁升级原理：
	在 锁对象 的 对象头 里面有一个 threadid 字段，在第一次访问的时候 threadid 为空，jvm 让其持有偏向锁，并将 threadid 设置为其 线程 id，再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁，通过 自旋循环 一定次数来获取锁(升级为重量级锁之前的自旋锁)，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了 synchronized 锁的升级。

	锁的升级的目的：锁升级是为了减低了锁带来的性能消耗。在 Java 6 之后优化 synchronized 的实现方式，使用了 偏向锁升级为轻量级锁 再升级 到重量级锁的方式，从而减低了锁带来的性能消耗。

【 参考：*** https://www.cnblogs.com/snow-man/p/10874464.html  synchronized 锁的升级
  synchronized 的基本认识：
	在多线程并发编程中 synchronized 一直是元老级角色，很 多人都会称呼它为重量级锁。但是，随着 JDK 1.6 对 synchronized 进行了各种优化之后，有些情况下它就并不 那么重，Java SE 1.6 中为了减少获得锁和释放锁带来的性 能消耗而引入的偏向锁和轻量级锁。

  synchronized 有三种方式来加锁，
	1. 修饰实例方法 public synchronized void test(){...} ，作用于 当前实例 加锁，进入同步代码前 要获得当前实例的锁
	2. static静态方法，作用于当前类对象加锁，进入同步代码前要 获得当前类对象的锁
	3. 修饰代码块 synchronized(obj){...}，指定加锁对象，对给定对象加锁，进入同 步代码库前要获得给定对象的锁。

    //修饰实例方法1, 锁作用域是this
    public synchronized void SyncMethod1() { ... }
    
    //修饰实例方法2,锁作用域是this.lock
    public void SyncMethod2() {
        synchronized (this.lock) {
        }
    }

    //修饰实例方法3, 锁作用域是this对象
    public synchronized void SyncMethod3() {
        synchronized (this) {
        }
    }
    
    //静态方法 锁作用域是整个class
    public static synchronized void SyncMethod4() { ... }

    //修饰代码块 
    public   void SyncMethod5() {
        //.... 代码逻辑
        synchronized (this.lock){
            //同步代码
        }
        //.... 代码逻辑
    }
    
    
    JDK1.6 之后做了一些优化，为了减少获得锁和释放锁带来 的性能开销，引入了偏向锁、轻量级锁的概念。
    因此大家 会发现在 synchronized 中，锁存在四种状态 分别是：无锁、偏向锁、轻量级锁、重量级锁；
    锁的状态 根据竞争激烈的程度从低到高不断升级。
    
    JVM优化synchronized的运行机制，当JVM检测到不同的竞争状态时，就会根据需要自动切换到合适的锁，这种切换就是锁的升级。
    升级是不可逆的，也就是说只能从低到高，也就是偏向-->轻量级-->重量级，不能够降级

　　锁级别：无锁->偏向锁->轻量级锁->重量级锁
  
     1.偏向锁：在 锁对象 的 对象头 里面有一个 threadid 字段，在第一次访问的时候 threadid 为空，jvm 让其持有偏向锁，并将 threadid 设置为其 线程 id，再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则升级偏向锁为轻量级锁；
     
     2.轻量级锁：
       三个人轮流(错开使用时间)在一张椅子上吃饭。
     　
      当线程存在竞争时(多个线程，但不会同时竞争)，偏向锁的效率就会降低，因为当多条线程竞争同一个偏向锁时，会频繁产生偏向锁的撤销，所以此时应该升级为轻量级锁；
      轻量级锁当线程竞争锁失败时，线程不会阻塞进入自旋（升级为重量级锁之前的自旋锁），继续获取锁，当竞争非常激烈时，持续自旋而获取不到锁会消耗大量CPU资源，此时就会升级为重量级锁，重量级锁当获取锁失败线程会阻塞，重量级锁的缺点是线程上下文会频繁的切换。
         
     升级为轻量级锁的过程：
	1. 线程在自己的栈桢中创建锁记录 LockRecord。
	2. 将锁对象的对象头中的MarkWord复制到线程的刚刚创 建的锁记录中。
	3. 将锁记录中的 Owner 指针指向锁对象。
	4. 将锁对象的对象头的 MarkWord替换为指向锁记录的指 针。

     3. 自旋锁：火车厕所上两个人上厕所，一个进去1min，另一个走到门口2min。【硬件需支持两个以上的进程，而且 同步代码的内容执行时间较短】
          轻量级锁在加锁过程中，用到了自旋锁:
	  通过 自旋循环 一定次数来获取锁，执行一定次数之后(不用马上进入重量锁，升级为重量锁没拿到锁的线程会进入等待，需要唤醒等资源)，如果还没有正常获取到要使用的对象，此时就会把锁从轻量级升级为重量级锁，此过程就构成了 synchronized 锁的升级。
	  
       【轻量级锁 升级到 重量级锁之前，因为重量级锁的开销很大，尽量避免升级为重量级锁，所以中间会有一个 自旋锁的升级。】
	  
     4. 重量级锁：
	加了同步代码块以后，在字节码中会看到一个 monitorenter 和 monitorexit。
	每一个 JAVA 对象都会与一个监视器 monitor 关联，我们 可以把它理解成为一把锁，当一个线程想要执行一段被 synchronized 修饰的同步方法或者代码块时，该线程得先 获取到 synchronized 修饰的对象对应的 monitor。
	monitorenter 表示去获得一个对象监视器。monitorexit 表 示释放 monitor 监视器的所有权，使得其他被阻塞的线程 可以尝试去获得这个监视器 monitor 依赖操作系统的 MutexLock(互斥锁)来实现的, 线 程被阻塞后便进入内核（Linux）调度状态，这个会导致系 统在用户态与内核态之间来回切换，严重影响锁的性能。

	扩展：锁消除(如new StringBuffer.append().append()..)、锁粗化。 （参考：https://www.bilibili.com/video/BV1JJ411J7Ym?p=28）
	小结：什么是锁粗化？JVM会探测到一连串细小的操作都使用 同一个独对象加锁，将同步代码块的范围放大，放到这串操作的外面，这样只需要加一次锁即可。例如for循环append。
】


49.什么是死锁？
【两个线程互相等待对方释放锁的情况，就叫死锁】
当线程A持有独占锁a，并尝试取获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a的情况下，就会发生AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，称为死锁。


50.怎么防止死锁？

【谨慎用锁，手动释放锁】
 (1) 尽量使用tryLock(Long timeout，TimeUnit unit)的方法，设置超时时间，超时可以退出 防止死锁。
 	【tryLock() 区别是 Lock会一直等待，tryLock可以设置等待时，过时就停止】
 (2) 尽量使用 Java.util.concurrent 包下的并发类 代替自己手写锁。
 (3) 尽量降低锁的使用粒度，不要几个功能 用 同一把锁。
 (4) 尽量减少同步的代码块。


51.ThreadLocal 是什么？有哪些使用场景？
	ThreadLocal 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。
	ThreadLocal 的经典使用场景是数据库连接和 session 管理等。

使用：
public class MonitorUtil {
    private static ThreadLocal<Long> tl = new ThreadLocal<>();

    public static void start() {
        tl.set(System.currentTimeMillis());
    }

    //结束时打印耗时
    public static void finish(String methodName) {
        long finishTime = System.currentTimeMillis();
        System.out.println(methodName + "方法耗时" + (finishTime - tl.get()) + "ms");
    }
}

1.ThreadLocal 是什么？
  【参考：https://www.jianshu.com/p/6fc3bba12f38 ThreadLocal作用、场景、原理】
    在JDK 1.2的版本中就提供java.lang.ThreadLocal，ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。使用这个工具类可以很简洁地编写出优美的多线程程序。
    ThreadLocal并不是一个Thread，而是Thread的局部变量，也许把它命名为ThreadLocalVariable更容易让人理解一些。
    在JDK5.0中，ThreadLocal已经支持泛型，该类的类名已经变为ThreadLocal<T>。API方法也相应进行了调整，新版本的API方法分别是void set(T value)、T get()以及T initialValue()。
	
    1.1.ThreadLocal 的作用？
    	ThreadLocal是解决线程安全问题一个很好的思路，它通过为 每个线程 提供一个独立的变量副本 解决了 变量并发访问的冲突问题。在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

    1.1.2.ThreadLocal的应用场景？
    	在Java的多线程编程中，为保证多个线程对共享变量的安全访问，通常会使用synchronized来保证同一时刻只有一个线程对共享变量进行操作。这种情况下可以将类变量放到ThreadLocal类型的对象中，使变量在每个线程中都有独立拷贝，不会出现一个线程读取变量时而被另一个线程修改的现象。最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。在下面会例举几个场景。
	经典的使用场景是为每个线程分配一个 JDBC 连接 Connection。这样就可以保证每个线程的都在各自的 Connection 上进行数据库的操作，不会出现 A 线程关了 B线程正在使用的 Connection； 还有 Session 管理 等问题。
    
    ThreadLocal类中提供了几个方法：
	 T get()方法是用来获取ThreadLocal在当前线程中保存的变量副本，
	 void set()用来设置当前线程中变量的副本，
	 void remove()用来移除当前线程中变量的副本，
	 T initialValue()是一个protected方法，一般是用来在使用时进行重写的，它是一个延迟加载方法，下面会详细说明。

**描述： Java中的ThreadLocal类允许我们创建只能被 同一个线程 读写的变量。
 * 因此，如果一段代码含有一个ThreadLocal变量的引用，即使两个线程同时执行这段代码，
 * 它们也无法访问到对方的ThreadLocal变量。

使用案例：
public class ThreadLocalExsample {

	public static class MyRunnable implements Runnable {	
		/**
		 * 例化了一个ThreadLocal对象。我们只需要实例化对象一次，并且也不需要知道它是被哪个线程实例化。
		 * 虽然所有的线程都能访问到这个ThreadLocal实例，但是每个线程却只能访问到自己通过调用ThreadLocal的
		 * set()方法设置的值。即使是两个不同的线程在同一个ThreadLocal对象上设置了不同的值，
		 * 他们仍然无法访问到对方的值。
		 */	 
		 private ThreadLocal threadLocal = new ThreadLocal();

		 @Override
		 public void run() {
			 //一旦创建了一个ThreadLocal变量，你可以通过如下代码设置某个需要保存的值
			 threadLocal.set((int) (Math.random() * 100D));
			 try {
				Thread.sleep(2000);
			 } catch (InterruptedException e) {  }
			 //可以通过下面方法读取保存在ThreadLocal变量中的值
			 System.out.println("-------threadLocal value-------"+threadLocal.get());
		 }
	 }
	​
	 public static void main(String[] args) {
		 MyRunnable sharedRunnableInstance = new MyRunnable();
		 Thread thread1 = new Thread(sharedRunnableInstance);
		 Thread thread2 = new Thread(sharedRunnableInstance);
		 thread1.start();
		 thread2.start();
	 }
}
运行结果
-------threadLocal value-------38
-------threadLocal value-------88

    结论: ThreadLocal 中 set 和 get 操作的都是对应线程的 table数组，因此在不同的线程中访问同一个 ThreadLocal 对象的 set 和 get 进行存取数据是不会相互干扰的。


【参考：https://blog.csdn.net/meism5/article/details/90413860 ThreadLocal 是什么？有哪些使用场景？】
    在线程池中使用 ThreadLocal 为什么可能导致内存泄露呢？
	在线程池中线程的存活时间太长，往往都是和程序同生共死的，这样 Thread 持有的 ThreadLocalMap 一直都不会被回收，再加上 ThreadLocalMap 中的 Entry 对 ThreadLocal 是弱引用（WeakReference），所以只要 ThreadLocal 结束了自己的生命周期是可以被回收掉的。
	Entry 中的 Value 是被 Entry 强引用的，即便 value 的生命周期结束了，value 也是无法被回收的，导致内存泄露。

    线程池中，如何正确使用 ThreadLocal？
	在 finally 代码块中手动清理 ThreadLocal 中的 value，调用 ThreadLocal 的 remove()方法。


52.说一下 synchronized 底层实现原理？
   【monitorenter 和 monitorexit 指令】
   synchronized 是由一对 monitorenter/monitorexit 指令实现的，monitor 对象是同步的基本实现单元。
   
   synchronized底层原理 Java面试热点：synchronized原理剖析与优化 ：https://www.bilibili.com/video/BV1JJ411J7Ym?p=13 
笔记如下：（javap -v -p xxx.class）
synchronized修饰对象(同步代码块)：
	private static Object obj = new Object()；
	synchronized(obj){ //monitorenter指令
	   system.out.println();
	}		   //monitorexit指令
   1.monitorenter小结：
	synchronized的锁对象会关联一个monitor，这个monitor不是我们主动创建的，是JVM的线程执行到这个同步代码块，发现锁对象没有monitor就会创建monitor，monitor内部有两个重要的成员变量owner：拥有这把锁的线程，recursion：会记录线程拥有锁的次数。当一个线程拥有monitor后其他线程只能等待。
   2.monitorexit释放锁：
  	monitorexit插入在方法结束和异常处，JVM保证每个monitorenter必须对应一个monitorexit。
	能执行monitorexit指令的线程一定是之前拥有过当前对象的monitor锁的线程。
	执行monitorexit时会将monitor的recursion减1，当recursion减为0时，线程就释放锁。
   3.面试题synchronized出现异常会释放锁吗？
   	会，会释放锁。同步代码块里出现异常会自动释放锁。
	
synchronized修饰方法(同步方法)：
	public synchronized void test(){ 
	   system.out.println();
	}
    1.同步方法在反汇编后，会增加acc_synchronized修饰，会隐式调用monitorenter和monitorexit。在执行同步方法前会调用monitorenter，在执行完同步方法后会调用monitorexit。
    
小结：
	通过javap反汇编我们看到synchronized使用变成了monitorenter和monitorexit两个指令，每个锁对象都会关联一个monitor(监视器，它才是真正的锁对象)，它内部有两个重要的成员变量owner：会保存获得锁的线程，recursion会保存线程获得锁的次数，当执行到monitorexit时，recursion会减1，当计数器减到0时，这个线程会释放锁。

   
53.synchronized 和 volatile 的区别是什么？

【volatile 仍然是从主内存到工作内存的拷贝，只是volatile修饰会在工作内存修改后迅速同步到主内存，从而使变量在值发生改变时能尽快地让其他线程知道。
保证了每个线程值的可见性（指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值），有序性（即程序执行的顺序按照代码的先后顺序执行）；但是不能保证原子性（一个线程中，不论是for循环还是其他，都要独立一次执行完。其他线程不能插入）；
而 atomic包下提供了一些原子操作类，即对基本数据类型的 自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作】

参考1：https://www.iteye.com/blog/723242038-2069470 volatile 与 synchronized 区别
使用volatile关键字：用一句话概括volatile,它能够使变量在值发生改变时能尽快地让其他线程知道.
volatile详解：
   首先我们要先意识到有这样的现象,编译器为了加快程序运行的速度,对一些变量的写操作会先在（线程的工作内存中）寄存器或者是CPU缓存上进行,最后才写入内存（主内存）.
   而在这个过程,变量的新值对其他线程是不可见的.而volatile的作用就是使它修饰的变量的读写操作都必须在内存中进行!
volatile与synchronized：
    volatile本质是在告诉jvm当前变量在（工作内存）寄存器中的值是不确定的,需要从主存中读取,synchronized则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住.

参考2：https://blog.csdn.net/vking_wang/article/details/9982709 【Java线程】volatile的适用场景
   把代码块声明为 synchronized，有两个重要后果，通常是指该代码具有 原子性（atomicity）和 可见性（visibility）。
   原子性 意味着某个时刻，只有一个线程能够执行一段代码，这段代码通过一个monitor object保护。从而防止多个线程在更新共享状态时相互冲突。
   可见性 则更为微妙，它必须确保释放锁之前对共享数据做出的更改对于随后获得该锁的另一个线程是可见的。
      —— 如果没有同步机制提供的这种可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发许多严重问题。

volatile的使用条件:
   Volatile 变量具有 synchronized 的可见性特性，但是不具备原子性。这就是说线程 能够自动发现 volatile 变量的最新值。

   Volatile 变量可用于提供线程安全，但是只能应用于非常有限的一组用例：多个变量之间或者某个变量的当前值与修改后值之间没有约束。
   因此，单独使用 volatile 还不足以实现计数器、互斥锁或任何具有与多个变量相关的不变式（Invariants）的类（例如 “start <=end”）。

volatile的适用场景：
    模式 #1：状态标志
	也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。

	volatile boolean shutdownRequested;
	...
	public void shutdown() { 
	    shutdownRequested = true; 
	}
	public void doWork() { 
	    while (!shutdownRequested) { 
		// do stuff
	    }
	}
   模式 #2：一次性安全发布（one-time safe publication）
   模式 #3：独立观察（independent observation）
   模式 #4：“volatile bean” 模式
   模式 #5：开销较低的“读－写锁”策略


volatile关键字的作用：
  java volatile关键字作用及使用场景 ：参考：https://www.cnblogs.com/YLsY/p/11295732.html
	1. volatile关键字的作用：保证了变量的可见性（visibility）。
	被volatile关键字修饰的变量，如果值发生了变更，其他线程立马可见，避免出现脏读的现象。【每个线程都会有自己的线程空间（称为工作内存），一开始时会从主内存copy数据到自己的内存中，如果有变量是volatile修饰的，但变量在某个线程中发生改变，写到主内存中后，会通知其他线程该变量有变化，重新赋新值。】
	载如以下代码片段，isShutDown被置为true后，doWork方法仍有执行。如用volatile修饰isShutDown变量，可避免此问题。
	2. 为什么会出现脏读？
	Java内存模型规定所有的变量都是存在主内存当中，每个线程都有自己的工作内存。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。变量的值何时从线程的工作内存写回主存，无法确定。

/*
volatile 是变量修饰符；synchronized 是修饰类、方法、代码段。
volatile 仅能实现变量的修改可见性，不能保证原子性；而 synchronized 则可以保证变量的修改可见性和原子性。
volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
*/

【volatile 变量每次读取前必须先从主内存刷新最新的值，每次写入后必须立即同步回主内存当中。总结，volatile变量可以实现线程之间的通信。
  参考：https://www.cnblogs.com/yangwei20160911/p/6536758.html
  https://developer.aliyun.com/ask/134175?spm=a2c6h.13159736 volatile变量的读取每次都是从主内存中获取么？】

54.synchronized 和 Lock 有什么区别？
【 synchronized 可以用在 类、对象、方法、代码块；
   Lock只能用在代码块上；需要手动释放；
   tryLock() 区别：当所别其他线程占用，Lock会等待，tryLock不会等待，但tryLock可以设置等待时，过时就停止】

【synchronized 可以给类、方法、代码块加锁；而 lock 只能给代码块加锁。
 synchronized 不需要手动获取锁和释放锁，使用简单，发生异常会自动释放锁，不会造成死锁；
  而 lock 需要自己加锁和释放锁，如果使用不当没有 unLock()去释放锁就会造成死锁。
通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。】

【面试题：synchronized 与 Lock 的区别 (https://www.bilibili.com/video/BV1JJ411J7Ym?p=14)
	1.synchronized是关键字，Lock是一个接口。
	2.synchronized会自动释放锁，而Lock必须手动释放锁。
	3.synchronized是不可中断的，Lock可以中断也可以不中断。
	4.通过Lock可以知道线程有没有拿到锁（调用lock.tryLock()方法会返回true或false），而synchronized不能。
	5.synchronized能锁住方法和代码块，而Lock只能锁住代码块。
	6.Lock可以使用读锁(ReentrantReadWriteLock)提高多线程 读效率。（Lock有一个实现类ReentrantReadWriteLock允许多个线程读，但只能有一个线程写）
	7.synchronized是非公平锁(非公平即，不是按照先来后到的顺序来竞争锁，而是随机的)，ReentrantLock可以控制是否是公平锁。
】
      【 实现层面不一样。synchronized 是 Java 关键字，JVM层面 实现加锁和释放锁；Lock 是一个接口，在代码层面实现加锁和释放锁
	是否自动释放锁。synchronized 在线程代码执行完或出现异常时自动释放锁；Lock 不会自动释放锁，需要再 finally {} 代码块显式地中释放锁
	是否一直等待。synchronized 会导致线程拿不到锁一直等待；Lock 可以设置尝试获取锁或者获取锁失败一定时间超时
	获取锁成功是否可知。synchronized 无法得知是否获取锁成功；Lock 可以通过 tryLock 获得加锁是否成功
	功能复杂性。synchronized 加锁可重入、不可中断、非公平；Lock 可重入、可判断、可公平和不公平、细分读写锁提高效率
	参考：https://blog.csdn.net/meism5/article/details/90413893 synchronized 和 Lock 有什么区别？】

synchronized的使用：
	public class TestSynDemo {

	    private static Object obj = new Object();

	    // 1.synchronized 锁方法
	    public synchronized void test(){
		System.out.println("object == ");
	    }

	    // 2. synchronized 锁对象
	    public void testOne(){
		synchronized (obj) {
		    System.out.println("object == ");
		}
	    }

	    // 3. synchronized修饰代码块
	    public void testTwo(){
		synchronized (this) {
		    System.out.println("object == ");
		}
	    }
	    
	    // 4. synchronized 锁 类.class
	    public void testThree(){
		synchronized (TestSynDemo.class) {
		    System.out.println("object == ");
		}
	    }
	}
    

55.synchronized 和 ReentrantLock 区别是什么？

ReentrantLock可以控制是否是公平锁

使用：
  Lock lock = new ReentrantLock(); //Lock是接口，ReentrantLock是Lock的常用实现类
  try{
     lock. lock();...
  }

参考：https://blog.csdn.net/weixin_42739473/article/details/81220767 Lock和实现类中ReentrantLock中的lock()与tryLock()区别
  ReentrantLock是Lock的常用实现类，lock.lock()是以lock对象为锁，代码中使用的同一个lock对象，所以使用的是同一把锁，
  一个线程通过lock()方法拿到了锁，执行了System.out.println("执行了方法");打印出结果，之后没有调用lock.unlock()方法，所以没有释放锁，
  第二个线程在调用lock.lock()控制台就一直阻塞在那，说明lock.lock()是一个阻塞方法，而lock.tryLock()是不同，他是一个有返回值的方法，在没有设置longTime和TimeUnit时，即试图获取锁，没有获取到就返回false,获取到就返回true,这个方法是立即返回的，
  当你调用lock.lock(long time,TimeUnit unit)，意思就是 在指定时间范围内阻塞，规定时间范围内获取到锁就返回true,否则就返回false，注意：如果在当前线程中以将中断表示位设为true，例如Thread.currentThread().interrupt();执行lock.lock(long time,TimeUnit unit)就会报java.lang.InterruptedException异常

【同上面试题：synchronized 与 Lock 的区别 (https://www.bilibili.com/video/BV1JJ411J7Ym?p=14)
	1.synchronized是关键字，Lock是一个接口。
	2.synchronized会自动释放锁，而Lock必须手动释放锁。
	3.synchronized是不可中断的，Lock可以中断也可以不中断。
	4.通过Lock可以知道线程有没有拿到锁（调用lock.tryLock()方法会返回true或false），而synchronized不能。
	5.synchronized能锁住方法和代码块，而Lock只能锁住代码块。
	6.Lock可以使用读锁(ReentrantReadWriteLock)提高多线程 读效率。（Lock有一个实现类ReentrantReadWriteLock允许多个线程读，但只能有一个线程写）
	7.synchronized是非公平锁(非公平即，不是按照先来后到的顺序来竞争锁，而是随机的)，ReentrantLock可以控制是否是公平锁。
】


56.说一下 atomic 的原理？
     在多线程的场景中，我们需要保证数据安全，就会考虑同步的方案，通常会使用synchronized或者lock来处理，使用了synchronized意味着内核态的一次切换。这是一个很重的操作。

     有没有一种方式，可以比较便利的实现一些简单的数据同步，比如计数器等等。concurrent包下的atomic提供我们这么一种轻量级的数据同步的选择。

class MyThread implements Runnable {
 
    static AtomicInteger ai=new AtomicInteger(0);
 
    public void run() {
        for (int m = 0; m < 1000000; m++) {
            ai.getAndIncrement();
        }
    }
};
 
public class TestAtomicInteger {
    public static void main(String[] args) throws InterruptedException {
        MyThread mt = new MyThread();
 
        Thread t1 = new Thread(mt);
        Thread t2 = new Thread(mt);
        t1.start();
        t2.start();
        Thread.sleep(500);
        System.out.println(MyThread.ai.get());
    }

参考1：https://mp.weixin.qq.com/s/aw6OXC9wkxH42rCywNd7yQ Java并发编程包中atomic的实现原理
参考2：https://blog.csdn.net/wuzhiwei549/article/details/82621947 atomic的实现原理
在以上代码中，使用AtomicInteger声明了一个全局变量，并且在多线程中进行自增，代码中并没有进行显示的加锁。
以上代码的输出结果，永远都是 2000000。如果将AtomicInteger换成Integer，打印结果基本都是小于 2000000。
也就说明AtomicInteger声明的变量，在多线程场景中的自增操作是可以保证线程安全的。接下来我们分析下其原理。

原理：
   我们可以看一下AtomicInteger的代码，private volatile int value;他的值是存在一个volatile的int里面。
   而volatile只能保证这个变量的可见性。不能保证他的原子性。
   可以看到getAndIncrement这个类似i++的函数，可以发现，是调用了UnSafe中的getAndAddInt。
   进一步，我们可以发现实现方式： image.png
        如何保证原子性：自旋 + CAS（乐观锁）。在这个过程中，通过compareAndSwapInt比较更新value值，如果更新失败，重新获取旧值，然后更新。

优缺点：
    CAS相对于其他锁，不会进行内核态操作，有着一些性能的提升。但同时引入自旋，当锁竞争较大的时候，自旋次数会增多。cpu资源会消耗很高。
    换句话说，CAS+自旋适合使用在低并发有同步数据的应用场景。
    

Java 8做出的改进和努力：
    在Java 8中引入了4个新的计数器类型，LongAdder、LongAccumulator、DoubleAdder、DoubleAccumulator。他们都是继承于Striped64。
    在LongAdder 与AtomicLong有什么区别？
    Atomic* 遇到的问题是，只能运用于低并发场景。因此LongAddr在这基础上引入了分段锁的概念。可以参考《JDK8系列之LongAdder解析》一起看看做了什么。 大概就是当竞争不激烈的时候，所有线程都是通过CAS对同一个变量（Base）进行修改，当竞争激烈的时候，会将根据当前线程哈希到对于Cell上进行修改（多段锁）。

    可以看到大概实现原理是：通过CAS乐观锁保证原子性，通过自旋保证当次修改的最终修改成功，通过降低锁粒度（多段锁）增加并发性能。    

【volatile 仍然是从主内存到工作内存的拷贝，只是volatile修饰会在工作内存修改后迅速同步到主内存，从而使变量在值发生改变时能尽快地让其他线程知道。
保证了每个线程值的可见性（指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值），有序性（即程序执行的顺序按照代码的先后顺序执行）；但是不能保证原子性（一个线程中，不论是for循环还是其他，都要独立一次执行完。其他线程不能插入）；
而 atomic包下提供了一些原子操作类，即对基本数据类型的 自增（加1操作），自减（减1操作）、以及加法操作（加一个数），减法操作（减一个数）进行了封装，保证这些操作是原子性操作】
public class CountTest {
    // 请求总数
    public static int clientTotal = 5000;
    public static volatile int count = 0;

    public static void main(String[] args) throws Exception {
        //使用CountDownLatch来等待计算线程执行完
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        //开启clientTotal个线程进行累加操作
        for(int i=0;i<clientTotal;i++){
            new Thread(){
                public void run(){
                    count++;//自加操作
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(count);
    }
}
====================================
public class CountTest {
    // 请求总数
    public static int clientTotal = 5000;
    public static AtomicInteger count = new AtomicInteger(0); //上面的程序修改处 1

    public static void main(String[] args) throws Exception {
        //使用CountDownLatch来等待计算线程执行完
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        //开启clientTotal个线程进行累加操作
        for(int i=0;i<clientTotal;i++){
            new Thread(){
                public void run(){
                    count.incrementAndGet();//先加1，再get到值	//上面的程序修改处 2
                    countDownLatch.countDown();
                }
            }.start();
        }
        //等待计算线程执行完
        countDownLatch.await();
        System.out.println(count);
    }
}

** 参考：https://www.cnblogs.com/java-chen-hao/p/9968544.html 并发编程（一）—— volatile关键字和 atomic包

扩展：
** https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650121285&idx=1&sn=7cf9b5badd6d38b57ccfbfed63d3aad1&chksm=f36bb964c41c307218914cab6c1592f649281460e4ec1f85627c1311441c8a43ee1854440552&scene=21#wechat_redirect 乐观锁的一种实现方式——CAS()

锁存在的问题
    Java在JDK1.5之前都是靠synchronized关键字保证同步的，这种通过使用一致的锁定协议来协调对共享状态的访问，可以确保无论哪个线程持有共享变量的锁，都采用独占的方式来访问这些变量。独占锁其实就是一种悲观锁，所以可以说synchronized是悲观锁。

悲观锁机制存在以下问题：
	1.在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。
	2.一个线程持有锁会导致其它所有需要此锁的线程挂起。
	3.如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。
    
    而另一个更加有效的锁就是乐观锁。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。
    与锁相比，volatile变量是一个更轻量级的同步机制，因为在使用这些变量时不会发生上下文切换和线程调度等操作，但是volatile不能解决原子性问题，因此当一个变量依赖旧值时就不能使用volatile变量。因此对于同步最终还是要回到锁机制上来。

乐观锁：
    乐观锁（ Optimistic Locking）其实是一种思想。相对悲观锁而言，乐观锁假设认为数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。
    上面提到的乐观锁的概念中其实已经阐述了他的具体实现细节：主要就是两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是Compare and Swap(CAS)。

CAS：
    CAS是项乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。

CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”这其实和乐观锁的冲突检查+数据更新的原理是一样的。

***这里再强调一下，乐观锁是一种思想。CAS是这种思想的一种实现方式。

Java对CAS的支持：
    在JDK1.5 中新增java.util.concurrent(J.U.C)就是建立在CAS之上的。相对于对于synchronized这种阻塞算法，CAS是非阻塞算法的一种常见实现。所以J.U.C在性能上有了很大的提升。
    我们以java.util.concurrent中的AtomicInteger为例，看一下在不使用锁的情况下是如何保证线程安全的。主要理解getAndIncrement方法，该方法的作用相当于 ++i 操作。


四、反射

57.什么是反射？
【JDK原生动态代理（java动态代理） 和 cgLib动态代理】
	反射 是在Java运行状态中，对于任意一个类，都能够知道这个类的属性和方法；对于一个对象，都能够调用它的任意一个方法和属性，这种动态获取的信息 以及 动态调用对象的方法 的功能 称为Java语言的反射机制。


58.什么是 java 序列化？什么情况下需要序列化？
【Java序列化是内存中 对象 数据的状态 能够被读取出来】

Java序列化是 为了 保存 各种对象在内存中 的状态，并且可以把 保存的对象状态 再读取出来。
以下情况需要使用Java序列化：
	1.想把 内存中的对象状态 保存到 一个文件中 或者数据库中的时候；
	2.想用套接字 在网络上传送 对象的时候；
	3.想通过RMI（远程方法调用） 传输 对象的时候。

序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化,将数据分解成字节流，以便存储在文件中或在网络上传输
参考：https://blog.csdn.net/meism5/article/details/90413987 什么是 java 序列化？什么情况下需要序列化？
序列化：将 Java 对象转换成字节流的过程。
反序列化：将字节流转换成 Java 对象的过程。
当 Java 对象需要在网络上传输 或者 持久化存储到文件中时，就需要对 Java 对象进行序列化处理。
序列化的实现：类实现 Serializable 接口，这个接口没有需要实现的方法。实现 Serializable 接口是为了告诉 jvm 这个类的对象可以被序列化。

注意事项：
	某个类可以被序列化，则其子类也可以被序列化
	声明为 static 和 transient 的成员变量，不能被序列化。static 成员变量是描述类级别的属性，transient 表示临时数据
	反序列化读取序列化对象的顺序要保持一致


59.动态代理是什么？有哪些应用？
【程序运行期间，动态生成的代理类即为动态代理。动态代理的应用有 spring aop，Java注解对象获取 等】
 动态代理运行时 动态生成的 代理类。
 

60.怎么实现动态代理？

参考：https://blog.csdn.net/meism5/article/details/90413999 怎么实现动态代理？（包括 Chapter 4、JDK 动态代理、Chapter 5、CGLib 动态代理）

JDK 原生动态代理 和 cglib 动态代理。JDK 原生动态代理是基于接口实现的，而 cglib 是基于继承当前类的子类实现的。
【 1.动态代理：代理类在程序运行时创建的代理方式被成为动态代理。 
   代理模式上讲的(包括我们上面)是静态代理的例子中，代理类(studentProxy)是自己定义好的，在程序运行之前就已经编译完成。
   然而动态代理，代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的“指示”动态生成的。
   相比于静态代理，动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类中的方法。】

参考：https://www.cnblogs.com/gonjan-blog/p/6685611.html (博客园)java动态代理实现与原理详细分析
1.使用JDK动态代理：
	创建StuInvocationHandler类（代理类），实现InvocationHandler接口，这个类中持有一个被代理对象的实例target。
	InvocationHandler中有一个invoke方法，所有执行代理对象的方法都会被替换成执行invoke方法。再在invoke方法中执行被代理对象target的相应方法。
	
	当然，在代理过程中，我们在真正执行被代理对象的方法前加入自己其他处理。
	这也是Spring中的AOP实现的主要原理，这里还涉及到一个很重要的关于java反射方面的基础知识。

 public class ProxyTest {
    public static void main(String[] args) {
        
        //创建一个实例对象，这个对象是被代理的对象（真实对象）
        Person zhangsan = new Student("张三");
        
        //创建一个与代理对象相关联的InvocationHandler
        InvocationHandler stuHandler = new StuInvocationHandler<Person>(zhangsan);
        
        //创建一个代理对象stuProxy来代理zhangsan，代理对象的每个执行方法都会替换执行Invocation中的invoke方法
        Person stuProxy = (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler)；

       //代理执行上交班费的方法
        stuProxy.giveMoney();
    }
}
	我们创建了一个需要被代理的Person学生张三（真实对象），将zhangsan对象传给了stuHandler（自定义InvocationHadle，关联了Person真实对象）中，
	我们在创建代理对象stuProxy时，将stuHandler作为参数了的，上面也有说到所有执行代理对象的方法都会被替换成执行invoke方法，
	也就是说，最后执行的是StuInvocationHandler中的invoke方法。
	所以在看到下面的运行结果也就理所当然了。
	
**上面说到，动态代理的优势在于可以 很方便的对 代理类的函数 进行统一的处理，而不用修改每个代理类中的方法。是因为所有被代理执行的方法，都是通过在InvocationHandler中的invoke方法调用的，所以我们只要在invoke方法中统一处理，就可以对所有被代理的方法进行相同的操作了。
  例如，这里的方法计时，所有的被代理对象执行的方法都会被计时，然而我只做了很少的代码量。
  即，如果一个真实类就写一个代理类，第一多个真是类就会有很多代理类；第二是一个真是类中如果有多个方法需要计时(加强)，则代理类重写的每个方法都要写加强，耗时耗力。而使用InvocationHandler只需要一个，这就是动态代理。

  我们可以对InvocationHandler看做一个中介类，中介类持有一个被代理对象，在invoke方法中调用了被代理对象的相应方法。通过聚合方式持有被代理对象的引用，把外部对invoke的调用最终都转为对被代理对象的调用。
  代理类调用自己方法时，通过自身持有的中介类对象来调用中介类对象的invoke方法，从而达到代理执行被代理对象的方法。也就是说，动态代理通过中介类实现了具体的代理功能。
  五、总结
	生成的代理类：$Proxy0 extends Proxy implements Person，我们看到代理类(stuProxy)继承了Proxy类，所以也就决定了java动态代理只能对接口进行代理，Java的继承机制注定了这些动态代理类们无法实现对class的动态代理。
	上面的动态代理的例子，其实就是AOP的一个简单实现了，在目标对象的方法执行之前和执行之后进行了处理，对方法耗时统计。Spring的AOP实现其实也是用了Proxy和InvocationHandler这两个东西的。


2. cglib 动态代理
	为了解决 JDK的动态代理 无法代理不实现接口的类的问题，可以使用 CGLib 的实现动态代理。
	CGLib（Code Generator Library）是一个强大的、高性能的代码生成库。底层使用了ASM（一个短小精悍的字节码操作框架）来操作字节码生成新的类。
详细关于 CGLib 的疑难点：
	pom.xml依赖：1.cglib+asm(即网上的asm-3.3.1，而不是org.ow2.asm)依赖包 	2.Student(被代理类)需要空构造函数
		<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>3.2.10</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/asm/asm -->
		<dependency>
			<groupId>asm</groupId>
			<artifactId>asm</artifactId>
			<version>3.3.1</version>
		</dependency>

简单使用：
	1.新建代理类CgLibProxy 实现MethodInterceptor接口，重写intercept方法，
	其中methodProxy.invokeSuper(object, objects);为执行对应被代理类实际方法。
	2.使用：【参考https://www.cnblogs.com/xrq730/p/6661692.html】
	public static void main(String[] args) {
		/** 这是使用Cglib的通用写法，
		 *  setSuperclass表示设置要代理的类，
		 *  setCallback表示设置回调即MethodInterceptor的实现类，使用create()方法生成一个代理对象，注意要强转一下，因为返回的是Object。
		*/
		// Student student = new Student("silence"); 不需要
		CgLibProxy cgLibProxy = new CgLibProxy();

		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(Student.class);
		enhancer.setCallback(cgLibProxy);
		Student student = (Student)enhancer.create();

		student.display();
		student.updateName("hello");
		student.display();
	}
	
【代码使用也可参考： https://www.cnblogs.com/jqyp/archive/2010/08/20/1805041.html java动态代理（JDK和cglib）】

Spring AOP中JDK和CGLib动态代理商哪个更快？ https://www.songma.com/news/txtlist_i22903v.html
	
    总结：
	在1.8版本中运行屡次，基本都可以得到一致的测试结果，那就是 JDK动态代理商 已经比 CGLib动态代理商 快了！
	以后再有人问你Spring AOP，不要简单的说JDK动态代理商和CGLib这两个了，是时候的可以抛出来对两者之间区别的了解，是有加分的哦！

一、基本概念
	首先，我们知道Spring AOP的底层实现有两种方式：一种是JDK动态代理商，另一种是CGLib的方式。
	自Java 1.3以后，Java提供了动态代理商技术，允许开发者在运行期创立接口的代理商实例，后来这项技术被使用到了Spring的很多地方。

	JDK动态代理商主要涉及java.lang.reflect包下边的两个类：Proxy和InvocationHandler。
	其中，InvocationHandler是一个接口，可以通过实现该接口定义横切逻辑，并通过反射机制调使用目标类的代码，动态地将横切逻辑和业务逻辑编织在一起。
	JDK动态代理商的话，他有一个限制，就是它只能为接口创立代理商实例，而对于没有通过接口定义业务方法的类，如何创立动态代理商实例哪？答案就是CGLib。
	CGLib采使用底层的字节码技术，全称是：Code Generation Library，CGLib可以为一个类创立一个子类，在子类中采使用方法阻拦的技术阻拦所有父类方法的调使用并顺势织入横切逻辑。

二、JDK 和 CGLib动态代理商区别
    1、JDK动态代理商具体实现原理：
    【参考：https://www.cnblogs.com/yeyang/p/10087293.html 或者 https://blog.csdn.net/yhl_jxy/article/details/80586785】
	通过实现InvocationHandler接口创立自己的调使用解决器；
	通过为Proxy类( 通过调用Proxy.newProxyInstance()生成代理对象 )指定ClassLoader对象和一组interface来创立动态代理商；
	通过反射机制获取动态代理商类的 构造函数，其唯一参数类型就是调使用解决器 接口类型；(调用代理类的构造器，构造器参数为InvocationHandler的接口实现类)
	通过构造函数创立动态代理商类实例，构造时调使用解决器对象作为参数参入；
【生成代理对象的方法调用链为：
	Proxy.newProxyInstance()----->getProxyClass0()----->ProxyClassFactory.apply()----->ProxyGenerator.generateProxyClass()。
  我们下面依次对各个阶段进行分析：
      Proxy.newProxyInstance() //指定ClassLoader对象和一组interface来生成代理对象
      getProxyClass0() //根据提供的接口（在我们的例子中，即Subject接口），生成代理类
      final Constructor<?> cons = cl.getConstructor(constructorParams); //调用代理类的构造器，构造器参数为InvocationHandler的接口实现类（在我们的例子中，即JdkProxySubject）
】
    JDK动态代理商是 面向接口 的代理商模式，假如被代理商目标没有接口那么Spring也无能为力，Spring通过Java的反射机制生产被代理商接口的新的匿名实现类，重写了其中AOP的加强方法。

    2、CGLib动态代理商：
	CGLib是一个强大、高性能的Code生产类库，可以实现运行期动态扩展java类，Spring在运行期间通过 CGlib继承要被动态代理商的类，重写父类的方法，实现AOP面向切面编程呢。
	
    3、两者比照：
	JDK动态代理商是面向接口的。
	CGLib动态代理商是通过 字节码底层 继承 要代理商类 来实现（假如被代理商类被final关键字所修饰，那么抱歉会失败）。

    4、用注意：
	假如要被代理商的对象 是个实现类，那么Spring会用JDK动态代理商来完成操作（Spirng默认采使用JDK动态代理商实现机制）；
	假如要被代理商的对象 不是个实现类, 那么Spring会强制用CGLib来实现动态代理商。



五、对象拷贝

61.为什么要使用克隆？

62.如何实现对象克隆？

63.深拷贝和浅拷贝区别是什么？



六、Java Web

64.jsp 和 servlet 有什么区别？
【jsp本质也是servlet】
参考：https://blog.csdn.net/anwarkanji/article/details/90526155 jsp和servlet区别
1.Servlet：
	Servlet 是一种服务器端的Java应用程序，具有独立于平台和协议的特性，可以生成动态的Web页面。它担当客户请求（Web浏览器或其他HTTP客户程序）与服务器响应（HTTP服务器上的数据库或应用程序）的中间层。 Servlet是位于Web服务器内部的服务器端的Java应用程序，与传统的从命令行启动的Java应用程序不同，Servlet由Web服务器进行加载，该Web服务器必须包含支持Servlet的Java虚拟机。

2.JSP：
	JSP 全名为 Java Server Pages，中文名叫java服务器页面，其根本是一个简化的Servlet设计。 
	JSP(JavaServer Pages)是一种动态页面技术，它的主要目的是将表示逻辑从Servlet中分离出来。

3.相同点：
	jsp经编译后就变成了servlet，jsp本质就是servlet，jvm只能识别java的类，不能识别jsp代码，web容器将jsp的代码编译成jvm能够识别的java类。

4.不同点：
	JSP侧重视图，Sevlet主要用于控制逻辑。

	Servlet中没有内置对象 。
	JSP中的内置对象都是必须通过HttpServletRequest对象，HttpServletResponse对象以及HttpServlet对象得到。

**JS和JSP相比较：虽然JS可以在客户端动态生成HTML，但是很难与服务器交互，因此不能提供复杂的服务。如：访问数据库和图像处理等等。
JSP在HTML中用<% %>里面实现。JS在HTML中用<Scrippt></Script>实现。

JSP 是 servlet 技术的扩展，本质上就是 servlet 的简易方式。
servlet 和 JSP 最主要的不同点在于，servlet 的应用逻辑是在 Java 文件中，并且完全从表示层中的 html 里分离开来，
而 JSP 的情况是 Java 和 html 可以组合成一个扩展名为 JSP 的文件。
JSP 侧重于视图，servlet 主要用于控制逻辑。


65.jsp 有哪些内置对象？作用分别是什么？
	request：用户端请求，此请求会包含来自来自GET/POST请求的参数
	response：网页传回用户端的回应
	pageContext：网页的属性是在这里管理的
	session：与请求有关的
	application：servlet正在执行的内容
	out：用来传送回应的输出
	page：JSP网页本身
	config：servlet的架构部件
	exception：针对错误网页，未捕捉的例外

JSP 有 9 大内置对象：
	request：封装客户端的请求，其中包含来自 get 或 post 请求的参数；
	response：封装服务器对客户端的响应；
	pageContext：通过该对象可以获取其他对象；
	session：封装用户会话的对象；
	application：封装服务器运行环境的对象；
	out：输出服务器响应的输出流对象；
	config：Web 应用的配置对象；
	page：JSP 页面本身（相当于 Java 程序中的 this）；
	exception：封装页面抛出异常的对象。


66.说一下 jsp 的 4 种作用域？


67.session（HttpSession） 和 cookie 有什么区别？
存储位置不同：session用于服务器端（后台）存储数据，cookie用于浏览器（前端）存储数据；
安全性不同：session存储在服务器中，相对安全，cookie 安全性一般，因为cookie存储在浏览器中，可以被伪造和修改；
容量和个数限制：cookie 有容量限制，每个站点下的 cookie 也有个数限制。
	       单个cookie保存的数据<=4KB，一个站点最多保存20个Cookie。
	       对于session来说并没有上限，但出于对服务器端的性能考虑，session内不要存放过多的东西，并且设置session删除机制。
存储的多样性：session 可以存储在 Redis 中、数据库中、应用程序中；而 cookie 只能存储在浏览器中。
跨域支持上不同：cookie支持跨域名访问。
	       session不支持跨域名访问。
参考：https://www.cnblogs.com/mark5/p/11641131.html cookie和session的区别有哪些


(HttpSessiion)使用参考：https://github.com/nameof/RedisSSO 
基于Redis封装自定义HttpSession，实现跨域单点登录和Session共享，支持android客户端扫码登陆，并使用redis模拟消息队列进行注销消息的发送。


** 参考：https://blog.csdn.net/qq_42651904/article/details/85543640 Java中Cookie的使用（Cookie 和Session的区别）
区别	Cookie	Session
存在:
	Cookie是客户端技术，通常保存在客户端，即本地，IE浏览器把Cookie信息保存在类似于C:\windows\cookies的目录下。
	因为Cookie在客户端所以可以编辑伪造，不是十分安全。
	Session是服务器端技术，在服务端，利用这个技术，服务器在运行时可以为每一个用户的浏览器创建一个其独享的session对象，
	由于session为用户浏览器独享，所以用户在访问服务器的web资源时，可以把各自的数据放在各自的session中，当用户再去访问服务器中的其它web资源时，其它web资源再从用户各自的session中取出数据为用户服务。
	
存储数据：	Cookie只能存储 String 类型的对象；	Session 能够存储任意的 java 对象
性能：	Cookie存在客户端对服务器没影响；	Session过多时会消耗服务器资源，大型网站会有专门Session服务器
作用域：	Cookie通过设置指定作用域只能在指定作用域有效；	Session在整个网页都有效
作用时间：	Cookie可以通过 setMaxAge设置有效时间，即使浏览器关闭了仍然存在；	关闭网页Session就结束了


68.说一下 session 的工作原理？
	session 的工作原理是客户端登录完成之后，服务器会创建对应的 session，
	session 创建完之后，会把 session 的 id 发送给客户端，客户端再存储到浏览器中。
	这样客户端每次访问服务器时，都会带着 sessionid，服务器拿到 sessionid 之后，在内存找到与之对应的 session 这样就可以正常工作了。


69.如果客户端禁止 cookie 能实现 session 还能用吗？
可以，session只是依赖cookie存储sessionID，如果cooki禁用了，可以使用url中添加sessionID的方式保证session能正常使用。


70.spring mvc 和 struts 的区别是什么？
【拦截级别：struts2 是类级别的拦截；spring mvc 是方法级别的拦截。】

71.如何避免 sql 注入？
	使用预处理 PreparedStatement。
	使用正则表达式过滤掉字符中的特殊字符。

【2. mybatis防止sql注入】
参考：https://blog.csdn.net/alan_liuyue/article/details/88314299
  1.简介：SQL注入就是客户端在向服务器发送请求的时候，sql命令通过表单提交或者url字符串拼接传递到后台持久层，最终达到欺骗服务器执行恶意的SQL命令；
防止：
    (1) 前端：前端表单进行参数格式控制；
    (2) 后端：2.1 后台进行参数格式化，过滤所有涉及sql的非法字符；
           2.2 持久层使用参数化的持久化sql，使用预编译语句集，切忌使用拼接字符串；


  2. mybatis防止sql注入：
	mybatis是一款优秀的持久层框架，在防止sql注入方面，mybatis启用了预编译功能，在所有的SQL执行前，都会先将SQL发送给数据库进行编译，执行时，直接替换占位符"?"即可；

	mybatis在处理传参的时候，有两种处理方式，一种是#，另一种是$；
	  #{XXX}，将传入参数都当成一个字符串，会自动加双引号，已经经过预编译，安全性高，能很大程度上防止sql注入；
	  ${XXX}，该方式则会直接将参数嵌入sql语句，未经过预编译，安全性低，无法防止sql注入；



72.什么是 XSS 攻击，如何避免？

73.什么是 CSRF 攻击，如何避免？



七、异常

74.throw 和 throws 的区别？

【throw 用于try-catch或代码中直接抛出一个异常，throws用于方法声明定义可能抛出异常】
  throw是真实抛出一个异常（代码中用）、throws是声明可能抛出一个异常（方法声明用），像extends和implements


75.final、finally、finalize 有什么区别？

【 final 是修饰符，修饰类、方法、变量；被final修饰的类不能被继承、方法不能被重写/修改、变量不能修改；
   finally 是 try-catch-finally 最后一部分，可以省略，若存在，finall一定会执行，即使try中有return了，还是会执行finally再返回；
   finalized(定案、终结) 是 Object 类的一个方法，垃圾回收器执行时会调用 被回收对象的该方法 】


76.try-catch-finally 中哪个部分可以省略？

【catch 或 finally可以省略；但不能同时省略】
try-catch-finally 其中catch 或 finally 都可以省略，但是不能同时省略，也就是说有try的时候，后面必须有一个catch或finally。


77.try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

【会，finally完再return】finally 一定会执行，即使是 catch 中 return 了，catch 中的 return 会等 finally 中的代码执行完之后，才会执行。


78.常见的异常类有哪些？
NullPointException、IOException、ClassCastException 类转换异常、NoSuchMethodException 方法不存在异常



八、网络

79.http 响应码 301 和 302 代表的是什么？有什么区别？
	301：永久重定向。
	302：暂时重定向。
	它们的区别是，301 对搜索引擎优化（SEO）更加有利；302 有被提示为网络拦截的风险。

百度参考：
  301转向(或叫301重定向，301跳转)是当用户或搜索引擎向网站服务器发出浏览请求时，服务器返回的HTTP数据流中头信息(header)中的状态码的一种，表示本网页永久性转移到另一个地址。

参考菜鸟：
	301	Moved Permanently	
		永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
	302	Found	
		临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI


80.forward 和 redirect 的区别？

【 forward 转发，参数会携带(可以共享 request 里的数据)，url地址栏不会发生改变；
   redirect 重定向，参数不会携带(不能共享)，url地址栏会改变 】
 (1) 地址栏Url：forward 转发 地址栏 不会发生改变，redirect 重定向会改变；
 (2) request 数据共享：forward 可以共享 request的数据，redirect不能；
 (3) 效率：forward的效率要 比 redirect高。


81.简述 tcp 和 udp的区别？
【两种协议都是传输层协议，为应用层提供信息载体。
  TCP与UDP的适用场景
    TCP：对数据传输的质量有较高要求，但对实时性要求不高。比如HTTP，HTTPS，FTP等传输文件的协议以及POP，SMTP等邮件传输的协议，应选用TCP协议。
    UDP：只对数据传输的 实时性 要求较高，但不对传输质量有要求。比如视频传输、实时通信等，应选用UDP协议。】

【  一、TCP协议：(可靠：三次握手+四次挥手)
      (Transmission Control Protocol / Internet Protocol 传输控制协议,传输数据前需要先建立连接,是一种可靠的、基于字节流的传输层通信协议)
	位于传输层， 提供可靠的字节流服务。
	所谓的字节流服务（Byte Stream Service） 是指， 为了方便传输， 将大块数据分割成以报文段（segment） 为单位的数据包进行管理。
	而可靠的传输服务是指， 能够把数据准确可靠地传给对方。
    
    二、UDP协议：
      (User Datagram Protocol 用户数据报协议,传输数据的时候不需要建立连接,是一种面向事务的简单不可靠的信息传送协议。)
	无连接协议，也称透明协议，也位于传输层。

    三、两者区别：
	1） TCP提供面向连接的传输，通信前要先建立连接（三次握手机制）； UDP提供无连接的传输，通信前不需要建立连接。
	2） TCP提供可靠的传输（有序，无差错，不丢失，不重复）； UDP提供不可靠的传输。
	3） TCP面向字节流的传输，因此它能将信息分割成组，并在接收端将其重组； UDP是面向数据报的传输，没有分组开销。
	4） TCP提供拥塞控制和流量控制机制； UDP不提供拥塞控制和流量控制机制。
】

tcp 和 udp 是 OSI 模型中的运输层中的协议。
tcp 提供可靠的通信传输，而 udp 则常被用于让广播和细节控制交给应用的通信传输。
两者的区别大致如下：
	tcp 面向连接，udp 面向非连接即发送数据前不需要建立链接；
	tcp 提供可靠的服务（数据传输），udp 无法保证；
	tcp 面向字节流，udp 面向报文；
	tcp 数据传输慢，udp 数据传输快；


82.tcp 为什么要三次握手，两次不行吗？为什么？【TCP建立连接为什么是三次握手？】

【 TCP的三次握手(建立连接)，四次分手(断开连接)分别是什么意思？
   参考：通俗大白话来理解TCP协议的三次握手和四次分手 (https://github.com/jawil/blog/issues/14)

   **三次握手不是TCP本身的要求, 而是为了满足"在不可靠信道上可靠地传输信息"这一需求所导致的。
   进行三次握手来保证连接是双工的。 】即哔哩哔哩上说的信道传输信息不是100%可靠的

【 三次握手： 1.客户端 --> 建立连接好吗？ 2. 服务器 --> 同意，可以  3.客户端 --> 好的，建立连接 ；完成。 
   四次分手： 1.客户端 --> 要断开连接！？ 2. 服务器 --> 同意，可以  3.客户端 --> 好的，断开连接 。4.服务器 --> 等待确认，断开连接。
   参考：https://www.cnblogs.com/yuilin/archive/2012/11/05/2755298.html#!comments 】


83.说一下 tcp 粘包是怎么产生的？



84.OSI 的七层模型都有哪些？

【应用层(QQ等应用) --> 表示层(打包员) --> 会话层(建立通信) --> 传输层(传送员) --> 网络层(规划路线) --> 数据链路层(路线执行) --> 物理层(0、1元数据)】

OSI七层模式简单通俗理解：https://blog.csdn.net/xzongyuan/article/details/10371331

掘金参考：https://juejin.im/post/59eb06b1f265da430f313c7f

物理层：利用传输介质为数据链路层提供物理连接，实现比特流的透明传输。
数据链路层：负责建立和管理节点间的链路。
网络层：通过路由选择算法，为报文或分组通过通信子网选择最适当的路径。
传输层：向用户提供可靠的端到端的差错和流量控制，保证报文的正确传输。
会话层：向两个实体的表示层提供建立和使用连接的方法。
表示层：处理用户信息的表示问题，如编码、数据格式转换和加密解密等。
应用层：直接向用户提供服务，完成用户希望在网络上完成的各种工作。


85.get 和 post 请求有哪些区别？
	get 请求会被浏览器主动缓存，而 post 不会。
	get 传递参数有大小限制，而 post 没有。
	post 参数传输更安全，get 的参数会明文限制在 url 上，post 不会。

【get 参数会显示在地址栏，post不会；post比get安全(为什么?)；】

最佳参考： https://www.cnblogs.com/logsharing/p/8448446.html

GET在浏览器回退时是无害的，而POST回退会再次提交请求。
GET产生的URL地址可以被Bookmark(可收藏为书签)，而POST不可以。
GET请求会被浏览器主动cache(缓存)，而POST不会，除非手动设置。
GET请求 只能 进行url编码，而POST支持 多种编码方式 。
GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
GET请求在URL中传送的 参数是有长度限制 的，而POST么有。
对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
GET参数通过URL传递，POST放在Request body中。

重点：
      GET和POST是什么？HTTP协议中的两种发送请求的方法。HTTP是什么？HTTP是基于TCP/IP的关于数据如何在万维网中如何通信的协议。
      HTTP的底层是TCP/IP。所以GET和POST的底层也是TCP/IP，也就是说，GET/POST都是TCP链接。GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上是完全行的通的。 

      HTTP只是个行为准则，而TCP才是GET和POST怎么实现的基本。
     (如果你用GET服务，在request body偷偷藏了数据，不同服务器的处理方式也是不同的，有些服务器会帮你卸货，读出数据，有些服务器直接忽略，所以，虽然GET可以带request body，也不能保证一定能被接收到哦。)

     *** GET和POST还有一个重大区别，简单的说：   GET产生一个TCP数据包；POST产生两个TCP数据包。(对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）； 而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。)


86.如何实现跨域？

所谓同源是指，域名，协议，端口均相同，不明白没关系，举个栗子：
http://www.123.com/index.html 调用 http://www.123.com/server.php （非跨域）
http://www.123.com/index.html 调用 http://www.456.com/server.php （主域名不同:123/456，跨域）
http://abc.123.com/index.html 调用 http://def.123.com/server.php （子域名不同:abc/def，跨域）
http://www.123.com:8080/index.html 调用 http://www.123.com:8081/server.php （端口不同:8080/8081，跨域）


【JSONP是JSON with Padding的略称。
 它是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式）】
   参考：https://www.cnblogs.com/itmacy/p/6958181.html 跨域问题：解决跨域的三种方案
	当前端页面与后台运行在不同的服务器时，就必定会出现跨域这一问题，本篇简单介绍解决跨域的三种方案，部分代码截图如下，仅供参考：
	方式一：使用ajax的jsonp
	前端代码 & 服务器代码
	 使用该方式的缺点：请求方式只能是get请求

	方式二：使用jQuery的jsonp插件
	插件下载网址：https://github.com/jaubourg/jquery-jsonp
	前端代码 & 服务器代码
	 使用该方式的特点：与方式一相比，请求方式不只局限于get请求，还可以是post请求，但从服务器从获取的数据依然是jsonp格式

	方式三：使用cors
	前端代码 & 服务器代码
	**使用该方式的特点：与前两种方式相比，前端代码和未处理跨域前一样，即普通的ajax请求，但服务器代码添加了一段解决跨域的代码

	    // 设置：Access-Control-Allow-Origin头，处理Session问题
		response.setHeader("Access-Control-Allow-Origin", request.getHeader("Origin"));
		response.setHeader("Access-Control-Allow-Credentials", "true");
		response.setHeader("P3P", "CP=CAO PSA OUR");
		if (request.getHeader("Access-Control-Request-Method") != null && "OPTIONS".equals(request.getMethod())) {
		    response.addHeader("Access-Control-Allow-Methods", "POST,GET,TRACE,OPTIONS");
		    response.addHeader("Access-Control-Allow-Headers", "Content-Type,Origin,Accept");
		    response.addHeader("Access-Control-Max-Age", "120");
		}

	cors高级使用：在springmvc中配置拦截器
	创建跨域拦截器实现HandlerInterceptor接口，并实现其方法，在请求处理前设置头信息，并放行
	（在springmvc的配置文件中配置拦截器，注意拦截的是所有的文件）


87.说一下 JSONP 实现原理？

**jsonp：JSON with Padding，它是利用script标签的 src 连接可以访问不同源的特性，加载远程返回的“JS 函数”来执行的。

参考：https://www.cnblogs.com/soyxiaobi/p/9616011.html 彻底弄懂jsonp原理及实现方法
【JSONP是JSON with Padding的略称。
  它是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式）】

　<script>标签的src属性并不被同源策略所约束，所以可以获取任何服务器上脚本并执行。
  四、JSONP的实现模式--CallBack
	刚才的小例子讲解了跨域的原理，我们回上去再看看JSONP的定义说明中讲到了javascript callback的形式。那我们就来修改下代码，如何实现JSONP的javascript callback的形式。

程序A-20001中sample的部分代码：
	<script type="text/javascript">
		//回调函数
		function callback(data) {
		    alert(data.message);
		}
	</script>
	<script type="text/javascript" src="http://localhost:20002/test.js"></script>

程序B-20002中test.js的代码：
	//调用callback函数，并以json数据形式作为阐述传递，完成回调
	callback({message:"success"});

这其实就是JSONP的简单实现模式，或者说是JSONP的原型：创建一个回调函数，然后在远程服务上调用这个函数并且将JSON 数据形式作为参数传递，完成回调。

将JSON数据填充进回调函数，这就是JSONP的JSON+Padding的含义吧。

一般情况下，我们希望这个script标签能够动态的调用，而不是像上面因为固定在html里面所以没等页面显示就执行了，很不灵活。我们可以通过javascript动态的创建script标签，这样我们就可以灵活调用远程服务了。



九、设计模式

88.说一下你熟悉的设计模式？
单例模式：保证对象实例只创建一次，节省系统开销。
工厂模式（简单工厂、抽象工厂）：解耦代码。

1.简单工厂：用来生产同一等级结构中的任意产品，对于增加新的产品，无能为力。 （新建一工厂类，if-else判断的形式）
举例：【PhoneFactory类：手机代工厂（Factory）
	public class PhoneFactory {
	    public Phone makePhone(String phoneType) {
		if(phoneType.equalsIgnoreCase("MiPhone")){
		    return new MiPhone();
		}
		else if(phoneType.equalsIgnoreCase("iPhone")) {
		    return new IPhone();
		}
		return null;
	    }
	}
】

2.工厂方法：用来生产同一等级结构中的固定产品，支持增加任意产品。（新增产品时，就新增对应的工厂类）
【定义一个抽象工厂，其定义了产品的生产接口，但不负责具体的产品，将生产任务交给不同的派生类工厂。这样不用通过指定类型来创建对象了。
举例：【 AppleFactory类：生产苹果手机的工厂（ConcreteFactory2）
	public class AppleFactory implements AbstractFactory {
	    @Override
	    public Phone makePhone() {
		return new IPhone();
	    }
	}

3.抽象工厂：用来生产不同产品族的全部产品，对于增加新的产品，无能为力；支持增加产品族。


89.简单工厂和抽象工厂有什么区别？

简单工厂：用来生产同一等级结构中的任意产品，对于增加新的产品，无能为力。 （新建一工厂类，if-else判断的形式）
工厂方法：用来生产同一等级结构中的固定产品，支持增加任意产品。
抽象工厂：用来生产不同产品族的全部产品，对于增加新的产品，无能为力；支持增加产品族。


十、Spring/Spring MVC

90.为什么要使用 spring？

spring 提供 ioc 技术，容器会帮你管理依赖的对象，从而不需要自己创建和管理依赖对象了，更轻松的实现了程序的解耦。
spring 提供了面向切片编程，这样可以更方便的处理某一类的问题。
spring 提供了事务支持，使得事务操作变的更加方便。
更方便的框架集成，spring 可以很方便的集成其他框架，比如 MyBatis、hibernate 等。

【  参考：**** https://blog.csdn.net/weixin_44175121/article/details/90297426
spring( Spring Framework)：
     简单定义它为一个轻量级的控制反转（IoC）和面向切面（AOP）的容器，Java 开发框架。

     首先，Spring是一个生态体系（也可以说是技术体系），是集大成者，它包含了Spring Framework、Spring Boot、Spring Cloud等（还包括Spring Cloud data flow、spring data、spring integration、spring batch、spring security、spring hateoas），可以参考链接：https://spring.io/projects 。
     其中，Spring Framework是整个spring生态的基石 ，它可是硬生生的消灭了Java官方主推的企业级开发标准EJB，从而实现一统天下。Spring官方对Spring Framework简短描述：为依赖注入、事务管理、WEB应用、数据访问等提供了核心的支持。Spring Framework专注于企业级应用程序的“管道”，以便开发团队可以专注于应用程序的业务逻辑。

    提醒，千万不要把Spring和Spring Framework搞混淆了，很多文章都错误的定义了spring：spring是一个一站式的轻量级的java开发框架，核心是控制反转（IoC）和面向切面（AOP），针对于开发的WEB层(springMVC)、业务层(IoC)、持久层(jdbcTemplate)等都提供了多种配置解决方案。这是Spring Framework的定义，至于Spring，是整个生态。


spring boot ：   
    boot之前，WEB项目基于spring framework，项目目录一定要是标准的WEB-INF + classes + lib，而且大量的xml配置。如果说，以前搭建一个SSH架构的Web项目需要1个小时，那么现在应该10分钟就可以了。

    Spring boot 是 Spring 的一套快速配置脚手架，可以基于spring boot 快速开发单个微服务，Spring Boot，看名字就知道是Spring的引导，就是用于启动Spring的，使得Spring的学习和使用更方便。不仅适合替换原有的工程结构，更适合微服务开发。
   
    有一个starter概念，其实就是个jar包，想用SpringMVC了，想用jpa了，想用security了，就把对应的starter配置一下就好了。

    官方定义：即Spring Boot为快速启动且最小化配置的spring应用而设计。


spring cloud ： 
    Spring Cloud事实上是一整套基于Spring Boot 的微服务解决方案 。它为开发者提供了很多工具，用于快速构建分布式系统的一些通用模式，例如：配置管理、注册中心、服务发现、限流、网关、链路追踪等。

总结：
    笔者参与的项目也是基于Spring Cloud体系搭建的微服务。笔者认为Spring Cloud的 名气要大于它的作用 ，你俯瞰一下Spring Cloud的整个微服务生态，你会发现真的 不可替代 的组件又有几个？甚至它的一些组件，笔者压根不会考虑将它引入项目中，比如：

    Spring Cloud Sleuth：它是链路追踪解决方案，很明显，我只会考虑Skywalking、Pinpoint、CAT。
    Spring Cloud Config ：它是一个配置中心解决方案，无论是携程的apollo、还是百度的disconf，都远比它强大好用的多。
    另外，Spring Cloud netflix的核心组件hystrix已经停更，你可否还记得dubbo当年停更被喷成什么样？
    网关也并不是非Spring Cloud netflix下的zuul不可。非Spring Cloud生态下还有优秀的kong、 Traefik、soul都是不错的选择。
    Spring Cloud的一些方案给我的感觉更像一个 半成品 。

    综上，笔者得出的结论是： Spring Boot是大势所趋 ，而且它就像当年Spring Framework干掉EJB一样，干掉WEB容器+WAR的开发模式，统一现在的Java企业级应用开发标准。至于Spring Cloud？请 谨慎 选择每一个引入项目的组件，毕竟它的每一个微服务组件都面对很多优秀的开源可替代方案。】



91.解释一下什么是 aop？

【apo即切面编程】
   aop 是面向切面编程，通过预编译的方式 和 运行期动态代理 实现程序功能的统一维护 的一种技术。
   简单来说，就是 统一处理某一 "切面(类)" 的问题的编程思想，比如 统一处理日志、异常等。
【实践： 类加 @Aspect 、
	声明方法：   @Pointcut (需要加强的一些方法) 、 @Before 进入类前执行，  @AfterReturning、   
		    @AfterThrowing(后置异常通知)、@After (后置最终通知，final增强，不管是抛出异常或者正常退出都会执行)、 
		    @Around(环绕通知,环绕增强，相当于MethodInterceptor) 】


92.解释一下什么是 ioc？

【ioc 即控制反转，是spring 的核心，对于spring框架来说，就是由spring来负责控制对象的生命周期 和 对象间的关系。

  简单来说，控制 指的是当前对象 对内部成员的控制权；控制反转就是，这种控制不由当前对象管理了，由其他(类，第三方容器)来管理。 】
  Ioc — Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

   ●谁控制谁，控制什么：传统Java SE程序设计，我们直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象；而IoC是有专门一个容器来创建这些对象，即由Ioc容器来控制对象的创建；谁控制谁？当然是IoC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。
   参考：https://blog.csdn.net/Tritoy/article/details/81010595

【参考： https://blog.csdn.net/weixin_42447959/article/details/84591807
   从spring开始有了bean的概念，并且用Java Bean去管理bean。我们之前创建一个对象都是new出来的吧，而Spring可以将类作为bean配置在xml，类文件上用@Bean注解就可以不用new来创建对象了，Spring容器可以自己去管理对象的创建并注入和对象的销毁。再然后更方便了，也可以不用在xml中配置各种<bean>标签了，直接通过不同类型的注解如@Component，@Respository，@Service，@Controller等标注在不同类上表示不同类型的bean。如上就是IOC（控制反转）。】



93.spring 有哪些主要模块？【区分 spring frameWork有哪些主要模块？ 】

【 spring包括 spring frenwork（包括 spring core、spring dao、spring aop、spring context、spring web、spring MVC、spring ORM）、spring data、sspring boot、Spring Cloud、pring security、spring session等。

 首先，Spring是一个生态体系（也可以说是技术体系），是集大成者，它包含了Spring Framework、Spring Boot、Spring Cloud等（还包括Spring Cloud data flow、spring data、spring integration、spring batch、spring security、spring hateoas），可以参考链接：https://spring.io/projects 。
     其中，Spring Framework是整个spring生态的基石 ，它可是硬生生的消灭了Java官方主推的企业级开发标准EJB，从而实现一统天下。Spring官方对Spring Framework简短描述：为依赖注入、事务管理、WEB应用、数据访问等提供了核心的支持。Spring Framework专注于企业级应用程序的“管道”，以便开发团队可以专注于应用程序的业务逻辑。】


可参考：https://blog.csdn.net/meism5/article/details/90446643

spring core：框架的最基础部分，提供 ioc 和依赖注入特性；【提供了Ioc容器，对bean进行管理】。
spring aop：提供了面向切面编程实现，让你可以自定义拦截器、切点等。
spring dao：提供了JDBC的抽象层，它可以消除冗长的JDBC编码和解析数据库厂商特有的错误代码，还提供声明式事务管理方法。
spring context：构建于 core 封装包基础上的 context 封装包，提供了一种框架式的对象访问方法【提供上下文信息】。
spring web：提供了针对web开发的集成特性，如文件上传。
spring MVC：提供了 Web 应用的 Model-View-Controller 全功能/的实现。


94.spring 常用的注入方式有哪些？
1.setter注入； 2.注解注入； 3. 构造方法注入


95.spring 中的 bean 是线程安全的吗？

【结论： 不是线程安全的 】
（参考：https://www.cnblogs.com/myseries/p/11729800.html Spring 中的bean 是线程安全的吗？）

spring 中的 bean 默认是单例模式，spring 框架并没有对单例 bean 进行多线程的封装处理。
（@Scope(value = "prototype") // 加上@Scope注解，他有2个取值：单例-singleton 多实例-prototype(prototype:原型，每次创建一个新对象)）

实际上大部分时候 spring bean 无状态的（比如 dao 类），所有某种程度上来说 bean 也是安全的，
(也就是线程中的操作不会对Bean的成员执行查询以外的操作，那么这个单例Bean是线程安全的。)
但如果 bean 有状态的话（比如 view model 对象），那就要开发者自己去保证线程安全了，
最简单的就是改变 bean 的作用域，把“singleton”变更为“prototype”，这样请求 bean 相当于 new Bean()了，所以就可以保证线程安全了。

·有状态就是有数据存储功能。
·无状态就是不会保存数据。

【参考：https://blog.csdn.net/qq_29645505/article/details/88432001 Spring中的Bean是线程安全的吗？
 注： Spring容器本身并没有提供线程安全的策略，因此是否线程安全完全取决于Bean本身的特性。
 使用ThreadLocal的好处
 使得多线程场景下，多个线程对这个单例Bean的成员变量并不存在资源的竞争，因为ThreadLocal为每个线程保存线程私有的数据。】

【参考：https://www.cnblogs.com/goody9807/p/7472127.html Spring中Bean的五个作用域
曾经面试的时候有面试官问我spring的controller是单例还是多例，结果
我傻逼的回答当然是多例，要不然controller类中的 非静态变量 如何保证是线程安全的，这样想起似乎是对的，但是不知道（主要是我没看过
spring的源码，不知道真正的内在意图）为什么spring的controller是单例的。
  最佳实践：
	1、不要在controller中定义成员变量。
	2、万一必须要定义一个非静态成员变量时候，则通过注解@Scope("prototype")，将其设置为多例模式。】


96.spring 支持几种 bean 的作用域？

spring 支持 5 种作用域，如下：
	1.singleton：spring ioc 容器中只存在一个 bean 实例，bean 以单例模式存在，是系统默认值；
	2.prototype：每次从容器调用 bean 时都会创建一个新的示例，既每次 getBean()相当于执行 new Bean()操作；

    Web 环境下的作用域：
	3.request：每次 http 请求都会创建一个 bean； 【@Scope(value = "request")】
	4.session：同一个 http session 共享一个 bean 实例；
	5.global-session：用于 portlet 容器，因为每个 portlet 有单独的 session，globalsession 提供一个全局性的 http session。
    注意： 使用 prototype 作用域需要慎重的思考，因为频繁创建和销毁 bean 会带来很大的性能开销。


97.spring 自动装配 bean 有哪些方式？

参照下面：103.@Autowired 的作用是什么？
使用注解自动装配：
    如果不想在xml文件中使用autowire属性来启用自动装配，还可以直接在类定义中使用@Autowired或@Resource来装配bean。

【no：默认值，表示没有自动装配，应使用显式 bean 引用进行装配。
byName：它根据 bean 的名称注入对象依赖项。
byType：它根据类型注入对象依赖项。
构造函数：通过构造函数来注入依赖项，需要设置大量的参数。
autodetect：容器首先通过构造函数使用 autowire 装配，如果不能，则通过 byType 自动装配。】


98.spring 事务实现方式有哪些？

【注解 和 xml配置方式 --- 都是声明式】
声明式事务：声明式事务也有两种实现方式，注解方式(在类上添加@Transaction) 和 基于XML配置文件的方式。
编码式事务：提供编码的形式管理 和 维护事务。(即 手动编写代码的方式咯)


99.说一下 spring 的事务隔离？【区分 数据库的隔离级别，多一个默认数据库而已，其他一样】

【 事务四大特性：A原子性、C一致性、I隔离性、D持久性；
   (数据库)事务四大隔离级别：读未提交、读已提交、可重复读、序列化； spring有5种(多一种默认使用数据库的)
   数据库(事务)问题：脏读、幻读、不可重复读】

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致：

ISOLATION_DEFAULT：用底层数据库的设置隔离级别，数据库设置的是什么我就用什么；
ISOLATION_READUNCOMMITTED：未提交读，最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）；
ISOLATION_READCOMMITTED：读已提交，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别；(Oracle也是)
ISOLATION_REPEATABLEREAD：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；
ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。


脏读：事务A 读取到了 事务B 未提交的数据。 
幻读：这里经常会和不可重复读弄混淆，首先明白一点幻读并不是说“一个事务内进行两次相同操作居然得到了不一样的结果” 直接从例子入手：

时间	T1（事务1）						T2（事务2）
T1	select * from Table where id=1; 【此时读取为0】	
T2							insert into Table(id,name) values(1,“A”) ;
T1	insert into Table(id,name) values(1,“A”) ;
	检索id=1的数据，发现没有，插入。
	【此时插入失败，因为select * 不为0了】
T2							干扰 T1事务 的进行，在T1事务插入数据之前，先插入数据提交。
T1	此时T1插入数据，会报主键冲突。	

**对于T1来说第一次的查询并不能支持插入操作，但实际上是可以的，T1这里就是发生了幻读，数据就像凭空出现的一样。

​ 所以 mysql 的幻读并非什么读取两次返回结果集不同，而是事务在插入事先检测不存在的记录时，惊奇的发现这些数据已经存在了，之前的检测读获取到的数据如同鬼影一般。

​ 这里要灵活的理解读取的意思，第一次select是读取，第二次的 insert 其实也属于隐式的读取，只不过是在 mysql 的机制中读取的，插入数据也是要先读取一下有没有主键冲突才能决定是否执行插入。

幻读，是指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。

不可重复读：事务A中多次读取，其中第二次读取事务B 提交的数据，与第一次不一样。即指在一个事务内，多次读同一数据。



100.说一下 spring mvc 运行流程？ 【全局是dispatcherServlet 在控制流程】

【 接受前端请求 --> 匹配mapping，查询对应controller方法 --> 方法业务逻辑处理 -->
   返回ModelAndView(模型和视图) --> 查询对应视图解析器 --> 视图渲染返回给客户端。】


1. spring mvc 将请求发送到 dispatcherServlet。
2. dispatcherServlet 查询一个/多个 HandlerMapping， 找到处理请求的Controller。
3. dispatcherServlet 再次 把请求提交到对应的 Controller。
4. Controller 进行业务逻辑处理后，返回一个 模型和视图 ModelAndView。
5. dispatcherServlet 查询一个/多个 视图解析器viewResolver，找到 modelAndView对象指定 的视图对象。
6. 视图对象 负责渲染 返回给客户端。


101.spring mvc 有哪些组件？

【 前端接受器（前置控制器dispatcherServlet+映射控制器handlerMapping）、
   处理器controller、
   模型和视图(ModelAndView)、
   试图解析器(ViewResolver)。】

【 接受前端请求 --> 匹配mapping，查询对应controller方法 --> 方法业务逻辑处理 -->
   返回ModelAndView(模型和视图) --> 查询对应视图解析器 --> 视图渲染返回给客户端 】


102.@RequestMapping 的作用是什么？
  **将http请求 映射到 相应的类/方法上。


103.@Autowired 的作用是什么？

 【注入依赖对象，完成自动装配工作】
 @Autowired 它可以对 类成员变量、方法 及 构造函数 进行标注，完成自动装配的工作，通过@Autowired的使用来消除 set/get方法。



十一、Spring Boot/Spring Cloud

104.什么是 spring boot？

【 详细见90  spring boot是spring的一套快速搭建项目、配置脚手架；
   可以基于spring boot快速开发单个微服务；据官方定义，spring boot为快速启动且最小化配置的spring应用而设计的。】

Spring Boot是为spring服务的，用来简化 新spring应用的 初始搭建 以及 开发 的过程。



105.为什么要用 spring boot？

【官方定义 spring boot，是为 快速启动 且 最小化配置的 spring应用 而设计的。
 spring boot是一套快速搭建spring应用的脚手架。】

 配置简单、独立运行、自动装配、易上手、无代码生成 和 xml 配置、提升开发效率、 提供应用监控、


106.spring boot 核心配置文件是什么？
【bootstrap和application配置文件（.yml 或 .applicatin）】

bootstrap (. yml 或者 . properties)：
	boostrap 由父 ApplicationContext 加载的，比 applicaton 优先加载，且 boostrap 里面的属性不能被覆盖；
	application (. yml 或者 . properties)：用于 spring boot 项目的自动化配置。


107.spring boot 配置文件有哪几种类型？它们有什么区别？
【bootsrap和application；bootstrap加载顺序优先于application，且 boostrap 里面的属性不能被覆盖；】

配置文件有 . properties 格式和 . yml 格式，它们主要的区别是书法风格不同。



108.spring boot 有哪些方式可以实现热部署？
【devtools】

使用 devtools 启动热部署，添加 devtools 库，在配置文件中把 spring. devtools. restart. enabled 设置为 true；
使用 Intellij Idea 编辑器，勾上自动编译或手动重新编译。


109.jpa 和 hibernate 有什么区别？
【hibernate是基于jpa实现的】
jpa，全称是Java Persistence API，hibernate是基于jpa的具体实现。



110.什么是 spring cloud？

【 spring cloud ： 
      Spring Cloud事实上是一整套基于Spring Boot 的微服务解决方案 。它为开发者提供了很多工具，用于快速构建分布式系统的一些通用模式，例如：配置管理、注册中心、服务发现、限流、网关、链路追踪等。】

   Spring Cloud 是一系列框架的有序集合。它利用Spring Boot 的开发便利性 巧妙的简化了分布式系统基础设施 的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用spring boot的开发风格做到 一键启动和部署。



111.spring cloud 断路器的作用是什么？
【 一个微服务项目中，不外乎调用别人的接口 和 提供接口给别人调用；
   在messcat(或一般)项目中，controller就是提供给别人调用的，那么feign下的接口则是调用别人的接口；
   在feign下的接口中，用到了 @FeignClient + fallbackFactory,其用意就是feign调用别人的接口出现异常或失败时，进入fallbackFactory类返回指定的数据参数。
】

参考：https://blog.csdn.net/HD243608836/article/details/96010684 和 https://www.cnblogs.com/achengmu/p/9911808.html 
spring cloud: Hystrix（六）：feign的注解@FeignClient：fallbackFactory（类似于断容器）与fallback方法

SpringCloud + Feign + Hystrix使用FallbackFactory统一处理，查看服务调用异常或失败，进入熔断降级处理的原因

  fallbackFactory（类似于断容器）与fallback方法：
    feign的注解@FeignClient：fallbackFactory与fallback方法不能同时使用，这个两个方法其实都类似于Hystrix的功能，当网络不通时返回默认的配置数据.
    fallback方法的使用：
	1.在入口文件开启feign注解功能。  @EnableFeignClients
	2.写一个访问spring-boot-user服务的接口，同时在@FeignClient注解中使用fallback默认返回方法（断容器）
	fallback=HystrixClientFallback.class
	
	@FeignClient(name="spring-boot-user", fallback=HystrixClientFallback.class)
	public interface UserFeignClient {
	    // 两个坑：1. @GetMapping不支持   2. @PathVariable得设置value
	    @RequestMapping(value="/simple/{id}", method=RequestMethod.GET)
	    public User findById(@PathVariable("id") Long id);
	}

	3.1 方式1 - 写HystrixClientFallback类，并实现UserFeignClient类，当网络不通或者访问失败时，返回固定/默认内容
	@Component
	public class HystrixClientFallback  implements UserFeignClient{
	    @Override
	    public User findById(Long id) {
		// TODO Auto-generated method stub
		User user = new User();
		user.setId(0L);
		return user;
	    }
	}
   
    （fallbackFactory方法的使用）
	3.2 方式2 - 写HystrixClientFallbackFactory类，和HystrixClientWithFallbackFactory类
	HystrixClientWithFallbackFactory类继承UserFeignClient类（这步骤可以不写，然后在下面方法中直接换成 new UserFeignClient()）

	public interface HystrixClientWithFallbackFactory extends UserFeignClient { }

	HystrixClientFallbackFactory实现FallbackFactory类，并使用内部匿名方法类，继续UserFeignClient
		@Component
		public class HystrixClientFallbackFactory implements FallbackFactory<UserFeignClient> {
		    @Override
		    public UserFeignClient create(Throwable arg0) {
			// TODO Auto-generated method stub
			return new HystrixClientWithFallbackFactory() {   //这步可以换成直接new接口：new UserFeignClient()
			    @Override
			    public User findById(Long id) {
				// TODO Auto-generated method stub
				User user = new User();
				user.setId(-1L);
				return user;
			    } 
			};
		    }
		}

 】

当一个服务调用另一个服务由于网络原因或者自身原因 出现问题时,调用者就会等待被调用者的响应(调用方就会一直等待),当更多的服务请求到这些资源时,导致更多的请求等待,这样就会发生连锁效应（雪崩效应）,断路器就是解决这一问题。
 
在分布式架构中，断路器模式 的作用也是类似的，当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。


112.spring cloud 的核心组件有哪些？
【Eruke 服务的注册和发现、Rubbin 客户端调用服务、Feign 实现负载均衡的客户端调用、Zuul 网关管理（链路路由）】



十二、Hibernate

113.为什么要使用 hibernate？
·hibernate对JDBC进行了封装，大大简化了数据访问层的繁琐和重复性代码。
·hibernate是一个主流的持久化框架，是一个优秀的ORM（对象关系映射(Object Relational Mapping)）实现，它很大程度的简化了dao层编码工作。


114.什么是 ORM 框架？

ORM（Object Relation Mapping）对象关系映射，是把数据库中的关系数据映射成为程序中的对象。
使用 ORM 的优点：提高了开发效率降低了开发成本、开发更简单更对象化、可移植更强。


115.hibernate 中如何在控制台查看打印的 sql 语句？
 在 conf 配置文件中 把  hibernate.show_SQL 设置为 true 即可。
 但不建议开启，开启会降低程序运行效率。


116.hibernate 有几种查询方式？

三种：HQL 、 原生SQL  、 条件查询 Criteria。【criteria 标准的意思】



117.hibernate 实体类可以被定义为 final 吗？
【可以，但不建议】


118.在 hibernate 中使用 Integer 和 int 做映射有什么区别？【和 Integer 和 int 的区别一个意思】

【Integer 是对象类型，默认值为null； int是基本类型，默认值为0】
Integer 类型 为对象，它的值可以为null； 而 Int 属于基础数据类型，值不能为null。



119.hibernate 是如何工作的？【有两个文件：hibernate.cfg.xml 和 hibernate.hbm.xml】

 1. cfg --- Hibernate的核心配置文件
（反映了持久化类和数据库表的映射信息，Hibernate的配置文件主要用来 配置数据库连接 以及 Hibernate运行时所需要的各个属性的值）。
 2. hbm --- Hibernate加载映射。

1.读取cfg配置文件 -->
2.读取映射文件，创建sessionFactory -->   3.打开session(会话) -->    
4.创建事务 -->  5.进行持久化操作(执行业务代码) -->  6.提交事务 -->
7.关闭session -->   8.关闭sessionFactory


120.get()和 load()的区别？
 
 参考：https://www.cnblogs.com/cc11001100/p/6883790.html Hibernate中get()和load()的区别（good）
 **【 Hibernate中 根据Id单条查询 获取对象的方式有两种，分别是get()和load()，来看一下这两种方式的区别
      1.get() 立即查询加载；
      	当get()方法被调用的时候就会立即发出SQL语句：
		Hibernate:
		    select
			user0_.ID as ID1_1_0_,
			user0_.CREATETIME as CREATETI2_1_0_,
			user0_.UPDATETIME as UPDATETI3_1_0_,
			user0_.USERNAME as USERNAME4_1_0_,
			user0_.PASSWD as PASSWD5_1_0_
		    from
			USER user0_
		    where
			user0_.ID=?
	并且返回的对象也是实际的对象：
	使用get()和普通的单条查询并没有多大的区别。
      2.load() 延迟查询加载:
      	当调用load()方法的时候会 返回一个目标对象的代理对象，在这个代理对象中只存储了目标对象的ID值，
	只有当调用 除ID值以外 的属性值 的时候 才会 发出SQL查询的。
	
	比如：当我们尝试下面的代码时：
		User user=session.load(User.class, "1");
		System.out.println(user.getId()); //只访问ID属性
	因为我们只访问了ID属性，这个在代理对象中是已经存在的了，所以并不需要再去数据库中查询，因此并不会发出SQL查询语句。
	
	当使用到除ID以外的属性的时候，会发出SQL查询语句，比如尝试执行下面的代码：
		User user=session.load(User.class, "1");
		System.out.println(user.getUsername());
	会发现控制台打印了SQL查询语句：
		Hibernate:
		    select
			user0_.ID as ID1_1_0_,
			user0_.CREATETIME as CREATETI2_1_0_,
			user0_.UPDATETIME as UPDATETI3_1_0_,
			user0_.USERNAME as USERNAME4_1_0_,
			user0_.PASSWD as PASSWD5_1_0_
		    from
			USER user0_
		    where
			user0_.ID=?
	这个时候再看代理对象的话，会发现target已经被填充上了：	

	3. Exception
	   数据查询时，没有 OID 指定的对象，
	   get() 返回 null，继续获取属性会报空指针异常；
	   load() 返回一个代理对象。继续获取属性时ObjectNotFoundException异常：。
	即，查询结果为空(又继续获取属性)时：
		get()抛出NullPointerException
		load()抛出ObjectNotFoundException

	4. 缓存：
		get()和load()都会使用缓存，都是首先从一级缓存Session中查找，当找不到的时候再去二级缓存中查找，
		当查询不到的时候get()返回的是null，而load()则返回代理对象。
  】
  
1.数据查询时，没有 OID 指定的对象，get() 返回 null；load() 返回一个代理对象。
2.load()支持延迟加载；get() 不支持延迟加载。


121.说一下 hibernate 的缓存机制？
【一级缓存和二级缓存；一级缓存自动开启，二级缓存需要手动开启】
参考：https://www.cnblogs.com/lone5wolf/p/11065155.html Hibernate的缓存机制
一级缓存：
　　1、使用一级缓存的目的是 为了减少对数据库的访问次数，从而提升hibernate的执行效率；
  （当执行一次查询操作的时候，执行第二次查询操作，先检查缓存中是否有数据，如果有数据就不查询数据库，直接从缓存中获取数据）；
　　2、Hibernate中的一级缓存，也叫做session的缓存，它可以在session范围内减少数据库的访问次数，只在session范围内有效，session关闭，一级缓存失败；
　　3、一级缓存的特点，只在session范围有效，作用时间短，效果不是特别明显，在短时间内多次操作数据库，效果比较明显。
　　4、当调用session的save/saveOrUpdate/get/load/list/iterator方法的时候，都会把对象放入session缓存中；
　　5、session的缓存是由hibernate维护的，用户不能操作缓存内容；如果想操作缓存内容，必须通过hibernate提供的evict/clear方法操作
　　6、缓存相关的方法（在什么情况下使用上面方法呢？批量操作情况下使用，如Session.flush();先与数据库同步，Session.clear(); 再清空一级缓存内容）：
　　　　session.flush();让一级缓存与数据库同步；
　　　　session.evict();清空一级缓存中指定的对象；
　　　　session.clear();清空一级缓存中所有的对象；
    
【hibernate和mybatis的一级缓存第一次查询后会缓存，若查询后执行了insert、delete、update就会抹除一级缓存，此时再查询相同的条件仍会查询数据库。
 参考 https://blog.csdn.net/weixin_41847891/article/details/100623375 缓存篇】

hibernate 常用的缓存有一级缓存和二级缓存：

一级缓存：也叫 Session 缓存，只在 Session 作用范围内有效，不需要用户干涉，由 hibernate 自身维护，可以通过：evict(object)清除 object 的缓存；clear()清除一级缓存中的所有缓存；flush()刷出缓存；

二级缓存：应用级别的缓存，在所有 Session 中都有效，支持配置第三方的缓存，如：EhCache。


122.hibernate 对象有哪些状态？
 【 游离态 、持久态。   瞬时态 --  持久态 -- 游离态】
临时/瞬时状态：直接 new 出来的对象，该对象还没被持久化（没保存在数据库中），不受 Session 管理。
持久化状态：   当调用 Session 的 save/saveOrupdate/get/load/list 等方法的时候，对象就是持久化状态。
游离状态：     Session 关闭之后对象就是游离状态（ 代码执行session.getTransaction().commit();后）。


123.在 hibernate 中 getCurrentSession 和 openSession 的区别是什么？

参考：https://blog.csdn.net/chuck_kui/article/details/54845235 Hibernate 中的 openSession和getCurrentSession 方法的区别
在进行配置信息管理时，我们一般进行一下简单步骤：
Configuration cfg = new Configuration(); // 获得配置信息对象
SessionFactory sf = cfg.configure().buildSessionFactory(); //解析并建立Session工厂

1. Session session = sf.getCurrentSession(); // 获得Session
2. Session session = sf.openSession(); // 打开Session

对于上述的两个方法，有以下区别：
  1. openSession 从字面上可以看得出来，是打开一个新的session对象，而且每次使用都是打开一个新的session，假如连续使用多次，则获得的session不是同一个对象，并且使用完需要调用close方法关闭session。
  2. getCurrentSession ，从字面上可以看得出来，是获取当前上下文一个session对象，当第一次使用此方法时，会自动产生一个session对象，并且连续使用多次时，得到的session都是同一个对象，这就是与openSession的区别之一，简单而言，getCurrentSession 就是：如果有已经使用的，用旧的，如果没有，建新的。

**注意：在实际开发中，往往使用getCurrentSession多，因为一般是处理同一个事务（即是使用一个数据库的情况），所以在一般情况下比较少使用openSession或者说openSession是比较老旧的一套接口了；

getCurrentSession 会绑定当前线程，而 openSession 则不会。
getCurrentSession 事务是 Spring 控制的，并且不需要手动关闭，而 openSession 需要我们自己手动开启和提交事务。


124.hibernate 实体类必须要有无参构造函数吗？为什么？
   【必须】

    原因： Hibernate框架会调用这个默认构造方法来构造实例对象，即Class类的newInstance方法 ，这个方法就是通过调用默认构造方法来创建实例对象的 。

    当查询的时候返回的实体类是一个对象实例，是Hibernate动态通过反射生成的。反射的Class.forName(“className”).newInstance()需要对应的类提供一个无参构造方法，必须有个无参的构造方法将对象创建出来，单从Hibernate的角度讲 他是通过反射创建实体对象的 所以没有默认构造方法是不行的，另外Hibernate也可以通过有参的构造方法创建对象。
    
    提醒一点
如果你没有提供任何构造方法，虚拟机会自动提供默认构造方法（无参构造器），但是如果你提供了其他有参数的构造方法的话，虚拟机就不再为你提供默认构造方法，这时必须手动把无参构造器写在代码里，否则new Xxxx()是会报错的，所以默认的构造方法不是必须的，只在有多个构造方法时才是必须的，这里“必须”指的是“必须手动写出来”。

hibernate 中每个实体类必须提供一个无参构造函数，因为 hibernate 框架要使用 reflection api（反射），通过调用 Class类的newInstance() 来创建实体类的实例，如果没有无参的构造函数就会抛出异常。



十三、Mybatis
(**参考 :https://blog.csdn.net/weixin_41847891/article/details/100623375 
第十三模块 mybatis 中 #{}和 ${}的区别、mybatis 有几种分页方式、mybatis 逻辑分页和物理分页的区别、mybatis 是否支持延迟加载、延迟加载的原理、一级缓存和二级缓存)

125.mybatis 中 #{}和 ${}的区别是什么？

【 #{}，将传入的参数都作为字符串处理，自动加上双引号，经过预编译处理，安全行强，可以避免sql注入；
   ${}，将参数直接嵌入持久层的sql，未经过预编译处理，存在sql注入的隐患】

\#{}是预编译处理，${}是字符替换。 
在使用 #{}时，MyBatis 会将 SQL 中的 #{}替换成“?”，配合 PreparedStatement 的 set 方法赋值，这样可以有效的防止 SQL 注入，保证程序的运行安全。


(**参考 :https://blog.csdn.net/weixin_41847891/article/details/100623375 
第十三模块 mybatis 中 #{}和 ${}的区别、mybatis 有几种分页方式、mybatis 逻辑分页和物理分页的区别、mybatis 是否支持延迟加载、延迟加载的原理、一级缓存和二级缓存)

#{}是预编译处理，${}是字符串替换；
Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；
Mybatis在处理${}时，就是把sql中的${}替换成变量的值；
使用#{}可以有效的防止SQL注入，提高系统安全性。


126.mybatis 有几种分页方式？
1.物理分页
    物理分页就是数据库本身提供了分页方式，如MySQL的limit，oracle的rownum ，好处是效率高，不好的地方就是不同数据库有不同的搞法。

2.逻辑分页
    逻辑分页利用游标分页，好处是所有数据库都统一，坏处就是效率低。

    两种分页方式，分别是逻辑分页和物理分页；
    	物理分页 是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。
	逻辑分页 是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。

**参考：https://www.jianshu.com/p/2d6e8ff4d85a mybatis（1）—逻辑分页和物理分页
区别：见连接图。（推荐使用物理查询，减少内存占用）
1. Mybatis实现分页的方法
使用RowBounds对象进行逻辑（逻辑内存中）分页，它是针对ResultSet结果集执行的内存分页。
使用pageHelper插件进行物理分页（其实是依赖物理数据库实体）。

2. Mybatis使用pageHelper实现分页的原理
强烈推荐阅读——浅析pagehelper分页原理(https://blog.csdn.net/qq_21996541/article/details/79796117)

本质上两个知识点：
  1.将pageNum和pageSize封装为page对象，保存在ThreadLocal中，实现线程间数据隔离。
  2.Pagehelper实现了Mybatis的Interceptor接口，调用拦截StatementHandler（Sql语法的构建处理）方法，按照物理库的不同重构SQL实现分页。

插件拦截的对象：
	1.Executor：拦截执行器的方法（log记录）
	2.StatementHandler：sql语法构建处理
	3.ParameterHandler：拦截参数的处理
	4.ResultSetHandler：拦截结果集的处理


127.RowBounds 是一次性查询全部结果吗？为什么？
【PageHelper是物理查询 -  通过拦截器加数据库limit等进行分页查询；
  RowBounds是逻辑查询 - 数据量大的时候压力较大】

参考：https://blog.csdn.net/wangzhetianxia8/article/details/88783431 mybatis 分页 RowBounds和PageHelper性能评测
原理分析
 PageHelper：物理分页， 通过拦截器加 limit 语句进行分页
 RowBounds： 逻辑分页，数据量大的时候压力较大
总结：  1.使用RowBounds最大好处就是节省了在xml再拼装limit
       2.Mybatis的逻辑分页比较简单，简单来说就是取出所有满足条件的数据，然后舍弃掉前面offset条数据，然后再取剩下的数据的limit条

【**但是，RowBounds 表面是在“所有”数据中检索数据，其实并非是一次性查询出所有数据，
  因为 MyBatis 是对 jdbc 的封装，在 jdbc 驱动中有一个 Fetch Size 的配置，它规定了每次最多从数据库查询多少条数据，
  假如你要查询更多数据，它会在你执行 next()的时候，去查询更多的数据。
  就好比你去自动取款机取 10000 元，但取款机每次最多能取 2500 元，所以你要取 4 次才能把钱取完。
  只是对于 jdbc 来说，当你调用 next()的时候会自动帮你完成查询工作。这样做的好处可以有效的防止内存溢出。】


128.mybatis 逻辑分页和物理分页的区别是什么？（RowBounds和PageHelper）
	1.物理分页
		物理分页就是数据库本身提供了分页方式，如MySQL的limit，oracle的rownum ，好处是效率高，不好的地方就是不同数据库有不同的搞法。
	2.逻辑分页
		逻辑分页利用游标分页，好处是所有数据库都统一，坏处就是效率低。

	  两种分页方式，分别是逻辑分页和物理分页；
		物理分页 是从数据库查询指定条数的数据，弥补了一次性全部查出的所有数据的种种缺点，比如需要大量的内存，对数据库查询压力较大等问题。
		逻辑分页 是一次性查询很多数据，然后再在结果中检索分页的数据。这样做弊端是需要消耗大量的内存、有内存溢出的风险、对数据库压力较大。
	
	hibernate采用的是物理分页；

129.mybatis 是否支持延迟加载？延迟加载的原理是什么？
	mybatis支持延迟加载; 设置 lazyLoadingEnabled=true 即可。
	适用场景
	一对一，多对一   立即加载
	一对多，多对多   延迟加载

	延迟加载的原理 是调用的时候触发加载，而不是在初始化的时候就加载信息。
	比如调用 a. getB(). getName()，这个时候发现 a. getB() 的值为 null，此时会单独触发事先保存好的关联 B 对象的 SQL，先查询出来 B，
	然后再调用 a. setB(b)，而这时候再调用 a. getB(). getName() 就有值了，这就是延迟加载的基本原理。
	
	**当然了，不光是Mybatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。


130.说一下 mybatis 的一级缓存和二级缓存？
一级缓存：
  缓存目的 是为了减少对数据库的访问次数，提升数据库的执行效率。
  mybatis缓存机制包括：1.一级缓存 2.二级缓存
  
  【hibernate和mybatis的一级缓存第一次查询后会缓存，若查询后执行了insert、delete、update就会抹除一级缓存，此时再查询相同的条件仍会查询数据库。
 参考 https://blog.csdn.net/weixin_41847891/article/details/100623375 缓存篇】

 【为什么本地项目调用两次接口查询两次数据库，而没有使用一级缓存？
  参考：https://blog.csdn.net/ex_tang/article/details/83155031 mybatis中一级缓存
  
  **注意：调controller接口不论是否一级缓存都需要查数据库的，以上以下说的是再同一个service方法里面写了两个同样查询的代码，这样+@Transactional后有使用一级缓存，不会查两遍。（涉及两个session的话应该使用二级缓存。待验证）
  
   2、spring中结合 mybatis中，默认情况下，数据库处于自动提交模式，每一条sql语句处于一个单独的事务中，语句执行完毕时，如果执行成功则隐式提交事务。
   而mybatis的一级缓存在这种情况下是无效的，想要一级缓存起作用，则要开启事务：
   
   下面Service层中的代码同样对同一个数据查询了两次，这次开启了事务管理
	@Autowired
	private LsjmUserMapper lsjmUserMapper;

	@Override
	@Transactional   // 开启事务控制，当前，spring配置文件中得先配置好
	public LsjmUser getUser() {
		// 第一次查询
		LsjmUser user = lsjmUserMapper.getUserByName("300");
		System.out.println(user.toString());

		// 第二次查询
		LsjmUser user1 = lsjmUserMapper.getUserByName("300");
		System.out.println(user1.toString());
		return user;
	}
	
	前台页面触发Service后：控制台打印日志：
	可以看出来第一次查询时，构造了一个SqlSession对象，从数据库查询数据，然后将查询的结果存储到一级缓存SqlSession中，
	第二次查询时，直接Fetched SqlSession，而不是再重新建一个，此时就是从缓存中直接取数据了.
  】
  
二级缓存： 
  二级缓存是mapper级别的缓存，也就是多个sqlSession之间可以实现数据的共享。
  二级缓存默认是不开启，所以在使用二级缓存时需要在做以下配置：

一级缓存和二级缓存的示意图参考：https://blog.csdn.net/ex_tang/article/details/83155031 mybatis中一级缓存
（一级缓存是 sqlSession级别的缓存; 二级缓存是 mapper级别的缓存）

注意：
	1 当执行insert、delete、update时清除二级缓存，因为一级缓存都不存在了，何来二级缓存。（数据缓存到一级再到二级）
	2 当没有session.close()时，没有二级缓存，因为压根没有写入，
	3 ，没有开启相关配置文件，没有二级缓存，因为其默认是关闭的


131.mybatis 和 hibernate 的区别有哪些？
	灵活性：    MyBatis 更加灵活，自己可以写 SQL 语句，使用起来比较方便。
	可移植性：  MyBatis可移植性差。MyBatis 有很多自己写的 SQL，因为每个数据库的 SQL 可以不相同，所以可移植性比较差。
	学习和使用门槛：MyBatis 入门比较简单，使用门槛也更低。
	二级缓存：  MyBatis二级缓存不是很友好。hibernate 拥有更好的二级缓存，它的二级缓存可以自行更换为第三方的二级缓存。


132.mybatis 有哪些执行器（Executor）？


133.mybatis 分页插件的实现原理是什么？


134.mybatis 如何编写一个自定义插件？



十四、RabbitMQ

135.rabbitmq 的使用场景有哪些？

136.rabbitmq 有哪些重要的角色？

137.rabbitmq 有哪些重要的组件？

138.rabbitmq 中 vhost 的作用是什么？

139.rabbitmq 的消息是怎么发送的？

140.rabbitmq 怎么保证消息的稳定性？

141.rabbitmq 怎么避免消息丢失？

142.要保证消息持久化成功的条件有哪些？

143.rabbitmq 持久化有什么缺点？

144.rabbitmq 有几种广播类型？

145.rabbitmq 怎么实现延迟消息队列？

146.rabbitmq 集群有什么用？

147.rabbitmq 节点的类型有哪些？

148.rabbitmq 集群搭建需要注意哪些问题？

149.rabbitmq 每个节点是其他节点的完整拷贝吗？为什么？

150.rabbitmq 集群中唯一一个磁盘节点崩溃了会发生什么情况？

151.rabbitmq 对集群节点停止顺序有要求吗？



十五、Kafka

152.kafka 可以脱离 zookeeper 单独使用吗？为什么？

不可以，因为kafka需要依赖于zookeeper来管理和协调kafka的节点服务器。【所以kafka有内置的zookeeper的】

**Zookeeper作用：管理broker、consumer。创建Broker后，向zookeeper注册新的broker信息，实现在服务器正常运行下的水平拓展。Topic的注册，zookeeper会维护topic与broker的关系，通/brokers/topics/topic.name节点来记录。


153.kafka 有几种数据保留的策略？
*kafka 有两种数据保存策略：按照过期时间保留和按照存储的消息大小保留。


154.kafka 同时设置了 7 天和 10G 清除数据，到第五天的时候消息达到了 10G，这个时候 kafka 将如何处理？
*这个时候 kafka 会执行数据清除工作，时间和大小不论那个满足条件，都会清空数据。


155.什么情况会导致 kafka 运行变慢？
·cpu 性能瓶颈
·磁盘读写瓶颈
·网络瓶颈

156.使用 kafka 集群需要注意什么？
·集群的数量不是越多越好，最好不要超过 7 个，因为节点越多，消息复制需要的时间就越长，整个群组的吞吐量就越低。
·集群数量最好是单数，因为超过一半故障集群就不能用了，设置为单数容错率更高。


十六、Zookeeper

157.zookeeper 是什么？

158.zookeeper 都有哪些功能？

159.zookeeper 有几种部署模式？

160.zookeeper 怎么保证主从节点的状态同步？

161.集群中为什么要有主节点？

162.集群中有 3 台服务器，其中一个节点宕机，这个时候 zookeeper 还可以使用吗？

163.说一下 zookeeper 的通知机制？



十七、MySql

164.数据库的三范式是什么？【数据库脏读、事务的四大特性、(数据库)四大隔离级别、三大范式】

参考：https://blog.csdn.net/qq_34569497/article/details/79064208?utm_source=distribute.pc_relevant.none-task

在实际开发中最为常见的设计范式有三个：

1．第一范式(确保每列保持原子性)
	第一范式是最基本的范式。如果数据库表中的所有字段值都是不可分解的原子值，就说明该数据库表满足了第一范式。

	第一范式的合理遵循需要根据系统的实际需求来定。比如某些数据库系统中需要用到“地址”这个属性，本来直接将“地址”属性设计成一个数据库表的字段就行。但是如果系统经常会访问“地址”属性中的“城市”部分，那么就非要将“地址”这个属性重新拆分为省份、城市、详细地址等多个部分进行存储，这样在对地址中某一部分操作的时候将非常方便。这样设计才算满足了数据库的第一范式，如下表所示。

2．第二范式(确保表中的每列都和主键相关)
	第二范式在第一范式的基础之上更进一层。第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。

比如 要设计一个订单信息表，因为订单中可能会有多种商品，所以要将订单编号和商品编号作为数据库表的联合主键，如下表所示。

	（这个 违反了第二范式）。这样就产生一个问题：这个表中是以订单编号和商品编号作为联合主键。这样在该表中商品名称、单位、商品价格等信息不与该表的主键相关，而仅仅是与商品编号相关。所以在这里违反了第二范式的设计原则。

	正确如下： 而如果把这个订单信息表进行拆分，把商品信息分离到另一个表中，把订单项目表也分离到另一个表中，就非常完美了。如下所示。

	这样设计，在很大程度上减小了数据库的冗余。如果要获取订单的商品信息，使用商品编号到商品信息表中查询即可。

3．第三范式(确保每列都和主键列直接相关,而不是间接相关)
	第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。

比如在设计一个订单数据表的时候，可以将客户编号作为一个外键和订单表建立相应的关系。而不可以在订单表中添加关于客户其它信息（比如姓名、所属公司等）的字段。如下面这两个表所示的设计就是一个满足第三范式的数据库表。

这样在查询订单信息的时候，就可以使用客户编号来引用客户信息表中的记录，也不必在订单信息表中多次输入客户信息的内容，减小了数据冗余。



165.一张自增表里面总共有 7 条数据，删除了最后 2 条数据，重启 mysql 数据库，又插入了一条数据，此时 id 是几？

	id	
	1	
	....	
	6	
	7	X
	8	X
	?	
【看表的类型 / 数据引擎，MYSQL 表类型（存储引擎） 有：MyIASM (一直连续）和 InnoDB (删除会出现 不连续) 不重启的情况 】

 (1)InnoDB 表只会把自增主键的最大 id 记录在内存中，所以重启之后会导致最大 id 丢失。
 (2)如果是InnoDB 类型/引擎，Id会不重复的继续，新增过的Id会被保存在缓存中，insert获取+1，所以不重启缓存还在的话正常，重启缓存清空则从表中取最大缓存。  MyIASM 也是记录当前Id，不过是保存在文件中，重启不会丢失；

所以，InnoDB 是 id = 7 重启了，最大的缓存清空了，取条数+1；
      MyISAM 是 id = 9

一般情况下，我们创建的表的类型是InnoDB，如果新增一条记录（不重启mysql的情况下），这条记录的id是8；
	   但是如果重启（文中提到的）MySQL的话，这条记录的ID是6。
	   因为InnoDB表只把自增主键的最大ID记录到内存中，所以重启数据库或者对表OPTIMIZE操作，都会使最大ID丢失。

	但是，如果我们使用表的类型是MylSAM，那么这条记录的ID就是8。
	因为MylSAM表会把自增主键的最大ID记录到数据文件里面，重启MYSQL后，自增主键的最大ID也不会丢失。

** 注：如果在这7条记录里面删除的是中间的几个记录（比如删除的是3,4两条记录），重启MySQL数据库后，insert一条记录后，ID都是8。因为内存或者数据库文件存储都是自增主键最大ID

ing


166.如何获取当前数据库版本？

【show version 不对；】
 MySQL：select version()
 Oracle： select * from v$version
 SQL Server：select @@version


167.说一下 ACID 是什么？【ACID是事务的四大特性/属性；区分事务的隔离级别；】

（** 重点理解：程序是获取数据库连接，通过这个连接使用数据库的事务）

【  A 为原子性（Atomicity），一个事务中的所有操作，要么全部完成，要么都不完成，不会在中间某个环节结束 。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务没被执行过。 即，事务不可分割，不可约简。 
    即，事务是一个不可分割的单位。
    C 为一致性 （Consistency），事务前后，数据库的完整性没有被破坏，表示写入资料必须完全符合所有的预设约束、触发器，级联回滚等。
    即，事务结束后系统的状态是一致的。
    I 为独立/隔离性（Isolation），事务之间互不干扰，并发执行的事务之间无法看到彼此的中间状态。
     数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。
     事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
    D 为持久性（Durability），事务执行完后，对数据库的修改的影响是永久的，即便系统故障也不会丢失。 】


168.char 和 varchar 的区别是什么？

【char固定长度，一般用于密码的MD5值存储，varchar可以变化长度】
 	char(n)：固定长度类型，比如定义 char(10)，当输入''abc" 3个字符时，占用的空间还是10个字节，其他7个是空字节。

        char 优点：效率高；   缺点：占用空间；   适用场景：存储密码的MD5值，固定长度的使用char非常合适。
	varchar(n)：可变长度， 存储的值是每个值占用的字节 ，再加上一个记录其长度的字节  的空间大小(长度)。

**所以，从内存空间上 考虑varchar比较适合；  
       从效率上考虑 char比较适合，二者使用需要权衡。


169.float 和 double 的区别是什么？【区分 基本数据类型 和 MySQL】
	float 单精度浮点数 最多可以存储 8 位的十进制数，并在内存中占 4 字节。
	double 双精度浮点数 最可可以存储 16 位的十进制数，并在内存中占 8 字节。 

	float和double都是浮点型，Float单精度浮点数，Double双精度浮点数，而decimal是定点型

参考如下：https://www.cnblogs.com/liutianci/p/8443372.html

基本数据类型float和double的区别
	float : 单精度浮点数   double : 双精度浮点数
	
两者的主要区别如下：
　　01.在内存中占有的字节数不同
　　　　float 单精度浮点数 在机内存占4个字节（8位）
　　　　double 双精度浮点数 在机内存占8个字节 （16位）

　　02.有效数字位数不同
　　　　单精度浮点数 有效数字 8位
　　　　双精度浮点数 有效数字 16位

　　03.数值取值范围
　　　　单精度浮点数的表示范围：-3.40E+38~3.40E+38
　　　　双精度浮点数的表示范围：-1.79E+308~-1.79E+308

　　04.在程序中处理速度不同
　　　　一般来说，CPU处理单精度浮点数的速度比处理双精度浮点数快

    如果不声明，默认小数为double类型，所以如果要用float的话，必须进行强转
　　例如：float a=1.3; 会编译报错，正确的写法 float a = (float)1.3;或者float a = 1.3f;（f或F都可以不区分大小写）

注意：float是8位有效数字，第7位数字将会四舍五入

【**在 MySQL 中，有一条忠告，就是 「 不要使用浮点类型，不要使用浮点类型，不要使用浮点类型 」，如果真要使用，那么也请使用 DECIMAL类型。】



170.mysql 的内连接、左连接、右连接有什么区别？

【  内连接 两张表匹配的数据 都组合；
    左连接 是以左边为基础，匹配 右边的数据；
    右链接 是右边表为基础，匹配 左表的数据】

关键字：内连接 inner join、左连接 left join、右连接 right join

  含义： 内连接 两张表 匹配的关联数据 都组合 显示出来； 
	左连接是左边的表全部显示出来，右边的表显示出符合条件的数据；
	右链接是右边的表全部显示出来，左边的表显示符合条件的数据。


171.mysql 索引是怎么实现的？

【创建索引：create index index_name on  table_name(column_name,column_name) include(score)    索引是基于B+树实现的 】

  索引是满足 某种特定查找算法的 数据结构，而这些数据结构会以某种方式 指向数据，从而实现高效快速查找数据。

  具体来说MySQL中的索引，不同的数据引擎实现有所不同，但目前主流的数据库引擎 的索引都是B+树实现的，B+树的搜索效率可以达到二分法的性能，找到数据区域后就找到了完整的数据结构了，所有索引的性能也是更好的。



172.怎么验证 mysql 的索引是否满足需求？
 【explain】



173.说一下数据库的事务隔离？【重点理解：程序是获取数据库连接，通过这个连接使用数据库的事务】

  事务的隔离体现在多个并发事务时，即隔离性的体现。

  **区分并发事务可能出现的问题：脏读、不可重复读、幻读。

  事务4特性ACID，其中 I 为独立/隔离性（Isolation），事务之间互不干扰，并发执行的事务之间无法看到彼此的中间状态。

  数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。
  事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

参考：https://blog.csdn.net/qq_38238296/article/details/88363017

事务的隔离级别
	6.1.读未提交（read uncommitted）
    ​ 可以读取未提交的数据，会出现上面的脏读现象（此隔离级别不会使用，不做解释）

	6.2.读已提交（read committed）（Oracle & SQL Server 的默认级别）
    ​ 不能读取未提交的数据，这里就防止了脏读现象，但是避免不了不可重复读

	6.3.可重复读（repeatable-read）（MySQL/InnoDB下的默认级别）
    ​ 针对当前读，RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)，不存在幻读现象。

	6.4.串行化（serializable） (MySQL/InnoDB下不建议使用)
    ​ 从MVCC并发控制退化为基于锁的并发控制。不区别快照读与当前读，所有的读操作均为当前读，读加读锁 (S锁)，写加写锁 (X锁)。

    ​ Serializable隔离级别下，读写冲突，因此并发度急剧下降，在MySQL/InnoDB下不建议使用。


有关mysql加锁情况这篇文章有详细的解释：
【 补充：行锁 加锁粒度最小，开销大，可能会出现死锁。行级锁按照使用方式分为共享锁和排他锁。
参考：https://blog.csdn.net/qq_38238296/article/details/88362999
首先对mysql锁进行划分：
    按照锁的粒度划分：行锁、表锁、页锁
    按照锁的使用方式划分：共享锁、排它锁（悲观锁的一种实现）
    还有两种思想上的锁：悲观锁、乐观锁。
    InnoDB中有几种行级锁类型：Record Lock、Gap Lock、Next-key Lock
    Record Lock：在索引记录上加锁
    Gap Lock：间隙锁
    Next-key Lock：Record Lock+Gap Lock  】



174.说一下 mysql 常用的引擎？
【Mysql 常用数据表类型/数据引擎 有：MyIASM 和 InnoDB。】

	MyISAM 引擎：MyISAM是MySQL 的 默认引擎，ISAM即 Indexed Sequential Access Method（有索引的顺序访问方法）。
	它不提供事务的支持，也不支持行级锁和外键，但支持全文搜索。如果事务回滚将会造成不完全回滚，从而不具备原子性。
	优点：访问速度快，对事务的完整性没有要求 或者 以select、insert为主的应用基本上都可以使用这个引擎来创建。 因此当执行 insert、update语句时，需要锁定这个表，所以会导致效率降低。 
	和InnoDB不同的是，MyISAM引擎保存了表的行数，当进行select count(*)操作时，可以直接的读取已保存的行数值而不需要进行全表扫描。  所以，如果表的读操作远远多于写操作时，并且不需要事务的支持的，可以将 MyIASM 作为数据库引擎的首选。

	InnoDB 引擎：InnoDB引擎提供了对数据库ACID事务的支持，并且还提供了行级锁和外键的约束，它设计的目的就是为了处理大数据容量的数据库系统。MySQL运行时，InnoDB会在内存中建立缓冲池，用于缓冲数据和索引。 
	但是InnoDB不支持全文搜索，同时启动比较慢，不会保存表的行数，所以进行select count(*)的时，需要进行全表扫描。 由于锁的粒度小，写的操作不会锁定 全表，所以在并发高的情况下使用会提升效率。

**补充： DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。



175.说一下 mysql 的行锁和表锁？

【行锁锁住某一行数据；表锁锁定一整张表；
  mysql有两种引擎，InnoDB默认是行锁，也支持表锁，效率高; MyISAM只有表锁，效率低

 表锁：开销小，加锁快，不会出现死锁。锁的颗粒度大，发生锁冲突（死锁）的概率高，并发度最低（效率低）
 行锁：开销大，加锁慢，会出现死锁。锁力度小，发生锁冲突（死锁）的概率低，并发度最高（效率高）】



176.说一下乐观锁和悲观锁？

乐观锁：每次拿数据时不担心别人会修改数据，所以不会上锁，但是在提交更新时会判断一下此期间别人有没有去更新这个数据（和数据库的对比版本是否一致，一致则没修改）。
悲观锁：每次操作拿数据时都认为别人会修改数据，所以每次在拿数据时都会上锁，这样别人想拿这个数据就会被阻止，直到锁被释放。

【 由于乐观锁没有了 锁等待，提高了吞吐量，所以 乐观锁适合 多读少写 的场景。
   悲观锁由于 阻塞原因，所以导致吞吐量不高。悲观锁更适用于 多写少读 的情况】

    数据库的乐观锁需要自己实现，在表里面添加一个version字段，每次修改成功值加1，这样每次修改的时候先对比一下，自己拥有的version和数据库现在的version是否一致，如果不一致就不修改（一致就修改），这样就实现了乐观锁。

【意思说 现在数据库的锁都是悲观锁？是的，平时的开发并没用到锁，使用数据库的锁要 "xxx for update"】

悲观锁的实现方式：
	通过数据库锁机制实现，即对查询语句添加for update关键字。如： select * from table where id = 1 for update;
	当一个请求A开启事务并执行此sql同时未提交事务时，另一个线程B发起请求，此时B将阻塞在加了锁的查询语句上，直到A请求的事务提交或者回滚，B才会继续执行，保证了访问的隔离性。

【**讲解+乐观锁使用案例 https://blog.csdn.net/qidasheng2012/article/details/83007103

乐观锁优点： 由于在检测数据冲突时并不依赖数据库本身的锁机制，不会影响请求的性能，当产生并发且并发量较小的时候只有少部分请求会失败。
     缺点：一需要对表的设计增加额外的字段，增加了数据库的冗余，另外，当应用并发量高的时候，version值在频繁变化，则会导致大量请求失败，影响系统的可用性。我们通过上述sql语句还可以看到，数据库锁都是作用于同一行数据记录上，这就导致一个明显的缺点，在一些特殊场景，如大促、秒杀等活动开展的时候，大量的请求同时请求同一条记录的行锁，会对数据库产生很大的写压力。

悲观锁优点：悲观锁可以严格保证数据访问的安全。
     缺点：每次请求都会额外产生加锁的开销且未获取到锁的请求将会阻塞等待锁的获取，在高并发环境下，容易造成大量请求阻塞，影响系统可用性。另外，悲观锁使用不当还可能产生死锁的情况。
】


参考：https://blog.csdn.net/qq_38238296/article/details/88362999
在mysql/InnoDB中使用悲观锁：
​ 首先我们得关闭mysql中的autocommit属性，因为mysql默认使用自动提交模式，也就是说当我们进行一个sql操作的时候，mysql会将这个操作当做一个事务并且自动提交这个操作。



177.mysql 问题排查都有哪些手段？



178.如何做 mysql 的性能优化？
为索引字段创建索引；避免使用 select *，列出需要查询的字段；垂直分割分表（分库分表）；选择正确的存储引擎。




十八、Redis

179.redis 是什么？都有哪些使用场景？
【非关系型数据库，一般用于缓存】

Redis 是一个用C语言开发的高速缓存数据库。
Redis使用场景：
   -- 记录帖子的点赞数、点击数、评论数；
   -- 缓存近期热帖；
   -- 缓存文章详情信息；
   -- 记录用户会话信息；


180.redis 有哪些功能？
【数据缓存功能；分布式锁功能；支持数据持久化；支持事务；支持消息队列】


181.redis 和 memecache 有什么区别？


182.redis 为什么是单线程的？

因为 cpu 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存或者网络带宽。
既然单线程 容易实现，而且 cpu 又不会成为瓶颈，那就顺理成章地采用单线程的方案了。
关于 Redis 的性能，官方网站也有，普通笔记本轻松处理每秒几十万的请求。

而且单线程并不代表就慢 nginx 和 nodejs 也都是高性能单线程的代表。


183.什么是缓存穿透？怎么解决？

参考：https://baijiahao.baidu.com/s?id=1619572269435584821&wfr=spider&for=pc 实例解读什么是Redis缓存穿透、缓存雪崩和缓存击穿
  Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。
  其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。

  另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也都有比较流行的解决方案。
  本篇文章，并不是要更加完美的解决这三个问题，也不是要颠覆业界流行的解决方案。而是，从实际代码操作，来演示这三个问题现象。之所以要这么做，是因为，仅仅看这些问题的学术解释，脑袋里很难有一个很形象的概念，有了实际的代码演示，可以加深对这些问题的理解和认识。

  1.缓存穿透
	缓存穿透，是指查询一个数据库一定不存在的数据。正常的使用缓存流程大致是，数据查询先进行缓存查询，如果key不存在或者key已经过期，再对数据库进行查询，并把查询到的对象，放进缓存。如果数据库查询对象为空，则不放进缓存。
	想象一下这个情况，如果传入的参数为-1，会是怎么样？这个-1，就是一定不存在的对象。就会每次都去查询数据库，而每次查询都是空，每次又都不会进行缓存。假如有恶意攻击，就可以利用这个漏洞，对数据库造成压力，甚至压垮数据库。即便是采用UUID，也是很容易找到一个不存在的KEY，进行攻击。
	
	解决方案：最简单粗暴的方法如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），把这个空结果(null)也进行缓存，但它的过期时间会很短，最长不超过五分钟。（缓存中key不为null，但是value为null，纸样师为了避免为null每次还是查数据库）

  2.缓存雪崩
	缓存雪崩，是指在某一个时间段，缓存集中过期失效。（缓存全部失效，缓存为空，全部查询直接查数据库）

	产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。

	小编在做电商项目的时候，一般是采取不同分类商品，缓存不同周期。在同一分类中的商品，加上一个随机因子。这样能尽可能分散缓存过期时间，而且，热门类目的商品缓存时间长一些，冷门类目的商品缓存时间短一些，也能节省缓存服务的资源。
	
	其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。
	因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，那么那个时候数据库能顶住压力，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。

  3.缓存击穿
	缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。
	小编在做电商项目的时候，把这货就成为“爆款”。

	其实，大多数情况下这种爆款很难对数据库服务器造成压垮性的压力。达到这个级别的公司没有几家的。所以，务实主义的小编，对主打商品都是早早的做好了准备，让缓存永不过期。即便某些商品自己发酵成了爆款，也是直接设为永不过期就好了。

参考：https://blog.csdn.net/kongtiao5/article/details/82771694 缓存穿透、缓存击穿、缓存雪崩区别和解决方案
   缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。
   和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
 
 

184.redis 支持的数据类型有哪些？
redis支持的数据类型：String（字符串）、List（列表）、hash（hashmap字典）、Set（集合）、zSet（有序集合）。


185.redis 支持的 java 客户端都有哪些？
	支持的 Java 客户端有 Redisson、jedis、lettuce 等。


186.jedis 和 redisson 有哪些区别？

187.怎么保证缓存和数据库数据的一致性？

188.redis 持久化有几种方式？



189.redis 怎么实现分布式锁？

@Autowired private RedisTemplate redisTemplate; 
RedisLock lock = new RedisLock(redisTemplate, "ms-ha-lianjia", 10000, 20000); 
try{ 
  if(lock.lock()) {
     //发送每周日活数据邮件 
     mailTimingTaskService.sendDailyLiveData(startDate,endDate); 
  } 
  log.info("Sending daily live data mail end"); 
}catch (Exception e) { e.printStackTrace(); }
finally { lock.unlock(); }

redis分布式锁 其实就是 在系统中占一个“坑”，其他程序也要占“坑”的时候，占用成功的就可以继续执行，失败了的就只能放弃 或者 稍后重试。
占坑 一般 使用 setnx() 指令，只允许被一个程序占有，使用完调用del 释放锁。

【https://baijiahao.baidu.com/s?id=1623086259657780069&wfr=spider&for=pc
布式锁的实现方案中，都是针对单节点Redis而言的，然而在生产环境中，我们使用的通常是Redis集群，并且每个主节点还会有从节点。
由于Redis的 主从复制 是 异步的，因此上述方案在Redis集群的环境下也是有问题的。】继续TODO研究



190.redis 分布式锁有什么缺陷？

191.redis 如何做内存优化？

192.redis 淘汰策略有哪些？

193.redis 常见的性能问题有哪些？该如何解决？




十九、JVM

194.说一下 jvm 的主要组成部分？及其作用？

	1.类加载器（ClassLoader）
	2.运行时数据区（Runtime Data Area）
	3.执行引擎（Execution Engine）
	4.本地库接口（Native Interface）

	组件的作用： 首先通过类加载器（ClassLoader）会把 Java 代码转换成字节码，
		运行时数据区（Runtime Data Area）再把字节码加载到内存中，而字节码文件只是 JVM 的一套指令集规范，并不能直接交个底层操作系统去执行，因此需要特定的命令 解析器 执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由 CPU 去执行，
		而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。

参考：https://www.cnblogs.com/rong0912/p/12047674.html jvm主要组成部分及其作用
jvm主要组成部分及其作用
    class Files(类文件)
    --> Class Loader(类加载器) 
    --> Runtime Data Area(运行时数据区)【
	  (1)Stack(虚拟机 栈)：虚拟机栈中执行每个方法的时候，都会创建一个栈桢用于存储局部变量表，操作数栈，动态链接，方法出口等信息。
	  (2)Heap(Java 堆)：堆是java对象的存储区域。
	    	任何用new字段分配的java对象实例和数组，都被分配在堆上，
	    	java堆可用-Xms和-Xmx进行内存控制，jdk1.7以后，运行时常量池从方法区移到了堆上。
		Java堆内存结构又分： 新生代和老年代。
	  (3)Method Area(方法 区)：用于存储已被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。
	  (4)PC Register(程序计数器)、
	  (5)Native Method Stack(本地方法栈)】
    --> Execution Engine(执行引擎) 
    --> Native Interface(本地库接口) --> Native Libraies(本地库接口)
 线程私有的：  程序计数器    虚拟机栈     本地方法栈

	1.类加载器（Class Loader）：加载类文件到内存。Class loader只管加载，只要符合文件结构就加载，至于能否运行，它不负责，那是有Exectution Engine 负责的。
	2.执行引擎（Execution Engine）：也叫解释器，负责解释命令，交由操作系统执行。
	3.本地库接口（Native Interface）：本地接口的作用是融合不同的语言为java所用
	4.运行时数据区（Runtime Data Area）：

 		（1）堆。堆是java对象的存储区域，任何用new字段分配的java对象实例和数组，都被分配在堆上，java堆可用-Xms和-Xmx进行内存控制，jdk1.7以后，运行时常量池从方法区移到了堆上。
		（2）方法区：用于存储已被虚拟机加载的类信息，常量，静态变量，即时编译器编译后的代码等数据。
		  误区：方法区不等于永生代
		  很多人原因把方法区称作“永久代”（Permanent Generation），本质上两者并不等价，只是HotSpot虚拟机垃圾回收器团队把GC分代收集扩展到了方法区，或者说是用来永久代来实现方法区而已，这样能省去专门为方法区编写内存管理的代码，但是在Jdk8也移除了“永久代”，使用Native Memory来实现方法区。
		（3）虚拟机 栈：虚拟机栈中执行每个方法的时候，都会创建一个栈桢用于存储局部变量表，操作数栈，动态链接，方法出口等信息。
		（4）本地方法栈：与虚拟机发挥的作用相似，相比于虚拟机栈为Java方法服务，本地方法栈为虚拟机使用的Native方法服务，执行每个本地方法的时候，都会创建一个栈帧用于存储局部变量表，操作数栈，动态链接，方法出口等信息。
		（5）程序计数器。指示Java虚拟机下一条需要执行的字节码指令。


195.说一下 jvm 运行时数据区？
	【JVM主要组成部分有：类加载器、执行(解析)引擎、运行时数据区、本地库接口
	其中，运行时数据区Runtime Data Area中包括了 
		Java堆(Heap)：Java对象的储存区域。是JVM中内存最大的一块，保存new出来的对象、
		Java栈(Stack)：执行方法时，会创建一个栈帧来储存局部变量、
		方法区、
		虚拟机栈、
		程序计数器】

    不同虚拟机的运行时数据区可能略微有所不同，但都会遵从 Java 虚拟机规范， Java 虚拟机规范规定的区域分为以下 5 个部分：
	1.程序计数器（Program Counter Register）：当前线程所执行的字节码的行号指示器，字节码解析器的工作是通过改变这个计数器的值，来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能，都需要依赖这个计数器来完成；
	2.Java 虚拟机栈（Java Virtual Machine Stacks）：用于存储局部变量表、操作数栈、动态链接、方法出口等信息；
	3.本地方法栈（Native Method Stack）：与虚拟机栈的作用是一样的，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的；
	4.Java 堆（Java Heap）：Java 虚拟机中内存最大的一块，是被所有线程共享的，几乎所有的对象实例都在这里分配内存；
	5.方法区（Methed Area）：用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译后的代码等数据。


196.说一下堆栈的区别？
【 对象 保存在 堆内存中；
   堆是java对象的存储区域，任何用new字段分配的java对象实例和数组，都被分配在堆上，
   java堆可用-Xms和-Xmx进行内存控制，jdk1.7以后，运行时常量池从方法区移到了堆上。
 
   栈 用于存储 局部变量表、操作数栈、动态链接、方法出口等信息；
   虚拟机栈中执行每个方法的时候，都会创建一个栈桢用于存储局部变量表，操作数栈，动态链接，方法出口等信息。
 】
	功能方面：堆是用来存放对象的，栈是用来执行程序的。
	共享性：堆是线程共享的，栈是线程私有的。
	空间大小：堆大小远远大于栈。（堆是JVM中内存最大的一块）


197.队列和栈是什么？有什么区别？
【队列先进先出，栈后进先出】就是栈和队列，栈是先进后出，队列先进先出。（数据结构学过）

参考：https://blog.csdn.net/u010955843/article/details/21294901 链表，队列和栈的区别（详细讲了之间的介绍和优劣势）

1.栈和队列：
	队列（Queue）：是限定只能在表的一端进行插入和另一端删除操作的线性表
	栈（Stack）：是限定之能在表的一端进行插入和删除操作的线性表
2.队列和栈的规则:
	队列：先进先出
	栈：  先进后出
3.队列和栈的遍历数据速度:
	队列：基于地址指针进行遍历，而且可以从头部或者尾部进行遍历，但不能同时遍历，无需开辟空间，因为在遍历的过程中不影响数据结构，所以遍历速度要快
	栈：只能从顶部取数据，也就是说最先进入栈底的，需要遍历整个栈才能取出来，遍历数据时需要微数据开辟临时空间，保持数据在遍历前的一致性。
	

队列和栈都是被用来 预存储数据的。
队列允许先进先出检索元素，但也有例外的情况，Deque 接口允许从两端检索元素。
栈和队列很相似，但它运行对元素进行后进先出进行检索。


198.什么是双亲委派模型？


199.说一下类加载的执行过程？
【类文件 由类加载器CLass Loader加载，实现把Java代码转换为字节码，然后由运行时数据区Runtime Data Area把字节码加载到内存中，这时需要 执行(解析)引擎 把字节码翻译成 底层系统指令，再交由CPU执行，而这个过程需要调用其他语言的本地库接口 来实现整个程序的功能】

类装载分为以下 5 个步骤：
	1.加载：根据查找路径找到相应的 class 文件然后导入；
	2.链接
		(1) 检查：检查加载的 class 文件的正确性；
		(2) 准备：给类中的静态变量分配内存空间；
		(3) 解析：虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址；
	3.初始化：对静态变量和静态代码块执行初始化工作。

参考：https://www.cnblogs.com/jxxblogs/p/12208112.html 类加载的执行过程
1.加载：根据查找路径找到相应的 class 文件然后导入; 是把class字节码文件从各个来源通过类加载器装载入内存中。
2.检查/验证：检查加载的 class 文件的正确性; 主要是为了保证加载进来的字节流 符合虚拟机规范 ，不会造成安全错误。
3.准备：给类中的 静态变量 分配内存空间; 主要是为类变量（注意，不是实例变量）分配内存，并且赋予初值。
4.解析：虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为一个标示，而在直接引用直接指向内存中的地址; 
	     将常量池内的符号引用替换为直接引用的过程。
5.初始化：对静态变量和静态代码块执行初始化工作。 这个阶段主要是对类变量初始化，是执行类构造器的过程。


200.怎么判断对象是否可以被回收？

参考：https://www.jianshu.com/p/b1b6eecb6224 GC:如何确定一个对象是否可以被回收

众所周知，java引入了垃圾回收机制（GC），将程序员比较头疼的内存管理问题轻松加愉快的解决了。
我们不用再关心程序什么时候去释放内存，什么时候去回收内存了，那么GC是怎么判断程序中的对象是不是一个垃圾，要不要回收的呢？

引用的概念:
    简单的理解，如果一个变量的类型是 类类型，而非基本类型，那么该变量又叫做引用，类类型创建的对象都可以称为引用。

1.引用计数 判断:
    上面讲过了什么是引用，其实 java在GC时也会去看这个对象有没有任何引用与之关联，如果存在引用关系则表示这个对象还有用，不能被回收，
    如果不存在引用则可以基本定性为可被回收的对象了。使用此方式效率确实很高，
    但是有个致命的缺点，无法解决循环引用的问，例如下面这段代码：

	public class Main {
	    public static void main(String[] args) {
		MyObject object1 = new MyObject();
		MyObject object2 = new MyObject();
		object1.object = object2;
		object2.object = object1;
		object1 = null;
		object2 = null;
	    }
	}

	class MyObject{
	    public Object object = null;
	}
    可以看到将object1和object2赋值为null，也就是说object1和object2指向的对象已经不可能再被访问，但是由于它们互相引用对方，导致它们的引用计数都不为0，那么垃圾收集器就永远不会回收它们。因此在Java中并没有采用这种方式。

2.正向可达 性 判断：
    为了解决循环引用的问题，java中采取了正向可达的方式，
    主要是通过Roots对象作为起点进行搜索，搜索走过的路径称为“引用链”(检查对象通过=赋值的)，当一个对象到 Roots 没有任何的引用链相连时时，证明此对象不可用，当然被判定为不可达的对象 不一定 就会成为可回收对象。
    被判定为不可达的对象要成为可回收对象必须 至少经历两次标记过程，如果在这两次标记过程中仍然没有逃脱成为可回收对象的可能性，则基本上就真的成为可回收对象了，能否被回收其实主要还是要看finalize()方法有没有与引用链上的对象关联，如果在finalize()方法中有关联则自救成功，改对象不可被回收，反之如果没有关联则成功被二次标记成功，就可以称为要被回收的垃圾了。
    Roots对象：
	面试常问的问题，Roots对象有哪些？
	1、引用栈帧中的本地变量表的所有对象；
	2、引用方法区中静态属性的所有对象；
	3、引用方法区中常量的所有对象；
	4、引用Native方法的所有对象。


一般有两种方法来判断：
    1.引用计数器：为每个对象创建一个引用计数，有 对象引用 时计数器 +1，引用被释放时计数 -1，当计数器为 0 时就可以被回收。
    	       它有一个缺点不能解决循环引用的问题；举例如上。
    2.可达性分析：从 GC Roots 开始向下搜索，搜索所走过的路径称为“引用链”。当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是可以被回收的。



201.java 中都有哪些引用类型？
【String、Long、Float、Integer、Double、Boolean】

参考：https://www.cnblogs.com/frankcui/p/12492973.html JVM - java 中都有哪些引用类型？
1.什么叫引用reference
	Object o = new Object();
	这个 o,我们可以称之为对象引用,而 new Object()我们可以称之为在内存 中产生了一个对象实例。
	当写下 o=null 时,只是表示 o 不再指向堆中 object 的对象实例,不代表这个对象实例不存在了。
	示意图见连接。

2.强软弱虚引用
	强引用（strongreference）就是指在程序代码之中普遍存在的,类似“Object obj=new Object()” 这类的引用,只要强引用还存在,垃圾收集器永远不会回收掉被引用的对象实例。

	软引用（softreference）是用来描述一些还有用但并非必需的对象。对于软引用关联着的对象, 在系统将要发生内存溢出异常之前,将会把这些对象实例列进回收范围之中进行 第二次回收。如果这次回收还没有足够的内存,才会抛出内存溢出异常。在 JDK 1.2 之后,提供了 SoftReference 类来实现软引用。

	弱引用（weakreference）也是 用来描述非必需对象的,但是它的强度比软引用 更弱一些,被弱 引用关联的对象实例只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时, 无论当前内存是否足够,都会回收掉只被弱引用关联的对象实例。在 JDK 1.2 之 后,提供了 WeakReference 类来实现弱引用。

	虚引用（phantomreference）也称为幽灵引用或者幻影引用,它是 最弱 的一种引用关系。一个对象 实例是否有虚引用的存在,完全不会对其生存时间构成影响,也无法通过虚引用 来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能在这个对象 实例被收集器回收时收到一个系统通知。在 JDK 1.2 之后,提供了 PhantomReference 类来实现虚引用。


JVM中的java引用类型：
	强引用：发生 gc 的时候不会被回收。
	软引用：有用但不是必须的对象，在发生内存溢出之前会被回收。
	弱引用：有用但不是必须的对象，在下一次GC时会被回收。
	虚引用（幽灵引用/幻影引用）：无法通过虚引用获得对象，用 PhantomReference 实现虚引用，虚引用的用途是在 gc 时返回一个通知。


202.说一下 jvm 有哪些垃圾回收算法？



203.说一下 jvm 有哪些垃圾回收器？
【gc】



204.详细介绍一下 CMS 垃圾回收器？

205.新生代垃圾回收器和老生代垃圾回收器都有哪些？有什么区别？

206.简述分代垃圾回收器是怎么工作的？


207.说一下 jvm 调优的工具？

参考：https://www.cnblogs.com/ityouknow/p/6437037.html jvm系列(七):jvm调优-工具篇
	jvm监控分析工具一般分为两类，一种是jdk自带的工具，一种是第三方的分析工具。
	jdk自带工具一般在jdk bin目录下面，以exe的形式直接点击就可以使用，其中包含分析工具已经很强大，几乎涉及了方方面面，
	  但是我们最常使用的只有两款：jconsole.exe和jvisualvm.exe；
	  jconsole：连接进去之后，就可以看到jconsole概览图和主要的功能：概述、内存、线程、类、VM、MBeans
	  jvisualvm：VisualVM 是一个工具，它提供了一个可视界面，用于查看 Java 虚拟机 (JVM) 上运行的基于 Java 技术的应用程序（Java 应用程序）的详细信息。
	  VisualVM 是javajdk自带的最牛逼的调优工具了吧，也是我平时使用最多调优工具，几乎涉及了jvm调优的方方面面。
	第三方的分析工具有很多，各自的侧重点不同，比较有代表性的：MAT(Memory Analyzer Tool)、GChisto等。

连接远程需要在服务器终端设置环境变量，主要是便于每次启动项目时使用；然后配置用户和密码文件。后可以进行GC操作、检查死锁。
【参考https://blog.csdn.net/box_clf/article/details/88344631  基于Springboot项目使用jconsole远程监控JVM】

	JDK 自带了很多监控工具，都位于 JDK 的 bin 目录下，其中最常用的是 jconsole 和 jvisualvm 这两款视图监控工具。
 	  jconsole：用于对 JVM 中的内存、线程和类等进行监控；
	  jvisualvm：JDK 自带的全能分析工具，可以分析：内存快照、线程快照、程序死锁、监控内存的变化、gc 变化等。


208.常用的 jvm 调优的参数都有哪些？
-Xms 初始化内存
-Xmx 最大内存

