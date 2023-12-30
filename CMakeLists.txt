if (POLICY CMP0114)
    # 'ExternalProject' step targets fully adopt their steps, support new CMake build system
    cmake_policy(SET CMP0114 NEW)
endif()

set(TARGET DirectXCompiler)

include(ExternalProject)

if(WIN32)
    add_library(${TARGET} STATIC src/d3dcomp.cpp)
else()
    add_library(${TARGET} STATIC src/dummy.cpp)
endif()

if(WIN32)

    if(${CMAKE_SIZEOF_VOID_P} STREQUAL "4")
        set(WIN_ARCH "x86")
        set(BINARIES_MD5 126d272b93bf866d2752e0dab9cef5a8)
    elseif(${CMAKE_SIZEOF_VOID_P} STREQUAL "8")
        set(WIN_ARCH "x64")
        set(BINARIES_MD5 b021edca36b29b2b4a1b770ceac840a6)
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
    set(BINARIES_MD5 f1f1e00c6605066e86e45ecc77904d53)

else() # Linux

    set(BINARIES_DIR "${CMAKE_CURRENT_SOURCE_DIR}/binaries/Linux")
    set(BINARIES_MD5 267c9e7efece17b931826fb5cc287e3a)

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
