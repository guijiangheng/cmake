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

CMakeLists.txt在不同的配置下为目标main设置不同的输出名。需要注意的是变量CMAKE_BUILD_TYPE默认是没有设置的，最后编译得到的结果和Debug配置相同，但是由于该变量没有设置，所以两条set_target_properties命令并不起作用，输出文件名还是main.out。如果想要设置生效务必设置CMAKE_BUILD_TYPE的值。有两种办法设置该变量的值，一种是通过命令set(CMAKE_BUILD_TYPE Release)，另一种是通过命令行cmake .. -DCMAKE_BUILD_TYPE=Release，这样输出文件名才会变成renderer_release。
