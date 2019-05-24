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

list(APPEND CONAN_OPTIONS "qt:commercial=False") # opensource
list(APPEND CONAN_OPTIONS "qt:shared=True") # Shared, not static
list(APPEND CONAN_OPTIONS "qt:GUI=True") # Support for GUI applications
list(APPEND CONAN_OPTIONS "qt:widgets=True") # Support for GUI applications (Widgets are the base classes we use)

list(APPEND CONAN_OPTIONS "qt:openssl=True") # For QNetwork
list(APPEND CONAN_OPTIONS "qt:with_pcre2=True")
list(APPEND CONAN_OPTIONS "qt:with_harfbuzz=True")

# Database stuff, we don't use QSql at all actuallly
list(APPEND CONAN_OPTIONS "qt:with_sqlite3=False") # sqlite
list(APPEND CONAN_OPTIONS "qt:with_mysql=False")  # MySQL
list(APPEND CONAN_OPTIONS "qt:with_pq=False")     # PostgreSQL
list(APPEND CONAN_OPTIONS "qt:with_odbc=False")   # Oracle Database

# I think that's why produces Qt Positioning
list(APPEND CONAN_OPTIONS "qt:qtlocation=True")

# Stuff we definitely want to enable because it's disabled, and we are actively using find_package to find them below
list(APPEND CONAN_OPTIONS "qt:qtwebengine=True")
list(APPEND CONAN_OPTIONS "qt:qtwebview=True")
list(APPEND CONAN_OPTIONS "qt:qtwebchannel=True")

list(APPEND CONAN_OPTIONS "qt:qttools=False")

list(APPEND CONAN_OPTIONS "qt:qtmultimedia=False")
# Already turned off by qtmultimedia=False
list(APPEND CONAN_OPTIONS "qt:with_sdl2=False")
list(APPEND CONAN_OPTIONS "qt:with_openal=False")

list(APPEND CONAN_OPTIONS "qt:qtdoc=False")
list(APPEND CONAN_OPTIONS "qt:qtsensors=False")
list(APPEND CONAN_OPTIONS "qt:qtconnectivity=False")
list(APPEND CONAN_OPTIONS "qt:qtwayland=False")
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
# Deal with qrc files automatically
set(CMAKE_AUTORCC ON)

#get_target_property(QT_INCLUDE_DIRS CONAN_PKG::qt INTERFACE_INCLUDE_DIRECTORIES)
#get_target_property(QT_LIB_DIRS CONAN_PKG::qt INTERFACE_LIBRARY_DIRECTORIES)
#message("QT_LIB_DIRS=${QT_LIB_DIRS}")
message("CONAN_LIB_DIRS_QT=${CONAN_LIB_DIRS_QT}")

find_package(Qt5Core 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Core)

find_package(Qt5Widgets 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Widgets)

# find_package(Qt5Sql 5.12.3 CONFIG REQUIRED)
# list(APPEND QT_LIBS Qt5::Sql)

find_package(Qt5Network 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Network)

find_package(Qt5Xml 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Xml)

find_package(Qt5Concurrent 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Concurrent)

find_package(Qt5PrintSupport 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::PrintSupport)

find_package(Qt5Gui 5.12.3 CONFIG REQUIRED)
list(APPEND QT_LIBS Qt5::Gui)

if( WIN32 )
  find_package(Qt5WinExtras 5.12.3 CONFIG REQUIRED)
  list(APPEND QT_LIBS Qt5::WinExtras)
endif()

find_package(Qt5Quick 5.12.3 CONFIG REQUIRED)

find_package(Qt5QuickWidgets 5.12.3 CONFIG REQUIRED)

find_package(Qt5Qml 5.12.3 CONFIG REQUIRED)

find_package(Qt5Positioning 5.12.3 CONFIG REQUIRED) # > Missing!

find_package(Qt5WebEngine 5.12.3 CONFIG REQUIRED) # > Missing!
list(APPEND QT_WEB_LIBS Qt5::WebEngine)
list(APPEND QT_WEB_LIBS Qt5::WebEngineCore)

find_package(Qt5WebEngineWidgets 5.12.3 CONFIG REQUIRED) # > Missing!
list(APPEND QT_WEB_LIBS Qt5::WebEngineWidgets)

find_package(Qt5WebChannel 5.12.3 CONFIG REQUIRED)
list(APPEND QT_WEB_LIBS Qt5::WebChannel)

# add_definitions(${Qt5Widgets_DEFINITIONS} ${QtQml_DEFINITIONS} ${${Qt5Quick_DEFINITIONS}})

set(${PROJECT_NAME}_SRC
    main.cpp
)

set(${PROJECT_NAME}_qrc
  qml.qrc
)

qt5_add_resources(${PROJECT_NAME}_qrcs ${${PROJECT_NAME}_qrc})

message("${PROJECT_NAME}_SRC=${${PROJECT_NAME}_SRC}")
message("${PROJECT_NAME}_qrcs=${${PROJECT_NAME}_qrcs}")


add_executable(${PROJECT_NAME} ${${PROJECT_NAME}_SRC} ${${PROJECT_NAME}_qrcs})
target_link_libraries(${PROJECT_NAME}
  Qt5::WebEngine
  Qt5::Qml )
