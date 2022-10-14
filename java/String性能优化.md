## String性能优化

首先来看一段小段代码如下:
```java
    String str1 = "abc";//1
    String str2 = new String("abc");//2
    String str3 = str2.intern();//3

    //输出结果如下
    System.out.println(str1 == str2);// false
    System.out.println(str2 == str3);//false
    System.out.println(str1 == str3);//false
```

我们的一场程序中, `String`类是最占内存的, 于是JVM底层一个哈希(Hash)缓存机制,叫做`字符串常量池`.

- 对于1，JVM 首先会检查该对象是否在字符串常量池中，如果在，就返回该对象引用，否则新的字符串将在常量池中被创建。这种方式可以减少同一个值的字符串对象的重复创建，节约内存。

- 对于2，首先在编译类文件时，`abc`常量字符串将会放入到常量结构中，在类加载时，`abc`将会在常量池中创建；其次，在调用 new 时，JVM 命令将会调用 `String` 的构造函数，同时引用常量池中的`abc` 字符串，在堆内存中创建一个 `String` 对象；最后，str 将引用 `String` 对象。

- 对于3, 调用`intern`方法，会去查看字符串常量池中是否有等于该对象的字符串，如果没有，就在常量池中新增该对象，并返回该对象引用；如果有，就返回常量池中的字符串引用。堆内存中原有的对象由于没有引用指向它，将会通过垃圾回收器回收


## 使用 String.intern节省内存
比如我做的项目中, 会存到大量的重读字符串, 比如`countryCode`, `currency`, `address`, 所以就可以使用`String.intern()`


## String.intern滥用会有坑

下面来看一个例子:
```java
    // -XX:+PrintStringTableStatistics
    int size = 10000000;// 1000万
    long begin = System.currentTimeMillis();
    List<String> list = IntStream.rangeClosed(1, size)
            .mapToObj(i-> String.valueOf(i).intern())
            .collect(Collectors.toList());
    System.out.println( "exec time: " + (System.currentTimeMillis() - begin));
```

运行结果如下:
```
exec time: 30641

SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     16237 =    389688 bytes, avg  24.000
Number of literals      :     16237 =    729048 bytes, avg  44.900
Total footprint         :           =   1278824 bytes
Average bucket size     :     0.811
Variance of bucket size :     0.820
Std. dev. of bucket size:     0.906
Maximum bucket size     :         6
StringTable statistics:
Number of buckets       :     60013 =    480104 bytes, avg   8.000
Number of entries       :  10002435 = 240058440 bytes, avg  24.000
Number of literals      :  10002435 = 560144800 bytes, avg  56.001
Total footprint         :           = 800683344 bytes
Average bucket size     :   166.671
Variance of bucket size :    55.369
Std. dev. of bucket size:     7.441
Maximum bucket size     :       196
```

可以看到, size = 1000万下, 代码执行要30秒左右.其中`PrintStringTableStatistics`的jvm参数打印相关的`statistics`,可以看到，在`StringTable statistics:`,默认的String的bucket size是`60013`,但是却要存放一千多万的`entries`,这会有严重哈希冲突,导致性能问题。

然后再加上`-XX:StringTableSize=10000000`参数重跑,结果如下:
```
exec time: 7551

SymbolTable statistics:
Number of buckets       :     20011 =    160088 bytes, avg   8.000
Number of entries       :     16237 =    389688 bytes, avg  24.000
Number of literals      :     16237 =    729048 bytes, avg  44.900
Total footprint         :           =   1278824 bytes
Average bucket size     :     0.811
Variance of bucket size :     0.820
Std. dev. of bucket size:     0.906
Maximum bucket size     :         6
StringTable statistics:
Number of buckets       :  10000000 =  80000000 bytes, avg   8.000
Number of entries       :  10002433 = 240058392 bytes, avg  24.000
Number of literals      :  10002433 = 560144616 bytes, avg  56.001
Total footprint         :           = 880203008 bytes
Average bucket size     :     1.000
Variance of bucket size :     1.584
Std. dev. of bucket size:     1.258
Maximum bucket size     :         9
```
可以看到执行速度上来了。

## 总结
String.intern 别滥用,会有坑, 用的好就提高内存使用率和性能,用得不好就会反而性能下降,要具体问题具体分析.