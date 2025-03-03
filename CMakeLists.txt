cmake_minimum_required(VERSION 2.8)

set(QUICKJS_PATH ./src/jsruntime/deps/quickjs)
set(LIBUV_PATH ./src/deps/libuv)
set(LV_CONF_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src/deps/lv_conf.h CACHE INTERNAL "LV_CONF_PATH")

if (${CMAKE_BUILD_TYPE} MATCHES Debug)
    set(CMAKE_CXX_FLAGS "-O0 -Wall -g -ggdb -std=c++11")
    set(CMAKE_C_FLAGS "-O0 -Wall -g -ggdb -std=gnu99")
elseif (${CMAKE_BUILD_TYPE} MATCHES Release)
    set(CMAKE_CXX_FLAGS "-O0 -Wall -std=c++11")
    set(CMAKE_C_FLAGS "-O0 -Wall -std=gnu99")
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin" AND ${CMAKE_BUILD_PLATFORM} MATCHES arm)
    set(MACOSX TRUE)
endif()

if (${CMAKE_BUILD_TYPE} MATCHES Debug AND (${CMAKE_BUILD_PLATFORM} MATCHES x86 OR MACOSX))
add_definitions(-DIS_SIM)
elseif(${CMAKE_BUILD_PLATFORM} MATCHES arm)
add_definitions(-DIS_DEVICE)
endif()

add_subdirectory(./src/deps/libuv EXCLUDE_FROM_ALL)
add_subdirectory(./src/jsruntime)
add_subdirectory(./src/deps/lvgl EXCLUDE_FROM_ALL)
add_subdirectory(./src/deps/lv_drivers EXCLUDE_FROM_ALL)
add_subdirectory(./src/render)

include_directories(
    ./src
    ./src/deps
    ./src/deps/lvgl
    ${LIBUV_PATH}/include
    ${QUICKJS_PATH}/include
    ./src/utils
    ./src/engine
    ./src/jsruntime
    ./src/render
    ${CMAKE_CURL_INCLUDE_DIRS}
)

if (${CMAKE_BUILD_TYPE} MATCHES Debug AND (${CMAKE_BUILD_PLATFORM} MATCHES x86 OR MACOSX))
    find_package(SDL2 REQUIRED 
        SDL2
    )
    find_package(OpenSSL REQUIRED)
    message(OPENSSL_SSL_LIBRARIES is ${OPENSSL_SSL_LIBRARIES})
    string(STRIP ${SDL2_LIBRARIES} SDL2_LIBRARIES)
    find_package(CURL REQUIRED)
    string(STRIP ${CURL_LIBRARIES} CURL_LIBRARIES)
    add_executable(lvgljs
        ./src/utils/utils.c
        ./src/engine/hal/simulator/mouse_cursor_icon.c
        ./src/engine/hal/simulator/simulator.cpp
        ./src/engine/engine.cpp
    )
    target_include_directories(lv_drivers PUBLIC ${SDL2_INCLUDE_DIRS}/../)

    target_link_libraries(lvgljs INTERFACE OpenSSL::Crypto)
    target_link_libraries(lvgljs INTERFACE OpenSSL::SSL)

    target_link_libraries(lvgljs
        jsruntime
        ${CURL_LIBRARIES}
        uv_a
        dl
        pthread
        render
        lvgl
        lv_drivers
        ${SDL2_LIBRARIES}
    )
elseif (${CMAKE_BUILD_PLATFORM} MATCHES arm)
    add_executable(lvgljs
        ./src/utils/utils.c
        ./src/engine/hal/device/device.cpp
        ./src/engine/engine.cpp
    )
    target_link_libraries(lvgljs
        jsruntime
        ${CMAKE_CURL_LIB}
        ${CMAKE_CRYPTO_LIB}
        ${CMAKE_SSL_LIB}
        uv_a
        dl
        pthread
        render
        lvgl
        lv_drivers
    )
endif()