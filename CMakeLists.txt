# <Copyright Holder>. Copyright (C) <Copyright Year(s)>. <License>.

#
# Project Properties
#
cmake_minimum_required (VERSION 2.6.2)
project (projectname)
set (APPLICATION_NAME "Project Name")
set (APPLICATION_CODENAME "${PROJECT_NAME}")
set (APPLICATION_COPYRIGHT_YEARS "2013")
set (APPLICATION_VERSION_MAJOR 1)
set (APPLICATION_VERSION_MINOR 0)
set (APPLICATION_VERSION_PATCH 0)
set (APPLICATION_VERSION_TYPE SNAPSHOT)
set (APPLICATION_VERSION_STRING "${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}-${APPLICATION_VERSION_TYPE}")
set (APPLICATION_VENDOR_ID "com.vacuum-cleaner")
set (APPLICATION_VENDOR_NAME "Vacuum Cleaner")
set (APPLICATION_VENDOR_URL "vacuumcleaner.com")
set (APPLICATION_ID "${APPLICATION_VENDOR_ID}.${PROJECT_NAME}")


#
# Debugging Options
#
set (CMAKE_VERBOSE_MAKEFILE 0) # Use 1 for debugging, 0 for release

#
# Project Output Paths
#
set (EXECUTABLE_OUTPUT_PATH "${PROJECT_BINARY_DIR}/bin")
set (LIBRARY_OUTPUT_PATH "${PROJECT_BINARY_DIR}/lib")

#
# Project Search Paths
#
list (APPEND CMAKE_PREFIX_PATH "${PROJECT_SOURCE_DIR}")
list (APPEND CMAKE_PREFIX_PATH "${PROJECT_SOURCE_DIR}/tools")
list (APPEND CMAKE_PREFIX_PATH "${PROJECT_SOURCE_DIR}/thirdparty")
set (CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/tools/share/cmake")
include_directories ("${PROJECT_SOURCE_DIR}/include")

#
# Locate Project Prerequisites 
#
set (Boost_ADDITIONAL_VERSIONS "1.39" "1.39.0" "1.40" "1.40.0" "1.41" "1.41.0" "1.42" "1.42.0" "1.43" "1.43.0" "1.44" "1.44.0" "1.45" "1.45.0" "1.46" "1.46.0" "1.47" "1.47.0" "1.48" "1.48.0" "1.49" "1.49.0" "1.50" "1.50.0")
find_package (Boost 1.37 COMPONENTS "date_time" "filesystem" "graph" "iostreams" "program_options" "regex" "serialization" "signals" "system" "thread" "wserialization" REQUIRED)
include_directories (${Boost_INCLUDE_DIRS})
link_directories (${Boost_LIBRARY_DIRS})
find_package (Qt4 REQUIRED)
include (UseQt4)
find_package (Log4Cxx REQUIRED)
include_directories (${Log4Cxx_INCLUDE_DIRS})
link_directories (${Log4Cxx_LIBRARY_DIRS})
find_package (UnitTestPlusPlus REQUIRED)
include_directories (${UnitTestPlusPlus_INCLUDE_DIRS})
link_directories (${UnitTestPlusPlus_LIBRARY_DIRS})

#
# Special hack for switching to 32-bit mode on Snow
# Leopard if Qt has been compiled for 32-bit, only.
#
if (APPLE)
  if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    execute_process (
      COMMAND file "${QT_QTCORE_LIBRARY}"
      COMMAND grep "x86_64" -c
      OUTPUT_VARIABLE QT4_SUPPORTS_X86_64_OUTPUT
    )
    if (NOT QT4_SUPPORTS_X86_64_OUTPUT MATCHES "0")
      set (QT4_SUPPORTS_X86_64 1)
    else (NOT QT4_SUPPORTS_X86_64_OUTPUT MATCHES "0")
      set (QT4_SUPPORTS_X86_64 0)
    endif (NOT QT4_SUPPORTS_X86_64_OUTPUT MATCHES "0")

    if (QT4_SUPPORTS_X86_64)
      set (CMAKE_OSX_ARCHITECTURES x86_64)
    else (QT4_SUPPORTS_X86_64)
      set (CMAKE_OSX_ARCHITECTURES i386)
    endif (QT4_SUPPORTS_X86_64)
  endif (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
endif ()


#
# Configure Files
#
file (GLOB_RECURSE CONFIGINPUTS1 include/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS2 include/*.h.in.cmake)
file (GLOB_RECURSE CONFIGINPUTS3 src/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS4 src/*.h.in.cmake)
file (GLOB_RECURSE CONFIGINPUTS5 test/*.in.h.cmake)
file (GLOB_RECURSE CONFIGINPUTS6 test/*.h.in.cmake)
list (APPEND CONFIGINPUTS "${CONFIGINPUTS1}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS2}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS3}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS4}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS5}")
list (APPEND CONFIGINPUTS "${CONFIGINPUTS6}")
foreach (CONFIGINPUT ${CONFIGINPUTS})
  string (REPLACE ".in.h.cmake" ".h" CONFIGOUTPUT1 "${CONFIGINPUT}")
  string (REPLACE ".h.in.cmake" ".h" CONFIGOUTPUT2 "${CONFIGOUTPUT1}")
  configure_file ("${CONFIGINPUT}" "${CONFIGOUTPUT2}")
endforeach ()

#
# Add Build Targets
#
add_subdirectory (src)
add_subdirectory (test)

#
# Add Install Targets
#
if (EXISTS "${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}" AND IS_DIRECTORY "${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}")
  install (DIRECTORY "${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}" DESTINATION "include")
endif ()

#
# Add Documentation Targets
#
include (DocumentationTargets)
