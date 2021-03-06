#### JVM内存区域

##### JVM内存区域（内存模型、运行时数据区域）

<img src="https://snailclimb.gitee.io/javaguide/docs/java/jvm/pictures/java%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F/JVM%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA%E5%9F%9F.png" style="zoom:50%;" />

- **程序计数器：** 通过改变这个计数器的值来选取下一条需要执行的字节码指令

  **它是唯一一个不会出现 `OutOfMemoryError` 的内存区域**

- **Java虚拟机栈：**存放Java方法的参数，变量和局部变量等信息

- **本地方法栈**：用于native方法使用

- **堆**：存放对象实例和数组，GC的主要区域

- **方法区**：被虚拟机加载的类信息、常量、静态变量等（1.8后变成了元空间，放在直接内存中）

##### 创建对象的五步

- **类加载检查：**检查常量池中是否有这个类的符号引用，并检查是否加载初始化过；若没有，执行类加载
- **分配内存**
  - **方法**
    - **指针碰撞法（内存规整）**：将内存分为已使用和未使用，用一个指针相隔，分配时就将指针移动指定大小
    - **空闲列表（内存不规整）：**维护一个可用列表，从列表中找出一块可用的内存
  - **并发问题**
    - CAS机制保证分配动作的原子性
    - 每个线程预先在堆分配一块内存，给对象分配时就在每个线程自己的这块内存上分配，用完了再分配新的
- **初始化0值**：将分配的内存置0，以便能够直接使用
- **设置对象头**：包含GC分代年龄、hashCode、锁状态标志等信息
- **执行init**：按照程序员的意愿初始化字段

##### 对象内存布局

- **对象头**
  - **自身运行时数据（Mark Word）**：GC分代年龄、hashCode、锁状态标志等
  - **类型指针**：指向类元数据，表示它是哪个类的实例
- 实例数据
- 对齐填充：要求对象大小为8字节整数倍，对象头为8字节或16字节(取决于机器位数)

##### 对象的访问方式

- **句柄**：在堆中分出一块内存为句柄，引用存的就是句柄的地址，句柄再指向实例
- **直接指针**：引用直接保存的就是实例地址

#### GC垃圾回收

##### 对象死亡判断的2种方法

- **引用计数法**

- **可达性分析**（图）：从GC ROOT向下搜索，若该对象不可达，则不可用；后判断是否有必要执行finalize方法（覆盖finalize方法或者已经执行过了）；若必要就二次标记，没有建立其他引用就回收；

  **GC ROOT**

  - **栈**：虚拟机栈、本地方法栈中引用的对象
  - **方法区**：类静态属性、常量引用的对象

##### Full GC（大对象进入）

- 回收堆、方法区
- 条件
  - **老年代空间不够**：新生代大对象、大数组转移
  - **方法区空间不够**

##### 四种引用（强、软、弱、虚）

- **强引用**：不会回收
- **软引用**：内存不够回收
- **弱引用**：发现就回收（不管内存够不够）
- **虚引用**：跟没有引用一样，必须配合引用队列使用，用于跟踪对象回收的活动

##### 无用的常量和类

- **常量**：没有String引用的常量会从常量池中回收
- **类**
  - 无实例
  - 无加载该类的ClassLoader
  - 无Class对象的使用

##### GC算法

- **标记-清除**：标记得是**不需要回收的对象**，然后清除没有标记的对象
  - 效率低、碎片化高
- **复制算法**：将内存分为两部分，每次使用其中一块，用完后就将存活对象复制到另外一块，然后清除。
  - 效率高
- **标记-整理**：标记后，将标记对象移动到另一端，清除边界以外的对象
- **分代收集**：将内存分代，然后采用不同的上述几种算法；**新生代-复制法**，**标记法**

##### GC优化

尽量在新生代回收，避免过大过多的对象进入老年代

- 合理设置年龄大小，以及老年代的空间大小，避免Full GC

##### GC收集器

- 初代
  - Serial：单线程，需要停止用户线程
  - ParNew：多线程版本
- 现行
  - **CMS**：注重用户体验，**以获取最短回收停顿时间为目标**。采用**标记-清除算法**
    - **步骤**
      - **初始标记**：停下其他线程，标记**直接与Root相连的对象**
      - **并发标记**：从Root记录可达对象，但由于用户线程在更新，没有实时性；
      - **重新标记**：修正上一步更新的记录（会停止用户线程）
      - **并发清除**
  - **G1**：面向服务器，**以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.**
    - **特点**
      - G1将堆分为多个Region，在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region
      - **并行与并发**：充分利用多CPU减少停顿
      - **分代收集**：G1保留了分代的概念
      - **空间整合**：整体是”标记-整理“，局部是”复制“
      - **可预测的停顿**：使用者可以明确指定GC时间

#### 类加载过程

##### **加载**

- 通过全类名获取二进制字节流并转换为方法区的数据机构
- 在内存中生成Class对象作为数据访问入口

##### **验证**

<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/%E9%AA%8C%E8%AF%81%E9%98%B6%E6%AE%B5.png" style="zoom:50%;" />

##### **准备**

- 正式为类变量分配内存并设置类变量初始值的阶段，初始值除了final其余均为0（null）

##### **解析**

- 将符号引用变为直接引用；

##### **初始化**

- 执行字节码（主动使用类才会初始化）

#### 类生命周期

​	<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-11/%E7%B1%BB%E5%8A%A0%E8%BD%BD%E8%BF%87%E7%A8%8B-%E5%AE%8C%E5%96%84.png" style="zoom:50%;" />

#### 类加载器

##### **BootstrapClassLoader(启动类加载器)** 

- 最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。

##### **ExtensionClassLoader(扩展类加载器)** 

- 主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。

##### **AppClassLoader(应用程序类加载器)** 

- 面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。

#### 双亲委派模型



<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-6/classloader_WPS%E5%9B%BE%E7%89%87.png" style="zoom:50%;" />

- 加载时将请求委派给父加载器，检查是否加载过。如果不能处理才由自己加载；总之**就是自底向上的检查是否加载过，自顶向下的尝试加载类**
- 优点
  - 避免类重复加载
  - 防止Java的API被篡改，比如用户自定义Object类，运行时出现多个Object类

