# this file generates a compile_commands.json in the same directory as the
# CMakeLists.txt that included this. the compile_commands.json lets IDEs know to
# look for your headers in include/. i made this orginally for my neovim config.

# if we are using visual studio then we need complex commands for generation.
if(MSVC)
    # store all cpp files in CPP_FILES
    file(GLOB_RECURSE CPP_FILES "${CMAKE_SOURCE_DIR}/src/*.cpp")

    # build include flags starting with the local project include
    set(INCLUDE_FLAGS "-I${CMAKE_SOURCE_DIR}/include")

    # generate JSON string
    set(JSON_CONTENT "[\n")
    set(FIRST_ENTRY TRUE)

    # build rest of include flags adding fetch content deps from
    # build/_deps/*/include/
    file(GLOB DEPS_DIRS "${CMAKE_BINARY_DIR}/_deps/*")
    foreach(DEPS_DIR ${DEPS_DIRS})
        get_filename_component(DIR_NAME ${DEPS_DIR} NAME)
        # check if directory doesn't end with "-build" and has an include
        # subdirectory.
        if(NOT DIR_NAME MATCHES ".*-build$" AND EXISTS "${DEPS_DIR}/include")
            string(APPEND INCLUDE_FLAGS " -I${DEPS_DIR}/include")
        endif()
    endforeach()

    foreach(CPP_FILE ${CPP_FILES})
        if(NOT FIRST_ENTRY)
            string(APPEND JSON_CONTENT ",\n")
        endif()

        string(APPEND JSON_CONTENT "  {\n")
        string(APPEND JSON_CONTENT "    \"directory\": \"${CMAKE_SOURCE_DIR}\",\n")
        string(APPEND JSON_CONTENT "    \"command\": \"clang++ ${INCLUDE_FLAGS} -std=c++${CMAKE_CXX_STANDARD} -c\",\n")
        string(APPEND JSON_CONTENT "    \"file\": \"${CPP_FILE}\"\n")
        string(APPEND JSON_CONTENT "  }")

        set(FIRST_ENTRY FALSE)
    endforeach()
    string(APPEND JSON_CONTENT "\n]\n")

    # create compile_commands.json with string
    file(WRITE "${CMAKE_SOURCE_DIR}/compile_commands.json" "${JSON_CONTENT}")
else()
    # if we are using some other compiler like gcc , then there is a simple
    # command that we can use.

    set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # this command creates the json file

    add_custom_command(TARGET gemm POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${CMAKE_BINARY_DIR}/compile_commands.json
        ${CMAKE_SOURCE_DIR}/compile_commands.json
    COMMENT "Copying compile_commands.json to project root"
    )
endif()
