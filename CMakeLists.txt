cmake_minimum_required(VERSION 3.0)

project(mymath VERSION 1.0)

set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11" ) 

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/lib)        # 动态库
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/static)     # 静态库
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/build/bin)        # 执行文件
 
# 如果想生成动态库，使用下面的语句
add_library(mymath SHARED src/mymath.cpp)
 
#VERSION 指代动态库版本，SOVERSION 指代 API 版本
target_include_directories(mymath PRIVATE  ${CMAKE_SOURCE_DIR}/include) 
set_target_properties(mymath PROPERTIES PUBLIC_HEADER ${CMAKE_SOURCE_DIR}/include/mymath.h)
set_target_properties(mymath PROPERTIES VERSION 1.0.1 SOVERSION 1)
 
# 如果同时要生成静态库
add_library(mymath_static STATIC src/mymath.cpp)
target_include_directories(mymath_static PRIVATE  ${CMAKE_SOURCE_DIR}/include) 
set_target_properties(mymath_static PROPERTIES PUBLIC_HEADER ${CMAKE_SOURCE_DIR}/include/mymath.h)
set_target_properties(mymath_static PROPERTIES OUTPUT_NAME "mymath")


install(TARGETS msgs
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION bin
)

install(DIRECTORY lib/
  DESTINATION lib
)

# 生成可执行文件
add_executable(mymathapp src/mymathApp.cpp)
target_link_libraries(mymathapp mymath)
target_include_directories(mymathapp PRIVATE ${CMAKE_SOURCE_DIR}/include)
