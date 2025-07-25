# EnTT

cmake_minimum_required(VERSION 3.15.7)

# Read project version

set(ENTT_VERSION_REGEX "#define ENTT_VERSION_.*[ \t]+(.+)")
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/src/entt/config/version.h" ENTT_VERSION REGEX ${ENTT_VERSION_REGEX})
list(TRANSFORM ENTT_VERSION REPLACE ${ENTT_VERSION_REGEX} "\\1")
string(JOIN "." ENTT_VERSION ${ENTT_VERSION})

# Project configuration

project(
    EnTT
    VERSION ${ENTT_VERSION}
    DESCRIPTION "Gaming meets modern C++ - a fast and reliable entity-component system (ECS) and much more"
    HOMEPAGE_URL "https://github.com/skypjack/entt"
    LANGUAGES C CXX
)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
endif()
    
message(VERBOSE "*")
message(VERBOSE "* ${PROJECT_NAME} v${PROJECT_VERSION} (${CMAKE_BUILD_TYPE})")
message(VERBOSE "* Copyright (c) 2017-2025 Michele Caini <michele.caini@gmail.com>")
message(VERBOSE "*")

# CMake stuff

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

# Compiler stuff

option(ENTT_USE_LIBCPP "Use libc++ by adding -stdlib=libc++ flag if available." OFF)
option(ENTT_USE_SANITIZER "Enable sanitizers by adding -fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined flags if available." OFF)
option(ENTT_USE_CLANG_TIDY "Enable static analysis with clang-tidy" OFF)

if(ENTT_USE_LIBCPP)
    if(NOT WIN32)
        include(CheckCXXSourceCompiles)
        include(CMakePushCheckState)

        cmake_push_check_state()

        set(CMAKE_REQUIRED_FLAGS "${CMAKE_REQUIRED_FLAGS} -stdlib=libc++")

        check_cxx_source_compiles("
            #include<type_traits>
            int main() { return std::is_same_v<int, char>; }
        " ENTT_HAS_LIBCPP)

        cmake_pop_check_state()
    endif()

    if(NOT ENTT_HAS_LIBCPP)
        message(VERBOSE "The option ENTT_USE_LIBCPP is set but libc++ is not available.")
    endif()
endif()

if(ENTT_USE_SANITIZER)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
        set(ENTT_HAS_SANITIZER TRUE CACHE BOOL "" FORCE)
        mark_as_advanced(ENTT_HAS_SANITIZER)
    endif()

    if(NOT ENTT_HAS_SANITIZER)
        message(VERBOSE "The option ENTT_USE_SANITIZER is set but sanitizer support is not available.")
    endif()
endif()

if(ENTT_USE_CLANG_TIDY)
    find_program(ENTT_CLANG_TIDY_EXECUTABLE "clang-tidy")

    if(NOT ENTT_CLANG_TIDY_EXECUTABLE)
        message(VERBOSE "The option ENTT_USE_CLANG_TIDY is set but clang-tidy executable is not available.")
    endif()
endif()

# Add EnTT target

include(GNUInstallDirs)

add_library(EnTT INTERFACE)
add_library(EnTT::EnTT ALIAS EnTT)

target_include_directories(
    EnTT
    INTERFACE
        $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src>
        $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/tools>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_compile_features(EnTT INTERFACE cxx_std_17)

if(ENTT_HAS_LIBCPP)
    target_compile_options(EnTT BEFORE INTERFACE -stdlib=libc++)
endif()

if(ENTT_HAS_SANITIZER)
    target_compile_options(EnTT INTERFACE $<$<CONFIG:Debug>:-fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined>)
    target_link_libraries(EnTT INTERFACE $<$<CONFIG:Debug>:-fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined>)
endif()

if(ENTT_CLANG_TIDY_EXECUTABLE)
    set(CMAKE_CXX_CLANG_TIDY "${ENTT_CLANG_TIDY_EXECUTABLE};--config-file=${EnTT_SOURCE_DIR}/.clang-tidy;--header-filter=${EnTT_SOURCE_DIR}/src/entt/.*")
endif()

# Add EnTT goodies

option(ENTT_INCLUDE_HEADERS "Add all EnTT headers to the EnTT target." OFF)
option(ENTT_INCLUDE_TOOLS "Add EnTT tools files to the EnTT target." OFF)
option(ENTT_INCLUDE_NATVIS "Add EnTT natvis files to the EnTT target." OFF)

if(ENTT_INCLUDE_HEADERS)
    target_sources(
        EnTT
        INTERFACE
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/config/config.h>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/config/macro.h>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/config/version.h>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/container/dense_map.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/container/dense_set.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/container/table.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/container/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/algorithm.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/any.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/attribute.h>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/bit.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/compressed_pair.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/enum.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/family.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/hashed_string.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/ident.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/iterator.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/memory.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/monostate.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/ranges.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/tuple.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/type_info.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/type_traits.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/core/utility.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/component.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/entity.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/group.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/handle.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/mixin.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/helper.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/organizer.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/ranges.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/registry.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/runtime_view.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/snapshot.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/sparse_set.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/storage.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entity/view.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/graph/adjacency_matrix.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/graph/dot.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/graph/flow.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/graph/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/locator/locator.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/adl_pointer.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/container.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/context.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/factory.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/meta.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/node.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/pointer.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/policy.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/range.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/resolve.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/template.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/type_traits.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/meta/utility.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/poly/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/poly/poly.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/process/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/process/process.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/process/scheduler.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/resource/cache.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/resource/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/resource/loader.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/resource/resource.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/signal/delegate.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/signal/dispatcher.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/signal/emitter.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/signal/fwd.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/signal/sigh.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/entt.hpp>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/fwd.hpp>
    )
endif()

if(ENTT_INCLUDE_TOOLS)
    target_sources(
        EnTT
        INTERFACE
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/tools/entt/davey/davey.hpp>
    )
endif()

if(ENTT_INCLUDE_NATVIS)
    if(MSVC)
        set(ENTT_HAS_NATVIS TRUE CACHE BOOL "" FORCE)
        mark_as_advanced(ENTT_HAS_NATVIS)
    endif()

    if(NOT ENTT_HAS_NATVIS)
        message(VERBOSE "The option ENTT_INCLUDE_NATVIS is set but natvis files are not supported. They will not be added to the target.")
    endif()
endif()

if(ENTT_HAS_NATVIS)
    target_sources(
        EnTT
        INTERFACE
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/config.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/config.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/container.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/container.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/core.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/core.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/entity.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/entity.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/graph.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/graph.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/locator.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/locator.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/meta.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/meta.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/poly.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/poly.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/process.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/process.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/resource.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/resource.natvis>
            $<BUILD_INTERFACE:${EnTT_SOURCE_DIR}/src/entt/natvis/signal.natvis>
            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/entt/natvis/signal.natvis>
    )
endif()

# Install EnTT and all related files

option(ENTT_INSTALL "Install EnTT and all related files." OFF)

if(ENTT_INSTALL)
    # Install pkg-config file

    include(JoinPaths)

    set(EnTT_PKGCONFIG ${CMAKE_CURRENT_BINARY_DIR}/entt.pc)

    join_paths(EnTT_PKGCONFIG_INCLUDEDIR "\${prefix}" "${CMAKE_INSTALL_INCLUDEDIR}")

    configure_file(
        ${EnTT_SOURCE_DIR}/cmake/in/entt.pc.in
        ${EnTT_PKGCONFIG}
        @ONLY
    )

    install(
        FILES ${EnTT_PKGCONFIG}
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
    )

    # Install EnTT

    include(CMakePackageConfigHelpers)

    install(
        TARGETS EnTT
        EXPORT EnTTTargets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )

    write_basic_package_version_file(
        EnTTConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
    )

    configure_package_config_file(
        ${EnTT_SOURCE_DIR}/cmake/in/EnTTConfig.cmake.in
        EnTTConfig.cmake
        INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/EnTT/cmake
    )

    export(
        EXPORT EnTTTargets
        FILE ${CMAKE_CURRENT_BINARY_DIR}/EnTTTargets.cmake
        NAMESPACE EnTT::
    )

    install(
        EXPORT EnTTTargets
        FILE EnTTTargets.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/EnTT/cmake
        NAMESPACE EnTT::
    )

    install(
        FILES
            ${PROJECT_BINARY_DIR}/EnTTConfig.cmake
            ${PROJECT_BINARY_DIR}/EnTTConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/EnTT/cmake
    )

    install(
        DIRECTORY src/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.hpp"
        PATTERN "*.natvis"
    )

    export(PACKAGE EnTT)
endif()

# Tests and testbed

option(ENTT_BUILD_TESTING "Enable building tests." OFF)
option(ENTT_BUILD_TESTBED "Enable building testbed." OFF)

if(ENTT_BUILD_TESTING OR ENTT_BUILD_TESTBED)
    set(ENTT_ID_TYPE std::uint32_t CACHE STRING "Type of identifiers to use for tests and testbed")
    set(ENTT_CXX_STD cxx_std_17 CACHE STRING "C++ standard revision to use for tests and testbed")

    # Tests and tesetbed do not work together because SDL gets confused with EnTT tests
    if(ENTT_BUILD_TESTING)
        option(ENTT_FIND_GTEST_PACKAGE "Enable finding gtest package." OFF)
    
        option(ENTT_BUILD_BENCHMARK "Build benchmark." OFF)
        option(ENTT_BUILD_EXAMPLE "Build examples." OFF)
        option(ENTT_BUILD_LIB "Build lib tests." OFF)
        option(ENTT_BUILD_SNAPSHOT "Build snapshot test with Cereal." OFF)
    
        include(CTest)
        enable_testing()
        add_subdirectory(test)
    elseif(ENTT_BUILD_TESTBED)
        add_subdirectory(testbed)
    endif()
endif()

# Documentation

option(ENTT_BUILD_DOCS "Enable building with documentation." OFF)

if(ENTT_BUILD_DOCS)
    add_subdirectory(docs)
endif()
