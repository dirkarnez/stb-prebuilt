cmake_minimum_required(VERSION 3.3.2)
project(stb LANGUAGES C CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/CMake")


set(VERSION "${ENSMALLEN_VERSION_MAJOR}.${ENSMALLEN_VERSION_MINOR}.${ENSMALLEN_VERSION_PATCH}")

# Create library target.
add_library(stb INTERFACE)
target_include_directories(stb INTERFACE
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  $<INSTALL_INTERFACE:include>)

# Set warning flags for target.
if(MSVC)
  target_compile_options(ensmallen INTERFACE $<BUILD_INTERFACE:/Wall>)
else()
  target_compile_options(ensmallen INTERFACE $<BUILD_INTERFACE:-Wall -Wpedantic -Wunused-parameter>)
endif()

# Set helper variables for creating the version, config and target files.
include(CMakePackageConfigHelpers)
set(ENSMALLEN_CMAKE_DIR "lib/cmake/stb" CACHE STRING "Installation directory for cmake files, relative to ${CMAKE_INSTALL_PREFIX}.")
set(VERSION_CONFIG "${PROJECT_BINARY_DIR}/stb-config-version.cmake")
set(PROJECT_CONFIG "${PROJECT_BINARY_DIR}/stb-config.cmake")
set(TARGETS_EXPORT_NAME stb-targets)

# Generate the version, config and target files into the build directory.
write_basic_package_version_file(${VERSION_CONFIG}
    VERSION ${VERSION}
    COMPATIBILITY AnyNewerVersion)
configure_package_config_file(${PROJECT_SOURCE_DIR}/CMake/ensmallen-config.cmake.in
    ${PROJECT_CONFIG}
    INSTALL_DESTINATION ${ENSMALLEN_CMAKE_DIR})
export(TARGETS ensmallen NAMESPACE ensmallen::
    FILE ${PROJECT_BINARY_DIR}/${TARGETS_EXPORT_NAME}.cmake)

# Install version, config and target files.
install(FILES ${PROJECT_CONFIG} ${VERSION_CONFIG}
    DESTINATION ${ENSMALLEN_CMAKE_DIR})
install(EXPORT ${TARGETS_EXPORT_NAME} DESTINATION ${ENSMALLEN_CMAKE_DIR}
    NAMESPACE ensmallen::)

# Export the targets and install the header files.
install(TARGETS ensmallen EXPORT ${TARGETS_EXPORT_NAME} DESTINATION lib)
install(DIRECTORY "${CMAKE_SOURCE_DIR}/include/ensmallen_bits"
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include"
        PATTERN "*~" EXCLUDE
        PATTERN "*.sw*" EXCLUDE)
install(FILES ${CMAKE_SOURCE_DIR}/include/ensmallen.hpp
        DESTINATION "${CMAKE_INSTALL_PREFIX}/include")

# Enable testing and build tests.
enable_testing()
add_subdirectory(tests)
