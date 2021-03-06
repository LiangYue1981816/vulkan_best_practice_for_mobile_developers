# Copyright (c) 2019, Arm Limited and Contributors
#
# SPDX-License-Identifier: MIT
#
# Permission is hereby granted, free of charge,
# to any person obtaining a copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software,
# and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
# INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
# WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
#
 
cmake_minimum_required(VERSION 3.6)

project(framework LANGUAGES C CXX)

set(FRAMEWORK_FILES
    # Header Files
    common.h
    utils.h
    gui.h
    stats.h
    glsl_compiler.h
    spirv_reflection.h
    gltf_loader.h
    fence_pool.h
    semaphore_pool.h
    command_record.h
    command_replay.h
    render_target.h
    graphics_pipeline_state.h
    resource_binding_state.h
    cache_resource.h
    cache_resource.inl
    render_frame.h
    render_context.h
    vulkan_sample.h
    # Source Files
    common.cpp
    utils.cpp
    gui.cpp
    profiler.cpp
    stats.cpp
    glsl_compiler.cpp
    spirv_reflection.cpp
    gltf_loader.cpp
    fence_pool.cpp
    semaphore_pool.cpp
    command_record.cpp
    command_replay.cpp
    render_target.cpp
    graphics_pipeline_state.cpp
    resource_binding_state.cpp
    render_frame.cpp
    render_context.cpp
    vulkan_sample.cpp)

set(FONT_FILES
    # Source Files
    fonts/roboto.cpp)

set(SCENE_GRAPH_FILES
    # Header Files
    scene_graph/component.h
    scene_graph/node.h
    scene_graph/scene.h
    scene_graph/script.h
    # Source Files
    scene_graph/component.cpp
    scene_graph/node.cpp
    scene_graph/scene.cpp
    scene_graph/script.cpp)

set(SCENE_GRAPH_COMPONENT_FILES
    # Header Files
    scene_graph/components/camera.h
    scene_graph/components/perspective_camera.h
    scene_graph/components/image.h
    scene_graph/components/material.h
    scene_graph/components/mesh.h
    scene_graph/components/pbr_material.h
    scene_graph/components/sampler.h
    scene_graph/components/sub_mesh.h
    scene_graph/components/texture.h
    scene_graph/components/transform.h
    # Source Files
    scene_graph/components/camera.cpp
    scene_graph/components/perspective_camera.cpp
    scene_graph/components/image.cpp
    scene_graph/components/material.cpp
    scene_graph/components/mesh.cpp
    scene_graph/components/pbr_material.cpp
    scene_graph/components/sampler.cpp
    scene_graph/components/sub_mesh.cpp
    scene_graph/components/texture.cpp
    scene_graph/components/transform.cpp)

set(SCENE_GRAPH_SCRIPTS_FILES
    # Header Files
    scene_graph/scripts/free_camera.h
    scene_graph/scripts/node_animation.h
    # Source Files
    scene_graph/scripts/free_camera.cpp
    scene_graph/scripts/node_animation.cpp)

set(CORE_FILES
    # Header Files
    core/device.h
    core/shader_module.h
    core/pipeline_layout.h
    core/pipeline.h
    core/descriptor_set_layout.h
    core/descriptor_pool.h
    core/descriptor_set.h
    core/queue.h
    core/command_pool.h
    core/swapchain.h
    core/command_buffer.h
    core/buffer.h
    core/image.h
    core/image_view.h
    core/framebuffer.h
    core/render_pass.h
    # Source Files
    core/device.cpp
    core/image.cpp
    core/shader_module.cpp
    core/pipeline_layout.cpp
    core/pipeline.cpp
    core/descriptor_set_layout.cpp
    core/descriptor_pool.cpp
    core/descriptor_set.cpp
    core/queue.cpp
    core/command_pool.cpp
    core/swapchain.cpp
    core/command_buffer.cpp
    core/buffer.cpp
    core/image.cpp
    core/image_view.cpp
    core/framebuffer.cpp
    core/render_pass.cpp)

set(PLATFORM_FILES
    # Header Files
    platform/application.h
    platform/platform.h
    platform/thread_pool.h
    platform/concurrent_queue.h
    platform/concurrent_queue.inl
    platform/input_events.h
    platform/configuration.h
    # Source Files
    platform/application.cpp
    platform/platform.cpp
    platform/thread_pool.cpp
    platform/input_events.cpp
    platform/configuration.cpp)

# Add files based on platform
if(ANDROID)
    list(APPEND PLATFORM_FILES
        # Header Files
        platform/android/android_platform.h
        # Source Files
        platform/android/android_platform.cpp)
else()
    list(APPEND PLATFORM_FILES
        # Header Files
        platform/glfw_platform.h
        # Source Files
        platform/glfw_platform.cpp)

    if(WIN32)
        list(APPEND PLATFORM_FILES
        # Header Files
        platform/windows/windows_platform.h
        # Source Files
        platform/windows/windows_platform.cpp)
    else()
        list(APPEND PLATFORM_FILES
        # Header Files
        platform/linux/linux_platform.h
        # Source Files
        platform/linux/linux_platform.cpp)
    endif()
endif()

source_group("\\" FILES ${FRAMEWORK_FILES})
source_group("fonts\\" FILES ${FONT_FILES})
source_group("platform\\" FILES ${PLATFORM_FILES})
source_group("core\\" FILES ${CORE_FILES})
source_group("scene_graph\\" FILES ${SCENE_GRAPH_FILES})
source_group("scene_graph\\components\\" FILES ${SCENE_GRAPH_COMPONENT_FILES})
source_group("scene_graph\\scripts\\" FILES ${SCENE_GRAPH_SCRIPTS_FILES})

set(PROJECT_FILES
    ${PLATFORM_FILES}
    ${FRAMEWORK_FILES}
    ${FONT_FILES}   
    ${CORE_FILES}
    ${SCENE_GRAPH_FILES}
    ${SCENE_GRAPH_COMPONENT_FILES}
    ${SCENE_GRAPH_SCRIPTS_FILES})

add_library(${PROJECT_NAME} STATIC ${PROJECT_FILES})

# compiler flags based on compiler type
if(NOT MSVC)
    target_compile_options(${PROJECT_NAME} PUBLIC -fexceptions)
endif()

if(${VKB_VALIDATION_LAYERS})
    target_compile_definitions(${PROJECT_NAME} PUBLIC VKB_VALIDATION_LAYERS)
endif()

target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# Link third party libraries
target_link_libraries(${PROJECT_NAME}
    volk
    imgui
    tinygltf
    glm
    glslang
    SPIRV
    vma
    hwcpipe
    spirv-cross-glsl
    glslang-default-resource-limits)

# Link platform specific libraries
if(ANDROID)
    target_link_libraries(${PROJECT_NAME} log android native_app_glue)
else()
    target_link_libraries(${PROJECT_NAME} glfw)
endif()
