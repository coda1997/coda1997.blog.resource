---
title: Profiling Tools
date: 2017-11-30
categories: java
---



Here I give four different kinds of Java profiler tools.

- TPTP
- CodePro Profiler
- YourKit Profiler
- JProfiler

#### TPTP

TPTP ( Test and Performance Tools Platform) is a official plugin provided by eclipse. It provides testing, tracing, performance testing, performance analyzing and so on. Also TPTP is a extensive developing framework.

##### Pros

- Thread telemetry
- CPU snapshot contains:
  - The composition of the package
  - Method call relationship, represented as a tree
  - Hot list: to show method or package taking CPU time

##### Cons

- Memory snapshot: to provide data on memory allocation, which is much less than other tools
- Source code positioning


<!--more-->

#### CodePro Profiler

CodePro Profiler is a premier Java software testing tool for Eclipse developers who are concerned about improving software quality and reducing developments costs and schedules.

##### Pros

- Telemetry: CPU, memory, thread, loaded classes, garbage collection telemetries
- CPU snapshot contains:
  - The composition of the package
  - Method call relationship
  - Hot list
- Memory snapshot contains:
  - memory allocation of classes, including the number of objects, shallow size and retain size (Shallow size means the size of object itself except reference, which is contained by retain size)
  - Lists of objects, which result in memory leak probably

##### Cons

- Source code positioning: only to position classes and member variables, can not position methods
- Performance: It may performance a bit bad when the program is a large scale and taking BCI model. What's more, it's quite slow to get memory leak lists.
- When profiling huge application, stack overflowing occurs very easily.

#### YourKit Java Profiler

YourKit is a tool for profiling Java and .net applications. With  YourKit solutions, both CPU and memory profiling have come to the highest professional level, where one can profile even huge applications with maximum productivity and zero overhead.

##### Pros

- YourKit is not a plugin but a application we can use without eclipse.
- CPU snapshot contains:
  - The composition of the package
  - Method call relationship
  - Hot list
- Source code positioning: to position all of classes and member variables, and methods
- Performance: In BCI model, it performs not bad, a little bit slower than native codes.

##### Cons

- Memory snapshot: compared with CodePro, it leaks checking objects resulting in memory leaking.

#### JProfiler

JProfiler is a commercially licensed Java profiling tool developed by ej-technologies GmbH, targeted at Java EE and Java SE application. It supports local profiling and remote profiling and enables both memory profiling to assess memory usage and dynamic allocation leaks and CPU profiling to assess thread conflicts.

##### Pros

- Memory snapshot contains:
  - memory allocation of classes, including the number of objects, shallow size.
- High performance: almost the same with native codes

##### Cons

- CPU snapshot: compared with CodePro, it doesn't have a method allocation tree. It just show by the thread.
- Source code positioning: only to position classes and member variables, can not position methods

#### Conclusion

- TPTP is a plugin base on eclipse, providing some sample functions, which is suitable for lightweight applications. 
- CodePro provides more function than TPTP. It can be used for applications which is insensitive to performance developed on eclipse because it affect the performance of applications.
- YourKit Java Profiler and JProfiler provides various functions and have a little performance cost. It would be used for applications which request high performance.



