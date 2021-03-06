cmake_minimum_required(VERSION 3.1.3)

project(GSL CXX)

include(ExternalProject)
find_package(Git)

# Use GNUInstallDirs to provide the right locations on all platforms
include(GNUInstallDirs)

# creates a library GSL which is an interface (header files only)
# Target must be renamed to avoid conflics with GNU Scientific Library
add_library(GSL INTERFACE)

# determine whether this is a standalone project or included by other projects
set(GSL_STANDALONE_PROJECT OFF)
if (CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(GSL_STANDALONE_PROJECT ON)
endif ()

# when minimum version required is 3.8.0 remove if below
# both branches do exactly the same thing
if (CMAKE_VERSION VERSION_LESS 3.7.9)
    if (NOT MSVC)
        include(CheckCXXCompilerFlag)
        CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
        if(COMPILER_SUPPORTS_CXX14)
            target_compile_options(GSL INTERFACE "-std=c++14")
        else()
            message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++14 support. Please use a different C++ compiler.")
        endif()

    endif()
else ()
    # set the GSL library to be compiled only with c++14
    target_compile_features(GSL INTERFACE cxx_std_14)
    # on *nix systems force the use of -std=c++XX instead of -std=gnu++XX (default)
    set(CMAKE_CXX_EXTENSIONS OFF)
endif()

# add definitions to the library and targets that consume it
target_compile_definitions(GSL INTERFACE
    $<$<CXX_COMPILER_ID:MSVC>:
        # remove unnecessary warnings about unchecked iterators
        _SCL_SECURE_NO_WARNINGS
    >
)

# add include folders to the library and targets that consume it
target_include_directories(GSL INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

if (MSVC_IDE)
    option(VS_ADD_NATIVE_VISUALIZERS "Configure project to use Visual Studio native visualizers" TRUE)
else()
    set(VS_ADD_NATIVE_VISUALIZERS FALSE CACHE INTERNAL "Native visualizers are Visual Studio extension" FORCE)
endif()

# add natvis file to the library so it will automatically be loaded into Visual Studio
if(VS_ADD_NATIVE_VISUALIZERS)
  target_sources(GSL INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/GSL.natvis>
    )
endif()

install(TARGETS GSL EXPORT Microsoft.GSLConfig
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
install(
    DIRECTORY include/gsl
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
# Make library importable by other projects
install(EXPORT Microsoft.GSLConfig NAMESPACE Microsoft.GSL:: DESTINATION share/Microsoft.GSL/cmake)
export(TARGETS GSL NAMESPACE Microsoft.GSL:: FILE Microsoft.GSLConfig.cmake)


option(GSL_TEST "Generate tests." ${GSL_STANDALONE_PROJECT})
if (GSL_TEST)
	enable_testing()
	add_subdirectory(tests)
endif ()
