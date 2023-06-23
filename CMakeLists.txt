cmake_minimum_required(VERSION 3.24)

set(VCPKG_OVERLAY_TRIPLETS "${CMAKE_CURRENT_LIST_DIR}/../my-vcpkg-triplets")
set(VCPKG_OVERLAY_PORTS "${CMAKE_CURRENT_LIST_DIR}/../my-vcpkg-ports")
set(VCPKG_TARGET_TRIPLET "x64-win-llvm-release")
set(VCPKG_HOST_TRIPLET "x64-win-llvm-release")
set(VCPKG_EXECUTABLE "${CMAKE_CURRENT_LIST_DIR}/../vcpkg/vcpkg.exe")
set(VCPKG_MANIFEST_DIR "${CMAKE_CURRENT_LIST_DIR}/build")
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_LIST_DIR}/../vcpkg/scripts/buildsystems/vcpkg.cmake")

list(APPEND vcpkg_target_ports "qtbase")
list(APPEND vcpkg_host_ports "root")
set(qtbase_FEATURES "gui;widgets")


function(create_vcpkg_manifest)
    cmake_parse_arguments(PARSE_ARGV 0 arg 
        "" 
        "OUTPUT"
        "PORTS;HOST_PORTS;OVERLAY_PORTS;OVERLAY_TRIPLETS"
    )
    set(json "{")
    string(APPEND json "\"name\":\"cling-vcpkg\",")

    string(APPEND json "\"dependencies\": [")
    list(JOIN arg_PORTS "\",\"" ports)
    string(APPEND json "\"${ports}\"")

    foreach(port IN LISTS arg_PORTS)
        if(DEFINED ${port}_FEATURES)
            set(remove "\"${port}\"")
            set(replacement "{ \"default-features\": false, \"name\":\"${port}\", \"features\":[")
            list(JOIN ${port}_FEATURES "\",\"" features)
            string(APPEND replacement "\"${features}\"]}")
            string(REPLACE "${remove}" "${replacement}" json "${json}")
        endif()
    endforeach()

    if(arg_PORTS)
        string(APPEND json ",")
    endif()
    list(JOIN arg_HOST_PORTS "\"}, { \"host\": true, \"default-features\": false, \"name\": \"" hports)
    string(APPEND json "{ \"host\": true, \"default-features\": false, \"name\": \"${hports}\" }")
    string(APPEND json "],")

    string(APPEND json "\"vcpkg-configuration\": {")
     string(APPEND json "\"overlay-ports\": [")
      list(JOIN arg_OVERLAY_PORTS "\",\"" oport)
      string(APPEND json "\"${oport}\"")
     string(APPEND json "],")
     string(APPEND json "\"overlay-triplets\": [")
      list(JOIN arg_OVERLAY_TRIPLETS "\",\"" otriplet)
      string(APPEND json "\"${otriplet}\"")
     string(APPEND json "]")
    string(APPEND json "}")

    string(APPEND json "}")

    file(WRITE "${arg_OUTPUT}" "${json}")
    execute_process(COMMAND "${VCPKG_EXECUTABLE}" format-manifest "${arg_OUTPUT}" 
        COMMAND_ERROR_IS_FATAL ANY
    )
endfunction()

create_vcpkg_manifest(
    OUTPUT "${VCPKG_MANIFEST_DIR}/vcpkg.json"
    PORTS ${vcpkg_target_ports}
    HOST_PORTS ${vcpkg_host_ports}
    OVERLAY_PORTS ${VCPKG_OVERLAY_PORTS}
    OVERLAY_TRIPLETS ${VCPKG_OVERLAY_TRIPLETS}
)

project(cling-vcpkg LANGUAGES C CXX)

find_package(Qt6 COMPONENTS Gui Widgets)
#TODO:add xeus-zmq: https://github.com/jupyter-xeus/xeus-zmq
#TODO:add xeus-cling: https://github.com/jupyter-xeus/xeus-cling/

#pragma cling add_include_path("inc_directory")
#pragma cling add_library_path("lib_directory")
#pragma cling load("libname")