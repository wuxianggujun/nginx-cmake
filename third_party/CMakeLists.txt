# 在文件开头设置全局编译选项
if(MSVC)
    # 统一所有库使用动态运行时库/MD，匹配OpenSSL的编译方式
    foreach(flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE)
        string(REGEX REPLACE "/MT" "/MD" ${flag_var} "${${flag_var}}")
    endforeach()
    
    # 确保在Debug模式中使用/MDd
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MDd")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")
endif()

# PCRE配置 - 确保使用动态运行时
if(EXISTS "${PCRE_DIR}/CMakeLists.txt")
    set(PCRE_STATIC_RUNTIME OFF CACHE BOOL "" FORCE)
    set(PCRE_BUILD_TESTS OFF CACHE BOOL "")
    set(PCRE_BUILD_PCREGREP OFF CACHE BOOL "")
    set(PCRE_BUILD_PCRECPP OFF CACHE BOOL "")
    set(PCRE_STATIC ON CACHE BOOL "")
    set(PCRE_SUPPORT_UTF ON CACHE BOOL "")
    set(PCRE_SUPPORT_UNICODE_PROPERTIES ON CACHE BOOL "")
    set(PCRE_BUILD_PCRE8 ON CACHE BOOL "")
    set(PCRE_BUILD_PCRE16 OFF CACHE BOOL "")
    set(PCRE_BUILD_PCRE32 OFF CACHE BOOL "")
    
    add_subdirectory(${PCRE_DIR})
    
    # 简化PCRE库设置
    set(PCRE_INCLUDE_DIRS ${PCRE_DIR}/include CACHE PATH "PCRE include directory")
    set(PCRE_LIBRARIES pcre CACHE PATH "PCRE library")
endif()

# ZLIB配置
if(EXISTS "${ZLIB_DIR}")
    set(ZLIB_BUILD_SHARED_LIBS OFF CACHE BOOL "")
    set(SKIP_INSTALL_ALL ON CACHE BOOL "")
    set(ZLIB_BINARY_DIR "${CMAKE_BINARY_DIR}/third_party/zlib" CACHE PATH "ZLIB binary directory")
    
    add_subdirectory(${ZLIB_DIR} ${ZLIB_BINARY_DIR})
    
    # 简化ZLIB库设置
    set(ZLIB_INCLUDE_DIRS 
        ${ZLIB_DIR}
        ${ZLIB_BINARY_DIR}
        CACHE PATH "ZLIB include directories"
    )
    set(ZLIB_LIBRARIES zlibstatic CACHE PATH "ZLIB library")
endif()

# OpenSSL配置 - 简化
set(OPENSSL_INCLUDE_DIRS "${OPENSSL_DIR}/include" CACHE PATH "OpenSSL include directory")
set(OPENSSL_LIBRARIES 
    "${OPENSSL_DIR}/lib/libssl.lib"
    "${OPENSSL_DIR}/lib/libcrypto.lib"
    CACHE PATH "OpenSSL libraries"
)

# 设置全局的第三方库变量
set(NGX_HAVE_PCRE ON CACHE BOOL "Enable PCRE support")
set(NGX_HAVE_ZLIB ON CACHE BOOL "Enable ZLIB support")
set(NGX_HAVE_OPENSSL ON CACHE BOOL "Enable OpenSSL support")