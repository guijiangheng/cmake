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

上面的项目编译后得到libfoo.so文件，注意`set(BUILD_SHARED_LIBS ON)`语句的位置，如果该语句在add_library命令之后，则仍将编译得到静态库。

还可以对目标使用`get_target_property`和`set_target_properties`命令，或者更一般的`get_property`和`set_property`命令读取或者设置目标的属性。`get_target_property`命令的语法如下：

```cmake
get_target_property(VAR target property)
```
该命令获取目标的属性存储在变量`VAR`中，如果没有找到属性值，`VAR`的值为`NOTFOUND`。该命令能够从任何已创建的目标中获取属性值，不仅仅是当前的CMakeLists.txt文件中的目标。`set_target_properties`命令的语法如下所示：

```cmake
set_target_properties(
    target1 target2 ...
    PROPERTIES
    prop1 value1
    prop2 value2
    ...
)
```
