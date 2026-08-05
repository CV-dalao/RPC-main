# 获取当前目录下的所有 .cc 源文件
# 注意：这已经包含了 Krpcheader.pb.cc，因此不需要再单独添加 PROTO_SRCS
file(GLOB SRC_FILES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/*.cc"
)

# 创建静态库
add_library(krpc_core STATIC
    ${SRC_FILES}
)

# 设置头文件路径
target_include_directories(krpc_core
    PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

# ZooKeeper 多线程客户端需要定义 THREADED，
# 否则 zoo_exists、zoo_create、zoo_get 等同步接口不会被声明
target_compile_definitions(krpc_core
    PRIVATE
        THREADED
)

# 使用 C++17，解决 MPMCQueue 的 aligned-new 警告
target_compile_features(krpc_core
    PUBLIC
        cxx_std_17
)

# 编译警告
target_compile_options(krpc_core
    PRIVATE
        -Wall
)

# 链接依赖库
target_link_libraries(krpc_core
    PUBLIC
        protobuf
        zookeeper_mt
        glog
        muduo_net
        muduo_base
        pthread
)
