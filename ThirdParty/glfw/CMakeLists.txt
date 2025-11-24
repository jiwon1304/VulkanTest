cmake_minimum_required(VERSION 3.20)

set(_GLFW_THIRDPARTY_ROOT ${CMAKE_CURRENT_LIST_DIR})
set(_GLFW_INCLUDE_DIR "${_GLFW_THIRDPARTY_ROOT}/include")

if(WIN32)
    set(_GLFW_LIB_DIR "${_GLFW_THIRDPARTY_ROOT}/lib/win64")
    set(_GLFW_BIN_DIR "${_GLFW_THIRDPARTY_ROOT}/bin/win64")
    set(_GLFW_IMPORTED_LOCATION "${_GLFW_BIN_DIR}/glfw3.dll")
    set(_GLFW_IMPORTED_IMPLIB "${_GLFW_LIB_DIR}/glfw3dll.lib")
elseif(APPLE)
    set(_GLFW_LIB_DIR "${_GLFW_THIRDPARTY_ROOT}/lib/mac")
    set(_GLFW_BIN_DIR "${_GLFW_THIRDPARTY_ROOT}/bin/mac")
    set(_GLFW_IMPORTED_LOCATION "${_GLFW_BIN_DIR}/libglfw.3.dylib")
    set(_GLFW_IMPORTED_IMPLIB "${_GLFW_LIB_DIR}/libglfw3.a")
else()
    message(FATAL_ERROR "Precompiled GLFW binaries are only provided for Windows in this workspace.")
endif()

add_library(ThirdParty::glfw SHARED IMPORTED GLOBAL)

set_target_properties(ThirdParty::glfw PROPERTIES
    IMPORTED_IMPLIB "${_GLFW_IMPORTED_IMPLIB}"
    IMPORTED_LOCATION "${_GLFW_IMPORTED_LOCATION}"
    INTERFACE_INCLUDE_DIRECTORIES "${_GLFW_INCLUDE_DIR}"
)

unset(_GLFW_THIRDPARTY_ROOT)
unset(_GLFW_INCLUDE_DIR)
unset(_GLFW_LIB_DIR)
unset(_GLFW_BIN_DIR)
unset(_GLFW_IMPORTED_LOCATION)
unset(_GLFW_IMPORTED_IMPLIB)
