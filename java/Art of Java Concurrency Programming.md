# Java并发编程的艺术

## Java对象头
如果对象是数组类型 JVM用3个word，如果是非数组类型 用2word来存储对象头
| 长度     | 内容                   | 说明                       |
|-        |-                       |-                          |
|32/64bit | Mark Word              | 存储对象的hashcode或锁信息等 |
|32/64bit | Class Metadata Address | 存储到对象类型数据的指针      |
|32/32bit | Array Length           | 数组的长度                  |  

### 下面是32bitJVM的Mark Word的存储结构和变化
| 锁状态  | 25bit          | 4bit        | 1bit 是否是偏向锁 | 2bit 锁标志位|
|-       |-               |-            |-                |-            |
|无锁禁用偏向| 对象的hashcode | 对象分代年龄 |  0              | 01          |
|偏向锁   | 线程ID  Epoch   | 对象分代年龄  |  1              | 01          |
|轻量锁   | 指向栈中锁记录的指针                            ||| 00          |
|重量锁   | 指向互斥量mutex（重量级锁）的指针                 ||| 10         |
|GC标记   | 空                                           ||| 11          |

> 偏向锁没有解锁 上偏向锁（CAS写入线程ID）失败时，认为竞争激烈，偏向锁撤销并升级为轻量锁（需要等待JVM safepoint）；上偏向锁成功后（mark word已偏向），
其他线程来竞争偏向锁时会导致偏向锁被撤销，是否升级取决于现在偏向的线程是否活着。
> 偏向锁被撤销后 对象头为 无锁禁用偏向状态。
> 偏向锁能否重新偏向？ 可以，Bulk Rebias机制，借用epoch（时间戳），对象所属的类class信息中， 也会保存一个epoch值，每次safepoint会更新class和锁定该class实例的线程中的epoch，当有线程需要尝试获取偏向锁时， 直接检查class中存储的epoch值是否与目标对象中存储的epoch值相等， 如果不相等，则说明该对象的偏向锁已经失效了， 可以尝试对此对象重新进行偏向操作

### Java锁与上厕所



## synchroized的原理和实现
Java中的每一个对象都可以作为锁
* 对于普通方法 锁是当前实例对象
* 对于静态同步方法 锁是当前类的Class对象
* 对于同步方法块 锁是synchronized括号内的对象

JVM基于进入和退出monitor对象来实现方法和代码块的同步 字节码指令 monitorenter/monitorexit 2者实现细节不一样

### 锁的状态 
* 无锁  
* 偏向锁 
    大多数情况由同一线程多次获得 此时
* 轻量级锁 
* 重量极锁

## happens-before
1. 程序顺序原则: 一个线程中的每个操作，happens-before于该线程中的任意后续操作
2. 监视器锁规则 monitor mutex: 对一个锁的解锁，happens-before于随后对这个锁的加锁
3. volatile变量规则: 对一个volatile变量的写，happens-before于任意后续对这个volatile的读
4. 传递性
5. start()原则: 如果线程A执行ThreadB.start()，那么A的start操作happens-before于线程B中的任意操作
6. join()原则: 如果线程A执行操作ThreadB.join()并成功返回，那么线程B的中任意操作happens-before于A的join

