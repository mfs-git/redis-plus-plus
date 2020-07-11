cmake_minimum_required(VERSION 3.6)
project(redis++ VERSION 1.1.2)

if (NOT DEFINED REDIS_PLUS_PLUS_CXX_STANDARD)
    set(REDIS_PLUS_PLUS_CXX_STANDARD 11)
endif()

message(STATUS "The CXX standard is c++${REDIS_PLUS_PLUS_CXX_STANDARD}")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++${REDIS_PLUS_PLUS_CXX_STANDARD} -Wall -W -Werror -fPIC")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/src/sw/redis++)

file(GLOB PROJECT_SOURCE_FILES "${PROJECT_SOURCE_DIR}/*.cpp")

set(STATIC_LIB static)
set(SHARED_LIB shared)

add_library(${STATIC_LIB} STATIC ${PROJECT_SOURCE_FILES})
add_library(${SHARED_LIB} SHARED ${PROJECT_SOURCE_FILES})

# hiredis dependency
find_path(HIREDIS_HEADER hiredis)
target_include_directories(${STATIC_LIB} PUBLIC ${HIREDIS_HEADER})
target_include_directories(${SHARED_LIB} PUBLIC ${HIREDIS_HEADER})

find_library(HIREDIS_LIB hiredis)
target_link_libraries(${SHARED_LIB} ${HIREDIS_LIB})

set_target_properties(${STATIC_LIB} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
set_target_properties(${SHARED_LIB} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})

set_target_properties(${STATIC_LIB} PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties(${SHARED_LIB} PROPERTIES CLEAN_DIRECT_OUTPUT 1)

option(REDIS_PLUS_PLUS_BUILD_TEST "Build tests for redis++" ON)

if(REDIS_PLUS_PLUS_BUILD_TEST)
    add_subdirectory(test)
endif()

# Install static lib.
install(TARGETS ${STATIC_LIB}
        ARCHIVE DESTINATION lib)

# Install shared lib.
install(TARGETS ${SHARED_LIB}
        LIBRARY DESTINATION lib)

#Install headers.
set(HEADER_PATH "sw/redis++")
file(GLOB HEADERS "${PROJECT_SOURCE_DIR}/*.h*")
install(FILES ${HEADERS} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${HEADER_PATH})


################### creating package ################
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(OS6432 "amd64")
else()
    set(OS6432 "i386")
endif()

EXECUTE_PROCESS(
        COMMAND python -c "import platform;print(platform.dist()[0])"
        OUTPUT_VARIABLE LINUX_DISTRO
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

EXECUTE_PROCESS(
        COMMAND python -c "import platform;print(platform.dist()[1])"
        OUTPUT_VARIABLE LINUX_VER
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

EXECUTE_PROCESS(
        COMMAND lsb_release -cs
        OUTPUT_VARIABLE LINUX_CODE_NAME
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

set(CPACK_GENERATOR "DEB")
set(CPACK_PACKAGE_VENDOR "https://github.com/sewenew/redis-plus-plus")
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}-${LINUX_CODE_NAME}-${LINUX_VER}-${OS6432}")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libhiredis-dev")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "https://github.com/sewenew/redis-plus-plus")
set(CPACK_PACKAGE_CONTACT "https://github.com/sewenew/redis-plus-plus")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "https://github.com/sewenew/redis-plus-plus")

include(CPack)
