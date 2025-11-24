cmake_minimum_required(VERSION 3.20)

add_library(glm-header-only INTERFACE)

target_include_directories(glm-header-only INTERFACE
    "${CMAKE_CURRENT_LIST_DIR}/include"
)

add_library(glm::glm ALIAS glm-header-only)
add_library(ThirdParty::glm ALIAS glm-header-only)
