## CMake语言
CMake文件是由CMake语言编写的，文件名为CMakeLists.txt或者后缀为.camke的文件。CMake源文件分为三类：
* 目录(CMakeLists.txt)
* 脚本(`<script>.cmake`)
* 模块(`<module>.cmake`)

单个的CMake脚本可以使用-P命令行选项，在CMake脚本模式下运行。脚本模式下只是运行源文件的中的命令，而不允许定义编译目标。下面的例子中，名为test.cmake的脚本包含以下内容：
``` cmake
# test.cmake
message(hello world)
add_executable(main main.cpp)
```
使用-P运行的截图如下所示：<br>
![script mode](./images/scriptmode.png)<br>

### Quoted Argument
CMake源文件由一条条CMake命令组成，命令的参数分为Quoted Argument和Unquoted Argument。Quoted Argument的内容被一对双引号括起来，转义字符和变量引用都会被替换，Quoted Argument被当成单独的一个参数。下面是一个引号参数的例子：
``` cmake
# test.cmake
set(variable hello world)
message(${variable})
message("${variable}")
message("This is a quoted argument containing multiple lines.
This is always one argument even though it contains a ; character.
Both \\-escape sequences and ${variable} references are evaluated.
The text does not end on an escaped double-quote like \".
It does end in an unescaped double quote.
")
```
运行的结果如下图：<br>
![quoted argument](./images/quotedarg.png)<br>

## 3.2 目标(Targets)

CMake中最重要的就是目标。目标表示可执行文件，库以及一些由CMake生成的工具。每个add_library，add_executable和add_custom_target命令都会产生一个目标。例如下面的语句将产生一个foo静态库。

``` cmake
add_library(foo STATIC foo.cpp)
```

STATIC表明这个库必须被编译成静态库，相应的SHARED表明该库必须被编译成动态链接库。如果add_library命令没有给出STATIC或者SHARED参数，则CMake将根据变量BUILD_SHARED_LIBS的值将库编译成静态库或动态库。没有设置该变量时，默认生成静态库。如下面的例子所示：

``` cmake
# CMakeLists.txt
cmake_minimum_required(VERSION)
project(foo)

set(BUILD_SHARED_LIBS ON)
add_library(foo foo.cpp)
```

```c++
// foo.cpp
void foo(int a, int b, int& c) {
    c = a + b;
}
```

上面的项目编译后得到libfoo.so文件，注意set(BUILD_SHARED_LIBS ON)语句的位置，如果该语句在add_library命令之后，则仍将编译得到静态库。

还可以对目标使用get_target_property和set_target_properties命令，或者更一般的get_property和set_property命令读取或者设置目标的属性。get_target_property命令的语法如下：

```cmake
get_target_property(VAR target property)
```
该命令获取目标的属性存储在变量VAR中，如果没有找到属性值，VAR的值为NOTFOUND。该命令能够从任何已创建的目标中获取属性值，不仅仅是当前的CMakeLists.txt文件中的目标。set_target_properties命令的语法如下所示：

```cmake
set_target_properties(
    target1 target2 ...
    PROPERTIES
    prop1 value1
    prop2 value2
    ...
)
```
可以设置目标的OUTPUT_NAME属性来设置目标的文件名，CMake要求逻辑上的目标名唯一，可以通过这种方法让两个目标的文件名相同（不含后缀）。还可以设置`<CONFIG>_OUTPUT_NAME`属性为不同的配置设置不同的文件名，比如DEBUG, RELEASE, MINSIZEREL, RELWITHDEBINFO配置。下面是两个例子。

### 3.2.1 get_property例子

``` cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(foo)

add_subdirectory(foo)
get_target_property(fooname foo OUTPUT_NAME)
message(${fooname})

add_executable(main main.cpp)
target_link_libraries(main foo)
```

``` cmake
# foo/CMakeLists.txt
add_library(foo foo.cpp)
set_target_properties(foo PROPERTIES OUTPUT_NAME bar)
```

foo/CMakeLists.txt添加了foo静态库，并未foo库设置了输出名bar，运行该配置会CMake打印信息bar，并在foo文件夹下生成libbar.a文件。如果将add_subdirectory命令放在message命令之后，则CMake会给出错误信息：<br>
`get_target_property() called with non-existent target "foo".`<br>
这是因为执行查询时，foo目标还没有被添加到项目中。还有一点需要注意的是，如果没有为foo目标设置输出名，则CMake将会输出fooname-NOTFOUND，不要想当然的认为没有设置OUTPUT_NAME属性则查询的结果是foo。

### 3.2.1 set_target_properties例子

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(foo)

add_subdirectory(foo)

add_executable(main main.cpp)
target_link_libraries(main foo)
set_target_properties(
    main
    PROPERTIES
    RELEASE_OUTPUT_NAME renderer_release
)
set_target_properties(
    main
    PROPERTIES
    DEBUG_OUTPUT_NAME renderer_debug
)

```

``` cmake
add_library(foo foo.cpp)
```

CMakeLists.txt在不同的配置下为目标main设置不同的输出名。需要注意的是变量CMAKE_BUILD_TYPE默认是没有设置的，虽然最后编译得到的结果和Debug配置相同，但是由于该变量没有设置，所以两条set_target_properties命令并不起作用，输出文件名还是main.out。如果想要设置生效务必设置CMAKE_BUILD_TYPE的值。有两种办法设置该变量的值，一种是通过命令set(CMAKE_BUILD_TYPE Release)，另一种是通过命令行cmake .. -DCMAKE_BUILD_TYPE=Release，这样输出文件名才会变成renderer_release。

## 3.5 变量和Cache

CMake变量的作用于和其他语言不同。变量对当前的CMakeLists文件，函数，子目录的CMakeLists文件，任何通过include命令引入的文件都有效。然而当CMake处理子目录和函数时会使用当前的变量初始化一个新的环境。因此任何在子环境中对变量所做的修改都不会影响父目录。如果想要修改父目录的变量，可以为set命令加上PARENT_SCOPE参数。修改父环境的变量值不会影响自环境的变量，如下面的代码所示：
```cmake
set(name gui)
set(name jiangheng PARENT_SCOPE)
message(STATUS ${name})
```
这段代码输出gui。

有些时候希望用户能够设置一些值，这些值只能存储在cache条目中，用户通过交互式命令行或者图形界面对这些cache条目进行设置。可以通过set命令设置cache变量，语法如下：
```cmake
set(VAR ON CACHE BOOL "doc str")
```
可选的变量类型包括：BOOL，PATH，FILEPATH和STRING。有些时候希望限制cache变量的取值，可以通过设置cache变量的STRINGS属性实现，例如：
``` cmake
set(CRYPTOBACKEND "OpenSSL" CACHE STRING "select a cryptography backend")
set_property(CACHE CRYPTOBACKEND PROPERTY STRINGS "OpenSSL" "LibTomCrypt" "LibDES")
```
这样CRYPTOBACKEND变量只能取OpenSSL，LibTomCrypt或LibDES了。如果变量存在cache中，仍然可以通过set命令设置变量的值。只有当环境中不存在变量，才会查找cache中的变量。因此set命令只是设置当前的环境而没有修改cache中的值。通常不希望CMakeLists文件对cache中的变量做修改，因为这些变量是由用户通过图形界面设置的。因此set(VAR ON CACHE BOOL "doc")只在cache中不存在变量时才起作用，否则没有任何效果。如果确实想要修改cache中的变量，需要加上FORCE选项。

# 第四章 编写CMakeLists文件

## if命令
if支持一下操作：<br>
if(VAR)：当变量不是空，0，FALSE，OFF或者NOTFOUND时为真<br>
if(NOT VAR)：取反<br>
if(VAR1 AND VAR2)：取与<br>
if(VAR1 OR VAR2)：取或<br>
if(COMMAND command-name)：如果给定的参数是命令则返回真<br>
if(DEFINED VAR)：如果变量存在，不论变量的值是什么，返回真<br>
if(EXISTS path)：如果文件名或者路径名存在返回真<br>
if(IS_DIRECTORY name)：如果参数是路径名返回真<br>
if(IS_ABSOLUTE name)：如果参数是绝对路径返回真<br>
if(FILE1 IS_NEWER_THAN FILE2)：如果FILE1比FILE2更新，返回真<br>
if(VAR MATCHES regex)：如果变量匹配正则表达式，返回真，注意直接使用变量名，而不是${VAR}<br>

## 优先级
括号的优先级最高，然后是类似EXISTS，COMMAND，DEFINED等前缀，之后是EQUAL，LESS，GREATER，STREQUAL，STRLESS，STRGREATER和MATCHES等比较操作，再之后是NOT，最后是AND和OR。

## 真假值
OFF，0，NO，FALSE，N，NOTFOUND，\*-NOTFOUND IGNORE被认为是false<br>
ON，1，YES，TRUE，Y被认为是真<br>
注意不区分大小写，因此true，True和TURE都被认为是真<br>
