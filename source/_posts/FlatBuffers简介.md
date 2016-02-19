---
title: FlatBuffers简介
date: 2016-02-19 20:20:42
categories: 
	- 技术
	- FlatBuffers
thumbnail: http://7xiovs.com1.z0.glb.clouddn.com/DPP_00032.JPG
tags: 
	- FlatBuffers
	- 序列化
---

## FlatBuffers简介

代码：https://github.com/google/flatbuffers/

文档：http://google.github.io/flatbuffers/


FlatBuffers是一个开源的、跨平台的、高效的、提供了C++/Java接口的序列化工具库。它是Google专门为游戏开发或其他性能敏感的应用程序需求而创建。 允许在不解析和解包就可以直接访问序列化数据，而且仍然很好地向上和向下兼容，这意味着序列化对象可以多版本共存。



### 支持的操作系统

* Android
* Windows
* MacOS X
* Linux


### 目前支持的编程语言

* C++
* C#
* Go
* Java
* JavaScript
* PHP
* Python

and many more in progress...

## 为什么要用FlatBuffers?

+ **对序列化数据的访问不需要打包和拆包**——它将序列化数据存储在缓存中，这些数据既可以存储在文件中，又可以通过网络原样传输，而没有任何解析开销；
+ **内存效率和速度**——访问数据时的唯一内存需求就是缓冲区，不需要额外的内存分配。 这里可查看详细的基准测试；
+ **扩展性、灵活性【多版本兼容】**——它支持的可选字段意味着不仅能获得很好的前向/后向兼容性（对于长生命周期的游戏来说尤其重要，因为不需要每个新版本都更新所有数据）；
+ **最小代码依赖**——仅仅需要自动生成的少量代码和一个单一的头文件依赖，很容易集成到现有系统中。再次，看基准部分细节；
+ **强类型设计**——尽可能使错误出现在编译期，而不是等到运行期才手动检查和修正；
+ **使用简单**——生成的C++代码提供了简单的访问和构造接口；而且如果需要，通过一个可选功能可以用来在运行时高效解析Schema和类JSON格式的文本；
+ **跨平台**——支持C++11、Java，而不需要任何依赖库；在最新的gcc、clang、vs2010等编译器上工作良好；

 
### 为什么不使用Protocol Buffers的，或者JSON

Protocol Buffers的确和FlatBuffers比较类似，但其主要区别在于FlatBuffers在访问数据前不需要解析/拆包这一步。 而且Protocol Buffers既没有可选的文本导入/导出功能，也没有Schemas语法特性（比如union）。同时，在工程中使用时，FlatBuffers的引用比Protocol Buffers方便很多，只需要包含两三个头文件即可。

JSON的可读性很好，而且当和动态类型语言（如JavaScript）一起使用时非常方便。然而在静态类型语言中序列化数据时，JSON不但具有运行效率低的明显缺点，而且会让你写更多的代码来访问数据（这个与直觉相反）。

与Protocol Buffers或JSON Parsing这样的可选方案相比，FlatBuffers的优势在于开销更小，这主要是由于它没有解析过程。http://google.github.io/flatbuffers/md__benchmarks.html



 
 