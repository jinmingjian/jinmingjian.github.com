---
layout: post
title: "Simple Revisit to 'Dart vs Java — the DeltaBlue Benchmark'"
tags: ["Dart", "Java", "benchmark"]
---

In these days, one benchmark article about "Dart vs Java" has pushed into the front page of Infoq [1], in which Dart attracts eyes again. As a performance addict and Javaer/Dartisan, I have a definite interesting to say something.

The first original benchmark result is posted by Nikolay Botev(a.k.a. bono8106)here [2]. To my surprise, Charles(a.k.a. headius), as a JRuby/JVM guy, does not complete to hit a homer:) The key here is, we still compare apples and oranges rather than author's "1-to-1 port" [2] [3].

I have done standalone porting [4] of Dart's DeltaBlue [5] based on the Nikolay's BenchmarkBase [6](I assume this measure class has no problem it self) in the day before yesterday.

The Nikolay's port indeed shows the speed of Java is ~12% slower than that of Dart the in my linux x64. As my little tweaking, the result has been reversed!


![results of Dart VM 0.1.2.0_r22600, linux x64](2013-05-15-simple-revisit-to-dart-vs-java-img/dartvm_linux64.png)

_ results of Dart VM 0.1.2.0_r22600 with linux x64 _

![results of OpenJDK 64-Bit Server VM 1.8.0-internal 25.0-b31](2013-05-15-simple-revisit-to-dart-vs-java-img/java8_linux64.png)

_ results of Dart VM 0.1.2.0_r22600 with linux x64 _

![results of Dart VM 0.1.2.0_r22600, linux 32bit](2013-05-15-simple-revisit-to-dart-vs-java-img/dartvm_linux32.png)

_ 32bit Dart VM gets not very much improvement _

OK, here is my note for this: 

1. the best correpsonding of const Strengths in Dart is not the String array, but enum. See the comment-out part of MyDeltaBlue [7], e), it works good 

2. beside the test DeltaBlue codes, the measure logic of BenchmarkBase is itself also in "a grotesque style". Or say, it is wrong. 
  * firstly, the elapsed time done by a while statement is not accurate obviously. It just guarantees that the elapsed time is below the limit in the last step, and above in the current step. 
  * then, the accuracy of elapsed time become worst in that it introduces some calculations between the run of bench logic. 
  * lastly, the unrelated activity of object will interfere with the execution engine.
The good example is Google itself's caliper [8]. Fortunately, xxgreg [9] has pointed out this problem.

3. the most key here is, that Dart List is not equivalent to Java List. Or "Dart list access [] operator to Java List.get()" is not very right.
  * Firstly, Dart List is special. From the aspect of accessor, it support common array index operator "[]", which is same to used in Java array. ArrayList does many checks in every call, which makes the port to ArrayList bad, It is better to see the Dart List as Java Array. 
  * The problem is how about the other methods in the List which is not . My idea is to add a lightweight ArrayList implementation [7] to imitate the Dart List. There is room to improve it more, but it is just Ok to illustrate "apples to oranges".
  * Secondly, Dart List is not an interface and Dart does not support explicit interface any more [10]. But Java explicit supports the interface. It is a little important for trciky micro benchmark. Because the explicit invocation to interface method in Java will result a invokeinterface bytecode which is more expensive than common invokevirtual no matter more or less(about ~5% in this benchmark).

Please note that, the Dart List is native implemeted(if I am not wrong?), but in Java, we use a Java source implemented version. The Java language add a more layer of restriction/indirection to the performance that the native, although it is powered by the Hotspot JIT. So, I guess my poorman's lightweight ArrayList implementation is more OK.

4. the another things, as Charles pointed out [2], the benchmark generate many many garbages, which is heavy for GC. So GC configuration takes much effect with the result. 
The trick is that these garbages are small. We do not need large heap, which reversely decrease GC efficiency in that scanning large heap is expensive itself. However, very small heap is not good as well in that the unstoped running of GC is also expensive. Here, I fix(Xms+Xmx) 512M as the heap size. It seems the Java GC is still one wonderful goods in the world.

5. summary of the benchmark characteristics: array indexing, GC, JIT compilation.

6. do I still compare apples and oranges?:) The final comparsion could be done by checking the generated machine codes(hi, man, only assembly codes accepted!).

Finally, it is very glad to see the Dart now can challenge Java at least in some aspects. Personally, I have predicted that Dart will go mainstream in 2-3 year. For the geniuses in Google, is this time too conservative?

Lastly, it will be fun to show one of my hobby time Dart based work to the public,
![a logo made by cubes](2013-05-15-simple-revisit-to-dart-vs-java-img/cubee_logo.png)

_ a logo made by cubes _

Simple, but the background of it is a high performance 2.5D HTML5 game engine. It can process 5,000,000+ unit cubes(in its virtual 3D world)/s without hardware acceleration in the last year.

Easily improve it into a minecraft thing like this:
![2.5D "minecraft" blocks](2013-05-15-simple-revisit-to-dart-vs-java-img/engine_early.png)

_ "2.5D 'minecraft' blocks _


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