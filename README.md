# JavaMultiThreadProgramming_Study

## 作者：冰红茶  
## 参考书籍：《Java核心技术》卷一和卷二 原书第10版 《Java多线程编程核心技术》
    
------    
    
记得一年前由于项目需要，急匆匆的拿一本《Java从入门到精通》边学边写StarCCM+的插件。该书实战性很强，就是内容比较浅薄。所以今天又啃起了编程思想，断断续续花了两周时间看了大概看了一半，剩余的内容后面将由JVM和多线程和Hadoop三本书补充完整。大数据的基础是Java和Linux，以后尽管进不了大数据的领域，也希望能借助此基础进入后端领域^_ ^

## 目录
## [一、线程](#13)
### [1.1 什么是线程](#1.1)
### [1.2 线程状态与线程属性](#1.2)
### [1.3 线程安全与非线程安全](#1.3)
### [1.3 同步](#13.3)
### [1.4 阻塞队列](#13.4)
### [1.5 线程安全的集合](#13.5)

        
------      
        
<h2 id='1'>一、线程</h2>
<h3 id='1.1'>1.1 什么是线程</h3>  
        
#### 1) 介绍
> - 通常，每一个任务称为一个线程，可以同时运行一个以上线程的程序称为多线程程序
> - 线程和进程的区别：每个进程拥有自己一整套变量，而线程则共享数据。共享变量使得线程之间的通讯比进程之间的通讯更加有效和容易，与进程相比，线程更加轻量级，创建，撤销一个线程比启动新进程的开销要小得多。
> - 可以理解为线程是进程中独立运行的子任务。
#### 2) 如何开启一个线程
> - 方法一：利用Runnable接口创建Thread类
> - 方法二：继承Thread类然后重写run()方法
> - 方法二：实现Runnable接口然后重写run()方法
> - 接口比继承要好的一个原因是，Java中只允许单根继承。何为单根继承，就是每次只能继承一个超类，而接口可以实现多个实现。
> - 不要直接调用Thread类或者Runnable对象的run方法，因为直接调用run方法只会执行同一个线程中的任务，而不会开启新的线程。开启新的线程应该调用Thread.start()方法
> - 值得注意的是，执行Thread.start()方法顺序不代表线程启动的顺序
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

#### 3) 中断线程
> - 两种结束的条件
>> - 当run方法执行到最后一条语句后，并经由执行return语句返回时
>> - 抛出异常 一般跟sleep()一同使用，在try-catch方法体中遇到异常立马进行catch(),try()中sleep()后面的语句不再运行。而且catch()里面的代码将放在最后进行
>> - 使用Stop方法或者suspend暴力终止，但是不建议使用这个已经被废弃的方法。调用这个方法会抛出java,lang.ThreadDeath异常，但是通常情况下，此异常不需要显示地捕捉。另外，如果强制让线程停止则有可能使一些清理性的工作得不到完成。另外一种情况是对锁定的对象进行了“解锁”，导致数据得不到同步的处理，出现数据不一致的问题。
>> - 使用interrupt()方法中断，但是这个方法不能完全杀死，仅仅是在当前线程中打了一个停止的标记，然后可以通过判断标志位 + return跳出线程
> - 在早期的Java版本中可以使用stop()方法进行线程终结，但是这个方法已经被弃用了，因为“没有任何语言方面的需求要求一个被中断的线程应该终止，中断一个程序不过是引起它的注意，被中断的线程可以决定如何相应中断”
> - 每个线程都有一个Boolean状态叫“中断状态”
>> - 使用静态的Thread.currentThread()方法可以获取当前的线程
>> - 然后使用isInterrupt()方法可以检查这个状态的标志位以判断线程是否被中断。但是如果线程被阻塞或者被中断的时候调用了sleep()或者wait()方法，就会由于无法检测而产生InterruptException异常
                
                // 这两种方法都可以用来检测线程是都被中断，但是前者是对线程的标志位是有影响的，使用后就会清除该线程的中断状态，中断状态会恢复为false；而后者则是无损的

                Thread.currentThread().interrupted();     //Tests whether the current thread has been interrupted. The interrupted status of the thread is cleared by this method. Inother words, if this method were to be called twice in succession, thesecond call would return false (unless the current thread wereinterrupted again, after the first call had cleared its interruptedstatus and before the second call had examined it).    

                Thread.currentThread().isInterrupted(); //Tests whether this thread has been interrupted. The interruptedstatus of the thread is unaffected by this method. 
>> - 使用interrupt()方法可以进行中断置位
                
                Thread.currentThread().interrupt();

> - 一个例子
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

                Runnable r = () -> {
                    System.out.println("start");
                    int i = 0;
                    try {
                        while(!Thread.currentThread().isInterrupted()) {
                            System.out.println("打开一个新的线程"+ i);
                            i++;
                            if(i >= 10) {
                                Thread.currentThread().interrupt();
                                Thread.sleep(1000);
                            }  
                        }
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }finally {
                        System.out.println("close!");
                    }
                };

                Worker w = new Worker(r);
                w.start();

                start
                打开一个新的线程0
                打开一个新的线程1
                打开一个新的线程2
                打开一个新的线程3
                打开一个新的线程4
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程9
                java.lang.InterruptedException: sleep interrupted
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:82)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                close!
#### 3) 中断语句顺序
> - 同步代码首先执行，异步最后执行
> - 同步线程加上sleep()方法后变成异步，异步之间看顺序
> - 遇到异常一般是异常内部的语句最后执行
#### 4) 暂停线程
> - suspend() 暂停线程
> - resume() 重启线程
> - 值得注意的是问题：
>> - 无法访问公共的同步对象：如果使用不当，极易造成公共的同步对象的占用，使得其他线程无法访问呢公共的同步对象，异步的顺序也会被跳过
>> - println()的同步问题：println()方法是同步的，加入你在被暂停的线程中出现println()方法，那么其他地方的println()方法也会被暂停
>> - 数据有时候无法同步，比如需要给两个或以上的实例域进行赋值时，中间突然被暂停，那么后续的值无法赋值了
#### 5) 其他API
> - Thread.currentThread() 使用静态的方法获取当前线程
> - isAlive()判断线程是否处于活动状态
> - join()等待线程终止后做某事
> - sleep(Long long) 线程休眠一段时间后再重启
> - getName() 获取线程的名字
> - getId() 取得线程的位移辨识
> - setDaemon(Boolean boolean) 设置守护线程
> - yield()方法 作用是放弃当前的资源，将他让给其他任务去占用CPU执行时间，但是放弃的时间不确定，当其他资源富余的时候再继续，一般这样一搞该线程的运行时间会明显增加

        
<h3 id='1.2'>1.2 线程状态与属性</h3>  
        
#### 1) 六种状态
> - New(新创建) 使用new进行创建的时候
> - Runnable(可运行) 调用start()方法后
> - Blocked(被阻塞)
>> - 等待获得内部的对象锁
>> - 等待条件满足
>> - 等待超时参数
> - Waiting(等待) 
> - Timed waiting(计时等待)
> - Termianted(被终止)  
> - ![图1-1 线程六种状态.jpg](https://github.com/hblvsjtu/Java_Study/blob/master/picture/%E5%9B%BE15-1%20%E4%B8%80%E4%B8%AA%E5%AF%B9%E8%B1%A1%E5%BA%8F%E5%88%97%E5%8C%96%E7%9A%84%E5%AE%9E%E4%BE%8B.jpg?raw=true)
            
#### 2) 线程属性
> - 线程的优先级 可以使用setPriority方法提高或者降低任何一个线程的优先级，优先等级从1~10从低到高，普通的是5。这个设置优先级要慎重，因为如果有几个高优先级的线程没有进入活动状态，那么低优先级的线程就不会运行而被“饿死”。但是不是说一定要等等级高的执行完才执行优先级低的线程，只是说优先级高的线程所获得的资源比较多
> - 线程的优先级以及继承，也就说用线程A去启动线程B，那么这两个线程的优先级相同
> - 优先级具有随机性，有时候低优先级的线程可能先执行完。
> - 守护线程 其实就是后台线程 可以使用 Thread.currentThread().setDaemon(true);进行设置。只要JVM中有非守护线程，那么守护线程就不会被回收，就像是“保姆”一样，最典型的应用就是GC

        
<h3 id='1.3'>1.3 线程安全与非线程安全</h3>  
        
#### 1) 不共享数据的情况
> - 创建多个实例，不同实例之间肯定是不会共享数据的
                
                /**
                 * 相同的线程类
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

                Runnable r = () -> {
                    System.out.println("start");
                    int i = 0;
                    try {
                        
                        while(!Thread.currentThread().isInterrupted()) {
                            System.out.println("打开一个新的线程"+ i);
                            i++;
                            if(i >= 10) {
                                Thread.currentThread().interrupt();
                                Thread.sleep(1000);
                            }  
                        }
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }finally {
                        System.out.println("close!");
                    }
                };
                Worker w1 = new Worker(r);
                Worker w2 = new Worker(r);
                Worker w3 = new Worker(r);
                Worker w4 = new Worker(r);
                Worker w5 = new Worker(r);
                w1.start();
                w2.start();
                w3.start();
                w4.start();
                w5.start();

                start
                start
                start
                打开一个新的线程0
                打开一个新的线程0
                start
                打开一个新的线程1
                打开一个新的线程1
                start
                打开一个新的线程0
                打开一个新的线程0
                打开一个新的线程2
                打开一个新的线程2
                打开一个新的线程0
                打开一个新的线程3
                打开一个新的线程3
                打开一个新的线程1
                打开一个新的线程1
                打开一个新的线程2
                打开一个新的线程4
                打开一个新的线程4
                打开一个新的线程1
                打开一个新的线程5
                打开一个新的线程5
                打开一个新的线程3
                打开一个新的线程2
                打开一个新的线程4
                打开一个新的线程6
                打开一个新的线程6
                打开一个新的线程2
                打开一个新的线程7
                打开一个新的线程7
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程3
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程8
                打开一个新的线程3
                打开一个新的线程9
                打开一个新的线程9
                打开一个新的线程8java.lang.InterruptedException: sleep interrupted

                打开一个新的线程4
                打开一个新的线程9
                打开一个新的线程4
                打开一个新的线程5
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程6
                打开一个新的线程7
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程8
                打开一个新的线程9
                打开一个新的线程9
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:83)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                close!
                java.lang.InterruptedException: sleep interrupted
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:83)

        
#### 2) 共享数据但不安全的情况
> - 共享数据但不安全的情况，称为“非线程安全”，主要指多个线程对同一个对象中的同一个实例变量进行操作的时候会出现值被更改，值不同步的情况，进而影响程序的执行流程。
> - 另外要出现线程安全还有一个前提时共享一个对象变量，这个很重要。否则就会变成数据不共享。
                
                // 实现Runnable接口的版本
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker implements Runnable {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    public void run() {
                        // TODO Auto-generated method stub
                //      super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 继承Thread的版本
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker extends Thread {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    public void run() {
                        // TODO Auto-generated method stub
                        super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 执行类
                Worker w = new Worker(0);
                Thread t1 = new Thread(w, "t1");
                Thread t2 = new Thread(w, "t2");
                Thread t3 = new Thread(w, "t3");
                Thread t4 = new Thread(w, "t4");
                Thread t5 = new Thread(w, "t5");
                t1.start();
                t2.start();
                t3.start();
                t4.start();
                t5.start();

                // 控制台
                由 t1计算，i的值为 1
                由 t3计算，i的值为 2
                由 t4计算，i的值为 4
                由 t5计算，i的值为 4
                由 t2计算，i的值为 5

        
#### 3) 共享数据且线程安全的情况
> - 共享数据且线程安全，加关键字synchronized 使得多个线程在执行run()方法的时候，以排队的方式进行。
> - 或者在此版本的代码中，我发现只要将数据变化的那一部分放在打印语句中，就不会出现“非线程安全”。这是因为Println()这个方法本书就是同步的。
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker extends Thread {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    synchronized public void run() {
                        // TODO Auto-generated method stub
                        super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 执行类
                Worker w = new Worker(0);
                Thread t1 = new Thread(w, "t1");
                Thread t2 = new Thread(w, "t2");
                Thread t3 = new Thread(w, "t3");
                Thread t4 = new Thread(w, "t4");
                Thread t5 = new Thread(w, "t5");
                t1.start();
                t2.start();
                t3.start();
                t4.start();
                t5.start();

                // 控制台
                由 t2计算，i的值为 1
                由 t5计算，i的值为 2
                由 t1计算，i的值为 3
                由 t3计算，i的值为 4
                由 t4计算，i的值为 5

> - 
> - 
<h3 id='1.3'>1.3 同步</h3>  
        
#### 1) 
------      
        
