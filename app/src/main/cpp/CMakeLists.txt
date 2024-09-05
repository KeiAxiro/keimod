cmake_minimum_required(VERSION 3.22.1)

# Declares the project name. The project name can be accessed via ${PROJECT_NAME},
# Since this is the top level CMakeLists.txt, the project name is also accessible
# with ${CMAKE_PROJECT_NAME} (both CMake variables are in-sync within the top level
# build script scope).
project("DarkTeam")

# Option to build examples and link log library
option(BNM_BUILD_EXAMPLES "Build examples" ON)
option(BNM_LINK_LOG "Link log library" OFF)

# Set C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Include directories
set(PUBLIC_INCLUDE_DIRS
        Canvas
        Includes
        hook/And64InlineHook
        hook/ByNameModding
        hook/Dobby
        hook/Substrate
        hook/KittyMemory
        Menu
)

# Compiler flags
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w -s -Wno-error=format-security -fvisibility=hidden -fpermissive -fexceptions")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -s -Wno-error=format-security -fvisibility=hidden -Werror -std=c++20 -Wno-error=c++11-narrowing -fpermissive -Wall -fexceptions")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--gc-sections,--strip-all")

# Add BNM library
add_library(HOOK STATIC
        hook/Substrate/hde64.c
        hook/Substrate/SubstrateDebug.cpp
        hook/Substrate/SubstrateHook.cpp
        hook/Substrate/SubstratePosixMemory.cpp
        hook/Substrate/SymbolFinder.cpp
        hook/KittyMemory/KittyMemory.cpp
        hook/KittyMemory/MemoryPatch.cpp
        hook/KittyMemory/MemoryBackup.cpp
        hook/KittyMemory/KittyUtils.cpp
        hook/And64InlineHook/And64InlineHook.cpp
        hook/ByNameModding/Tools.cpp
        hook/ByNameModding/fake_dlfcn.cpp
        hook/ByNameModding/Il2Cpp.cpp
)

# Include directories for BNM library
target_include_directories(HOOK PUBLIC ${PUBLIC_INCLUDE_DIRS})

# Link log library if enabled
if (BNM_LINK_LOG)
    target_link_libraries(HOOK PUBLIC log)
endif()

# Add main library
add_library(keiaxiro SHARED
        main.cpp
)

# Include directories for main library
target_include_directories(keiaxiro PUBLIC ${PUBLIC_INCLUDE_DIRS})

# Link libraries for main library
target_link_libraries(keiaxiro PUBLIC HOOK android log EGL GLESv2)

# Add examples if enabled
if (BNM_BUILD_EXAMPLES)
    add_library(BNM_Examples SHARED
            main.cpp
    )

    # Include directories for BNM_Examples library
    target_include_directories(BNM_Examples PUBLIC ${PUBLIC_INCLUDE_DIRS})

    # Link libraries for BNM_Examples library
    target_link_libraries(BNM_Examples PUBLIC HOOK android log EGL GLESv2)
endif()
