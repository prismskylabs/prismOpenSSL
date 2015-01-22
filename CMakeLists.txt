cmake_minimum_required(VERSION 2.8.8)

project(PrismOpenssl)
include (ExternalProject)
#
# OpenSSL
#
if(ISD)
    set(PLATFORM isd)
elseif(APPLE)
    set(PLATFORM osx)
elseif(WIN32)
    set(PLATFORM win32)
else()
    set(PLATFORM linux)
endif()

#Clean the install directories every time
#set(CLEAN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/OpenSSL/install/${PLATFORM})
#MESSAGE("DELETE " ${CLEAN_DIR})
#file(REMOVE_RECURSE ${CLEAN_DIR})

message("Configuring, and Building OpenSSL")
enable_language(CXX)

set(OPENSSL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(OPENSSL_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/install/${PLATFORM} CACHE INTERNAL "")
set(OPENSSL_INCLUDE_DIR  ${OPENSSL_ROOT_DIR}/include CACHE INTERNAL "")
set(OPENSSL_LIBRARY_DIR ${OPENSSL_ROOT_DIR}/lib CACHE INTERNAL "")
message("Openssl install to: ${OPENSSL_ROOT_DIR}")

if(WIN32)
    # for the windows build from the OpenSSL\build directory:
    # build>%VS100COMNTOOLS%vsvars32
    # build>cmake -G "NMake Makefiles" ..
    # nmake
    message("The joys of Windows. Time to fix those pesky carriage returns. Make sure you have sed installed and ActivePerl http://www.ActiveState.com")
    set(sed_cmd sed)
    set(sed_pattern -f "${CMAKE_CURRENT_SOURCE_DIR}/sedpattern.txt")
    set(sed_file -i "${OPENSSL_SOURCE_DIR}/util/mk1mf.pl")
    execute_process(COMMAND ${sed_cmd} ${sed_pattern} ${sed_file} )

    set(OPENSSL_LIBRARIES ${OPENSSL_LIBRARY_DIR}/ssleay.lib ${OPENSSL_LIBRARY_DIR}/libeay.lib CACHE INTERNAL "")

    ExternalProject_Add(openssl
        PREFIX              ${OPENSSL_ROOT_DIR}
        SOURCE_DIR          ${OPENSSL_SOURCE_DIR}
        CONFIGURE_COMMAND    perl ${OPENSSL_SOURCE_DIR}/Configure VC-WIN32 --openssldir=${OPENSSL_ROOT_DIR}
        COMMAND   "${OPENSSL_SOURCE_DIR}\\ms\\do_ms.bat"
        BUILD_COMMAND       nmake -s -f ${OPENSSL_SOURCE_DIR}\\ms\\nt.mak
        BUILD_IN_SOURCE     1
        INSTALL_COMMAND     nmake -s -f ${OPENSSL_SOURCE_DIR}\\ms\\nt.mak install
        WORKING_DIRECTORY   ${OPENSSL_SOURCE_DIR}
    )
else()
    # --openssldir=${OPENSSL_ROOT_DIR}: option is set in the ExternalProject_Add below
    if(APPLE)
        set(OPENSSL_CONFIG_CMD ${OPENSSL_SOURCE_DIR}/Configure darwin64-x86_64-cc )
    elseif(_openssl_target)
        set(OPENSSL_CONFIG_CMD ${OPENSSL_SOURCE_DIR}/Configure os/compiler:${_target_name})
    else(LINUX)
        set(OPENSSL_CONFIG_CMD ${OPENSSL_SOURCE_DIR}/config)
    endif()

    set(OPENSSL_LIBRARIES ${OPENSSL_LIBRARY_DIR}/libcrypto.a ${OPENSSL_LIBRARY_DIR}/libssl.a CACHE INTERNAL "")
    set (OPENSSL_BIN_DIR  ${OPENSSL_ROOT_DIR}/bin)
    if (NOT EXISTS ${OPENSSL_BIN_DIR})
        file (MAKE_DIRECTORY ${OPENSSL_BIN_DIR})
    endif ()
    set (BUILDEM_BIN_PATH     ${OPENSSL_BIN_DIR}:$ENV{PATH})
    set (BUILDEM_ENV_STRING   env PATH=${BUILDEM_BIN_PATH} )
    
    ExternalProject_Add(openssl
        PREFIX              ${OPENSSL_ROOT_DIR}
        SOURCE_DIR          ${OPENSSL_SOURCE_DIR}
        CONFIGURE_COMMAND   ${OPENSSL_CONFIG_CMD} --openssldir=${OPENSSL_ROOT_DIR}
        BUILD_COMMAND       ${OPENSSL_BULILD} make -j1  # Parallel builds on openssl are thoroughly broken.  Force -j1
        BUILD_IN_SOURCE     1
        INSTALL_COMMAND     ${BUILDEM_ENV_STRING} make -j1 install
        WORKING_DIRECTORY   ${CMAKE_CURRENT_SOURCE_DIR}/build
    )
endif()



