cmake_minimum_required(VERSION 2.8)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/Modules)

project(ship_server)
set(ship_server_VERSION_MAJOR 1)
set(ship_server_VERSION_MINOR 0)

set(ship_server_VERSION_STRING "${ship_server_MAJOR_VERSION}.${ship_server_MINOR_VERSION}.${ship_server_PATCH_VERSION}")


option(IPV6 "build application with IPv6 Support" ON) # ON is the default

include_directories(src)

set(LIBSYLVERANT_LIBRARY sylverant)
include_directories(${CMAKE_SOURCE_DIR}/lib/libsylverant/include)

set(LIBPSOARCHIVE_LIBRARY psoarchive)
include_directories(${CMAKE_SOURCE_DIR}/lib/libpsoarchive/include)

set(LIBMINI18N_LIBRARY mini18n)
include_directories(${CMAKE_SOURCE_DIR}/lib/libmini18n/src)

find_package(Threads REQUIRED)

find_package(ZLIB REQUIRED)
include_directories(${ZLIB_INCLUDE_DIR})

find_package(LibXml2 REQUIRED)
include_directories(${LIBXML2_INCLUDE_DIR})

find_package(Iconv REQUIRED)
include_directories(${ICONV_INCLUDE_DIR})

find_package(GnuTLS REQUIRED)
include_directories(${GNUTLS_INCLUDE_DIR})

IF(ICONV_SECOND_ARGUMENT_IS_CONST)
  add_definitions (-DICONV_CONST=const)
ELSE()
  add_definitions (-DICONV_CONST=)
ENDIF()

file(GLOB SOURCES src/*.c)

add_definitions (-Wall)

add_definitions (-DVERSION="${ship_server_VERSION_STRING}")
add_definitions (-DHAVE_LIBMINI18N)

if(IPV6)
  add_definitions(-DENABLE_IPV6)
endif(IPV6)

execute_process(COMMAND git log --oneline
                COMMAND wc -l
                COMMAND tr -d '[[:space:]]'
                OUTPUT_VARIABLE GIT_BUILD
                )

execute_process(COMMAND git show -s --pretty=format:%h
                OUTPUT_VARIABLE GIT_CHANGESET
                )

add_definitions(-DGIT_BUILD="${GIT_BUILD}")
add_definitions(-DGIT_CHANGESET="${GIT_CHANGESET}")



add_subdirectory(lib/libsylverant)
add_subdirectory(lib/libpsoarchive)
add_subdirectory(lib/libmini18n)

add_executable(sylverant-ship ${SOURCES})
target_link_libraries(sylverant-ship ${ZLIB_LIBRARY} ${LIBXML2_LIBRARIES} ${GNUTLS_LIBRARIES} ${LIBSYLVERANT_LIBRARY} ${LIBPSOARCHIVE_LIBRARY} ${LIBMINI18N_LIBRARY})
if(THREADS_HAVE_PTHREAD_ARG)
  target_compile_options(sylverant-ship PUBLIC "-pthread")
endif()
if(CMAKE_THREAD_LIBS_INIT)
  target_link_libraries(sylverant-ship "${CMAKE_THREAD_LIBS_INIT}")
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    install(TARGETS sylverant-ship RUNTIME DESTINATION bin)
    install(DIRECTORY l10n DESTINATION share/sylverant)
endif()