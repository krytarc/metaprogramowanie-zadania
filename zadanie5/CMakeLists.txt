cmake_minimum_required(VERSION 3.16)
project(MetaprogramowanieZadanie5 LANGUAGES CXX)

option(USE_CPP20 "Wlacz dodatkowy wariant C++20" OFF)
if(USE_CPP20)
    set(CMAKE_CXX_STANDARD 20)
else()
    set(CMAKE_CXX_STANDARD 17)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_executable(program main.cpp)
