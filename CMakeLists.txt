cmake_minimum_required(VERSION 3.2)

if (NOT DEFINED ENV{GITHUB_RUN_NUMBER})
    set(ENV{GITHUB_RUN_NUMBER} 1)
endif()
project(ipfilter VERSION 0.0.$ENV{GITHUB_RUN_NUMBER})

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in ${PROJECT_SOURCE_DIR}/version.h)

add_executable(ipfilter ${PROJECT_SOURCE_DIR}/src/main.cpp)
add_library(ipfilter_lib ${PROJECT_SOURCE_DIR}/libSrc/lib.cpp)
add_executable(test_version ${PROJECT_SOURCE_DIR}/test_version.cpp)

set_target_properties(ipfilter ipfilter_lib test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(ipfilter
    PRIVATE "${CMAKE_BINARY_DIR}"
)
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
    INCLUDE_DIRECTORIES ${PROJECT_SOURCE_DIR}/libSrc
)

target_link_libraries(ipfilter PRIVATE
    ipfilter_lib
)

target_include_directories(ipfilter PUBLIC
    ${PROJECT_SOURCE_DIR}/libSrc
)

target_link_libraries(test_version
    ${Boost_LIBRARIES}
    ipfilter_lib
)

if (MSVC)
    target_compile_option(ipfilter PRIVATE
        /W4
    )
    target_compile_options(ipfilter_lib PRIVATE
        /W4
    )
    target_compile_options(test_version PRIVATE
        /W4
    )
else ()
    target_compile_options(ipfilter PRIVATE
        -Wall -Wextra -pedantic 
    )
    target_compile_options(ipfilter_lib PRIVATE
        -Wall -Wextra -pedantic
    )
    target_compile_options(test_version PRIVATE
        -Wall -Wextra -pedantic
    )
endif()

install(TARGETS ipfilter RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT example@example.com)

include(CPack)

enable_testing()

add_test(ipfilter_tests test_version)
