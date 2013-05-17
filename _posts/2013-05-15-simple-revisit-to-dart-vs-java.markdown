---
layout: post
title: "Simple Revisit to 'Dart vs Java — the DeltaBlue Benchmark'"
tags: ["Dart", "Java", "benchmark"]
---

## Origin
In these days, one benchmark article about "Dart vs Java" has pushed into the front page of Infoq [1], where Dart attracts eyes again. As a performance addict and Javaer/Dartisan, I have a definite interesting to say something.

The first original benchmark result is posted by Nikolay Botev(a.k.a. bono8106) here [2]. To my surprise, Charles(a.k.a. headius), as a JRuby/JVM guy, does not complete to hit a homer:) 

**The key here is, the author of benchmark still compares apples and oranges rather than "1-to-1 port" [2]-[3].**

I have done a standalone porting [4] of Dart's DeltaBlue [5] based on the Nikolay's BenchmarkBase [6]. (I assume this measure class has no problem itself).


## Interesting Reversing
The Nikolay's port indeed shows the speed of Java is ~12% slower than that of Dart the in my linux x64. With my little tweaking, the result has been reversed!

![results of Dart VM 0.1.2.0_r22600, linux x64](/assets/img/posts/simple_img/dartvm_linux64.png)

_**results of Dart VM 0.1.2.0_r22600 with linux x64**_

![results of OpenJDK 64-Bit Server VM 1.8.0-internal 25.0-b31](/assets/img/posts/simple_img/java8_linux64.png)

_**results of Dart VM 0.1.2.0_r22600 with linux x64**_

Here, it is easy to figure out that, **Java 8 is ~15% faster than Dart nightly in my DeltaBlue testing**! P.S.: the running environment is linux 3.9.2 x86-64 + i7 3720M(IVB). 

The 32bit Dart VM does not change the conclusion much, as following:

![results of Dart VM 0.1.2.0_r22600, linux 32bit](/assets/img/posts/simple_img/dartvm_linux32.png)

_**32bit Dart VM does not get very much improvement**_


## Simple Notes
OK, here goes: 

1. it is *better to use the enum type* for translating const Strengths of Dart, rather than the String array. Dartisans from Java always miss enum See the comment-out part of MyDeltaBlue [7], e), it works good even from the point of performance. 

2. beside the test DeltaBlue codes, **the measure logic of BenchmarkBase is itself** also in "a grotesque style". Or say, it is **wrong**. 
  * firstly, the elapsed time done by a while statement is not accurate obviously. It just guarantees that the elapsed time is below the limit in the last step, and above in the current step. 
  * then, the accuracy of elapsed time become worst in that it introduces some calculations between the run of bench logic. 
  * lastly, the unrelated activity of object will interfere with the execution engine.
The good example is Google itself's caliper [8]. Fortunately, xxgreg [9] has pointed out this problem.

3. **the most key here is, that Dart List is not equivalent to Java List**. Or "Dart list access [] operator to Java List.get()" is not very right.
  * firstly, Dart List is special. From the aspect of accessor, it support common array index operator "[]", which is same to used in Java array. ArrayList does many checks in every call, which makes the port to ArrayList bad, It is better to see the Dart List as Java Array. 
  * the problem is how about the other methods in the List which is not . My idea is to add a lightweight ArrayList implementation [7] to imitate the Dart List. There is room to improve it more, but it is just Ok to illustrate "apples to oranges".
  * lastly, Dart List is not an interface and Dart does not support explicit interface any more [10]. But Java explicit supports the interface. It is a little important for trciky micro benchmark. Because the explicit invocation to interface method in Java will result a invokeinterface bytecode which is more expensive than common invokevirtual no matter more or less(about ~5% in this benchmark).

Please note that, the Dart List is native implemented(if I am not wrong?). But in Java, we use a Java source implemented version. The Java language adds a more layer of restriction/indirection to the performance than the native, although it is still powered by the Hotspot JIT. So, I guess my poorman's lightweight ArrayList implementation is more practical.

4. the another things, as Charles pointed out [2], the benchmark generates many many garbages, which is heavy for GC. So **GC configuration much affects the result**. 
The trick is that these garbages are small. We do not need large heap, which reversely decrease GC efficiency in that scanning large heap is expensive itself. However, very small heap is not good as well in that the unstoped running of GC is also expensive. Here, I fix(Xms+Xmx) 512M as the heap size. It seems the Java GC is still one wonderful goods in the world.

5. summary of the benchmark characteristics: array indexing, GC, JIT compilation.

6. do I still compare apples and oranges?:) Please note, **the Hotspot JVM is highly tunable** in all aspects. The Dart VM shows much less in this currently. We could check the generated machine codes for the final wisdom: Not sure whether it is easy for Dart VM, but simply for JVM.


## Blossom In Sight
Finally, it is very glad to see the Dart now can challenge Java at least in some aspects. I have personally predicted that **Dart will go mainstream in 2-3 year**. For all the geniuses in Google, is this time too conservative?

Lastly, it will be fun to show one of my hobby time Dart based work to the public,
![a logo made by cubes](/assets/img/posts/simple_img/cubee_logo.png)

_**a logo made by cubes**_

**Simple**, but the background of it is a **high performance** 2.5D HTML5 game engine. It can process 5,000,000+ unit cubes(in its virtual 3D world)/s without hardware acceleration in the last year.

Easily improve it into a minecraft thing like this:
![2.5D "minecraft" blocks](/assets/img/posts/simple_img/engine_early.png)

_**"2.5D 'minecraft' blocks**_

I hope to open source it with my mysterious backend in this year:) If you like to work with this kind of project or sponsor it, let me know!:)

_**Updated:**_

as Vyacheslav suggested, I re-benchmark both. there is indeed a slightly improvement form the timing result, especially for ia32. However, the basic result is still hold. 

![using the latest Dart SDK nightly](/assets/img/posts/simple_img/latest_benchmark_snapshot.png)

Nextl... I assume Vyacheslav is the export of Dart VM. As Vyacheslav read this blog, I plan to do the following tweakings: 
1. modify the timing to measuring the direct loop;(we do not use the threshhold value way)
2. revert the invokevirtual to invokeinterface by change the atual type from Array back to List.
3. compare the assembly codes.

thanks, all! 


[1]: http://www.infoq.com/news/2013/05/Dart-Java-DeltaBlue 
[2]: http://bonovox.be/blog/?p=128
[3]: http://www.reddit.com/r/programming/comments/1e2jhr/dart_vs_java_the_deltablue_benchmark/
[4]: https://github.com/jinmingjian/benchmark_harness_java 
[5]: https://github.com/dart-lang/benchmark_harness/
[6]: https://github.com/bono8106/benchmark_harness_java
[7]: https://github.com/jinmingjian/benchmark_harness_java/blob/master/deltablue/src/example/MyDeltaBlue.java
[8]: http://code.google.com/p/caliper
[9]: https://github.com/xxgreg/deltablue
[10]: http://www.dartlang.org/articles/m1-language-changes/#no-explicit-interfaces