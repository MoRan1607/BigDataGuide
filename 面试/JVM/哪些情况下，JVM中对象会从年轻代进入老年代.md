### 哪些情况下，JVM中对象会从年轻代进入老年代

新生代主要用于存储新创建的对象，而老年代主要用于存储生命周期较长的对象。

以下是对象从年轻代进入老年代的几种常见情况：

- **对象年龄达到阈值**：在年轻代的 Eden 区创建的新对象，在经历了一次 Minor GC（Minor Garbage Collection）后，如果仍然存活，它们会被移到 Survivor 区。在 Survivor 区，每次对象经历一次垃圾回收（未被回收），其年龄（即存活次数）就会增加。当对象的年龄达到某个设定的阈值（默认值通常是 15，但可以通过 JVM 参数 `-XX:MaxTenuringThreshold` 调整），该对象会被提升到老年代。
- **Survivor 区空间不足**：对象在Eden区和Survivor区之间移动。如果对象在Survivor区无法找到足够的空间存放，而且年龄达到一定阈值，则该对象会晋升到老年代。
- **大对象直接进入到老年代**：大对象（如大型数组或字符串）如果太大，因为占用幸存者区空间过大或大于幸存者区，可能导致频繁的垃圾回收，重复复制大对象增加GC时间问题。为了避免这种情况，JVM 可能会直接将大对象分配到老年代。可以通过 JVM 参数 `-XX:PretenureSizeThreshold` 设置直接分配到老年代的大对象的大小阈值，如果对象超过设置的大小会直接进入老年代，防止大对象进入年轻代造成重复GC。
- **对象动态年龄判断**：在某些情况下，当一批对象的总大小大大于Survivor区域的一半，此时大于该批对象中最大年龄的对象，直接进入老年代。


**[知识星球资料介绍](https://www.yuque.com/vxo919/gyyog3/ohvyc2e38pprcxkn?singleDoc=)**   

<p align="center">
<img src="https://github.com/MoRan1607/BigDataGuide/blob/master/Pics/%E6%98%9F%E7%90%83%E4%BC%98%E6%83%A0%E5%88%B8-30.jpg"  width="300" height="387"/>  
<p align="center">
</p>
</p>  
