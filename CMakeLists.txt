# we (Dreamler) have replaced all occurances of ${PROJECT_SOURCE_DIR} with ${CMAKE_CURRENT_SOURCE_DIR}
# we (Dreamler) have replaced all occurances of ${CMAKE_BINARY_DIR} with ${CMAKE_CURRENT_SOURCE_DIR}
#message("PROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}") # D:/emil/gitrepos/dreamler-client/cpp
#message("CMAKE_CURRENT_SOURCE_DIR=${CMAKE_CURRENT_SOURCE_DIR}") # D:/emil/gitrepos/dreamler-client/cpp/deps/poco
#message(FATAL_ERROR "stop")
#findstr /L /S /P /C:"target_link_libraries" *.txt

# POCO_BUILD_TYPE
# POCO_STATIC
# POCO_UNBUNDLED
# POCO_NO_LOCALE
#
# ENABLE_{COMPONENT}
# ENABLE_TESTS
# ENABLE_SAMPLES
# DISABLE_INTERNAL_OPENSSL

cmake_minimum_required(VERSION 3.2.0)

# project(Poco)
set(SUBPROJECT_NAME Poco)

option(POCO_VERBOSE_MESSAGES "Enable informational messages during configure" ON)

file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/libversion" SHARED_LIBRARY_VERSION)

# Read the version information from the VERSION file
file (STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/VERSION" PACKAGE_VERSION )
if(POCO_VERBOSE_MESSAGES)
    message(STATUS "Poco package version: ${PACKAGE_VERSION}")
endif()
string(REGEX REPLACE "([0-9]+)\\.[0-9]+\\.[0-9]+.*" "\\1" CPACK_PACKAGE_VERSION_MAJOR ${PACKAGE_VERSION})
string(REGEX REPLACE "[0-9]+\\.([0-9])+\\.[0-9]+.*" "\\1" CPACK_PACKAGE_VERSION_MINOR ${PACKAGE_VERSION})
string(REGEX REPLACE "[0-9]+\\.[0-9]+\\.([0-9]+).*" "\\1" CPACK_PACKAGE_VERSION_PATCH ${PACKAGE_VERSION})

set(COMPLETE_VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH})
set(RELEASE_NAME "Unstable-trunk")
set(PROJECT_VERSION ${COMPLETE_VERSION})
set(CMAKE_C_STANDARD 99)

# Put the libaries and binaries that get built into directories at the
# top of the build tree rather than in hard-to-find leaf
# directories. This simplifies manual testing and the use of the build
# tree rather than installed Boost libraries.

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
# Windows DLLs are "runtime" for CMake. Output them to "bin" like the Visual Studio projects do.
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin)

# Append our module directory to CMake
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
#list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

#################################################################################
# Setup C/C++ compiler options
#################################################################################

option(DISABLE_CPP14 "Disable C++14 if available" OFF)

if (DISABLE_CPP14)
    add_definitions(-DPOCO_DISABLE_CPP14)
else()
  # C++14 compiler flags
  include(CXX1x)
  check_for_cxx14_compiler(CXX14_COMPILER)

  # If a C++14 compiler is available, then set the appropriate flags
  if(CXX14_COMPILER)
    enable_cxx14()
  endif()
endif()

if(NOT MSVC_IDE)
    if(NOT CMAKE_BUILD_TYPE)
      set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
          "Choose the type of build, options are: None Debug Release" FORCE)
    endif()
    message(STATUS "Setting Poco build type - ${CMAKE_BUILD_TYPE}")
endif()

if (CMAKE_BUILD_TYPE STREQUAL "")
    set( CMAKE_BUILD_TYPE "RelWithDebInfo" )
endif()

# Include some common macros to simpilfy Poco CMake files
include(PocoMacros)

# Allow enabling and disabling components
option(ENABLE_CPPUNIT "Enable CppUnit" ON)
option(ENABLE_XML "Enable XML" ON)
option(ENABLE_JSON "Enable JSON" ON)
option(ENABLE_MONGODB "Enable MongoDB" OFF)
option(ENABLE_PDF "Enable PDF" OFF)
option(ENABLE_UTIL "Enable Util" ON)
option(ENABLE_NET "Enable Net" ON)

# allow disabling of internally built OpenSSL# (see below for details)
# if POCO pre-built OpenSSL directory is found, and DISABLE_INTERNAL_OPENSSL=OFF,
# the internal OpenSSL build will be used
option(DISABLE_INTERNAL_OPENSSL "Disable internal OpensSSL binaries use" OFF)
set(USING_INTERNAL_OPENSSL false)

if(MSVC)
  if((EXISTS "${CMAKE_CURRENT_SOURCE_DIR}\\openssl\\VS_120\\") AND NOT DISABLE_INTERNAL_OPENSSL)
    set(USING_INTERNAL_OPENSSL true)
    set(OPENSSL_ROOT_DIR "${CMAKE_CURRENT_SOURCE_DIR}\\openssl\\VS_120\\")
    ## windows xcopy does not tolerate forward-slash, must change this
    STRING(REGEX REPLACE "/" "\\\\" OPENSSL_ROOT_DIR ${OPENSSL_ROOT_DIR})
  endif()
endif(MSVC)

set(OPENSSL_FOUND false)
if(USING_INTERNAL_OPENSSL)
  message(STATUS "Using OpenSSL from ${OPENSSL_ROOT_DIR}")
  set(OPENSSL_INCLUDE_DIR "${OPENSSL_ROOT_DIR}include\\")
  set(OPENSSL_FOUND true)
else()
  find_package(OpenSSL QUIET)
  if(OPENSSL_FOUND)
    add_definitions( -DPOCO_EXTERNAL_OPENSSL)
    message(STATUS "Using OpenSSL from ${OPENSSL_ROOT_DIR}")
  endif()
endif()
if(OPENSSL_FOUND)
    option(ENABLE_NETSSL "Enable NetSSL" ON)
    option(ENABLE_CRYPTO "Enable Crypto" ON)
else()
    option(ENABLE_NETSSL "Enable NetSSL" OFF)
    option(ENABLE_CRYPTO "Enable Crypto" OFF)
endif()
if(WIN32)
    option(ENABLE_NETSSL_WIN "Enable NetSSL Windows" ON)
else()
    set(ENABLE_NETSSL_WIN OFF)
endif()
option(ENABLE_SQL "Enable SQL" ON)
option(ENABLE_SQL_SQLITE "Enable SQL SQlite" ON)
option(ENABLE_SQL_MYSQL "Enable SQL MySQL" OFF)
option(ENABLE_SQL_POSTGRESQL "Enable SQL PosgreSQL" OFF)
option(ENABLE_SQL_ODBC "Enable SQL ODBC" OFF)
option(ENABLE_SEVENZIP "Enable SevenZip" OFF)
option(ENABLE_ZIP "Enable Zip" OFF)
option(ENABLE_APACHECONNECTOR "Enable ApacheConnector" OFF)
option(ENABLE_CPPPARSER "Enable C++ parser" OFF)
option(ENABLE_POCODOC "Enable Poco Documentation Generator" OFF)
option(ENABLE_PAGECOMPILER "Enable PageCompiler" OFF)
option(ENABLE_PAGECOMPILER_FILE2PAGE "Enable File2Page" OFF)
option(ENABLE_REDIS "Enable Redis" OFF)

option(ENABLE_TESTS
  "Set to OFF|ON (default is OFF) to control build of POCO tests" OFF)

option(ENABLE_SAMPLES
  "Set to OFF|ON (default is OFF) to control build of POCO samples" OFF)

option(POCO_STATIC
  "Set to OFF|ON (default is OFF) to control build of POCO as STATIC library" ON)

option(POCO_UNBUNDLED
  "Set to OFF|ON (default is OFF) to control linking dependencies as external" OFF)

if(MSVC)
    option(POCO_MT
      "Set to OFF|ON (default is OFF) to control build of POCO as /MT instead of /MD" OFF)

    option(ENABLE_MSVC_MP
      "Set to OFF|ON (default is OFF) to control parallel build of POCO with MSVC" OFF)
endif()

# Uncomment from next two lines to force static or dynamic library, default is autodetection
if(POCO_STATIC)
    set(LIB_MODE_DEFINITIONS -DPOCO_STATIC)
    set(LIB_MODE STATIC)
    if(POCO_VERBOSE_MESSAGES)
        message(STATUS "Building static libraries")
    endif()
else(POCO_STATIC)
    set(LIB_MODE SHARED)
    if(POCO_VERBOSE_MESSAGES)
        message(STATUS "Building dynamic libraries")
    endif()
endif(POCO_STATIC)

if (ENABLE_TESTS)
    include(CTest)
    enable_testing()
    if(POCO_VERBOSE_MESSAGES)
        message(STATUS "Building with tests")
    endif()
elseif (POCO_VERBOSE_MESSAGES)
    message(STATUS "Building without tests")
endif ()

if (ENABLE_SAMPLES)
    if(POCO_VERBOSE_MESSAGES)
        message(STATUS "Building with samples")
    endif()
elseif (POCO_VERBOSE_MESSAGES)
    message(STATUS "Building without samples")
endif ()

if (POCO_UNBUNDLED)
    add_definitions( -DPOCO_UNBUNDLED)
    if(POCO_VERBOSE_MESSAGES)
        message(STATUS "Build with using external sqlite, libz, pcre, expat ...")
    endif()
elseif (POCO_VERBOSE_MESSAGES)
    message(STATUS "Build with using internal copy of sqlite, libz, pcre, expat, ...")
endif ()

include(DefinePlatformSpecifc)

# Collect the built libraries and include dirs, the will be used to create the PocoConfig.cmake file
set(Poco_COMPONENTS "")
set(Poco_ALL_INCLUDE_DIRS "") # added by Emil 2018-02-14

# Pthreads/threads support
find_package(Threads REQUIRED)

if (ENABLE_TESTS OR ENABLE_CPPUNIT)
    add_subdirectory_emil(CppUnit)
    list(APPEND Poco_COMPONENTS "CppUnit")
endif ()

add_subdirectory_emil(Foundation)
if(ENABLE_XML)
    add_subdirectory_emil(XML)
    list(APPEND Poco_COMPONENTS "XML")
endif()
if(ENABLE_JSON)
    add_subdirectory_emil(JSON)
    list(APPEND Poco_COMPONENTS "JSON")
endif()
if(ENABLE_MONGODB)
    add_subdirectory_emil(MongoDB)
    list(APPEND Poco_COMPONENTS "MongoDB")
endif()
if(ENABLE_PDF)
    add_subdirectory_emil(PDF)
    list(APPEND Poco_COMPONENTS "PDF")
endif()
if(ENABLE_UTIL)
    add_subdirectory_emil(Util)
    list(APPEND Poco_COMPONENTS "Util")
endif()
if(ENABLE_NET)
    add_subdirectory_emil(Net)
    list(APPEND Poco_COMPONENTS "Net")
endif()

#NetSSL


if(WIN32 AND ENABLE_NETSSL_WIN)
    add_subdirectory_emil(NetSSL_Win)
    list(APPEND Poco_COMPONENTS "NetSSL_Win")
endif(WIN32 AND ENABLE_NETSSL_WIN)

if(ENABLE_NETSSL OR ENABLE_CRYPTO)
  if(NOT OPENSSL_FOUND)
    find_package(OpenSSL REQUIRED)
  endif()
  if(OPENSSL_FOUND)
    if(MSVC)
        list(APPEND OPENSSL_LIBRARIES ws2_32 Crypt32)
    endif()
    include_directories("${OPENSSL_INCLUDE_DIR}")
    if(ENABLE_NETSSL)
      add_subdirectory_emil(NetSSL_OpenSSL)
      list(APPEND Poco_COMPONENTS "NetSSL_OpenSSL")
    endif()
    if(ENABLE_CRYPTO)
      add_subdirectory_emil(Crypto)
      list(APPEND Poco_COMPONENTS "Crypto")
    endif()
  endif(OPENSSL_FOUND)
endif(ENABLE_NETSSL OR ENABLE_CRYPTO)

if(ENABLE_SQL)
    add_subdirectory_emil(SQL)
    list(APPEND Poco_COMPONENTS "SQL")
endif()
if(ENABLE_SEVENZIP)
    add_subdirectory_emil(SevenZip)
    list(APPEND Poco_COMPONENTS "SevenZip")
endif()
if(ENABLE_ZIP)
    add_subdirectory_emil(Zip)
    list(APPEND Poco_COMPONENTS "Zip")
endif()
if(ENABLE_REDIS)
    add_subdirectory_emil(Redis)
    list(APPEND Poco_COMPONENTS "Redis")
endif()

find_package(APR)
find_package(Apache2)
if(APRUTIL_FOUND AND APACHE_FOUND)
    include_directories( "${APACHE_INCLUDE_DIR}" "${APRUTIL_INCLUDE_DIR}" )
    if(ENABLE_APACHECONNECTOR)
    add_subdirectory_emil(ApacheConnector)
    list(APPEND Poco_COMPONENTS "ApacheConnector")
    endif()
endif(APRUTIL_FOUND AND APACHE_FOUND)

if(ENABLE_CPPPARSER)
    add_subdirectory_emil(CppParser)
    list(APPEND Poco_COMPONENTS "CppParser")
endif()

if(ENABLE_POCODOC)
    add_subdirectory_emil(PocoDoc)
    list(APPEND Poco_COMPONENTS "PocoDoc")
endif()

if(ENABLE_PAGECOMPILER)
    add_subdirectory_emil(PageCompiler)
    list(APPEND Poco_COMPONENTS "PageCompiler")
endif()

if(ENABLE_PAGECOMPILER_FILE2PAGE)
    add_subdirectory_emil(PageCompiler/File2Page)
    list(APPEND Poco_COMPONENTS "File2Page")
endif()


# added by Emil 2018-02-14
#message("Poco_ALL_INCLUDE_DIRS ${Poco_ALL_INCLUDE_DIRS}")
set(Poco_ALL_INCLUDE_DIRS2 "")
foreach(subdir ${Poco_ALL_INCLUDE_DIRS})
	set(subdir2 "${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/include")
	list(APPEND Poco_ALL_INCLUDE_DIRS2 ${subdir2})
endforeach()
#message("Poco_ALL_INCLUDE_DIRS2 ${Poco_ALL_INCLUDE_DIRS2}")
set(Poco_ALL_INCLUDE_DIRS ${Poco_ALL_INCLUDE_DIRS2} PARENT_SCOPE) # will be used by our caller, in ../CMakeLists.txt - note that this ONLY sets the var in PARENT scope, not in this scope!


#############################################################
# Uninstall stuff see: http://www.vtk.org/Wiki/CMake_FAQ
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

#############################################################
# Enable packaging

include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Poco Libraries")
set(CPACK_PACKAGE_VENDOR "Applied Informatics Software Engineering GmbH")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "/usr/local")

include(CPack)

#############################################################
# cmake config files

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${SUBPROJECT_NAME}/${SUBPROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_file(cmake/${SUBPROJECT_NAME}Config.cmake.in "${CMAKE_CURRENT_BINARY_DIR}/${SUBPROJECT_NAME}/${SUBPROJECT_NAME}Config.cmake" @ONLY)
install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/${SUBPROJECT_NAME}/${SUBPROJECT_NAME}Config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${SUBPROJECT_NAME}/${SUBPROJECT_NAME}ConfigVersion.cmake
    DESTINATION
        "lib/cmake/${SUBPROJECT_NAME}"
    COMPONENT
        Devel
)

# in tree build settings
#configure_file(PocoBuildTreeSettings.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/PocoBuildTreeSettings.cmake @ONLY)

if(POCO_VERBOSE_MESSAGES)
    message(STATUS "CMake ${CMAKE_VERSION} successfully configured ${SUBPROJECT_NAME} using ${CMAKE_GENERATOR} generator")
    message(STATUS "Installation target path: ${CMAKE_INSTALL_PREFIX}")

    message(STATUS "")
    message(STATUS "Generated with config types: ${CMAKE_CONFIGURATION_TYPES}")
    message(STATUS "Setting Poco build type - ${CMAKE_BUILD_TYPE}")
    message(STATUS "")

    message(STATUS "C_FLAGS:  =${CMAKE_C_FLAGS}")
    message(STATUS "CMAKE_C_FLAGS_DEBUG:=${CMAKE_C_FLAGS_DEBUG}")
    message(STATUS "CMAKE_C_FLAGS_RELEASE:=${CMAKE_C_FLAGS_RELEASE}")
    message(STATUS "CMAKE_C_FLAGS_MINSIZEREL:=${CMAKE_C_FLAGS_MINSIZEREL}")
    message(STATUS "CMAKE_C_FLAGS_RELWITHDEBINFO:=${CMAKE_C_FLAGS_RELWITHDEBINFO}")
    message(STATUS "")
    message(STATUS "")
    message(STATUS "CXX_FLAGS:=${CMAKE_CXX_FLAGS}")
    message(STATUS "CMAKE_CXX_FLAGS_DEBUG:=${CMAKE_CXX_FLAGS_DEBUG}")
    message(STATUS "CMAKE_CXX_FLAGS_RELEASE:=${CMAKE_CXX_FLAGS_RELEASE}")
    message(STATUS "CMAKE_CXX_FLAGS_MINSIZEREL:=${CMAKE_CXX_FLAGS_MINSIZEREL}")
    message(STATUS "CMAKE_CXX_FLAGS_RELWITHDEBINFO:=${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")

    foreach(component ${Poco_COMPONENTS})
        message(STATUS "Building: ${component}")
    endforeach()
endif()
