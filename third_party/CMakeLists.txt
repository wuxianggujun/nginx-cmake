# 在文件开头设置全局编译选项
if(MSVC)
    # 统一所有库使用静态运行时库
    foreach(flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
        CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
        if(${flag_var} MATCHES "/MD")
            string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif()
    endforeach()
    
    # 更加严格地强制PCRE和ZLIB使用静态运行时
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
endif()

# PCRE配置 - 确保使用静态运行时
if(EXISTS "${PCRE_DIR}/CMakeLists.txt")
    # 强制PCRE使用静态运行时
    set(PCRE_STATIC_RUNTIME ON CACHE BOOL "")
    set(PCRE_BUILD_TESTS OFF CACHE BOOL "")
    set(PCRE_BUILD_PCREGREP OFF CACHE BOOL "")
    set(PCRE_BUILD_PCRECPP OFF CACHE BOOL "")
    set(PCRE_STATIC ON CACHE BOOL "")
    set(PCRE_SUPPORT_UTF ON CACHE BOOL "")
    set(PCRE_SUPPORT_UNICODE_PROPERTIES ON CACHE BOOL "")
    set(PCRE_BUILD_PCRE8 ON CACHE BOOL "")
    set(PCRE_BUILD_PCRE16 OFF CACHE BOOL "")
    set(PCRE_BUILD_PCRE32 OFF CACHE BOOL "")
    
    # 添加PCRE前确保静态运行时设置被应用
    add_subdirectory(${PCRE_DIR})
    
    # 设置PCRE的包含目录和库
    set(PCRE_INCLUDE_DIRS ${PCRE_DIR}/include CACHE PATH "PCRE include directory")
    
    set(PCRE_LIBRARIES pcre CACHE PATH "PCRE library")
    
    # 调试输出PCRE库信息
    message(STATUS "PCRE库: ${PCRE_LIBRARIES}")
endif()

# ZLIB配置
if(EXISTS "${ZLIB_DIR}")
    # 设置ZLIB选项
    set(ZLIB_BUILD_SHARED_LIBS OFF CACHE BOOL "")
    set(SKIP_INSTALL_ALL ON CACHE BOOL "")
    set(ZLIB_BINARY_DIR "${CMAKE_BINARY_DIR}/third_party/zlib" CACHE PATH "ZLIB binary directory")
    
    # 添加ZLIB子目录
    add_subdirectory(${ZLIB_DIR} ${ZLIB_BINARY_DIR})
endif()

# OpenSSL配置 - 调试版本
set(OPENSSL_ROOT_DIR ${OPENSSL_DIR})
set(OPENSSL_INCLUDE_DIR "${OPENSSL_DIR}/include")

# 调试输出OpenSSL库文件是否存在
file(GLOB SSL_LIB_FILES "${OPENSSL_DIR}/lib/libssl*.lib" "${OPENSSL_DIR}/lib/ssl*.lib")
file(GLOB CRYPTO_LIB_FILES "${OPENSSL_DIR}/lib/libcrypto*.lib" "${OPENSSL_DIR}/lib/crypto*.lib")

message(STATUS "SSL库文件: ${SSL_LIB_FILES}")
message(STATUS "Crypto库文件: ${CRYPTO_LIB_FILES}")

# 确保正确导入OpenSSL库
if(EXISTS "${OPENSSL_DIR}/lib/libssl.lib" AND EXISTS "${OPENSSL_DIR}/lib/libcrypto.lib")
    add_library(OpenSSL::SSL STATIC IMPORTED)
    set_target_properties(OpenSSL::SSL PROPERTIES 
        IMPORTED_LOCATION "${OPENSSL_DIR}/lib/libssl.lib"
        INTERFACE_INCLUDE_DIRECTORIES "${OPENSSL_INCLUDE_DIR}"
    )

    add_library(OpenSSL::Crypto STATIC IMPORTED)
    set_target_properties(OpenSSL::Crypto PROPERTIES 
        IMPORTED_LOCATION "${OPENSSL_DIR}/lib/libcrypto.lib"
        INTERFACE_INCLUDE_DIRECTORIES "${OPENSSL_INCLUDE_DIR}"
    )
    
    # 确保被链接的库具有正确的依赖关系
    set_property(TARGET OpenSSL::SSL APPEND PROPERTY 
        INTERFACE_LINK_LIBRARIES OpenSSL::Crypto ws2_32 crypt32 advapi32)
    
    set(OPENSSL_FOUND TRUE)
    message(STATUS "OpenSSL库配置完成")
else()
    message(FATAL_ERROR "OpenSSL库文件未找到: ${OPENSSL_DIR}/lib/libssl.lib, ${OPENSSL_DIR}/lib/libcrypto.lib")
endif()

# 设置全局的第三方库变量
set(NGX_HAVE_PCRE ON CACHE BOOL "Enable PCRE support")
set(NGX_HAVE_ZLIB ON CACHE BOOL "Enable ZLIB support")
set(NGX_HAVE_OPENSSL ${OPENSSL_FOUND} CACHE BOOL "Enable OpenSSL support")