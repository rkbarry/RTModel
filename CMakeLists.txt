cmake_minimum_required(VERSION 3.19)

project(RTModel LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(MINIMUM_GCC_VERSION "9.0")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS MINIMUM_GCC_VERSION)
        message(FATAL_ERROR "GCC must be ${MINIMUM_GCC_VERSION} or greater.")
    endif()
endif()

# Find directory of VBMicrolensing

find_package(Python3 COMPONENTS Interpreter Development)
set(Python3_EXECUTABLE "./python")

execute_process(
    COMMAND ${PYTHON_EXECUTABLE} fVBM.py   
    #-c "import sys, os, inspect; sys.path.extend(os.environ['PYTHONPATH'].split(os.pathsep));  mod = __import__('VBMicrolensing'); dir1=os.path.dirname(inspect.getfile(mod)); print(dir1.replace(\"\\\",\"/\"))"
    OUTPUT_VARIABLE VBMICROLENSING_DIR
    OUTPUT_STRIP_TRAILING_WHITESPACE
)
message(STATUS "Installation directory of VBMicrolensing: ${VBMICROLENSING_DIR}")


# Build executables
# =================

add_executable(Reader RTModel/lib/Reader.cpp)
target_include_directories(Reader PRIVATE RTModel/include)

add_executable(InitCond RTModel/lib/InitCond.cpp)
target_include_directories(InitCond PRIVATE RTModel/include)

add_executable(LevMar RTModel/lib/bumper.cpp RTModel/lib/LevMar.cpp RTModel/lib/LevMarFit.cpp ${VBMICROLENSING_DIR}/lib/VBMicrolensingLibrary.cpp)
target_include_directories(LevMar PRIVATE RTModel/include ${VBMICROLENSING_DIR}/lib)

add_executable(ModelSelector RTModel/lib/bumper.cpp RTModel/lib/ModelSelector.cpp)
target_include_directories(ModelSelector PRIVATE RTModel/include)

add_executable(Finalizer RTModel/lib/bumper.cpp RTModel/lib/Finalizer.cpp)
target_include_directories(Finalizer PRIVATE RTModel/include)

# Install executables
# ===================

if(SKBUILD)  # Only run if called through scikit-build-core. That is, don't run when called through CMake directly.
    set(binary_directory "${SKBUILD_PLATLIB_DIR}/RTModel/bin")

    install(TARGETS Reader DESTINATION ${binary_directory})
    install(TARGETS InitCond DESTINATION ${binary_directory})
    install(TARGETS LevMar DESTINATION ${binary_directory})
    install(TARGETS ModelSelector DESTINATION ${binary_directory})
    install(TARGETS Finalizer DESTINATION ${binary_directory})

    # Ensure data files are present in platlib.
    add_custom_command(TARGET Reader POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_directory
            "${CMAKE_CURRENT_LIST_DIR}/RTModel/data" "${SKBUILD_PLATLIB_DIR}/RTModel/data")
endif()
