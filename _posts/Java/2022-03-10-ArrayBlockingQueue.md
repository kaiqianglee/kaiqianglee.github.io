### 介绍

用Array实现的一个有界阻塞队列。队列中的元素按照FIFO（first-in-first-out）组织。队头的元素在队列中的时间最久，对尾的元素在队列中的时间最短。新的元素从队尾插入，从对头获取元素。

使用一个固定长度的Array存储元素，一旦创建，数组的容量不会改变。试图插入一个已满的队列会被阻塞，试图从一个空的队列取出元素也会阻塞。

支持公平策略处理等待的生产者和消费者线程。默认是非公平的。公平模式降低了吞吐但是减少了可变性，避免了饥饿线程的出现。

### API

boolean add(E e)

插入一个特定元素到队列尾部，成功插入返回true；队列已满抛出异常



boolean offer(E e)

插入一个特定元素到队列尾部，成功插入返回true；队列已满返回false



void put(E e) throws InterruptedException

插入一个特定元素到队列尾部，如果队列满了会阻塞



boolean offer(E e, long timeout, TimeUnit unit)

带有超时时间的offer



E poll()

取出队头元素。如果队列为空，返回null



E poll(long timeout, TimeUnit unit)

取出对头元素，如果队列为空，等待timeout 时间



E take() throws InterruptedException

取出队头元素。如果队列为空，阻塞等待



### 实现原理

ReentrantLock + 两个Condition notEmpty和notFull 实现。

