# IPC方式

## 使用Bundle

我们知道，四大组件中的三大组件（Activity、Service、Receiver)都是支持在Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以它可以方便地
在不同的进程间传输。基于这一点，当我们在一个进程中启动了另一个进程的Aciviy、Service和Receiver、我们就可以在Bundle中附加我们需要传输给远程进程的信息
并通过Intent发送出去。当然，我们传输的数据必须能够被序列化，比如基本类型、实现了Parcclable接口的对象、实现了Serializablc接口的对象以及一些Android
支持的特殊对象，具体内容可以看Bundle这个类，就可以看到所有它支持的类型。Bundle不支持的类型我们无法通过它在进程间传递数据。

## 使用文件共享

共享文件也是一种不错的进程间通信方式，两个进程通过读/写同一个文件来交换数据，比如A进程把数据写入文件，B进程通过读取这个文件来获取数据。我们知道，在
Windows上，一个文件如果被加了排斥锁将会导致其他线程无法对其进行访问，包括读写，而由于Android系统基于Linux，使得其并发读/写文件可以没有限制地进行，甚
至个线程同时对同一个文件进行写操作都是允许的，尽管这可能出问题。通过文件交换数据很好使用，除了可以交换一些文本信息外，我们还可以序列化一个对象到文件系
统中的时从另一个进程中恢复这个对象，下面就展示这种使用方法。

通过文件共享这种方式来共享数据对文件格式是没有具体要求的，比如可以是文本文件，也可以是XML文件，只要读/写双方约定数据格式即可。通过文件共享的方式也是有
局限性的，比如并发读/写的问题，像上面的那个例子，如果并发读/写，那么我们读出的内容就有可能不是最新的，如果是并发写的话那就更严重了。因此我们要尽量避免
并发写这种情况的发生或者考虑使用线程同步来限制多个线程的写操作。通过上面的分析，我们可以知道，文件共享方式适合在**对数据同步要求不高**的进程之间进行通
信，并且要妥善处理并发读/写的问题。

当然，SharedPreferences是个特例，众所周知，SharedPreferences是Android中提供的轻量级存储方案，它通过键值对的方式来存储数据，在底层实现上它采用XML文
件来存储键值对，每个应用的SharedPreferences文   都可以在当前包所在的data目录下查看到。一般来说，它的目录位于/data/data/package name/shared _prefs
目录下，其中package name表示的是当前应用的包名。从本质上来说，SharedPreferences也属于文件的一种，**但是由于系统对它的读/写有一定的缓存策略**，即在内
存中会有一份SharedPreferences文件的缓存，因此在多进程模式下，系统对它的读/写就变得不可靠，当面对高并发的读/写访问，Sharedpreferences 有很大几率会丢
失数据，因此，不建议在进程间通信中使用SharedPrefcrences.

## 使用Messenger
Mesenger可以翻译为信使，顾名思义，通过它可以在不同进程中传递Mesage对象，在Mesage中放入我们需要传递的数据，就可以轻松地实现数据的进程间传递了，
Mesenger是一种轻量级的IPC方案，它的底层实现是AIDL，为什么这么说呢，我们大致看一下Mesenger这个类的构造方法就明白了。下面是Messenger的两个构造方法，
从构造方法的实现上我们可以明显看出AIDL的痕迹，不管是IMessenger 还是Stub aslnterface，这种使用方法都表明它的底层是AIDL。

Mesxenger的使用方法很简单，它对AIDL做了封装，使得我们可以更简便地进行进程间通信。同时，由于它一次处理一个请求。因此在服务端我们不用表虑线程回步的问题，
这是因为服务端中不存在并发执行的情形。实现一个Mescnger有如下几个步骤,分为[服务端](https://github.com/jianjiandandande/Android-Demo/blob/master/aidldemo/src/main/java/com/example/aidldemo/service/MessagerService.java)和[客户端](https://github.com/jianjiandandande/Android-Demo/blob/master/aidldemo/src/main/java/com/example/aidldemo/MessagerActivity.java)

通过上面的例子可以看出，在Messenger中进行数据传递必须将数据放入Message中，而Messenger和Message都实现了Parcelable接口，因此可以跨进程传输。简单来说，
Message中所支持的数据类型就是Messenger所支持的传输类型。实际上，通过Messenger来传输Message，Message中能使用的载体只有what、argl、arg2、Bundle 以及
replyTo。Message中的另一个字段object在同一个进程中是很实用的，但是在进程间通信的时候，在Android2.2以前object字段不支持跨进程传输，即便是2.2以后，也
仅仅是**系统提供的实现了Parcelable接口的对象才能通过它来传输**。这就意味着我们自定义的Parcelable对象是无法通过object字段来传输的，读者可以试一下。非
系统的Parcelable对象的确无法通过object字段来传输，这也导致了object字段的实用性大大降低，所幸我们还有Bundle,Bundle中可以支持大量的数据类型。

## 使用AIDL

上面我们介绍了使用Messenger来进行进程间通信的方法，可以发现，Messenger是以串行的方式处理客户端发来的消息，如果大量的消息同时发送到服务端，服务端仍然
只能一个个处理，如果   大量的并发请求，那么用Messenger就不太合适了。同时，Messenger的作用主要是为了我递消息。很多时候我们可能需要跨进程调用服务端的方
法，这种情形用 Mesenger就无法做到了，但是我们可以使用AIDL来实现跨进程的方法调用。AIDL也是Mescnger的底层实现，因此Messenger本质上也是AIDL，只不过系统
为我们做了封装从而方便上层的调用而已。在上一节中，我们介绍了Binder的概念，大家对Binder也有了一定的了解，在Binder的基础上我们可以更加容易地理解AIDL。这
里先介绍使用AIDL来进行进程间通信的流程，分为[服务端](https://github.com/jianjiandandande/Android-Demo/blob/master/aidldemo/src/main/java/com/example/aidldemo/service/AIDLService.java)和[客户端](https://github.com/jianjiandandande/Android-Demo/blob/master/aidldemo/src/main/java/com/example/aidldemo/AIDLActivity.java)两个方面。

在AIDL文件中，并不是所有的数据类型都是可以使用的，那么到底AIDL文件支持哪些数据类型呢?如下所示:
* 基本数据类型（int、long、char、boolcan、double等）;
* String和CharSequence;
* List:只支持ArayList，里面每个元素都必须能够被AIDL支持;
* Map：只支持HashMap，里面的每个元素都必须被AIDL支持，包括key和value;
* Parcelable，所有实现了Parcelable接口的对象;
* AIDL：所有的AIDL接口本身也可以在AIDL文件中使用。
以上6种数据类型就是AIDL所支持的所有类型，其中自定义的Parclabe对象和AIDL对象必须要显式impot进来，不管它们是否和当前命AIDL文件位于同一个包内。

上边的例子就是AIDL的大致实现过程，但是有几点还需要再次说明一下。我们知道，客户端调用远程服务的方法，被词用的方法运行在服务端的inder线程池中，同
时客户端线程会被挂起，这个时候如果服务端方法执行比较耗时，就会导致客户端线程长间地阻塞在这里，而如果这个客户端线程是UI线程的话，就会导致客户端ANR，
这当然不是我们想要看到的。因此，如果我们明确知道某个远程方法是耗时的，那么就要避免在客户端的U1线程中去访问远程方法。由于客户端的onServiceConnected
和onServiceDisounected方法都运行在UI线程中，所以也不可以在它们里面直接调用服务端的耗时方法.这点要尤其注意。另外，由于服务端的方法本身就运行在服务
端的Binder线程池中，所以服务端方法本身就可以执行大量耗时操作，这个时候切记不要在服务端方法中开线程去进行异步任务，除非你明确知道自己在干什么，否则
不建议这么做。向类似的，如果在服务端调用了客户端的某一个方法，这个方法也是运行在客户端的Binder线程池之中。

## 使用ContentProvider

ContenProvider 是Androd中提供的专门用于不同应用间进行数据共享的方式，从这一点来看。它天生就适合进程间通信，和Messenger一样，ContentProvider 的底
层实现同样也是Binder，由此可见，Binder在Androd系统中是何等的重要，虽然ContentProvier的底层实现是Binder，但是它的使用过程要比AIDL简单许多，这是因
为系统已经为我们做了封装，使得我们无须关心底层细节即可轻松实现IPC.ConmtentProvidcr虽然使用起来很简单，包括自己创建一个ConmtentProvdr也不是什么难事，
尽管如此，它的细节还是相当多，比如CRUD操作、防止SQL注入和权限控制等。由于章节主题限制，在本节中，笔者暂时不对ContentProvider的使用细节以及工作机制
进行详细分析，而是为读者分析采用ContentProvider实现跨进程通信的一般原理。

系统预置了许多ContentProvider，比如通讯录信息、日程表信息等，要跨进程访问这些信息，只需要通过ContentResolver的query、update、insert和delete方法
即可。创建一个自定义的ContentProvider很简单，只需要继承ContentProvider类并实现六个抽象方法即可：onCreate、query、update、insert、delete和
gefType.这六个抽象方法都很好理解，onCreate代表ContentProvider的创建，一般来说我们需要做一些初始化工作；getType用来返回一个Uri请求所对应的MIME类型
（媒体类型），比如图片、视频等，这个媒体类型还是有点复杂的，如果我们的应用不关注这个选项，可以直接在这个方法中返回null或者“ * / * ”：剩下的四个方法对应于
CRUD操作,即实现对数据表的增删改查功能。根据Binder的工作原理，我们知道这六个方法均运行在ContentProvider的进程中，除了onCreate由系统回调并运行在主线程
里，其他五个方法均由外界回调并运行在Binder线程池中。(这部分后期会不断完善)
