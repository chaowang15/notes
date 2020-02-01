# CMake学习笔记

## 使用find_package查找链接库

- [CMake如何查找链接库---find_package的使用方法
](https://blog.csdn.net/u011092188/article/details/61425924)
- https://blog.csdn.net/bytxl/article/details/50637277
- [cmake教程4(find_package使用)
](https://blog.csdn.net/haluoluo211/article/details/80559341)
- [find_package的CMake官方文档，包含了详细的寻找方法](https://cmake.org/cmake/help/latest/command/find_package.html)
- [Find Modules的CMake官方文档，包含了FindLib.cmake的例子](https://cmake.org/cmake/help/latest/manual/cmake-developer.7.html#find-modules)

如果不知道你要用的链接库的位置，可以借助CMake的`find_package`命令来查找。

### 查找库文件的方式

`find_package(<LIBRARY_NAME>)`命令会按照以下的先后次序进行：

1. **首先寻找的是`Find<LIBRARY_NAME>.cmake`文件**。这种.cmake文件又被称为**Module mode**，即模板模式。该文件中定义了查找链接库的方法细节。其实，该文件内依然是CMake的语法命令集合。即，完全可以将它的内容全部移到CMakeLists.txt中。为了简便，CMake才定义了这种模块化文件方便查找。查找的顺序是：
- 首先会在模块路径`${CMAKE_MODULE_PATH}`的所有目录中寻找。不过，该变量**默认是空**。因此，如果你有自行定义的FindXXX.cmake文件的话，可以在CMakeLists.txt中直接设置该文件所在路径：
```shell
set(CMAKE_MODULE_PATH <your_FindLibrary_cmake_file_dir>)
find_package(library)
```
- 如果上面路径中没找到，再查看CMake自己安装的模块目录`<CMAKE_ROOT>/share/cmake-x.y/Modules/`，这里的`cmake-x.y`对应是你的CMake的安装版本，例如`cmake-3.12`。也就是说，你可以懒省事的将自己的FindXXX.cmake文件放到这个路径下，这样将来就不用每次都要像上面那样设置了。不过，这样其实**并不推荐**，因为通常CMake的Modules文件夹是私密的，而如果将你自己的Find文件放入的话会让该路径暴露在你的工程之下，这样并不好。因此，依然是推荐使用上面的设置CMAKE_MODULE_PATH路径的方法。

2. 如果找不到`Find<LIBRARY_NAME>.cmake`文件，那么接着会**查找`<LIBRARY_NAME>Config.cmake`或`<LIBRARY_NAME>-Config.cmake`文件**（这里的`LIBRARY_NAME`要么是全大写，要么是全小写）。这种.cmake文件被称为是**Config(uration) Mode**。查找路径有很多（详细参见上面给出的官方文档链接），都是在CMake定义的一些prefixes中查找，其中常见的有`~/.cmake/packages/`，`/usr/local/share/`，`/usr/local/lib/`等多个CMake的常见目录。例如Opencv的话，CMake会查找`/usr/local/share/OpenCV`中的`OpenCVConfig.cmake`或`opencv-config.cmake`。

不管找到了哪一种.cmake文件，它内部通常都会定义下面这些变量：

```shell
<NAME>_FOUND # 一个flag，标明该链接库已找到
<NAME>_INCLUDE_DIR 或 <NAME>_INCLUDE_DIRS 或 <NAME>_INCLUDES # 头文件所在路径文件夹
<NAME>_LIBRARIES 或 <NAME>_LIBRARIES 或 <NAME>_LIBS # 库文件(.so或.a)所在的路径全名称（即，包含库名称的全路径）
<NAME>_DEFINITIONS # 这个变量其实很少会定义。它包含用于编译的全部的FLAGS。具体参见本文后面的“传递FLAGS给编译器”的内容
```

#### 注意
- 通常情况下，对于一个链接库，只要它在安装时对应的FindLibrary.cmake文件也被放到了CMake的Modules目录下的话，那么上面的这几个变量都会被定义（通常都是一些比较大并且规范的库）。反之，如果FindLibrary.cmake文件存在，但却没有被放到放到了CMake的Modules目录下的话，那么这个Find文件就不是那么规范了，定义的变量名字也会有些奇怪。例如，G2O就是后者，它的cmake文件就没有定义G2O_LIBRARIES，而是分开定义了G2O_CORE_LIBRARY和G2O_STUFF_LIBRARY这两个库。
- 通常上面这些变量中的库的名称是**全大写**的，如 LIBFOO_FOUND，但还是有些包使用了库的**实际大小写**（注意不是全小写），如 LibFoo_FOUND。
- 使用`cmake --help-module-list`命令可以打印出CMake的Modules目录下的全部拥有FindLibrary.cmake文件的库的名字。不过，显然这并不是全部的安装库，因为有相当一部分库是只有Configure文件没有Find文件的。

### 使用外部链接库的方式

如果找到这个链接库，则可以通过在工程的顶层目录中的CMakeLists.txt 文件添加`include_directories(<LIBRARY_NAME>_INCLUDE_DIRS)`来包含库的头文件，并添加`target_link_libraries(Source_files <LIBRARY_NAME>_LIBRARIES)`命令将源文件与库文件链接起来。具体参加接下来这一章节。

为了能支持各种常见的库和包，CMake自带了很多模块。可以通过命令 cmake --help-module-list 得到你的CMake支持的模块的列表，或者直接查看模块路径。比如Ubuntu上，模块的路径是 /usr/share/cmake/Modules/ 。

  让我们以bzip2库为例。CMake中有个FindBZip2.cmake 模块。只要使用 find_package(BZip2) 调用这个模块，cmake会自动给一些变量赋值，然后就可以在CMakelists.txt中使用它们了。变量的列表可以查看cmake模块文件，或者使用命令 cmake –help-module FindBZip2 。

比如一个使用bzip2的简单程序，编译器需要知道 bzlib.h 的位置，链接器需要找到bzip2库（动态链接的话，Unix上是 libbz2.so 类似的文件，Windows上是 libbz2.dll ）。

```shell
cmake_minimum_required(VERSION 2.8)
project(helloworld)
add_executable(helloworld hello.c)
# set(CMAKE_MODULE_PATH <your_find_cmake_file_path>) # 看情况使用
find_package(BZip2)
if (BZIP2_FOUND)
  include_directories(${BZIP_INCLUDE_DIRS})
  target_link_libraries (helloworld ${BZIP2_LIBRARIES})
endif (BZIP2_FOUND)
```

### 寻找特定版本的链接库
很简单，直接放在库文件名后面，这样的话只会找满足版本要求的。
```shell
find_package( OpenCV 3.1.0 REQUIRED )
```

## 设置建立的工程类型是Debug或者Release

显式设置`CMAKE_BUILD_TYPE`变量为Debug或者Release：
```shell
SET(CMAKE_BUILD_TYPE Debug)
SET(CMAKE_BUILD_TYPE Release)
```

但是需要注意，上面的做法其实并不推荐，即，**不推荐直接在CMakeLists.txt中固定该变量**，因为这样会使得在外部建立工程时，无法更改类型了。通常推荐是在传入参数中修改它（always specify CMAKE_BUILD_TYPE explicitly on the command line）：
```shell
cmake -DCMAKE_BUILD_TYPE=Release <path_to_CMakeLists>
```

## 设置option
可以使用option来设置某个Flag的默认值，好处是可以在命令行设置这个Flag的初始值。很多链接库都在使用这种方法，例如打开或者关闭某一个flag等。

```shell
option(DEBUG_mode "ON for debug or OFF for release" OFF)
IF(DEBUG_mode)
add_definitions(-DDEBUG)
ENDIF()
```
上面默认关闭了debug模式。编译时，可以使用`cmake -DDEBUG_mode=ON ..`来打开它。

## 设置编译器

```shell
set(CMAKE_CXX_COMPILER "g++" ) # 或者"clang++"等
```
每个系统有默认的编译器类型。

## 传递FLAGS给编译器

和FLAGS设置相关：
- [中文的一个有关cmake流程的简单介绍](https://elloop.github.io/tools/2016-04-10/learning-cmake-2-commands)
- [gcc中有关Optimization Level -O1, O2, O3等的官方说明](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)
- [知乎上的有关Optimization Level -O1, O2, O3等的中文解释](https://www.zhihu.com/question/27090458)

GCC中和编译器相关的FLAGS参数有很多，可以参见：
- [GCC官方英文文档](https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html)
- [GCC官方文档：警告部分](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [15个最常用的GCC编译器参数](https://colobu.com/2018/08/28/15-Most-Frequently-Used-GCC-Compiler-Command-Line-Options/#%E9%80%9A%E8%BF%87-Wall%E5%8F%82%E6%95%B0%E5%90%AF%E7%94%A8%E6%89%80%E6%9C%89%E8%AD%A6%E5%91%8A)
- [gcc编译选项总结](https://blog.csdn.net/gatieme/article/details/21389603)

### 常见的编译器的FLAGS
- -std=c++11: 又或者是14,17等，设置C++的版本支持
- -g: 生成调试信息，使得GNU等调试器（例如gdb）可利用该信息
- -Wall: 启用所有警告信息；
- -w: 不生成任何警告；
- -Werror: 将所有警告转化为错误Error输出；
- -Wextra: 比-Wall增加了更多的警告判断，例如：将一个pointer <= 一个int；派生类和基类之间的一些不合理的设置等。
- -O0: 不进行任何优化处理；
- -O或-O1: 优化生成代码； 
- -O2: 进一步优化；
- -O3: 比-O2更进一步优化，包括 inline 函数。虽然编译速度会慢一点，但可执行程序运行速度会加快；

### 在CMakeLists.txt内添加Flags给编译器

https://blog.csdn.net/10km/article/details/51731959

1. 对于C++，设置`CMAKE_CXX_FLAGS`变量。对于C，设置`CMAKE_C_FLAGS`变量。
```shell
set(CMAKE_CXX_FLAGS   "-std=c++11"             # 设置c++11支持
                      "-g"                     # 调试信息
                      "-Wall")                  # 开启所有警告
# 如果要分行，可以用list命令将新内容链到变量已有内容的后面
set(CMAKE_CXX_FLAGS "-std=c++11")
list(APPEND CMAKE_CXX_FLAGS "-g")
list(APPEND CMAKE_CXX_FLAGS "-Wall")
```

其中，`CMAKE_CXX_FLAGS`是同时对Debug和Release模式设置相同的Flags。如果想要区别对待，可以用
```shell
set(CMAKE_CXX_FLAGS_DEBUG   "-O0" )             # 调试包不优化
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG " )   # 仅release包优化
```
不过注意，`CMAKE_CXX_FLAGS_DEBUG`是除了`CMAKE_CXX_FLAGS`外，在Debug配置下的额外参数。同样的，`CMAKE_CXX_FLAGS_RELEASE`也是除了`CMAKE_CXX_FLAGS`外，在Release配置下的额外参数。

2. 另一种方法是，使用`ADD_DEFINITIONS()`添加编译参数：
```shell
add_definitions(“-Wall -ansi –pedantic –g”)
add_definitions(LIBRARY_DEFINITIONS)
```
该命令的优点是，很多库已经在`find_package()`过程中就定义了编译参数到一个变量中，例如PCL库中的PCL_DEFINITIONS等，此时只要像上面这样一次性全部加进去就行了。不过注意，该命令会增加FLAGS到**所有的编译器**。因此有时候会很奇怪，例如增加`-std=c++11`到C编译器，编译时会出现Warning，虽然不会出错，但是不太好看。

3. 还有一种方法是`add_compile_options()`命令，用法和上面的`add_definitions`几乎完全一致。并且，它也是对所有编译器增加FLAGS。


### 添加Custom编译方式/在CMake命令行添加Flags给编译器

- https://stackoverflow.com/questions/44284275/passing-compiler-options-cmake
- https://stackoverflow.com/questions/11437692/how-to-add-a-custom-build-type-to-cmake-targeting-make

上面方法是将FLAGS固定在CMakeLists.txt中了。那么如果我们想要在命令行添加某个FLAG将如何？就像类似区分Debug和Release模式一样。例如，为了启动内存检测工具AddressSanitizer，需要添加-fsanitizer=address这个FLAG给编译器。而使用该工具的话会增加可执行程序的运行时间，因此我们并不希望默认使用它。那么如何将其在命令行中选择性加入？

首先，使用类似`-DCMAKE_CXX_FLAGS=<your_flag>`的方法的话是错误的，因为这会直接覆盖CMakeLists中的CMAKE_CXX_FLAGS已经定义的内容。我们想要将输入的FLAG用append的方式附在已有的FLAGS的后面。

上面的链接给出了一种方法：
```shell
 cmake -DCMAKE_CXX_FLAGS_YOUR_NAME:STRING="-fsanitizer=address" -DCMAKE_BUILD_TYPE=YOUR_NAME ..
```
这种做法的核心其实是：设置一个custom build type。设置的方法是：
- 定义一个CMAKE_CXX_FLAGS_YOUR_NAME变量，将一个自定义的YOUR_NAME符号附在CMAKE_CXX_FLAGS后面，然后`::STRING`后面是你要添加的FLAG名称，用引号包括住。这就是你默认的方法了。
- 然后使用-DCMAKE_BUILD_TYPE=YOUR_NAME启用自己定义的build type。

CMake会将CMAKE_CXX_FLAGS_YOUR_NAME的内容自动附在CMAKE_CXX_FLAGS后面。不过，这样的话似乎就无法设置CMAKE_BUILD_TYPE为Debug或者Release了。因此，这种方法其实也不是太好。有些库文件或者工具，例如Sanitizers，会在FindXXX.cmake文件中定义不同的option，用于使用不同的工具，这样更安全一些，更推荐使用。

## 找到路径中所有的cpp文件（虽然不推荐）

找到路径中全部cpp文件并设置到一个变量中：
```shell
file(GLOB helloworld_SRC
    "src/*.cpp"
    # "*.h" 
)
add_executable(helloworld ${helloworld_SRC})
```
如果要寻找更多类型，就像上面例子中的注释一样，直接加到`file()`中即可。

但是注意，其实上面这种做法**并不推荐**。这是因为，如果你的CMakeLists文件并没有改变，但是工程中新增或删除了cpp文件的话，CMake却不知道这些，它还认为你的源文件是不变的（因为它显式的生成了cpp list到MakeFile中），因此在编译时候会出错。所以，最好的方法还是一个一个将你的cpp文件显式的写到CMakeLists中，尤其是对于较大的工程，更是要这样。

https://stackoverflow.com/questions/3201154/automatically-add-all-files-in-a-folder-to-a-target-using-cmake

We do not recommend using GLOB to collect a list of source files from your source tree. If no CMakeLists.txt file changes when a source is added or removed then the generated build system cannot know when to ask CMake to regenerate.

## 各种工具的使用

### Sanitizers

- [Official Github link：其wiki页面中包括如何使用](https://github.com/google/sanitizers)
- [Sanitizers-cmake usage in Github](https://github.com/arsenm/sanitizers-cmake)

Sanitizers是一系列很好用的、C/C++的、和内存相关的检测工具。常见的几个工具是：
- **AddressSanitizer**：检测内存错误，例如释放已经释放的内存、再次access已经释放后的内存等；
- **LeakSanitizer**：检测内存泄露，例如未释放的内存；
- **MemorySanitizer**：检测未初始化的内存，例如stack或者heap等在写入之前就使用。

Sanitizers已经默认加到了gcc和LLVM中，因此无需额外安装。

若要使用AddressSanitizer，添加`-fsanitize=address`这个Flag给编译器就行了：
```shell
clang/gcc/g++ -fsanitize=address ...
```
然后在运行可执行文件，之后就会显示内存出错的地方。不过，可执行程序要比普通的没有Sanitizers时候要慢一点。

若要使用LeakSanitizer，添加`-fsanitize=leak`给编译器。

#### 在CMake中选择性决定是否使用Sanitizer

如果是确定使用Sanitizer的话，在CMakeLists.txt中将`-fsanitize=address`添加给CMAKE_C_FLAGS或者CMAKE_CXX_FLAGS就行了。不过，如果想要在cmake中将是否使用Sanitizers设为optional的话，这样就不行了。两种解决方法：
- 使用FindSanitizers.cmake文件。可以从本章节开头给出的”Sanitizers-cmake usage in Github“链接中下载。该链接同时给出了详细的在cmake中的使用方法。使用时，记得拷贝整个cmake_Modules文件夹中的全部Find文件。
- 既然是只需要增加一个`-fsanitize=XXX`，不妨采用上面"**添加FLAGS到编译器**"章节中所述的方法。