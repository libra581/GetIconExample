cmake_minimum_required(VERSION 3.16)
project(glib_example)

find_package(Qt5 COMPONENTS Widgets Svg REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(GLIB glib-2.0 gio-2.0 gobject-2.0)
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} ${GLIB_LDFLAGS}
    ${GTK3_LIBRARIES}
    ${QT_CORE_PRIVATE}
    Qt5::Widgets
    Qt5::Svg)

target_include_directories(${PROJECT_NAME} PRIVATE ${GLIB_INCLUDE_DIRS} ${GTK3_INCLUDE_DIRS})

set_target_properties(${PROJECT_NAME} PROPERTIES AUTOMOC TRUE)

target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)

custom_install_target(${PROJECT_NAME})