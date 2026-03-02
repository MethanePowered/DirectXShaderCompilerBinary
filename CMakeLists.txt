if (POLICY CMP0114)
    # 'ExternalProject' step targets fully adopt their steps, support new CMake build system
    cmake_policy(SET CMP0114 NEW)
endif()

set(TARGET DirectXCompiler)

include(ExternalProject)

if(WIN32)
    add_library(${TARGET} STATIC src/d3dcomp.cpp)
    if(MSVC)
        target_compile_options(${TARGET}
            PRIVATE
                /wd4297 # 'D3DCompileFromFile': function assumed not to throw an exception but does
        )
    endif()
else()
    add_library(${TARGET} STATIC src/dummy.cpp)
endif()

if(WIN32)

    if(${CMAKE_SIZEOF_VOID_P} STREQUAL "4")
        set(WIN_ARCH "x86")
        set(BINARIES_MD5 504f5d56cb5bd69d98320670f9dddfe5)
    elseif(${CMAKE_SIZEOF_VOID_P} STREQUAL "8")
        set(WIN_ARCH "x64")
        set(BINARIES_MD5 2669aaccaeab73465db3e923a3def521)
    else()
        message(FATAL_ERROR "Unknown architecture")
    endif()

    set(BINARIES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/binaries/Windows-${WIN_ARCH}")

    list(APPEND LIBRARIES
        "${BINARIES_DIR}/lib/dxcompiler.lib")
    list(APPEND PREREQUISITE_BINARIES_LIST
        "${BINARIES_DIR}/bin/dxcompiler.dll")

elseif(APPLE)

    set(BINARIES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/binaries/MacOS")
    set(BINARIES_MD5 ad1b4a64a6374de614e47c6300b0e665)

else() # Linux

    set(BINARIES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/binaries/Linux")
    set(BINARIES_MD5 aa77564f021bd4a61e41c58188aff27b)

endif()

ExternalProject_Add(${TARGET}Unpack
    URL ${BINARIES_DIR}.zip
    URL_MD5 ${BINARIES_MD5}
    SOURCE_DIR ${BINARIES_DIR}
    BUILD_BYPRODUCTS ${LIBRARIES}
    BUILD_IN_SOURCE TRUE
    EXCLUDE_FROM_ALL TRUE
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ""
    STEP_TARGETS build
)

add_dependencies(${TARGET} ${TARGET}Unpack-build)

set_target_properties(${TARGET} PROPERTIES PREREQUISITE_BINARIES "${PREREQUISITE_BINARIES_LIST}")

target_link_libraries(${TARGET}
    PUBLIC
        ${LIBRARIES}
)

target_include_directories(${TARGET}
    PUBLIC
        ./include
)

set_target_properties(${TARGET}
    PROPERTIES
    FOLDER Externals
)
