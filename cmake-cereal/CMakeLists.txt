cmake_minimum_required(VERSION 3.14)
project(cmake-cereal CXX)
set(CMAKE_CXX_STANDARD 17)

include(CMakePrintHelpers)
cmake_print_variables(PROJECT_SOURCE_DIR)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)

cmake_path(GET PROJECT_SOURCE_DIR PARENT_PATH ROOT_DIR)
cmake_path(APPEND INCLUDE_DIR ${ROOT_DIR} include)
cmake_path(APPEND SOURCE_DIR ${ROOT_DIR} src)
cmake_print_variables(ROOT_DIR)
cmake_print_variables(INCLUDE_DIR)
cmake_print_variables(SOURCE_DIR)

add_executable(${PROJECT_NAME} main.cpp)

target_compile_options(${PROJECT_NAME} PRIVATE -O3 -Wall -Wextra -Wno-psabi)
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_20)

include(FetchContent)

find_package(cereal CONFIG QUIET)
if (NOT cereal_FOUND)
    message(STATUS "Fetching cereal")
    FetchContent_Declare(
        cereal
        GIT_REPOSITORY https://github.com/USCiLab/cereal.git
        GIT_TAG         master
        SOURCE_SUBDIR build
    )
    FetchContent_GetProperties(cereal)
    if (cereal_POPULATED)
        message(STATUS
            ${PROJECT_NAME}
            " found already populated cereal dependency: "
            ${cereal_SOURCE_DIR}
        )
    else ()
        # Do not compile any cereal tests
        set(JUST_INSTALL_CEREAL ON)
        # Install cereal at ${CMAKE_INSTALL_PREFIX}
        set(CEREAL_INSTALL ${YGM_INSTALL})
        # Populate cereal
        FetchContent_MakeAvailable(cereal)
        # Include cereal root cmake boilerplate
        add_subdirectory(${cereal_SOURCE_DIR} ${cereal_BINARY_DIR})

        message(STATUS
            ${PROJECT_NAME}
            " cloned cereal dependency: "
            ${cereal_SOURCE_DIR}
        )
    endif ()
else ()
    message(STATUS
        ${PROJECT_NAME}
        " found installed cereal dependency: "
        ${cereal_DIR}
    )
endif ()
target_link_libraries(${PROJECT_NAME} PRIVATE cereal::cereal)
