cmake_minimum_required(VERSION 3.23)

project(stb LANGUAGES C CXX)

if (NOT CMAKE_C_STANDARD)
    set(CMAKE_C_STANDARD 99)
    set(CMAKE_C_STANDARD_REQUIRED ON)
endif()

option(STB_BUILD_SHARED "Build shared lib" OFF)

option(STB_IMAGE_WRITE_IMPLEMENTATION "Build stb_image_write" ON)
option(STB_IMAGE_IMPLEMENTATION "Build stb_image_write" ON)

set(STB_SOURCES "stb_implementation.c")

if(STB_IMAGE_WRITE_IMPLEMENTATION)
    list(APPEND STB_HEADERS "stb_image_write.h")
    list(APPEND STB_IMPLEMENTATION_DEFINES "STB_IMAGE_WRITE_IMPLEMENTATION")
endif()

if(STB_IMAGE_IMPLEMENTATION)
    list(APPEND STB_HEADERS "stb_image.h")
    list(APPEND STB_IMPLEMENTATION_DEFINES "STB_IMAGE_IMPLEMENTATION")
endif()

if (STB_BUILD_SHARED OR BUILD_SHARED_LIBS)
    add_library(stb SHARED)
else()
    add_library(stb STATIC)
endif()

add_library(stb::stb ALIAS stb)

target_sources(stb PRIVATE ${STB_SOURCES})
target_sources(stb PRIVATE FILE_SET HEADERS FILES ${STB_HEADERS})

target_include_directories(stb INTERFACE "${CMAKE_CURRENT_BINARY_DIR}/include")

# generate implementation file
if (NOT STB_HEADERS)
    message(FATAL_ERROR "No library implementation is selected.")
endif()

set(STB_IMPLEMENTATION_IN "${PROJECT_SOURCE_DIR}/cmake/stb_implementation.c.in")
set(STB_IMPLEMENTATION_OUT "${CMAKE_CURRENT_BINARY_DIR}/stb_implementation.c")

list(TRANSFORM STB_IMPLEMENTATION_DEFINES REPLACE "(.+)" "#define \\1")
string(REPLACE ";" "\n" STB_IMPLEMENTATION_DEFINES "${STB_IMPLEMENTATION_DEFINES}")

list(TRANSFORM STB_HEADERS REPLACE "(.+[.]h)$" "#include \"\\1\"" OUTPUT_VARIABLE STB_IMPLEMENTATION_INCLUDES)
string(REPLACE ";" "\n" STB_IMPLEMENTATION_INCLUDES "${STB_IMPLEMENTATION_INCLUDES}")

message(STATUS "Generating implementation file: ${STB_IMPLEMENTATION_OUT}")
configure_file("${STB_IMPLEMENTATION_IN}" "${STB_IMPLEMENTATION_OUT}" @ONLY)

# copy required headers
foreach(HEADER_FILE ${STB_HEADERS})
    message(STATUS "Copying header file to build dir: ${HEADER_FILE}")
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/${HEADER_FILE}" "${CMAKE_CURRENT_BINARY_DIR}/include/stb/${HEADER_FILE}" COPYONLY)
endforeach()
