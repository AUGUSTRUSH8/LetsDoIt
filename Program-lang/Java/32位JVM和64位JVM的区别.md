### 32位JVM和64位JVM的区别

先看两个参数，-Xms和-Xmx。参数-Xms设置了JVM初始的堆内存，默认是物理内存的 1/64；-Xmx设置了JVM最大的堆内存，默认是物理内存的 1/4。

直接在命令行运行java -X，可获得所有-X开头的参数的说明：

```xml
C:\Users\1024>java -X
-Xms<size>        设置初始 Java 堆大小
-Xmx<size>        设置最大 Java 堆大小
```

直接在命令行运行java -version，可知JVM是否是64位。不显示64-Bit的则表示32位，下面表示的是64位：

```xml
C:\Users\1024>java -version
java version "1.7.0_17"
Java(TM) SE Runtime Environment (build 1.7.0_17-b02)
Java HotSpot(TM) 64-Bit Server VM (build 23.7-b01, mixed mode)
```

在32位的模式下，引用是4个字节，2的32次方等于4294967296，也就是4G，JVM可寻址4G大小的内存，这也是32位JVM理论上的最大堆大小（-Xmx）。32位模式下，Linux系统JVM的最大堆大小可设置为2~3GB，Windows系统可设置为1.5GB。

而在64位模式下，引用是8个字节，JVM可寻址2的64次方字节的内存，这是非常大的一个数字，其JVM理论上的最大堆大小（-Xmx）也可以非常的大。

虽然64位JVM可以寻址更多的内存，但实际上，就像大象没有老鼠灵活一样，64位JVM的性能反而比32位JVM的性能差一些。根据Oracle JDK文档，在SPARC处理器上运行应用程序，64位的要比32位的性能差大约为10~20％；在AMD64和EM64T处理器上，性能差大约为0~15％，具体取决于应用程序中的指针访问次数。

一般当-Xmx必须大于2GB时，可以考虑从32位JVM更新到64位JVM。但要注意随着堆大小的增加，自动GC暂停时间（GC Pause time）也将开始变高，因为内存中会有更多的垃圾需要清理，这需要进行适当的GC调整，避免GC暂停时间太长。另外要注意的是，如果应用使用了JNI（Java Native Interface），也需要更新Native库，32位对应32位的库，64位对应64位的库。