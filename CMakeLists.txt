#
# OpenSSL
#
message("Configuring, and Building OpenSSL")
enable_language(CXX)
set(OPENSSL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(OPENSSL_ROOT_DIR ${CMAKE_CURRENT_BINARY_DIR}/OpenSSL CACHE INTERNAL "")
message("OPENSSL_ROOT_DIR: ${OPENSSL_ROOT_DIR}")
set(OPENSSL_DOWNLOAD_DIR "$ENV{TMPDIR}")
if(APPLE)
    set(OPENSSL_CONFIG_CMD 
        ${OPENSSL_SOURCE_DIR}/Configure darwin64-x86_64-cc 
        --openssldir=${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/mac)
elseif(WIN32)
    set(OPENSSL_DOWNLOAD_DIR "$ENV{TEMP}")
    set(OPENSSL_CONFIG_CMD 
        perl ${OPENSSL_SOURCE_DIR}/Configure VC-WIN32
        --openssldir=${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/windows)
elseif(_openssl_target)
    set(OPENSSL_CONFIG_CMD
        ${OPENSSL_SOURCE_DIR}/Configure os/compiler:${_target_name}
        --openssldir=${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/isd)
else(APPLE)
    set(OPENSSL_CONFIG_CMD
        ${OPENSSL_SOURCE_DIR}/config 
        --openssldir=${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/linux)
endif(APPLE)

message("OpenSSL config command: ${OPENSSL_CONFIG_CMD}")

if(WIN32)
    message("The joys of Windows. Time to fix those pesky carriage returns. Make sure you have sed installed.")
    set(sed_cmd sed)
    set(sed_pattern -f "${CMAKE_CURRENT_SOURCE_DIR}/../sedpattern.txt")
    set(sed_file -i "${OPENSSL_SOURCE_DIR}/util/mk1mf.pl")
    execute_process(COMMAND ${sed_cmd} ${sed_pattern} ${sed_file} )
    message("do_ms.bat")
    execute_process(COMMAND "${OPENSSL_SOURCE_DIR}/ms/do_ms.bat"
    WORKING_DIRECTORY ${OPENSSL_SOURCE_DIR})
    message("nt.mak ${OPENSSL_SOURCE_DIR}/ms/nt.mak")
    execute_process(COMMAND nmake.exe -f "${OPENSSL_SOURCE_DIR}/ms/nt.mak"
    WORKING_DIRECTORY ${OPENSSL_SOURCE_DIR}
    RESULT_VARIABLE ntmak_result)
    message("nt.mak result: ${ntmak_result}")
    message("nt.mak install ${OPENSSL_SOURCE_DIR}/ms/nt.mak")
    execute_process(COMMAND nmake.exe -f "${OPENSSL_SOURCE_DIR}/ms/nt.mak" install
    WORKING_DIRECTORY ${OPENSSL_SOURCE_DIR}
    RESULT_VARIABLE ntmakinstall_result)
    message("nt.mak install result: ${ntmakinstall_result}")
else()
    execute_process(COMMAND make -j8
        WORKING_DIRECTORY ${OPENSSL_ROOT_DIR}/src/${OPENSSL_EXTRACTED_DIR_NAME})
    execute_process(COMMAND make install
        WORKING_DIRECTORY ${OPENSSL_ROOT_DIR}/src/${OPENSSL_EXTRACTED_DIR_NAME})
endif()

link_directories("${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/lib")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_CURRENT_BINARY_DIR}/OpenSSL/lib -I${OPENSSL_INCLUDE_DIR}"
    CACHE STRING "")
set(OPENSSL_SSL_LIBRARIES ${OPENSSL_ROOT_DIR}/lib CACHE INTERNAL "")
set(OPENSSL_LIBRARIES ssl crypto CACHE INTERNAL "")
set(CMAKE_USE_OPENSSL 1 CACHE INTERNAL "")
INCLUDE_DIRECTORIES(${OPENSSL_INCLUDE_DIR})
LINK_DIRECTORIES(${OPENSSL_SSL_LIBRARIES})
set(CMAKE_PREFIX_PATH ${OPENSSL_SSL_LIBRARIES} ${CMAKE_PREFIX_PATH})
message("OpenSSL Include DIR: " ${OPENSSL_INCLUDE_DIR})
message("OpenSSL Root Dir: " ${OPENSSL_ROOT_DIR})
message("OpenSSL Lib Dir: " ${OPENSSL_SSL_LIBRARIES})
find_package(OpenSSL PATHS ${OPENSSL_ROOT_DIR} ${OPENSSL_SSL_LIBRARIES} ${OPENSSL_INCLUDE_DIR} NO_DEFAULT_PATH)
