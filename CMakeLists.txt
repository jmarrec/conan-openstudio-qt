cmake_minimum_required(VERSION 3.10)

project(conan-qt-test LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# The PROJECT_SOURCE_DIR here is OpenStudio/, which we will use instead of using CMAKE_SOURCE_DIR
set(PROJECT_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
# We will use BINARY_DIR instead of CMAKE_BINARY_DIR to allow this to be built as a submodule
set(PROJECT_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")


if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
   message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
   file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/v0.14/conan.cmake"
       "${CMAKE_BINARY_DIR}/conan.cmake")
endif()
include(${CMAKE_BINARY_DIR}/conan.cmake)


conan_check(VERSION 1.0.0 REQUIRED)

message(STATUS "RUNNING CONAN")

conan_add_remote(NAME bincrafters
  URL https://api.bintray.com/conan/bincrafters/public-conan)
conan_add_remote(NAME jmarrec
  URL https://api.bintray.com/conan/jmarrec/testing)

set(CONAN_OPTIONS "")
set(CONAN_BUILD "")

list(APPEND CONAN_OPTIONS "qt:shared=True")
list(APPEND CONAN_OPTIONS "qt:commercial=False")
list(APPEND CONAN_OPTIONS "qt:openssl=True")
list(APPEND CONAN_OPTIONS "qt:with_pcre2=True")
list(APPEND CONAN_OPTIONS "qt:with_harfbuzz=True")
list(APPEND CONAN_OPTIONS "qt:with_sqlite3=True")
list(APPEND CONAN_OPTIONS "qt:qtmultimedia=False")
list(APPEND CONAN_OPTIONS "qt:qttools=False")
list(APPEND CONAN_OPTIONS "qt:qtdoc=False")
list(APPEND CONAN_OPTIONS "qt:qtsensors=False")
list(APPEND CONAN_OPTIONS "qt:qtconnectivity=False")
list(APPEND CONAN_OPTIONS "qt:qtwayland=False")
list(APPEND CONAN_OPTIONS "qt:qtwebengine=True")
list(APPEND CONAN_OPTIONS "qt:qtpurchasing=False")

list(APPEND CONAN_BUILD "missing")

option(WITH_ICU BOOL ON)

if (WITH_ICU)
  # This recipe is the same as bincrafters', except I enabled ICU with ICU 64.2
  set(CONAN_QT "qt/5.12.3@jmarrec/testing")
  # Recipe has 2.3.0 which appears on the bincrafter repo but cannot be found...
  set(CONAN_HARFBUZZ "harfbuzz/2.4.0@bincrafters/stable")
  list(APPEND CONAN_OPTIONS "qt:with_icu=True")
else()
  set(CONAN_QT "qt/5.12.3@bincrafters/testing")
endif()

conan_cmake_run(REQUIRES
    ${CONAN_QT}
    ${CONAN_HARFBUZZ}
    BASIC_SETUP CMAKE_TARGETS NO_OUTPUT_DIRS
    OPTIONS ${CONAN_OPTIONS}
    BUILD ${CONAN_BUILD}
)

#set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -fPIC -fno-strict-aliasing")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -fno-strict-aliasing")

# all warnings
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -fPIC -fno-strict-aliasing -Winvalid-pch -Wnon-virtual-dtor")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}  -fPIC -fno-strict-aliasing -Winvalid-pch")

# Output stuff in Products
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/Products")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/Products")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/Products")


# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed
set(CMAKE_AUTOMOC ON)
# Create code from a list of Qt designer ui files
set(CMAKE_AUTOUIC ON)


#get_target_property(QT_INCLUDE_DIRS CONAN_PKG::qt INTERFACE_INCLUDE_DIRECTORIES)
#get_target_property(QT_LIB_DIRS CONAN_PKG::qt INTERFACE_LIBRARY_DIRECTORIES)
#message("QT_LIB_DIRS=${QT_LIB_DIRS}")
message("CONAN_LIB_DIRS_QT=${CONAN_LIB_DIRS_QT}")

find_package(Qt5Core CONFIG REQUIRED)
# TODO: This ends up empty!
message("Qt5=${Qt5}")

# TODO: This too!
find_package(Qt5Core 5.12.3 REQUIRED PATHS ${CONAN_LIB_DIRS_QT} NO_DEFAULT_PATH)
message("Qt5Core=${Qt5Core}")

# Tried some stuff
#set(pkg_qt CONAN_PKG::qt)
#message("CONAN_PKG::qt = ${pkg_qt}")
#message("QT_INCLUDE_DIRS=${QT_INCLUDE_DIRS}")
#message("==================================")
#message("CONAN_LIBS=${CONAN_LIBS}")

#find_package(Qt5Core CONFIG REQUIRED)
#message("Qt5Core=${Qt5Core}")

#message("CONAN_LIBS_QT=${CONAN_LIBS_QT}")

#find_package(Qt5Core 5.12.3 REQUIRED PATHS ${CONAN_LIB_DIRS_QT}/cmake/ NO_DEFAULT_PATH)
#message("Qt5Core=${Qt5Core}")

# This is the only way I have found to find libQt5Core.so
find_library(QT_LIBRARY NAMES Qt5Core PATHS ${CONAN_LIB_DIRS_QT})
message("QT_LIBRARY=${QT_LIBRARY}")

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} ${QT_LIBRARY} CONAN_PKG::qt)
