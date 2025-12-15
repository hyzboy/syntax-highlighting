cmake_minimum_required(VERSION 3.16)

set(KF_VERSION "6.22.0")
project(KSyntaxHighlighting VERSION ${KF_VERSION})

# Require C++20
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Compiler settings
if(MSVC)
    # Use UTF-8 for source files
    add_compile_options(/utf-8)
    # Enable standards-conforming preprocessor
    add_compile_options(/Zc:preprocessor)
    # Enable proper __cplusplus macro
    add_compile_options(/Zc:__cplusplus)
    # Increase warning level
    add_compile_options(/W3)
    # Suppress specific warnings
    add_compile_options(/wd4251) # needs to have dll-interface
    # Enable parallel builds
    add_compile_options(/MP)
else()
    # GCC/Clang warnings
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Position independent code for shared libraries
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include(GNUInstallDirs)
include(FeatureSummary)
include(GenerateExportHeader)
include(CMakePackageConfigHelpers)

set(KSYNTAXHIGHLIGHTING_VERSION_MAJOR 6)
set(KSYNTAXHIGHLIGHTING_VERSION_MINOR 22)
set(KSYNTAXHIGHLIGHTING_VERSION_PATCH 0)
set(KSYNTAXHIGHLIGHTING_VERSION ${KF_VERSION})
set(KSYNTAXHIGHLIGHTING_SOVERSION 6)

# Generate version header
set(ksyntaxhighlighting_version_header "${CMAKE_CURRENT_BINARY_DIR}/src/lib/ksyntaxhighlighting_version.h")
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/ksyntaxhighlighting_version.h.in"
    "${ksyntaxhighlighting_version_header}"
    @ONLY
)

# Generate package version file
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/KF6SyntaxHighlightingConfigVersion.cmake"
    VERSION ${KSYNTAXHIGHLIGHTING_VERSION}
    COMPATIBILITY SameMajorVersion
)

set(EXCLUDE_DEPRECATED_BEFORE_AND_AT 0 CACHE STRING "Control the range of deprecated API excluded from the build [default=0].")

#
# Dependencies
#
set(REQUIRED_QT_VERSION 6.8.0)
find_package(Qt6 ${REQUIRED_QT_VERSION} NO_MODULE REQUIRED COMPONENTS Core Network Test)
option(KSYNTAXHIGHLIGHTING_USE_GUI "Build components depending on QtGui" ON)
if(KSYNTAXHIGHLIGHTING_USE_GUI)
    find_package(Qt6 ${REQUIRED_QT_VERSION} NO_MODULE REQUIRED COMPONENTS Gui)
endif()
find_package(Qt6 ${REQUIRED_QT_VERSION} NO_MODULE QUIET OPTIONAL_COMPONENTS PrintSupport Widgets Quick)
set_package_properties(Qt6 PROPERTIES URL "http://qt-project.org/")
set_package_properties(Qt6Widgets PROPERTIES PURPOSE "Example application.")
set_package_properties(Qt6PrintSupport PROPERTIES PURPOSE "Example application.")
set_package_properties(Qt6Quick PROPERTIES PURPOSE "QtQuick bindings.")

# https://xerces.apache.org/index.html for validation
find_package (XercesC)
set_package_properties(XercesC PROPERTIES PURPOSE "Compile-time validation of syntax definition files.")

find_package(Perl REQUIRED)
set_package_properties(Perl PROPERTIES PURPOSE "Auto-generate PHP syntax definition files.")

find_package(Python)
set_package_properties(Python PROPERTIES PURPOSE "Auto-generate Jinja syntax definition files.")

#
# allow to install the "differently" licensed syntax xml files instead of putting them in a QRC and link them in
#
option(QRC_SYNTAX "Bundle the syntax definition files inside the library as resources" ON)
add_feature_info(SYNTAX_RESOURCE ${QRC_SYNTAX} "Bundle the syntax definition files inside the library as resources")

#
# allow to turn of lookup for syntax files and themes via QStandardPaths
#
option(NO_STANDARD_PATHS "Skip lookup of syntax and theme definitions in QStandardPaths locations" OFF)
add_feature_info(FEATURE_NO_STANDARD_PATHS ${NO_STANDARD_PATHS} "Skip lookup of syntax and theme definitions in QStandardPaths locations")

#
# Translations
# Note: Translation support requires manual handling without ECM

# tell the framework if it shall use the syntax files from the resource
if (QRC_SYNTAX)
    add_definitions(-DHAS_SYNTAX_RESOURCE)
endif()

# skip standard paths?
if (NO_STANDARD_PATHS)
    add_definitions(-DNO_STANDARD_PATHS)
endif()

# Set Qt deprecation warnings
add_definitions(-DQT_DISABLE_DEPRECATED_UP_TO=0x061000)

#
# Actually build the stuff
#
include_directories(${CMAKE_CURRENT_BINARY_DIR})
add_subdirectory(data)
add_subdirectory(src)
if(TARGET Qt6::Gui)
    add_subdirectory(examples)
    if (BUILD_TESTING)
        add_subdirectory(autotests)
    endif()
endif()

#
# CMake package config file generation
#
set(CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/KF6SyntaxHighlighting")

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KF6SyntaxHighlightingConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KF6SyntaxHighlightingConfig.cmake"
    INSTALL_DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
)

install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/KF6SyntaxHighlightingConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/KF6SyntaxHighlightingConfigVersion.cmake"
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel)

if(TARGET KF6SyntaxHighlighting)
    install(EXPORT KF6SyntaxHighlightingTargets
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        FILE KF6SyntaxHighlightingTargets.cmake
        NAMESPACE KF6::)
endif()

install(FILES "${ksyntaxhighlighting_version_header}"
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/KSyntaxHighlighting"
        COMPONENT Devel)

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)

# add target to update kate-editor.org syntax page + update site
add_custom_target(update_kate_editor_org
    COMMAND "${PERL_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/utils/update-kate-editor-org.pl" "${CMAKE_SOURCE_DIR}" "${CMAKE_BINARY_DIR}")
