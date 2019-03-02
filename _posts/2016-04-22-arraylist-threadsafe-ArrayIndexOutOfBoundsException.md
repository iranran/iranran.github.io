---
layout: post
title: ArrayList线程安全引起数组越界问题
category:
- Java基础知识
- 线程安全
feature_image: "https://picsum.photos/2560/600?image=770"
---

ArrayList不是线程安全的，那么如果用在多线程中会出现什么问题呢，下面的问题是在工作中踩的一个坑。

```
  List<Object> list = new ArrayList<>();
  stocklist.stream().forEach(stock->{
    ticket = api.getTicket(stock);
    list.add(ticket);
  });
 ```
 
 某天同事觉得这段代码跑得有点慢，就把stream换成parallelStream,结果就报如下错误：
 
 ```
 Exception in thread "Thread-1" Exception in thread "Thread-2" java.lang.ArrayIndexOutOfBoundsException: 15
     at java.util.ArrayList.elementData(ArrayList.java:418)
     at java.util.ArrayList.get(ArrayList.java:431)
     at java.lang.Thread.run(Thread.java:745)
 java.lang.ArrayIndexOutOfBoundsException: 15
     at java.util.ArrayList.add(ArrayList.java:459)
     at java.lang.Thread.run(Thread.java:745)
```


### 分析：

ArrayList是基于数组实现的，数组长度是固定的，当元素达到数组上限时容量就要自动增长。
add实现方法如下，首先检查数组长度是否可以再添加一个元素，如果不满足就扩容后添加此元素，然后再复制数组。

```

    public boolean add(E e) {
        ensureCapacityInternal(size + 1); 
        elementData[size++] = e; //数组赋值
        return true;
    }
    
    private void ensureExplicitCapacity(int minCapacity) {
            modCount++;
    
            // overflow-conscious code
            if (minCapacity - elementData.length > 0)
                grow(minCapacity);
        }
```

由于ArrayList是线程不安全的，假设当前数组容量是15，当前size=14

| 序号        | 线程1   |  线程2  |
| --------   | -----:  | ----:  |
| 1      | add   |   add     |
| 2        |  数组赋值 elementData[14]=e    |   |
| 3        |        |  数组赋值 elementData[15]=e  |
| 4       |        |  ArrayIndexOutOfBoundsException  |

可以看到第1步线程1和2都认为不需要扩容，线程1先执行add这时size=15数组已满无法再添加元素，而线程2执行第三步时自然就越界了。

### 解决办法

明白问题所在，解决办法就好弄了
- list.add(ticket); 加上锁
- 把ArrayList换成CopyOnWriteArrayList

<script async src="https://www.googletagmanager.com/gtag/js?id=UA-135360671-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-135360671-1');
</script>