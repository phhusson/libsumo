cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}")

# check for c++11 with gcc
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)

if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

# enable warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# main library
find_package(Threads REQUIRED)
include_directories(lib)
add_library(sumo
		lib/decode.cpp
		lib/control.cpp
		lib/common.cpp
		lib/realtime.cpp
		lib/image.cpp
)
target_link_libraries(sumo ${CMAKE_THREAD_LIBS_INIT})

if(UNIX AND NOT APPLE)
	find_library(RT rt)
    target_link_libraries(sumo ${RT})
endif()

# build analyzer only if PCAP was found
find_package(Pcap)
if(PCAP_FOUND)
	add_executable(analyze analyze/network.cpp analyze/main.cpp)
	target_link_libraries(analyze ${PCAP_LIBRARIES} sumo)
endif()

add_executable(test_sumo test/test.cpp)
target_link_libraries(test_sumo sumo)

# qsumo
set(CMAKE_AUTOMOC ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
#set(CMAKE_AUTOUIC ON) # works only with cmake >= 3.0

find_package(Qt5 COMPONENTS Core Gui Widgets REQUIRED)

set(qsumo_SRCS qsumo/qsumo.cpp qsumo/sumo-widget.cpp)

QT5_WRAP_UI(ui_widget.h qsumo/widget.ui)

add_executable(qs ${qsumo_SRCS} ui_widget.h)

qt5_use_modules(sumo Widgets)
target_link_libraries(qs sumo)


