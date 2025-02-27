# ~~~
# Copyright (c) 2014-2023 Valve Corporation
# Copyright (c) 2014-2023 LunarG, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ~~~
include(GoogleTest) # gtest_discover_tests

# GoogleTest is required for testing framework
find_package(GTest REQUIRED CONFIG)

# glslang is required for testing framework
find_package(glslang CONFIG REQUIRED)

add_executable(vk_layer_validation_tests)

target_sources(vk_layer_validation_tests PRIVATE
    ${VVL_SOURCE_DIR}/layers/convert_to_renderpass2.cpp
    ${VVL_SOURCE_DIR}/layers/generated/lvt_function_pointers.cpp
    ${VVL_SOURCE_DIR}/layers/generated/vk_format_utils.cpp
    ${VVL_SOURCE_DIR}/layers/generated/vk_safe_struct.cpp
    positive/android_hardware_buffer.cpp
    positive/best_practices.cpp
    positive/command.cpp
    positive/descriptors.cpp
    positive/dynamic_rendering.cpp
    positive/graphics_library.cpp
    positive/image_buffer.cpp
    positive/instance.cpp
    positive/other.cpp
    positive/pipeline.cpp
    positive/ray_tracing.cpp
    positive/ray_tracing_pipeline.cpp
    positive/render_pass.cpp
    positive/shaderval.cpp
    positive/sync.cpp
    positive/tooling.cpp
    positive/video.cpp
    layer_validation_tests.cpp
    vkerrormonitor.cpp
    vkerrormonitor.h
    vklayertests_amd_best_practices.cpp
    vklayertests_android_hardware_buffer.cpp
    vklayertests_arm_best_practices.cpp
    vklayertests_best_practices.cpp
    vklayertests_buffer.cpp
    vklayertests_command.cpp
    vklayertests_debug_printf.cpp
    vklayertests_descriptor_renderpass_framebuffer.cpp
    vklayertests_dynamic_rendering.cpp
	vklayertests_external_memory_sync.cpp
    vklayertests_gpu.cpp
    vklayertests_graphics_library.cpp
	vklayertests_image.cpp
    vklayertests_imageless_framebuffer.cpp
    vklayertests_instanceless.cpp
	vklayertests_memory.cpp
    vklayertests_nvidia_best_practices.cpp
	vklayertests_object_lifetime.cpp
    vklayertests_others.cpp
    vklayertests_pipeline_shader.cpp
    vklayertests_portability_subset.cpp
    vklayertests_query.cpp
    vklayertests_ray_tracing.cpp
    vklayertests_ray_tracing_gpu.cpp
    vklayertests_ray_tracing_pipeline.cpp
	vklayertests_sampler.cpp
	vklayertests_sparse.cpp
	vklayertests_sync.cpp
    vklayertests_video.cpp
    vklayertests_viewport_inheritance.cpp
    vklayertests_wsi.cpp
	vklayertests_ycbcr.cpp
    vkrenderframework.cpp
    vkrenderframework.h
    vksyncvaltests.cpp
    vktestbinding.cpp
    vktestframework.cpp
    ray_tracing_objects.h
    ray_tracing_objects.cpp
)

# gtest_discover_tests has problem with cross-compiling, but it is faster and more robust
if (CMAKE_CROSSCOMPILING)
    gtest_add_tests(TARGET vk_layer_validation_tests)
else()
    gtest_discover_tests(vk_layer_validation_tests DISCOVERY_TIMEOUT 100)
endif()

add_dependencies(vk_layer_validation_tests VkLayer_khronos_validation)
target_include_directories(vk_layer_validation_tests PRIVATE
    ${VVL_SOURCE_DIR}/layers
    .
)

if(${CMAKE_CXX_COMPILER_ID} MATCHES "(GNU|Clang)")
    target_compile_options(vk_layer_validation_tests PRIVATE
        -Wno-sign-compare
        -Wno-shorten-64-to-32
        -Wno-unused-parameter
        -Wno-missing-field-initializers
    )
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(vk_layer_validation_tests PRIVATE
            -Wno-sign-conversion
            -Wno-implicit-int-conversion
            -Wno-enum-enum-conversion
        )
    endif()
elseif(MSVC)
    # Disable some signed/unsigned mismatch warnings.
    target_compile_options(vk_layer_validation_tests PRIVATE /wd4267)

    if(MSVC_IDE)
        set_target_properties(vk_layer_validation_tests PROPERTIES VS_DEBUGGER_ENVIRONMENT "VK_LAYER_PATH=$<TARGET_FILE_DIR:VkLayer_khronos_validation>")
    endif()

    if (CMAKE_SIZEOF_VOID_P EQUAL 4)
        # Due to IHV driver issues, we need the extra 2GB of virtual address space for 32 bit testing
        target_link_options(vk_layer_validation_tests PRIVATE /LARGEADDRESSAWARE)
    endif()
endif()

target_link_libraries(vk_layer_validation_tests PRIVATE
    VkLayer_utils
    glslang::glslang
    glslang::OGLCompiler
    glslang::OSDependent
    glslang::MachineIndependent
    glslang::GenericCodeGen
    glslang::HLSL
    glslang::SPIRV
    glslang::SPVRemapper
    VVL-SPIRV-LIBS
    GTest::gtest
    GTest::gtest_main
    ${CMAKE_DL_LIBS}
    $<TARGET_NAME_IF_EXISTS:PkgConfig::XCB>
    $<TARGET_NAME_IF_EXISTS:PkgConfig::X11>
    $<TARGET_NAME_IF_EXISTS:PkgConfig::WAYlAND_CLIENT>
)

option(INSTALL_TESTS "Install tests")
if(INSTALL_TESTS)
    install(TARGETS vk_layer_validation_tests)
endif()

add_subdirectory(layers)
