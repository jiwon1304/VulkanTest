cmake_minimum_required(VERSION 3.20)

project(VulkanTest LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

find_package(Vulkan QUIET)

# Find Vulkan SDK manually if not found
if(NOT Vulkan_FOUND)
    find_path(VULKAN_INCLUDE_DIR
        NAMES vulkan/vulkan.h
        HINTS
            "$ENV{VULKAN_SDK}/Include"
            "/opt/homebrew/include"
            "/usr/local/include"
            "/usr/include"
    )

    find_library(VULKAN_LIBRARY
        NAMES vulkan-1 vulkan
        HINTS
            "$ENV{VULKAN_SDK}/Lib"
            "$ENV{VULKAN_SDK}/Lib32"
            "/opt/homebrew/lib"
            "/usr/local/lib"
            "/usr/lib"
    )

    # macOS fallback: try to find MoltenVK (Homebrew provides molten-vk / vulkan-loader)
    if(NOT VULKAN_INCLUDE_DIR AND NOT VULKAN_LIBRARY AND CMAKE_HOST_SYSTEM_NAME STREQUAL "Darwin")
        find_library(MOLTENVK_LIBRARY
            NAMES MoltenVK libMoltenVK
            HINTS
                "/opt/homebrew/lib"
                "/usr/local/lib"
        )

        find_path(VULKAN_INCLUDE_DIR
            NAMES vulkan/vulkan.h
            HINTS
                "/opt/homebrew/include"
                "/usr/local/include"
        )

        if(MOLTENVK_LIBRARY AND VULKAN_INCLUDE_DIR)
            set(VULKAN_LIBRARY ${MOLTENVK_LIBRARY})
        endif()
    endif()

    if(VULKAN_INCLUDE_DIR AND VULKAN_LIBRARY)
        add_library(Vulkan::Vulkan UNKNOWN IMPORTED)
        set_target_properties(Vulkan::Vulkan PROPERTIES
            IMPORTED_LOCATION "${VULKAN_LIBRARY}"
            INTERFACE_INCLUDE_DIRECTORIES "${VULKAN_INCLUDE_DIR}"
        )
        set(Vulkan_FOUND ON)
    else()
        message(FATAL_ERROR "Vulkan SDK not found. Please install the Vulkan SDK or set VULKAN_SDK.")
    endif()
endif()

add_subdirectory(ThirdParty)

# Bring in ImGui (core + GLFW/Vulkan backends) via FetchContent and build as a static target
include(FetchContent)
FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v1.89.8
)
FetchContent_MakeAvailable(imgui)

# If the imgui project didn't create an `imgui` target, create one from sources.
if(NOT TARGET imgui)
    add_library(imgui STATIC
        ${imgui_SOURCE_DIR}/imgui.cpp
        ${imgui_SOURCE_DIR}/imgui_draw.cpp
        ${imgui_SOURCE_DIR}/imgui_tables.cpp
        ${imgui_SOURCE_DIR}/imgui_widgets.cpp
        ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
        ${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp
    )

    target_include_directories(imgui
        PUBLIC
            ${imgui_SOURCE_DIR}
            ${imgui_SOURCE_DIR}/backends
    )

    # Link with Vulkan and GLFW (GLFW target provided by ThirdParty::glfw)
    target_link_libraries(imgui
        PUBLIC
            Vulkan::Vulkan
            ThirdParty::glfw
    )
endif()

if(NOT TARGET ThirdParty::imgui)
    add_library(ThirdParty::imgui ALIAS imgui)
endif()

add_executable(VulkanTest
    Source/main.cpp
)

target_include_directories(VulkanTest
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

target_link_libraries(VulkanTest
    PRIVATE
        Vulkan::Vulkan
        ThirdParty::glfw
    glm::glm
)

add_custom_command(TARGET VulkanTest POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:ThirdParty::glfw>
        $<TARGET_FILE_DIR:VulkanTest>
)
