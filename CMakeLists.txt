cmake_minimum_required(VERSION 3.30)

project(vcpkg-gdal-hello-world LANGUAGES CXX)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

find_package(GDAL CONFIG REQUIRED)

include(compiler_flags)

add_subdirectory(src)
