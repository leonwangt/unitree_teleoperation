# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License.

add_compile_options(-std=c++17)
set(CMAKE_BUILD_TYPE Release)
# Eigen
find_package(Eigen3 3.3 REQUIRED NO_MODULE)

message(STATUS "Current system architecture: ${CMAKE_SYSTEM_PROCESSOR}")



set(MuJoCo_Include_Dir ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco-3.1.5/include)
set(MuJoCo_Library_Dir ${CMAKE_CURRENT_SOURCE_DIR}/../mujoco-3.1.5/lib)
set(MuJoCo_Libraries mujoco)

link_directories(
    ${MuJoCo_Library_Dir}
    ${hx_idl_cpp_Include_Dir}
)

list(APPEND CMAKE_PREFIX_PATH "/usr/local/lib/cmake/lcm")
find_path(LCM_INCLUDE_DIR lcm/lcm.h)
find_library(LCM_LIBRARY lcm)

add_executable(kinect_teleoperate main.cpp)

target_include_directories(kinect_teleoperate 
    PRIVATE 
    ../sample_helper_includes 
    ${MuJoCo_Include_Dir}
    include/
    # unitree_sdk2
)

# Dependencies of this library
target_link_libraries(kinect_teleoperate PRIVATE 
    k4a
    k4abt
    k4arecord
    window_controller_3d::window_controller_3d
    glfw::glfw
    ${MuJoCo_Libraries}
    Eigen3::Eigen
    lcm
    # unitree_sdk2
)


list(APPEND CMAKE_PREFIX_PATH "/usr/local/lib")
list(APPEND CMAKE_PREFIX_PATH "/usr/local/include")
## Import Unitree SDK2 library
set(UNITREE_SDK_PATH "/usr/local/lib")
find_library(UNITREE_SDK_LIB unitree_sdk2 PATHS ${UNITREE_SDK_PATH} NO_DEFAULT_PATH)

if (NOT UNITREE_SDK_LIB)
    message(FATAL_ERROR "Unitree SDK library for the architecture is not found")
else ()
    message(STATUS "Unitree SDK library found at: ${UNITREE_SDK_LIB}")
endif ()

message(STATUS "Importing: ${UNITREE_SDK_LIB}")

find_package(Threads REQUIRED)
include_directories(/usr/local/include/ddscxx)

add_library(unitree_sdk2 STATIC IMPORTED GLOBAL)
set_target_properties(unitree_sdk2 PROPERTIES
        IMPORTED_LOCATION ${UNITREE_SDK_LIB})
target_link_libraries(unitree_sdk2 INTERFACE dds ddsc ddscxx Threads::Threads)
target_include_directories(unitree_sdk2 INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

        message(STATUS "Importing: ${CMAKE_CURRENT_LIST_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR}/libddsc.so")

        add_library(ddsc SHARED IMPORTED GLOBAL)
        set_target_properties(ddsc PROPERTIES
                IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR}/libddsc.so)
        set_target_properties(ddsc PROPERTIES IMPORTED_NO_SONAME TRUE)
        target_link_libraries(ddsc INTERFACE Threads::Threads)
        target_link_directories(ddsc INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR})
        target_include_directories(ddsc INTERFACE
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                $<INSTALL_INTERFACE:include>)
        
        message(STATUS "Importing: ${CMAKE_CURRENT_LIST_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR}/libddscxx.so")
        
        add_library(ddscxx SHARED IMPORTED GLOBAL)
        set_target_properties(ddscxx PROPERTIES
                IMPORTED_LOCATION ${CMAKE_CURRENT_LIST_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR}/libddscxx.so)
        set_target_properties(ddscxx PROPERTIES IMPORTED_NO_SONAME TRUE)
        target_link_libraries(ddscxx INTERFACE Threads::Threads)
        target_link_directories(ddscxx INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/lib/${CMAKE_SYSTEM_PROCESSOR})
        target_include_directories(ddscxx INTERFACE
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/ddscxx>
                $<INSTALL_INTERFACE:include>
                $<INSTALL_INTERFACE:include/ddscxx>)
