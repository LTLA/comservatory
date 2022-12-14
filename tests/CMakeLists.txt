include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/afd902e992b720d1b3e106bc5e425a5768872265.zip
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

set(CMAKE_CXX_STANDARD 17)

enable_testing()

add_executable(
    libtest
    src/convert_number.cpp
    src/convert_boolean.cpp
    src/convert_complex.cpp
    src/convert_string.cpp
    src/Field.cpp
    src/Parser.cpp
    src/TextReader.cpp
)

target_link_libraries(
    libtest
    gtest_main
    gmock_main
    comservatory
)

find_package(ZLIB)
if(ZLIB_FOUND)
    target_compile_definitions(libtest PRIVATE COMSERVATORY_USE_ZLIB=1)
    target_link_libraries(libtest ZLIB::ZLIB)
    target_sources(libtest PRIVATE src/GzipReader.cpp)
endif() 

include(CheckIncludeFiles)
check_include_files(filesystem HAVE_CXX_FS)
if (NOT HAVE_CXX_FS) 
    target_link_libraries(libtest stdc++fs) 
endif()

set(CODE_COVERAGE OFF CACHE BOOL "Enable coverage testing")
if(CODE_COVERAGE AND CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(libtest PRIVATE -O0 -g --coverage)
    target_link_options(libtest PRIVATE --coverage)
endif()

include(GoogleTest)
gtest_discover_tests(libtest)
