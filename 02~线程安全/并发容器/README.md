# Java 并发容器详解

## 概述

Java 并发包 (J.U.C) 提供了多种线程安全的容器类，用于在并发环境下安全地存储和操作数据。这些容器通过精心的设计和实现，在保证线程安全的同时，还能提供良好的性能。

## 常用并发容器

### 1. ConcurrentHashMap

ConcurrentHashMap 是线程安全的哈希表实现，是 Java 中最常用的并发容器之一。

#### 特点

- 线程安全，替代同步的 HashMap 或 Hashtable
- 采用分段锁机制，提供更好的并发性能
- 不允许 null 键和 null 值
- 弱一致性的迭代器

#### 使用示例

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key1", 1);
map.putIfAbsent("key2", 2);  // 仅当key不存在时才放入值
map.computeIfAbsent("key3", k -> k.length());  // 计算并放入新值
```

### 2. CopyOnWriteArrayList

CopyOnWriteArrayList 是一个线程安全的 ArrayList 变体，特别适合读多写少的场景。

#### 特点

- 写时复制策略，保证读操作不被阻塞
- 适用于读多写少的并发场景
- 内存占用较大
- 数据一致性是最终一致性

#### 使用示例

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("item1");
list.addIfAbsent("item2");

// 安全地进行迭代
for (String item : list) {
    System.out.println(item);
}
```

### 3. BlockingQueue 家族

BlockingQueue 接口的实现类用于在线程间安全地传递数据，常用于生产者-消费者模式。

#### ArrayBlockingQueue

```java
// 创建固定容量的阻塞队列
ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(100);
// 添加元素，如果队列满则等待
queue.put("item");
// 获取元素，如果队列空则等待
String item = queue.take();
```

#### LinkedBlockingQueue

```java
// 创建无界阻塞队列
LinkedBlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>();
// 添加任务
taskQueue.offer(new Task(), 1, TimeUnit.SECONDS);
// 获取任务
Task task = taskQueue.poll(1, TimeUnit.SECONDS);
```

### 4. ConcurrentSkipListMap

ConcurrentSkipListMap 是线程安全的有序映射表，可以替代 TreeMap。

#### 特点

- 维护键的自然顺序
- 支持高并发访问
- 基于跳表实现，提供了很好的查询性能

#### 使用示例

```java
ConcurrentSkipListMap<String, Integer> skipListMap = new ConcurrentSkipListMap<>();
skipListMap.put("A", 1);
skipListMap.put("B", 2);
skipListMap.put("C", 3);

// 按键的顺序遍历
for (Map.Entry<String, Integer> entry : skipListMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

### 5. DelayQueue

DelayQueue 是一个延迟队列，元素只有在其指定的延迟时间到期后才能被取出。

#### 使用示例

```java
class DelayedElement implements Delayed {
    private final long delayTime;
    private final long expire;

    public DelayedElement(long delay) {
        this.delayTime = delay;
        this.expire = System.currentTimeMillis() + delay;
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(expire - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed o) {
        return Long.compare(this.expire, ((DelayedElement) o).expire);
    }
}

DelayQueue<DelayedElement> delayQueue = new DelayQueue<>();
delayQueue.put(new DelayedElement(1000));  // 延迟1秒
DelayedElement element = delayQueue.take();  // 阻塞直到元素可用
```

## 使用建议

### 1. 选择原则

- 默认优先使用 ConcurrentHashMap，而不是 Hashtable
- 读多写少场景考虑使用 CopyOnWrite 容器
- 需要阻塞特性时使用 BlockingQueue
- 需要有序性时使用 ConcurrentSkipListMap

### 2. 性能考虑

- ConcurrentHashMap 的写性能优于同步的 HashMap
- CopyOnWrite 容器的写性能较差，但读性能很好
- BlockingQueue 适合生产者-消费者模式
- 合理设置初始容量，避免频繁扩容

### 3. 注意事项

- 并发容器提供的原子操作不能保证复合操作的原子性
- 迭代器是弱一致性的，可能不反映最新的修改
- 注意内存占用，特别是使用 CopyOnWrite 容器时
- 避免在单线程环境下使用并发容器，会带来不必要的开销

## 总结

并发容器是 Java 并发编程中的重要工具，合理使用可以大大简化并发程序的开发。但要注意，线程安全的容器不等同于线程安全的程序，开发者需要理解每种容器的特性和适用场景，在实际应用中做出正确的选择。

核心原则是：避免共享的可变状态，选择合适的并发容器，正确使用其提供的线程安全方法。
