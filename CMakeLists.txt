# Setup CMake project
cmake_minimum_required(VERSION 3.16)
project(wiringpi VERSION 2.61.1)

######################################################
# Installation paths
######################################################

set(LIB_INSTALL_DIR ${PROJECT_NAME}/lib CACHE PATH "Installation directory for libraries")
set(BIN_INSTALL_DIR ${PROJECT_NAME}/bin CACHE PATH "Installation directory for binaries")
set(INCLUDE_INSTALL_DIR wiringPi CACHE PATH "Installation directory for C++ headers")
set(DATA_INSTALL_DIR share/ CACHE PATH "Installation directory for data")

######################################################
# Create library
######################################################
# Find threads package: https://cmake.org/cmake/help/v3.2/module/FindThreads.html
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
find_package(Threads REQUIRED)

# Get WiringPi sources from the submodule (library only)
file(GLOB WIRINPI_SRC_FILES wiringPi/*.c)

# Suppress int to/from pointer cast warnings
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-pointer-to-int-cast -Wno-int-to-pointer-cast")

# Build static library
add_library(${PROJECT_NAME} STATIC ${WIRINPI_SRC_FILES})
target_include_directories(${PROJECT_NAME}
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/wiringPi>
    $<INSTALL_INTERFACE:${INCLUDE_INSTALL_DIR}>)

# Set library version property
set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION})
set_target_properties(${PROJECT_NAME} PROPERTIES SOVERSION ${PROJECT_VERSION_MAJOR})

# Link library
target_link_libraries(${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT} crypt m rt)

######################################################
# Install library
######################################################
# Install headers
install(DIRECTORY ${INCLUDE_INSTALL_DIR}
    EXPORT ${PROJECT_NAME}-targets
    DESTINATION ${PROJECT_NAME}/include
    FILES_MATCHING PATTERN "*.h")

include(CMakePackageConfigHelpers)

# Configure packges config file
configure_package_config_file(${PROJECT_SOURCE_DIR}/cmake/packaging/config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    INSTALL_DESTINATION ${LIB_INSTALL_DIR}/cmake/${PROJECT_NAME}
    PATH_VARS BIN_INSTALL_DIR INCLUDE_INSTALL_DIR LIB_INSTALL_DIR)

# Write packge version file
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion)

# Install config files
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
    DESTINATION ${LIB_INSTALL_DIR}/cmake/${PROJECT_NAME}
    COMPONENT cmake)
