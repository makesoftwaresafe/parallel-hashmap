#[===================================================================[
    parallel-hashmap library by Gregory Popovitch

    CMake projects that wish to use this library may do
    something like :

    include(FetchContent)
    FetchContent_Declare(
        parallel-hashmap
        GIT_REPOSITORY https://github.com/greg7mdp/parallel-hashmap.git
        GIT_TAG        v2.0.0    # adjust tag/branch/commit as needed
    )
    FetchContent_MakeAvailable(parallel-hashmap)

    ...
    include_directories(${parallel-hashmap_SOURCE_DIR})

#]===================================================================]

cmake_minimum_required(VERSION 3.13)

list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(DetectVersion)

if(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 11)            ## compile with C++11 support
endif()

if(NOT CMAKE_CXX_STANDARD_REQUIRED)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

if(NOT DEFINED PHMAP_MASTER_PROJECT)
    set(PHMAP_MASTER_PROJECT OFF)
    if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
        set(PHMAP_MASTER_PROJECT ON)
    endif()
endif()

project(phmap VERSION ${DETECTED_PHMAP_VERSION} LANGUAGES CXX)

## ----------------------------- options -----------------------------
option(PHMAP_INSTALL "Enable installation" ${PHMAP_MASTER_PROJECT})


set(PHMAP_DIR parallel_hashmap)
set(PHMAP_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_base.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_bits.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_config.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_dump.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_fwd_decl.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/phmap_utils.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/meminfo.h
                  ${CMAKE_CURRENT_SOURCE_DIR}/${PHMAP_DIR}/btree.h)

include(helpers)

add_library(${PROJECT_NAME} INTERFACE)

target_sources(${PROJECT_NAME} INTERFACE ${PHMAP_HEADERS})

target_include_directories(
     ${PROJECT_NAME} INTERFACE
     $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
     $<INSTALL_INTERFACE:include>)

if(PHMAP_INSTALL)
    include(GNUInstallDirs)
    include(CMakePackageConfigHelpers)

    install(
        DIRECTORY ${PROJECT_SOURCE_DIR}/${PHMAP_DIR}/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PHMAP_DIR})

    install(TARGETS ${PROJECT_NAME}
            EXPORT ${PROJECT_NAME}-targets)

    export(EXPORT ${PROJECT_NAME}-targets
           FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")
endif()

## ------------------------- building tests and examples -------------
option(PHMAP_BUILD_TESTS    "Whether or not to build the tests"    ${PHMAP_MASTER_PROJECT})
option(PHMAP_BUILD_EXAMPLES "Whether or not to build the examples" ${PHMAP_MASTER_PROJECT})

if(MSVC)
    add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:/bigobj>")
endif()

if (PHMAP_BUILD_TESTS OR PHMAP_BUILD_EXAMPLES)
    include_directories(${PROJECT_SOURCE_DIR})
endif()

if (PHMAP_BUILD_TESTS)

    if (NOT PHMAP_GTEST_LIBS)
        include(cmake/DownloadGTest.cmake)

        check_target(gtest)
        check_target(gtest_main)
        check_target(gmock)
        set(PHMAP_GTEST_LIBS gmock_main)
    endif()

    enable_testing()

    ## ---------------- regular hash maps ----------------------------
    phmap_cc_test(NAME container_memory SRCS "tests/container_memory_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME hash_policy_testing SRCS "tests/hash_policy_testing_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME node_hash_policy SRCS "tests/node_hash_policy_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME raw_hash_set SRCS "tests/raw_hash_set_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME raw_hash_set_allocator SRCS "tests/raw_hash_set_allocator_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    ## ---------------- regular hash maps ----------------------------
    phmap_cc_test(NAME flat_hash_set SRCS "tests/flat_hash_set_test.cc"
                  COPTS "-DUNORDERED_SET_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME flat_hash_map SRCS "tests/flat_hash_map_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME node_hash_map SRCS "tests/node_hash_map_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME node_hash_set SRCS "tests/node_hash_set_test.cc"
                  COPTS "-DUNORDERED_SET_CXX17" DEPS  ${PHMAP_GTEST_LIBS})

    ## --------------- parallel hash maps -----------------------------------------------
    phmap_cc_test(NAME parallel_flat_hash_map SRCS "tests/parallel_flat_hash_map_test.cc"
                  COPTS "-DUNORDERED_MAP_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME parallel_flat_hash_set SRCS "tests/parallel_flat_hash_set_test.cc"
                  COPTS "-DUNORDERED_SET_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME parallel_node_hash_map SRCS "tests/parallel_node_hash_map_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME parallel_node_hash_set SRCS "tests/parallel_node_hash_set_test.cc"
                  COPTS "-DUNORDERED_SET_CXX17" DEPS  ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME parallel_flat_hash_map_mutex SRCS "tests/parallel_flat_hash_map_mutex_test.cc"
                  COPTS "-DUNORDERED_MAP_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME dump_load SRCS "tests/dump_load_test.cc"
                  COPTS "-DUNORDERED_MAP_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    phmap_cc_test(NAME erase_if SRCS "tests/erase_if_test.cc"
                  COPTS "-DUNORDERED_MAP_CXX17" DEPS ${PHMAP_GTEST_LIBS})

    ## --------------- btree -----------------------------------------------
    phmap_cc_test(NAME btree SRCS "tests/btree_test.cc"
                  DEPS ${PHMAP_GTEST_LIBS})


endif()

if (PHMAP_BUILD_EXAMPLES)
    if(NOT MSVC)
        add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:-pedantic;-Wall;-Wextra;-Wcast-align;-Wcast-qual;-Wdisabled-optimization;-Winit-self;-Wlogical-op;-Wmissing-include-dirs;-Woverloaded-virtual;-Wredundant-decls;-Wshadow;-Wstrict-null-sentinel;-Wswitch-default;-Wno-unused>")
        if (NOT CMAKE_COMPILER_IS_GNUCC OR CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 5.0)
            add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:-Wno-unknown-warning-option;-Wno-gnu-zero-variadic-macro-arguments>")
        endif()
    else()
        add_compile_options("$<$<COMPILE_LANGUAGE:CXX>:/W4;/Zc:__cplusplus>")
    endif()

    set(THREADS_PREFER_PTHREAD_FLAG ON)
    find_package(Threads REQUIRED)

    add_executable(ex_allmaps examples/allmaps.cc phmap.natvis)
    add_executable(ex_basic examples/basic.cc phmap.natvis)
    add_executable(ex_bench examples/bench.cc phmap.natvis)
    add_executable(ex_emplace examples/emplace.cc phmap.natvis)
    if (MSVC)
        add_executable(ex_lazy_emplace_l examples/lazy_emplace_l.cc phmap.natvis)
    endif()
    add_executable(ex_serialize examples/serialize.cc phmap.natvis)
    #target_include_directories(ex_serialize PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../cereal/include>)
    add_executable(ex_hash_std examples/hash_std.cc phmap.natvis)
    add_executable(ex_hash_value examples/hash_value.cc phmap.natvis)
    add_executable(ex_hash examples/hash.cc phmap.natvis)
    add_executable(ex_two_files examples/f1.cc examples/f2.cc phmap.natvis)
    add_executable(ex_insert_bench examples/insert_bench.cc phmap.natvis)
    add_executable(ex_knucleotide examples/knucleotide.cc phmap.natvis)
    add_executable(ex_dump_load examples/dump_load.cc phmap.natvis)
    add_executable(ex_btree examples/btree.cc phmap.natvis)
    add_executable(ex_hash_bench examples/hash_bench.cc phmap.natvis)
    add_executable(ex_matt examples/matt.cc phmap.natvis)
    add_executable(ex_mt_word_counter examples/mt_word_counter.cc phmap.natvis)
    add_executable(ex_p_bench examples/p_bench.cc phmap.natvis)

    #set(Boost_INCLUDE_DIR /home/greg/dev/boost_1_82_0) # if boost installed in non-standard location
    set(Boost_USE_STATIC_LIBS OFF)
    set(Boost_USE_MULTITHREADED ON)
    set(Boost_USE_STATIC_RUNTIME OFF)

    # llil4map.cc - see https://www.perlmonks.com/?node_id=11149643
    # -------------------------------------------------------------
    find_package(OpenMP)
    find_package(Boost 1.70.0)
    if (OpenMP_FOUND AND Boost_FOUND AND "cxx_std_20" IN_LIST CMAKE_CXX_COMPILE_FEATURES)
       add_executable(ex_llil4map examples/llil4map.cc phmap.natvis)
       target_include_directories(ex_llil4map PRIVATE ${Boost_INCLUDE_DIRS})
       target_compile_features(ex_llil4map PUBLIC cxx_std_20)
       find_package(TBB COMPONENTS tbb)
       if (TBB_FOUND)
           target_link_libraries(ex_llil4map PRIVATE TBB::tbb)
       endif()
       target_link_libraries(ex_llil4map PRIVATE OpenMP::OpenMP_CXX)
       target_compile_options(ex_llil4map PRIVATE "${OpenMP_CXX_FLAGS}")

       file(COPY examples/llil_utils DESTINATION "${CMAKE_CURRENT_BINARY_DIR}")
    endif()

    if (Boost_FOUND)
       add_executable(ex_custom_pointer examples/custom_pointer.cc phmap.natvis)
       target_include_directories(ex_custom_pointer PRIVATE ${Boost_INCLUDE_DIRS})

       add_executable(ex_llil examples/llil.cc phmap.natvis)
       target_include_directories(ex_llil PRIVATE ${Boost_INCLUDE_DIRS})
       target_compile_features(ex_llil PUBLIC cxx_std_20)
       target_link_libraries(ex_llil Threads::Threads)
    endif()

    target_link_libraries(ex_knucleotide Threads::Threads)
    target_link_libraries(ex_bench Threads::Threads)
endif()
