CMAKE_MINIMUM_REQUIRED(VERSION 2.8.8)
PROJECT(OSQUERY)

if(APPLE)
  set(OS_COMPILE_FLAGS "-std=c++11 -stdlib=libc++")
  set(OS_WHOLELINK_PRE "-Wl,-all_load")
  set(OS_WHOLELINK_POST "")
else()
  if(EXISTS "/etc/redhat-release")
    set(CENTOS TRUE)
  else()
    set(UBUNTU TRUE)
  endif()

  set(OS_COMPILE_FLAGS "-std=c++11")
  set(OS_WHOLELINK_PRE "-Wl,-whole-archive")
  set(OS_WHOLELINK_POST "-Wl,-no-whole-archive")
endif()

if(APPLE)
  message("-- Building for OS X")
elseif(UBUNTU)
  message("-- Building for Ubuntu")
elseif(CENTOS)
  message("-- Building for CentOS")
endif()

if(BUILD_SHARED_LIBS)
  set(USER_COMPILE_FLAGS "-fPIC")
else()
  # Building/linking statically (no libosquery)
  set(USER_COMPILE_FLAGS "-static")
endif()

set(OS_COMPILE_FLAGS "${OS_COMPILE_FLAGS} ${USER_COMPILE_FLAGS}")

SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMake" ${CMAKE_MODULE_PATH})
FIND_PACKAGE(Gtest REQUIRED)
FIND_PACKAGE(Sqlite3 REQUIRED)
FIND_PACKAGE(BZip2 REQUIRED)
FIND_PACKAGE(Boost 1.55.0 REQUIRED)
FIND_PACKAGE(Glog REQUIRED)
FIND_PACKAGE(Gflags REQUIRED)
FIND_PACKAGE(Thrift 0.9.1 REQUIRED)
FIND_PACKAGE(Readline REQUIRED)
FIND_PACKAGE(OpenSSL REQUIRED)
FIND_PACKAGE(Threads REQUIRED)
FIND_PACKAGE(Snappy REQUIRED)
FIND_PACKAGE(RocksDB REQUIRED)
FIND_PACKAGE(Crypto REQUIRED)
FIND_PACKAGE(Dl REQUIRED)
if(NOT APPLE)
  FIND_PACKAGE(Unwind REQUIRED)
  FIND_PACKAGE(LZMA REQUIRED)
endif()

INCLUDE_DIRECTORIES("${CMAKE_SOURCE_DIR}/include")
INCLUDE_DIRECTORIES("${CMAKE_SOURCE_DIR}")
INCLUDE_DIRECTORIES("/usr/local/include")
LINK_DIRECTORIES("/usr/local/lib")

ENABLE_TESTING()

ADD_SUBDIRECTORY(osquery)
ADD_SUBDIRECTORY(tools)

# make docs
FIND_PACKAGE(Doxygen)
if(DOXYGEN_FOUND)
  ADD_CUSTOM_TARGET(
    docs
    ${DOXYGEN_EXECUTABLE} ${CMAKE_SOURCE_DIR}/Doxyfile
    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    COMMENT "Generating API documentation with Doxygen" VERBATIM
  )
endif(DOXYGEN_FOUND)

# make format-all
ADD_CUSTOM_TARGET(
  format-all
  find osquery include tools \( -name "*.h" -o -name "*.cpp" -o -name "*.mm" \) -exec clang-format -i {} +
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  COMMENT "Formatting all osquery code with clang-format" VERBATIM
)

ADD_CUSTOM_TARGET(
  format
  python tools/git-clang-format.py
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  COMMENT "Formatting code staged code changes with clang-format" VERBATIM
)

