# java基础
## 一.Object类中的方法
### 1.equals()
设计原则:
```
1.自反性:自己必须跟自己相等
2.传递性:x=y,y=z,则x=z
3.对称性:x.equals(y)则y.equals(x)
4.一致性:多次调用equal应返回同样的结果
5.null与任何非空不相等
```
一般编码步骤:
```
1.检测是否为同一个引用
if(this == otherObj) return true;

2.检测非空
if(otherObj == null) return false;

3.检测继承关系(即是否为同一个类)
1)若相等关系由父类决定,则
if(! (otherObj instanceof()) ) return false;
2)若相等关系由子类决定,则
if(getClass() != otherObj.getClass()) return false

4.将otherOBj转换成this相同的类型,并进行比较他们的属性是否一致
```
## HashCode()
注意事项:
此方法与equals()息息相关,重新定义了equals就必须重新定义HashCode;
原因是:**x.equals(y)==true,则x和y的HashCode就必须相等,即对象相等则hashcode一定相同.**,若不重写,则可能会出现equals相等,但hashcode不相等的错误

## toString()

## 二、HahshMap面试知识
### put方法过程：
1.计算hash值（高位与低位异或，让高位参与运算，尽可能的避免hash算法太垃圾）

2.寻址算法 hash & （n-1） （必须为2的次方数，因为2的次方数二进制为10000这种格
式，减一就会变成11111这种格式，与hash异或后
位置就是hash的低位，而且在保证在0-n之内）

- 在n为2的n次方的情况下，hash & （n-1） 等值于对n取余，因为取余就是右移几位，

3.查找当前桶（判断是否为红黑树结构，若是就采用红黑树的查找方法，否则就正常采用链表的方法）
	重复了就覆盖并且返回旧值，没有的话就新建Entry并插入链表

### 扩容：
* 条件：在容量超出阈值，默认16*0.75，
* 扩容时rehash，由于容量是*2，所以就是hash值向前添加一位，如原先hash值低位为1010，现在将它的前一位添加进来，变为01010或者1 1010，即
同一个桶的链表，有的保持位置不变，有的+n（原先容量），这样扩容也能达到减短链表的目的	

### 链表转换红黑树：
* 链表大于阈值（默认8）就调用treeifyBin方法转换，方法内判断容量是否大于64，若大于则变成红黑树，否则就只是扩容

##### static关键字

###### 修饰方法、变量

表示属于类，多个对象共享。调用时直接用类名就行，不需要创建实例

###### 修饰内部类

这个内部类独立与外围类，可以单独创建使用；不能使用任何外围类的非static变量

###### 修饰代码块

类的static代码块最先执行，只执行一次

###### import static

用于引入类的静态方法和成员，不用加类名就能直接使用