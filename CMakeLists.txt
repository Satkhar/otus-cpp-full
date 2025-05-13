cmake_minimum_required(VERSION 3.12)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VESRION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VESRION})

option(WITH_TESTS "Whether to build tests" ON)  # Переименовано с WITH_BOOST_TEST

# Подключаем Google Test через FetchContent
if(WITH_TESTS)
    include(FetchContent)
    FetchContent_Declare(
        googletest
        URL https://github.com/google/googletest/archive/refs/tags/release-1.12.1.tar.gz 
    )
    # ВАЖНО: Отключаем сборку ненужных компонентов
    set(BUILD_GMOCK OFF CACHE BOOL "Disable gmock" FORCE)
    set(INSTALL_GTEST OFF CACHE BOOL "Disable install" FORCE)
    FetchContent_MakeAvailable(googletest)
endif()

configure_file(version.h.in version.h)

add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)

set_target_properties(helloworld_cli helloworld PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(helloworld
    PRIVATE "${CMAKE_BINARY_DIR}"
)

target_link_libraries(helloworld_cli PRIVATE
    helloworld
)

# if(WITH_TESTS)
#     # Заменяем Boost.Test на Google Test
#     find_package(GTest REQUIRED)
    
#     add_executable(test_version test_version.cpp)
    
#     set_target_properties(test_version PROPERTIES
#         CXX_STANDARD 14
#         CXX_STANDARD_REQUIRED ON
#     )

#     target_link_libraries(test_version
#         GTest::GTest GTest::Main
#         helloworld
#     )
# endif()



# Настройка тестов
if(WITH_TESTS)
    add_executable(test_version test_version.cpp)
    
    target_link_libraries(test_version 
        PRIVATE 
            helloworld
            GTest::gtest_main  # Используем встроенную main()
    )
    
    # Автоматическое обнаружение тестов
    include(GoogleTest)
    gtest_discover_tests(test_version)
endif()

if (MSVC)
    target_compile_options(helloworld_cli PRIVATE
        /W4
    )
    target_compile_options(helloworld PRIVATE
        /W4
    )
    if(WITH_TESTS)
        target_compile_options(test_version PRIVATE
            /W4
        )
    endif()
else ()
    target_compile_options(helloworld_cli PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    if(WITH_TESTS)
        target_compile_options(test_version PRIVATE
            -Wall -Wextra -pedantic -Werror
        )
    endif()
endif()

install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)
set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")
set(CPACK_PACKAGE_CONTACT example@example.com)
include(CPack)

if(WITH_TESTS)
    enable_testing()
    add_test(NAME test_version COMMAND test_version)
endif()