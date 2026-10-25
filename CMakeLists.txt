# tokenizer.neuralnet.cpp — BPE / CharBPETokenizer（header-only）
#
# 内容：
#   domain_tokenizer.hpp        聚合入口
#   domain_tokenizer_base.hpp   Tokenizer 基类 + 并行预分词 + 词表持久化
#   domain_tokenizer_bpe.hpp    BPETokenizer
#   domain_tokenizer_charbpe.hpp CharBPETokenizer + make_tokenizer 工厂
#
# 依赖（均 submodule 位于 extern/）：
#   base       Result/断言/精度/常量
#   threadpool 并行预分词（parallel_for_samples / global_thread_pool）
#   kvrecord   词表二进制容器
#   dataset    词表文件格式（parse_nnvocab 等）
#
# 消费方式：
#   git submodule add .../{base,threadpool,kvrecord,dataset}.neuralnet.cpp.git extern/...
#   add_subdirectory(tokenizer)
#   target_link_libraries(app PRIVATE neuralnet::tokenizer)
#   #include <neuralnet.cpp/domain_tokenizer.hpp>
cmake_minimum_required(VERSION 3.20)
project(neuralnet_tokenizer LANGUAGES CXX)

# 顺序：先 base（定义 neuralnet_base），再 threadpool/kvrecord/dataset
#（各自检测目标已存在则跳过，避免重复 add_subdirectory）。父工程已定义则全跳过。
foreach(_dep base threadpool kvrecord dataset)
    if(NOT TARGET neuralnet_${_dep})
        add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/extern/${_dep}
                         ${CMAKE_CURRENT_BINARY_DIR}/extern/${_dep})
    endif()
endforeach()

add_library(neuralnet_tokenizer INTERFACE)
add_library(neuralnet::tokenizer ALIAS neuralnet_tokenizer)
target_include_directories(neuralnet_tokenizer INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>)
target_link_libraries(neuralnet_tokenizer INTERFACE
    neuralnet::base neuralnet::threadpool neuralnet::kvrecord neuralnet::dataset)
target_compile_features(neuralnet_tokenizer INTERFACE cxx_std_23)
