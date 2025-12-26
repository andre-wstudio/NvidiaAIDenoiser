cmake_minimum_required(VERSION 3.10)
project(Denoiser)

set(CMAKE_CXX_STANDARD 17)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

find_package(OptiX80)
# Add CUDA
find_package(CUDAToolkit 12.0 REQUIRED)

# OpenImageIO - use bundled version on Windows, system version on Linux
if(WIN32)
    set(OIIO_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/contrib/OpenImageIO/include)
    set(OIIO_LIBRARY ${PROJECT_SOURCE_DIR}/contrib/OpenImageIO/lib/OpenImageIO.lib)
    set(OIIO_BIN_DIR ${PROJECT_SOURCE_DIR}/contrib/OpenImageIO/bin)
else()
    find_package(OpenImageIO REQUIRED)
endif()

# Include directories
include_directories(${OPTIX80_INCLUDE_DIR} ${CUDAToolkit_INCLUDE_DIRS} ${OIIO_INCLUDE_DIR})

# Source files
set(SOURCES src/main.cpp)

# Executable
add_executable(${PROJECT_NAME} ${SOURCES})

# Link libraries
target_link_libraries(${PROJECT_NAME} ${OIIO_LIBRARY} CUDA::cudart)

# Windows-specific settings
if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE NOMINMAX)

    # Copy OpenImageIO DLLs to output directory
    file(GLOB OIIO_DLLS "${OIIO_BIN_DIR}/*.dll")
    foreach(DLL ${OIIO_DLLS})
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${DLL}"
            $<TARGET_FILE_DIR:${PROJECT_NAME}>)
    endforeach()
endif()
