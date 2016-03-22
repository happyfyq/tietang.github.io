---
title: FlatBuffers 使用指南
date: 2016-02-19 20:20:42
categories: 
	- 技术
	- FlatBuffers
thumbnail: http://7xiovs.com1.z0.glb.clouddn.com/IMG_20160218_122224.JPG
tags: 
	- FlatBuffers
	- 序列化
---

# FlatBuffers 使用指南

FlatBuffers序列化性能是protobuf的2倍，但size也是protobuf的2倍

## 编译源码

```
$ git clone https://github.com/google/flatbuffers.git
#切换到最新release版本
$ git checkout v1.2.0
```

### 安装cmake 

http://www.cmake.org.

```
for mac osx
$ brew install cmake
for centOS
$ sudo yum install cmake
```

### 用cmake构建project

```
cmake -G "Unix Makefiles"
cmake -G "Visual Studio 10"
cmake -G "Xcode"
```
在*nix系统，mac osx系统也建议使用 `cmake -G "Unix Makefiles"`，生成`Makefile`,之后`make & make install`
编译生成`flatc`并安装到系统。

```
$ cmake -G "Unix Makefiles"
$ make
$ make insall

```


## 使用schema编译器flatc来生成基础代码

```
$ cd samples
#在目录src中生成java代码
$flatc -j -o src monster.fbs 

```

### 编程语言参数:

+ --cpp, -c : Generate a C++ header for all definitions in this file (as filename_generated.h).
+ --java, -j : Generate Java code.
+ --csharp, -n : Generate C# code.
+ --go, -g : Generate Go code.
+ --python, -p: Generate Python code.
+ --javascript, -s: Generate JavaScript code.
+ --php: Generate PHP code.

### 其他常用选项：

+ -o PATH 指定源码输出目录
+ -I PATH 有include语句时，指定include目录


# 完整的参数

```
usage: flatc [OPTION]... FILE... [-- FILE...]
  -b              Generate wire format binaries for any data definitions.
  -t              Generate text output for any data definitions.
  -c              Generate C++ headers for tables/structs.
  -g              Generate Go files for tables/structs.
  -j              Generate Java classes for tables/structs.
  -s              Generate JavaScript code for tables/structs.
  -n              Generate C# classes for tables/structs.
  -p              Generate Python files for tables/structs.
  -o PATH         Prefix PATH to all generated files.
  -I PATH         Search for includes in the specified path.
  -M              Print make rules for generated files.
  --strict-json   Strict JSON: field names must be / will be quoted,
                  no trailing commas in tables/vectors.
  --defaults-json Output fields whose value is the default when
                  writing JSON
  --no-prefix     Don't prefix enum values with the enum type in C++.
  --scoped-enums  Use C++11 style scoped and strongly typed enums.
                  also implies --no-prefix.
  --gen-includes  (deprecated), this is the default behavior.
                  If the original behavior is required (no include
                  statements) use --no-includes.
  --no-includes   Don't generate include statements for included
                  schemas the generated file depends on (C++).
  --gen-mutable   Generate accessors that can mutate buffers in-place.
  --gen-onefile   Generate single output file for C#
  --raw-binary    Allow binaries without file_indentifier to be read.
                  This may crash flatc given a mismatched schema.
  --proto         Input is a .proto, translate to .fbs.
  --schema        Serialize schemas instead of JSON (use with -b)
FILEs may depend on declarations in earlier files.
FILEs after the -- must be binary flatbuffer format files.
Output files are named using the base file name of the input,
and written to the current directory or the path given by -o.
example: flatc -c -b schema1.fbs schema2.fbs data.json
```

## 写schema IDL文件

参考：http://google.github.io/flatbuffers/flatbuffers_guide_writing_schema.html
