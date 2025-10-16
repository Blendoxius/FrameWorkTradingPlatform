cmake_minimum_required(VERSION 3.23)
project(TradingPlatform LANGUAGES CXX VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Prefer maximum warnings in development
if(MSVC)
  add_compile_options(/W4 /permissive-)
else()
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets OpenGL)

qt6_add_executable(TradingPlatform
    src/main.cpp
    src/app/Application.h
    src/app/Application.cpp
    src/theme/Theme.h
    src/theme/Theme.cpp
    src/ui/MainWindow.h
    src/ui/MainWindow.cpp
    src/ui/ChartView.h
    src/ui/ChartView.cpp
    src/ui/NavigatorPanel.h
    src/ui/NavigatorPanel.cpp
    src/ui/ShortcutDot.h
    src/ui/ShortcutDot.cpp
)

target_include_directories(TradingPlatform PRIVATE src)

target_link_libraries(TradingPlatform
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    Qt6::OpenGL
)

# Enable high-DPI scaling by default
if (WIN32)
    add_definitions(-DQT_ENABLE_HIGHDPI_SCALING)
endif()