# 指定cmake的版本
cmake_minimum_required(VERSION 3.1)

# 项目名称
project(hello VERSION 1.0)


# 设置c++编译器
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11" )  

# 1、指定库的目录变量
set(libhello_src src/hello.cxx)
# 指定头文件搜索路径
include_directories("${PROJECT_SOURCE_DIR}/include")

# 2、添加库(对应的两个项目)
add_library( hello_shared SHARED ${libhello_src})
add_library( hello_static STATIC ${libhello_src})
#  按照一般的习惯，静态库名字跟动态库名字应该是一致的，只是扩展名不同；
# 即：静态库名为 libhello.a； 动态库名为libhello.so ；
# 所以，希望 "hello_static" 在输出时，不是"hello_static"，而是以"hello"的名字显示，故设置如下
# SET_TARGET_PROPERTIES (hello_static PROPERTIES OUTPUT_NAME "hello")

# 3、cmake在构建一个新的target时，会尝试清理掉其他使用这个名字的库，
# 因此，在构建libhello.a时，就会清理掉libhello.so.
# 为了回避这个问题，比如再次使用SET_TARGET_PROPERTIES定义 CLEAN_DIRECT_OUTPUT属性。
SET_TARGET_PROPERTIES (hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES (hello_shared PROPERTIES CLEAN_DIRECT_OUTPUT 1)

# 4、按照规则，动态库是应该包含一个版本号的，
# VERSION指代动态库版本，SOVERSION指代API版本。
#linux的动态库的命名格式是libfooSdk.so.x.y.z
#版本号遵循一个规则:
#    z: 最后一个z版本的变动一定是兼容的。
#    y: y版本升级一般向前兼容。所以这个y和z不能写死。
#    x: x版本变动一般是不兼容升级

SET_TARGET_PROPERTIES (hello_static PROPERTIES VERSION 1.1 SOVERSION 1)
SET_TARGET_PROPERTIES (hello_shared PROPERTIES VERSION 1.1 SOVERSION 1)

# 5、若将libhello.a, libhello.so.x以及hello.h安装到系统目录，才能真正让其他人开发使用，
# 本例中，将hello的共享库安装到<prefix>/lib目录；
# 将hello.h安装<prefix>/include/hello目录。
#INSTALL (TARGETS hello hello_shared LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
#INSTALL (TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
#INSTALL (FILES hello.h DESTINATION include/hello)


# ------------------------------------------------------------ #
# 设置VS会自动新建Debug和Release文件夹
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/Bin)

# 设置分别设置Debug和Release输出目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_CURRENT_SOURCE_DIR}/../../build/Debug)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/Lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_CURRENT_SOURCE_DIR}/Bin)
# ------------------------------------------------------------ #

# 设置编译器编译模式,以及设置变量：
SET(CMAKE_BUILD_TYPE "Debug" )
set(CMAKE_BUILD_TYPE "Release")
SET(HELLO_INCLUE /home/fan/dev/cmake/4-exer/)
SET(HELLO_SO /home/fan/dev/cmake/4-exer/build/libcalculate_shared.so)
INCLUDE_DIRECTORIES(${HELLO_INCLUE})
add_executable(main main.cpp)
target_link_libraries(main ${HELLO_SO})

${PROJECT_SOURCE_DIR} 这个变量是主Cmake文件的变量
${PROJECT_BINARY_DIR}这个变量是cmake -B生成的路径的变量
${CMAE_CURRENT_SOURCE_DIR}

# ------------------------------------------------------------ #

# 生成动态库
#用${SRC_LISTS}指定的所有的源文件生成一个库，名字叫libsugan
add_library(libsugan ${SRC_LISTS})    
#生成libsugan库需要链接 ${OpenCV_LIBS}、 ${PROJECT_SOURCE_DIR}/lib/libCommonUtilities.so、${PROJECT_SOURCE_DIR}/lib/libInuStreams.so

target_link_libraries(libsugan                 
    ${OpenCV_LIBS}
    ${PROJECT_SOURCE_DIR}/lib/libCommonUtilities.so
    ${PROJECT_SOURCE_DIR}/lib/libInuStreams.so
)
# ------------------------------------------------------------ #

# target_link_libraries 中的PRIVATE, PUBLIC, INTERFACE 区别
PUBLIC    在public后面的库会被Link到你的target中，并且里面的符号也会被导出，提供给第三方使用。

PRIVATE  在private后面的库仅被link到你的target中，并且终结掉，第三方不能感知你调了啥库

INTERFACE   在interface后面引入的库不会被链接到你的target中，只会导出符号。
# ------------------------------------------------------------ #
# public LIBRARY

#opencv 
find_package(OpenCV REQUIRED)
include_directories(${OPENCV_INCLUDE_DIRS})
target_link_libraries(MAIN ${OpenCV_LIBS})
# eigen
include_directories("/usr/include/eigen3")
# ------------------------------------------------------------ #

# 显示信息
message("hello world")
message(${PROJECT_SOURCE_DIR}/MathFunctions)

# ------------------------------------------------------------ #
# 设置option变量
option(USE_MYMATH "Use tutorial provided math implementation" ON)

if (USE_MYMATH)
    add_subdirectory(MathFunctions)
    list(APPEND EXTRA_LIBS MathFunctions)
    list(APPEND EXTRA_INCLUDES ${PROJECT_SOURCE_DIR})
endif()

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
target_include_directories(Tutorial PUBLIC 
                            ${EXTRA_INCLUDES}
                            ${PROJECT_BINARY_DIR})

# ------------------------------------------------------------ #
target_include_directories(MathFunctions
    INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR})

第一行代码是添加库名字，第二行是添加库的头文件，第一个参数是库的名字，

INTERFACE：第二个参数意思是当链接这个库的时候就要包含include第三个参数的头文件，但是自己的库不使用头文件，
PUBLIC：这个的意思是不光链接这个库的需要使用库头文件，自己也是用。一般这个使用的比较多。

第三个参数就是头文件的路径，
