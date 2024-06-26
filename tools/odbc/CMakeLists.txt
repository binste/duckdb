cmake_minimum_required(VERSION 3.12.0)

find_package(ODBC)

if(NOT ODBC_FOUND)
  message(FATAL_ERROR "No ODBC found; you need to install unixodbc-dev.")
endif()

set(LINK_LIB_LIST "")

if(ODBC_FOUND)
  find_library(ODBCINST_LIB NAMES odbcinst libodbcinst)
  if(ODBCINST_LIB)
    list(APPEND LINK_LIB_LIST ${ODBCINST_LIB})
    add_definitions(-DODBC_LINK_ODBCINST)
    message(STATUS "Found ODBCINST_LIB: " ${ODBCINST_LIB})
  endif()

  if(WIN32)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/bin)
    add_subdirectory(winsetup)
    list(APPEND LINK_LIB_LIST $<$<PLATFORM_ID:Windows>:odbccp32>)
    list(APPEND LINK_LIB_LIST
         $<$<PLATFORM_ID:Windows>:legacy_stdio_definitions>)
  endif()
endif()

add_definitions(-DNO_FRAMEWORKS)

include_directories(${ODBC_INCLUDE_DIRS} include)

add_subdirectory(src)

set(ALL_OBJECT_FILES ${ALL_OBJECT_FILES})

add_library(duckdb_odbc SHARED ${ALL_OBJECT_FILES} duckdb_odbc.def)

set_target_properties(duckdb_odbc PROPERTIES DEFINE_SYMBOL "DUCKDB_ODBC_API")
target_link_libraries(duckdb_odbc ${LINK_LIB_LIST} duckdb_static)

if(NOT CLANG_TIDY)
  add_subdirectory(test)
endif()
