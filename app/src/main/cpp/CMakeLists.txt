CMAKE_MINIMUM_REQUIRED(VERSION 3.0.0)

# Define the solutions name.
SET(NAME openppp2)
PROJECT(${NAME} C CXX) # CMAKE_CURRENT_SOURCE_DIR

# C/CXX compiler configurations.
SET(CMAKE_C_FLAGS "-fPIC -fvisibility=hidden -Wno-format -Wno-implicit-function-declaration")
SET(CMAKE_CXX_FLAGS "-fPIC -stdlib=libc++ -fvisibility=hidden -Wno-#pragma-messages -Wno-format -Wno-inconsistent-missing-override -Wno-pointer-bool-conversion -Wno-unsequenced -Wno-null-dereference -Wno-gnu-variable-sized-type-not-at-end -Wno-tautological-undefined-compare")

# Configure predefined compilation macros.
ADD_DEFINITIONS(-D_ANDROID)
ADD_DEFINITIONS(-DBUDDY_ALLOC_IMPLEMENTATION)
# ADD_DEFINITIONS(-D_ANDROID_REDEF_STD_IN_OUT_ERR)

# Set the tripartite library directorys.
SET(THIRD_PARTY_LIBRARY_DIR D:/android/cxx/3rd)
SET(PROJECT_ROOT_DIR "D:/android/cxx/openppp2")

# CPU.
SET(PLATFORM_CPU "unknown") # git clone https://github.com/liulilittle/openppp2-android-ndk-r20b-3rd-environment.git
IF (ANDROID_ABI STREQUAL "x86")
    SET(PLATFORM_CPU "x86")
ELSEIF (ANDROID_ABI STREQUAL "x86_64")
    SET(PLATFORM_CPU "x64")
ELSEIF (ANDROID_ABI STREQUAL "armeabi-v7a")
    SET(PLATFORM_CPU "armv7a")
ELSEIF (ANDROID_ABI STREQUAL "arm64-v8a")
    SET(PLATFORM_CPU "aarch64")
ENDIF ()
MESSAGE("cpu: ${PLATFORM_CPU}, abi: ${ANDROID_ABI}, output: ${LIBRARY_OUTPUT_PATH}")

# ENV.
SET(PLATFORM_DEBUG FALSE)
IF (CMAKE_BUILD_TYPE STREQUAL "Debug" OR BT MATCHES "Debug")
    SET(PLATFORM_DEBUG TRUE)
ENDIF ()

# C
IF (PLATFORM_DEBUG)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_DEBUG -g3")
ELSE ()
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
ENDIF ()

# C/CXX
IF (PLATFORM_DEBUG)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_DEBUG -g3")
ELSE ()
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
ENDIF ()

# Check C/CXX compiler supports.
INCLUDE(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
CHECK_CXX_COMPILER_FLAG("-std=c++1z" COMPILER_SUPPORTS_CXX1Z)

# If the current compiler environment is detected to support AES.
SET(__AES_NI__ FALSE)
IF(__AES_NI__)
    # Add in the compiler flag section.
    ADD_COMPILE_OPTIONS(-maes -msse2 -mpclmul)

    # Add the AES_NI compiler macro.
    ADD_DEFINITIONS(-D__AES_NI__)

    # Additional options for the old version of GCC.
    IF(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 8.0)
        ADD_COMPILE_OPTIONS(-march=westmere)
        ADD_COMPILE_OPTIONS(-fno-stack-protector)
    ENDIF()

    # AES is enabled, the source code of AES/128/256/CFB related to the library needs to be compiled.
    FILE(GLOB_RECURSE AES_NI_SOURCE_FILES
            ${PROJECT_ROOT_DIR}/common/aesni/impl/*.c
            ${PROJECT_ROOT_DIR}/common/aesni/impl/*.cpp)
ENDIF()

# According to the type of operating system to determine whether you need to check whether the compiler supports C/C++17, 
# On the MACOS system, the following check code is invalid, so directly specify the standard version of C/C++, 
# If an error or compilation does not pass, it means that you need to upgrade the LLVM GCC/CLANG compiler installed on MACOS.

# CXX17
SET(CMAKE_CXX_STANDARD 17)

# -rdynamic -Wl,-Bstatic -Wl,-Bdynamic -lstdc++ -lpthread -ldl -lz -lrt
SET(CMAKE_EXE_LINKER_FLAGS "-static-libstdc++ -rdynamic -Wl,-Bstatic")

# Set the compiled header file search directorys.
INCLUDE_DIRECTORIES(
        ${PROJECT_ROOT_DIR}/
        ${PROJECT_ROOT_DIR}/common
        ${PROJECT_ROOT_DIR}/common/json/include
        ${PROJECT_ROOT_DIR}/common/lwip/my
        ${PROJECT_ROOT_DIR}/common/lwip/include

        ${THIRD_PARTY_LIBRARY_DIR}/boost
        ${THIRD_PARTY_LIBRARY_DIR}/openssl/${PLATFORM_CPU}/include
)

# Set the third library connection directory to searchs.
LINK_DIRECTORIES(
        ${THIRD_PARTY_LIBRARY_DIR}/boost/${PLATFORM_CPU}
        ${THIRD_PARTY_LIBRARY_DIR}/openssl/${PLATFORM_CPU}/lib
)

SET(BUILD_SHARED_LIBS ON)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY BOTH)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE BOTH)
SET(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE BOTH)

# List of common cpp source files compilations for ppp/unix cmake solution projects.
FILE(GLOB_RECURSE SOURCE_FILES
        ${PROJECT_ROOT_DIR}/common/lwip/my/*.c
        ${PROJECT_ROOT_DIR}/common/lwip/core/*.c
        ${PROJECT_ROOT_DIR}/common/lwip/api/*.c

        ${PROJECT_ROOT_DIR}/common/aggligator/*.cpp
        ${PROJECT_ROOT_DIR}/common/base64/*.cpp
        ${PROJECT_ROOT_DIR}/common/chnroutes2/*.cpp
        ${PROJECT_ROOT_DIR}/common/json/src/*.cpp
        ${PROJECT_ROOT_DIR}/common/libtcpip/*.cpp
        ${PROJECT_ROOT_DIR}/common/dnslib/*.cpp
        ${PROJECT_ROOT_DIR}/common/unix/*.c
        ${PROJECT_ROOT_DIR}/common/unix/*.cpp
        ${PROJECT_ROOT_DIR}/common/aesni/aes.cpp

        ${PROJECT_ROOT_DIR}/ppp/*.c
        ${PROJECT_ROOT_DIR}/ppp/*.cpp

        ${PROJECT_ROOT_DIR}/linux/*.c
        ${PROJECT_ROOT_DIR}/linux/*.cpp

        ${PROJECT_ROOT_DIR}/android/libopenppp2.cpp)

# Add the compiled output binary files.
ADD_LIBRARY(${NAME} SHARED ${SOURCE_FILES} ${PLATFORM_SOURCE_FILES} ${AES_NI_SOURCE_FILES})

# Set the compilation output files path.
SET(LIBRARY_OUTPUT_PATH ${PROJECT_ROOT_DIR}/bin/${ANDROID_ABI})

# Set up library connections to dependent libraries.
TARGET_LINK_LIBRARIES(${CMAKE_PROJECT_NAME}
        libssl.a
        libcrypto.a

        dl

        libboost_system.a
        libboost_coroutine.a
        libboost_thread.a
        libboost_context.a
        libboost_regex.a
        libboost_filesystem.a)