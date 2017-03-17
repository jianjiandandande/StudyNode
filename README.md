# StudyNode
小型知识点总汇

### 1.[JAVA中的九种数据类型]
>java中包含了九种数据类型主要包括：boolean,byte,char,short,int,long, float,double,void.
*	boolean 的包裹类型为：Boolean，它的取值只有两种：true,false.
* void 包裹类型为 Void
*	byte 占一个字节，位数为8bit  包裹类型为：Byte，它的取值范围为-2^7～2^7 - 1.
*	char 占两个字节，位数为16bit 包裹类型为：Character，它的取值范围为：0～2^16 - 1
* short 占两个字节，位数为16bit 包裹类型为：Short,它的取值范围为：-2^15～2^15 - 1
* int 占四个字节，位数为32bit 包裹类型为：Integer,它的取值范围为：-2^31～2^31 - 1
* long 占八个字节，位数为64bit 包裹类型为 Long，它的取值范围为：-2^63～2^63- 1
* float 占四个字节，位数为32bit 包裹类型为 Float，它的取值范围为：1.4E-45～3.4028235E38
* double 占八个字节，位数为64bit 包裹类型为 Double，它的取值范围为：4.9E-324～1.7976931348623157E308


### 2.[java中的switch-case语句]
>switch(A),括号中A的取值只能是整型或者可以转换为整型的数值类型，比如byte、short、int、char、还有[枚举](http://baike.baidu.com/link?url=9VzlTPyBth5SrT03Fk8-eRPMqSxfeq33GYVM0EtHdJ-jSHakcvjhT6MX4yhgDKLxOCFXuqAn4fIGY6t-TGArjpGCm9nNdgt9moM6unpG4BYlzd3AUd3Da3VMViNXemhq)；需要强调的是：
* long类型是不能作用在switch语句上的。
* String类型在java7之前是不能作用在switch语句上的，但从java7开始，String类型可以作用在switch语句上了（这个新特性是在编译器这个层次上实现的）。
>case B：C；case是常量表达式，也就是说B的取值只能是常量（需要定义一个final型的常量）或者int、byte、short、char（比如1、2、3、200000000000（注意了这是整型）），case后的语句可以不用大括号，就是C不需要用大括号包裹着；


### 3.[equals与==的区别]
* 对于基本数据类型boolean,byte,char,short,int,long, float,double来说，变量直接存储的是“值”，因此在用关系操作符==来进行比较时，比较的就是 “值” 本身。
* 而对于非基本数据类型的变量，即引用变量。引用类型的变量存储的并不是 “值”本身，而是于其关联的对象在内存中的地址。 
>对于==，如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等；如果作用于引用类型的变量，则比较的是所指向的对象的地址。
>对于equals方法，注意：equals方法不能作用于基本数据类型的变量;如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址；诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。

### 4.[Object类的一些公用方法]
>Object类中一共有11个公用方法
![image](http://img.blog.csdn.net/20150924110146429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


### 5.[Java的四种引用(强软弱虚，以及它们的具体应用)]----不完善！！！
>强引用:强引用不会被GC回收，并且在java.lang.ref里也没有实际的对应类型，平时工作接触的最多的就是强引用。
Object obj = new Object();这里的**obj**引用便是一个强引用。如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。当内存空 间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。


>软引用：如果一个对象只具有软引用，那就类似于可有可物的生活用品。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只 要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。 软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。


>弱引用：如果一个对象只具有弱引用，那就类似于可有可物的生活用品。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回 收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。


>幽灵引用(虚引用)：虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用必须和引用队列 （ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。

### 6.[ ArrayList、LinkedList、Vector的区别]
* ArrayList，Vector，LinkedList都实现了List
* Arraylist和Vector是采用数组方式存储数据,Vector是线程安全的而ArrayList不是线程安全的
* LinkedList使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项前后项即可，插入数据较快,LinkedList是不同步的。

### 7.[ Hashcode的作用]
