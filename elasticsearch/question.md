### 1、JVM有哪些内存区域

* 程序计数器：当前线程所执行的字节码的行号指示器；
* 本地方法栈：执行本地方法时所创建的内存栈区，保存本地方法执行过程中的局部变量；
* Java虚拟机栈：执行Java方法时所创建的内存栈区；
* Java堆：最大的内存区域，新创建的对象都会分配在这区域；
* 方法区：主要存放常量、静态变量、类方法名等公有的信息。

### 2、Java内存泄漏

原因：长生命周期的对象持有短生命周期对象的引用就可能会发生内存泄漏。

### 3、强软弱虚引用

* 强引用：一般new出来的对象就是返回强引用，内存空间不够时，JVM也不会回收，宁愿抛出内存不够异常；
* 软引用：SoftReference，内存空间不够时，jvm gc时就会将其回收；
* 弱引用：WeakReference，jvm下次gc时就会将其回收，即使内存空间足够；
* 虚引用：PhantomReference，不能通过该引用来获取到对象，虚引用的作用仅是跟踪对象被垃圾回收器回收的活动。

### 4、新生代、老年代和永久代

* 永久代：主要存放Class和元数据的信息。

### 5、GC垃圾收集器

* Serial：最简单的，单线程处理；
* Parallel：并行的，多个线程执行；
* ParNew：
* CMS：并发的，针对老年代，标记清除算法，以牺牲吞吐量为代价来获取最短回收停顿时间的垃圾回收器，初始标记、并发标记、重新标记、并发清除；
* G1：并行的，标记整理算法，针对整个Java堆。

### 6、类加载的执行过程

* 加载：找到相应的class文件导入；
* 验证：检查加载的class文件的正确性；
* 准备：给类中的静态变量分配内存空间；
* 解析：虚拟机将常量池中的符号引用替换成直接引用的过程；
* 初始化：对静态变量和静态代码块执行初始化工作。

### 7、Bean的生命周期

* 实例化；
* 填充属性；
* 如果实现BeanNameAware接口，会调用setBeanName；
* 如果实现BeanFactoryAware接口，会调用setBeanFactory，将beanFactory注入；
* 如果实现ApplicationContextAware接口，会调用setApplicationContext，将applicationContext注入；
* 如果实现BeanPostProcessor接口，会调用postProcessBeforeInitialization方法；
* 如果实现InitializingBean接口，会调用afterPropertiesSet方法。如果使用initMethod声明初始化方法，该方法也会被调用；
* 如果实现BeanPostProcessor接口，会调用postProcessAfterInitialization方法；
* 如果实现DisposableBean接口，会调用destroy方法，如果使用destroy-method声明了销毁方法，该方法也会被调用。

### 8、Spring如何解决循环依赖问题

三级缓存



