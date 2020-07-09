cmake_minimum_required(VERSION 3.0.0)

project(openvino_pedestrian_tracker)

set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/build)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

find_package(ngraph REQUIRED)
find_package(InferenceEngine REQUIRED)
find_package(OpenCV REQUIRED)
include_directories(${CMAKE_SOURCE_DIR})
include_directories(${CMAKE_SOURCE_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/common)
add_subdirectory(common)
add_subdirectory(gflags)

add_executable(${PROJECT_NAME} main.cpp src/cnn.cpp src/detector.cpp src/distance.cpp src/kuhn_munkres.cpp src/tracker.cpp src/utils.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE ${InferenceEngine_LIBRARIES} ${OpenCV_LIBS} ${NGRAPH_LIBRARIES} gflags monitors)

if (WIN32)
    if (NOT "${CMAKE_SIZEOF_VOID_P}" EQUAL "8")
        message(FATAL_ERROR "Only 64-bit supported on Windows")
    endif()

    set_property (DIRECTORY APPEND PROPERTY COMPILE_DEFINITIONS _CRT_SECURE_NO_WARNINGS)
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_SCL_SECURE_NO_WARNINGS -DNOMINMAX")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc") #no asynchronous structured exception handling
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /LARGEADDRESSAWARE")

    if (TREAT_WARNING_AS_ERROR)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /WX") #treating warnings as errors
    endif ()

    if (${CMAKE_CXX_COMPILER_ID} STREQUAL MSVC)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4251 /wd4275 /wd4267") #disable some warnings
    endif()
else()
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror") #treating warnings as errors
    if (APPLE)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-error=unused-command-line-argument")
    elseif(UNIX)
        set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wuninitialized -Winit-self")
        if(NOT ${CMAKE_CXX_COMPILER_ID} STREQUAL Clang)
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wmaybe-uninitialized")
        endif()
    endif()
endif()

if(MSVC)
set_target_properties( ${PROJECT_NAME} PROPERTIES  VS_DEBUGGER_WORKING_DIRECTORY "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
endif()

file(COPY ${PROJECT_SOURCE_DIR}/models DESTINATION ${CMAKE_SOURCE_DIR}/build/Debug)
file(COPY ${PROJECT_SOURCE_DIR}/models DESTINATION ${CMAKE_SOURCE_DIR}/build/Release)
file(COPY ${PROJECT_SOURCE_DIR}/test_videos DESTINATION ${CMAKE_SOURCE_DIR}/build/Debug)
file(COPY ${PROJECT_SOURCE_DIR}/test_videos DESTINATION ${CMAKE_SOURCE_DIR}/build/Release)
file(COPY ${PROJECT_SOURCE_DIR}/run_test.cmd DESTINATION ${CMAKE_SOURCE_DIR}/build/Debug)
file(COPY ${PROJECT_SOURCE_DIR}/run_test.cmd DESTINATION ${CMAKE_SOURCE_DIR}/build/Release)