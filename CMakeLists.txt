cmake_minimum_required(VERSION 2.8)
set(CMAKE_CXX_STANDARD 14)
set(proxygen-http-server-sources
    HTTPServer.cpp
    HTTPServer.h
    HTTPServerAcceptor.cpp
    HTTPServerAcceptor.h
    PushHandler.h
    RequestHandler.h
    RequestHandlerAdaptor.cpp
    RequestHandlerAdaptor.h
    RequestHandlerFactory.h
    ResponseBuilder.h
    ResponseHandler.h
    ScopedHTTPServer.h
    SignalHandler.cpp
    SignalHandler.h
)

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-register")
endif()

set(httpDir ${CMAKE_CURRENT_SOURCE_DIR}/proxygen/lib/http)
set(generatorEnv FBCODE_DIR=${CMAKE_CURRENT_SOURCE_DIR} INSTALL_DIR=${httpDir} HEADERS_LIST=${httpDir}/HTTPCommonHeaders.txt)

add_custom_command(
    OUTPUT ${httpDir}/HTTPCommonHeaders.h
    COMMAND ${generatorEnv} ${httpDir}/gen_HTTPCommonHeaders.h.sh
    VERBATIM
)

add_custom_command(
    OUTPUT ${httpDir}/HTTPCommonHeaders.cpp
    COMMAND ${generatorEnv} ${httpDir}/gen_HTTPCommonHeaders.cpp.sh
    VERBATIM
)

foreach(f ${proxygen-http-server-sources})
    list(APPEND proxygen-http-server-paths "${CMAKE_CURRENT_SOURCE_DIR}/proxygen/httpserver/${f}")
endforeach()

list(APPEND proxygen-http-server-paths ${httpDir}/HTTPCommonHeaders.h ${httpDir}/HTTPCommonHeaders.cpp)

add_library(httpserver ${proxygen-http-server-paths})
# add_dependencies(proxygen-http-server ${httpDir}/HTTPCommonHeaders.h ${httpDir}/HTTPCommonHeaders.cpp)
find_package(wangle REQUIRED)
find_package(folly REQUIRED)
find_package(fizz REQUIRED)
find_package(Threads REQUIRED)
target_include_directories(httpserver PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> $<INSTALL_INTERFACE:include/proxygen>)
target_link_libraries(httpserver Folly::folly wangle::wangle)
install(
    TARGETS httpserver
    EXPORT proxygen
    DESTINATION ${CMAKE_INSTALL_PREFIX}
)

# public headers
foreach(f ${proxygen-http-server-paths})
    string(REGEX MATCH "\\.h$" f ${f})
    IF(f)
        LIST(APPEND httpserverHeaders ${f})
    ENDIF()
endforeach()

install(
    FILES
        ${httpserverHeaders}
    DESTINATION
        include
)

# install(
#     EXPORT proxygen
#     NAMESPACE proxygen::
#     DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/proxygen
# )

export(EXPORT proxygen FILE "${CMAKE_CURRENT_BINARY_DIR}/proxygen/proxygen-targets.cmake")