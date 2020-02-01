# Useful tools collection

## Clang, LLVM, GCC:
- [GCC，LLVM，Clang编译器对比](https://www.cnblogs.com/qoakzmxncb/archive/2013/04/18/3029105.html)

## AddressSanitizer
- [Official Github: https://github.com/google/sanitizers](https://github.com/google/sanitizers)
- [Sanitizers-cmake](https://github.com/arsenm/sanitizers-cmake)

**AddressSanitizer** (aka ASan) is a memory error detector (memory addressability) for C/C++. There is also LeakSanitizer to detect memory leak.

## GLog
- [使用 Google 的 glog 日志库
](http://senlinzhan.github.io/2017/10/07/glog/)
- https://blog.csdn.net/handsome_for_kill/article/details/69808446

GLog是Google开发的一款很好的LOG日志工具。用法上面的链接中基本都有了。补充一点，默认的Log文件的路径是`/tmp`。

## gflags
- [https://github.com/gflags/gflags](https://github.com/gflags/gflags)

The **gflags** package contains a C++ library that implements commandline flags processing. It includes built-in support for standard types such as string and the ability to define flags in the source file in which they are used. 

## gtest
- [https://github.com/google/googletest](https://github.com/google/googletest)
- [Doc: https://github.com/google/googletest/blob/master/googletest/docs/primer.md](https://github.com/google/googletest/blob/master/googletest/docs/primer.md)

**googletest** is a testing framework developed by the Testing Technology team with Google's specific requirements and constraints in mind. No matter whether you work on Linux, Windows, or a Mac, if you write C++ code, googletest can help you. And it supports any kind of tests, not just unit tests.

## anaconda usage

### Ref:
- [Official document about managing environment](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)

### `conda list`
List all packages installed in the environment

### `conda info --env`
List all existing conda environments

### `conda install <package>`
Install a package
