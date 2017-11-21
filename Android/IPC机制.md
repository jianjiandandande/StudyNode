# IPC机制
IPC(Inter-Process Communication)，含义为进程间通信或者跨进程通信，指的是两个进程之间进行数据交换的过程。
 * 线程：CPU调度的最小单元，同时是一种有限的系统资源。
 * 进程：一个执行单元。 

 ## Android中使用多进程(一个应用内部)的方法：
  * 在AndroidMMenifest中指定:android:process属性，除此之外没有别的办法，也就是说，我们没有办法给一个线程或者实体类指定其运行时所在的进程。
  * (非常规的使用方法)：通过JNI在native层fock一个新的进程---特殊情况。
 ## Android使用多进程所带来的问题：
  * 静态成员和单例模式完全失效
  * 线程同步机制完全失效
  * SharedPreferences的可靠性下降
  * Application会多次重建

  原因：
   * Android为每一个进程分配一个单独的虚拟机，不同的虚拟机在内存分配上有不同的地址空间。对于一个应用，它开启了多个进程，同时它内部包含了某
  一个实体类，那么分配不同的进程之后，每一个进程中都会有一个这样的实体类的备份，这就导致静态成员以及单例模式的失效。
   * 线程同步机制的前提是两个线程位于同一块内存当中
   * SharedPreferences不支持两个进程同时执行写操作，否则会导致一定几率的数据丢失(SharedPreferences的底层是通过读写xml文件来实现的，并发写很明显会存在问题)。
   * 系统在创建新的进程的时候要为进程分配虚拟机，这个过程其实就是启动一个应用的过程。再重启应用的过程中，Application自然会被重建。

  ## IPC基础概念介绍

  ### 序列化
  
  由于不同的进程有着不同的内存区域，并且它们只能访问自己的那一块内存区域，所以我们不能像平时那样，传一个句柄过去就完事了——句柄指向的是一个内存区域，
  现在目标进程根本不能访问源进程的内存，那把它传过去又有什么用呢？所以我们必须将要传输的数据转化为能够在内存之间流通的形式。这个转化的过程就叫做序列
  化与反序列化。数据实现序列化操作之后，就可以通过Intent和Binder进行传递了。

  ### Serializable接口

  想要通过Serializable接口来实现序列化，方法其实很简单，只需要让具体的类实现Serializable接口，然后在类的声明当中声明一个serialVersionUID(这个
  东西其实不是必须的)即可。
  接下来，我为大家写一个例子：
  ```
      public class User implements Serializable{
      
          private static final long serlalVeratonUID =519067123721295773L;
          public int userld;
          public String userlame;
          publicboolean lsMale;
          ...
      }

      //序列化过程
      User user=new User(0,"jake",true);
      ObjectoutputStream out =new ObjectOutputStream(
      new FileOutputStream("cache.txt"));
      out.writeObject(user);
      out.close(）;
      
      //反序列化过程
      ObjectInputStream in=new objectInputStream(
      new FileInputStream("cache.txt"));
      User newUser=(User)in.readobject();
      in.close(）;

    
  ```
通过例子，我们可以发现，所谓的序列化就是将一个具体的对象转化为了流，并保存在了文件当中，反序列化的过程与之相反。

刚开始提到，即使不指定 scrialVersionUITD也可以实现序列化，那到底要不要指定呢？如果指定的话，serialVersionUID后面那一长串数字又是什么含义呢？
我们要明白，系统既然提供了这个serialVersionUID，那么它必须是有用的。这个serial VersionUID是用来辅助序列化和反序列化过程的，原则上序列化后的
数据中的 serial VersionUID只有和当前类的scrialVersionUID相同才能够正常地被反序列化。serialVersionUID的详细工作机制是这样的：序列化的时候系
统会把当前类的serialVersionUID写入序列化的文件中（也可能是其他中介），当反序列化的时候系统会去检测文件中的serialVersionUID，看它是否和当前类的
scrialVersionUID一致，如果一致就说明序列化的类的版本和当前类的版本是相同的，这个时候可以成功反序列化；否则就说明当前类和序列化的类相比发生了某些
变换，比如成员变量的数量、类型可能发生了改变，这个时候是无法正常反序列化的，因此会报如下错误：
```
    java.10.ImvalldclassException:Main;local class incomestib1e:2Freag
    1ocalclass serial-
    classdesc serialVersionUID-8711368828010083044
    Vers1onUTD=8711368828010083043
    
```
一般来说，我们应该手动指定serialVersionUID的值，比如1L，也可以让编译器根据当前类的结构自动去生成它的hash值，这样序列化和反序列化时两者的
serialVersionum是相同的，因此可以正常进行反序列化。如果不手动指定serialVersionUID的值，反序列化时当前类有所改变，比如增加或者删除了某些成
员变量，那么系统就会重新计算当前类的hash 值并把它赋值给serialversionUID，这个时候当前类的serialVersionUTD就和序列化的数据中的
scrialVersionUID不一致，于是反序列化失败，程序就会出现crash，所以，我们可以明显感觉到scrialVersionUID的作用，当我们手动指定了它以后，就可
以在很大程度上避免反序列化过程的失败。比如当版本升级后，我们可能删除了某个成员变量也可能增加了一些新的成员变量，这个时候我们的反向序列化过程
仍然能够成功，程序仍然能够最大限度地恢复数据，相反，如果不指定scrialVersionUID的话，程序则会挂掉。当然我们还要考虑另外一种情况，如果类结构发
生了非常规性改变，比如修改了类名，修改了成员变量的类型，这个时候尽管scrialVersionUID验证通过了，但是反序列化过程还是会失败，因为类结构有了毁
灭性的改变，根本无法从老版本的数据中还原出一个新的类结构的对象。根据上面的分析，我们可以知道，给serialVersionUID指定为1L或者采用Eclipse根据
当前类结构去生成的hash值，这两者并没有本质区别，效果完全一样。以下两点需要特别提一下，首先**静态成员变量**属于类不属于对象，所以不会参与序列化过程；
其次用transient关键字标记的成员变量不参与序列化过程。另外，系统的默认序列化过程也是可以改变的，通过实现如下两个方法即可重写系统默认的序列化和反序列
化过程，具体怎么去重写这两个方法就是很简单的事了，这里就不再详细介绍了，毕竟这不是本章的重点，而且大部分情况下我们不需要重写这两个方法。
```
    private vold writeobject (Java,io.objectoutputStream out)throws IOExceptiont{
        //write‘this'to 'out'.
    }
    
    private yoid readobject (java,Lo.0hjectrnputstream in)throws IOExceptiont,ClassNotFoundException{
        //具体的反序列化操作
    }
```

  ### Parcelable接口

  Parcelable接口是安卓特有的序列化接口，一个具体的类只要实现Parcelable接口就可以进行序列化了。
  这里先说一下Parcel，Parcel内部包装了可序列化的数据，可以在Binder中自由传输。从上述代码中可以看出，在序列化过程中需要实现的功能有序列化、反序列化
  和内容描述。序列化功能由write ToParcel方法来完成，最终是通过Parcel中的一系列write方法来完成的：反序列化功能由CREATOR来完成，其内部标明了如何创
  建序列化对象和数组，并通过Parcel的一系列read方法来完成反序列化过程：内容描述功能由describeContents方法来完成，几乎在所有情况下这个方法都应该返回
  0，仅当当前对象中存在文件描述符时，此方法返回1。
  Intent、Bundle、Bitmap等都是系统提供的实现了Parcelable接口的可序列化的类。

  ### Serializable与Parcelable的区别
  
  Serializable是Java中的序列化接口，使用起来简单，但是相应的开销比较大，序列化和反序列化过程，都需要进行大量的I/O操作。而Parcelable是Android中
  的序列化方式。它的缺点是用起来会稍微麻烦一些，但是它的效率比较高。Parcelable主要用于内存序列化上，而将对象序列化到存储设备中或者将对象序列化后通过
  网络传输时，Parcelable接口实现这个过程是比较复杂的，所以这两种情况下，推荐使用Serializable。

  ### Binder
  直观来说，Binder是Android中的一个类，它实现了IBinder接口。从IPC角度来说，Binder是Android中的一种跨进程通信方式，Binder还可以理解为一和虚拟的物理
  设备，一商设备驱动是de/bimder，该重信方式在七inux中没有：从Android Framework角度来说，Binder 是ServiceManager连接各种Manager（ActivityManager、
  WindowManager等等）和ManagerService 的桥梁：从Android 应用层来说，Binder是客户端和服务端进行通信的媒介。当bindService的时候，服务会返回一个包含
  了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。
  Android开发中，Binder主要用在Service当中，包括AIDL和Messenger,其中，普通的Service中的Binder不涉及进程间的通信，而且Messenger的底层是由AIDL来实
  现的，我看过有一个大神对[AIDL的整个过程的分析](http://blog.csdn.net/luoyanglizi/article/details/51980630)，里面分析的很不错，推荐大家也来看看。
  




























  
