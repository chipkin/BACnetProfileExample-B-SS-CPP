# =============================================================================
# CMake build for the BACnet B-SS (Smart Sensor) C++ example.
#
# This example links the CAS BACnet Stack by COMPILING IT FROM SOURCE: it builds
# every file in the stack's source/ folder straight into the executable and
# calls the stack's C API (CASBACnetStackDLL.h) directly. There is no DLL/.so to
# build, copy, or ship first - everything compiles into one self-contained
# executable.
#
# NOTE: the first build compiles the entire stack (~460 source files), so it
# takes a few minutes. Subsequent incremental builds are fast.
#
# Build:
#   cmake -B build -S .
#   cmake --build build --config Release
# Run (from the example folder):
#   ./build/BACnetExampleBSS          (Linux/macOS)
#   .\build\Release\BACnetExampleBSS.exe   (Windows)
# =============================================================================

cmake_minimum_required(VERSION 3.15)
project(BACnetProfileExample_CPP_B_SS CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Locate the repo pieces relative to this folder -------------------------
# The shared helper lives ONCE at examples/common/ (the single source of truth,
# used by every example). Override with -D EXAMPLES_COMMON_DIR=... if you have
# relocated it (e.g. when taking this example out as a standalone project).
if(NOT DEFINED EXAMPLES_COMMON_DIR)
    set(EXAMPLES_COMMON_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../common")
endif()

# Where the CAS BACnet Stack lives. Defaults to the submodule in this repo.
# If you copy this example into a standalone project, point CMake at your own
# copy of the stack instead:
#     cmake -B build -S . -D CAS_STACK_DIR=/path/to/cas-bacnet-stack
if(NOT DEFINED CAS_STACK_DIR)
    set(CAS_STACK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../submodules/cas-bacnet-stack")
endif()
set(CAS_STACK_SOURCE_DIR "${CAS_STACK_DIR}/source")

if(NOT EXISTS "${CAS_STACK_SOURCE_DIR}/CASBACnetStackDLL.h")
    message(FATAL_ERROR
        "CAS BACnet Stack source not found at: ${CAS_STACK_SOURCE_DIR}\n"
        "Did you clone submodules? Run:  git submodule update --init --recursive\n"
        "Or pass -D CAS_STACK_DIR=/path/to/cas-bacnet-stack for a standalone build.")
endif()

# --- Gather the CAS BACnet Stack source -------------------------------------
# Compiling from source: grab every .cpp/.c in the stack's source/ folder. The
# example calls the stack's C API (CASBACnetStackDLL.h) directly, so no adapter
# layer is needed.
file(GLOB CAS_STACK_SOURCES
    "${CAS_STACK_SOURCE_DIR}/*.cpp"
    "${CAS_STACK_SOURCE_DIR}/*.c")

# --- The shared example helper ----------------------------------------------
set(COMMON_SOURCES
    "${EXAMPLES_COMMON_DIR}/CASExampleHelper.cpp"
    "${EXAMPLES_COMMON_DIR}/SimpleUDP.cpp")

# --- The executable ---------------------------------------------------------
add_executable(BACnetExampleBSS
    main.cpp
    ${COMMON_SOURCES}
    ${CAS_STACK_SOURCES})

target_include_directories(BACnetExampleBSS PRIVATE
    "${EXAMPLES_COMMON_DIR}"
    "${CAS_STACK_SOURCE_DIR}")

# --- Platform sockets -------------------------------------------------------
# ws2_32  = Winsock (UDP); iphlpapi = GetAdaptersInfo (local broadcast address).
if(WIN32)
    target_link_libraries(BACnetExampleBSS PRIVATE ws2_32 iphlpapi)
endif()
