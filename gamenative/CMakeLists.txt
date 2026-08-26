cmake_minimum_required(VERSION 3.22.1)
project(GameNative_MaliG57)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -Wno-unused-function -Wimplicit-function-declaration -DANDROID")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -std=c++17 -fvisibility=hidden -DANDROID")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS_RELEASE} -s")

# add_subdirectory(src/virglrenderer)

add_library(winlator SHARED
    src/winlator/sysvshared_memory.c
    src/winlator/xconnector_epoll.c
    src/winlator/alsa_client.c
    src/winlator/gpu_helper.c
    src/winlator/wine_registry_editor.c
    src/winlator/patchelf_wrapper.cpp)

target_link_libraries(winlator
    log android jnigraphics aaudio
    EGL GLESv2 GLESv3)

target_link_options(winlator PRIVATE -Wl,-z,max-page-size=16384)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/window_vert.h
    COMMAND glslangValidator -V --vn window_vert_code -x
            -o ${CMAKE_CURRENT_BINARY_DIR}/window_vert.h
            ${CMAKE_CURRENT_SOURCE_DIR}/src/winlator/window.vert
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/winlator/window.vert)

add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/window_frag.h
    COMMAND glslangValidator -V --vn window_frag_code -x
            -o ${CMAKE_CURRENT_BINARY_DIR}/window_frag.h
            ${CMAKE_CURRENT_SOURCE_DIR}/src/winlator/window.frag
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/winlator/window.frag)

add_library(vulkan_renderer SHARED
    src/winlator/vulkan_jni.cpp
    src/winlator/VulkanRendererContext.cpp
    src/winlator/VulkanRendererScanout.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/window_vert.h
    ${CMAKE_CURRENT_BINARY_DIR}/window_frag.h)

target_include_directories(vulkan_renderer PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_options(vulkan_renderer PRIVATE -Wall -Wextra -fvisibility=hidden -std=c++17)
target_link_libraries(vulkan_renderer log android vulkan dl atomic)
target_link_options(vulkan_renderer PRIVATE -Wl,-z,max-page-size=16384)
