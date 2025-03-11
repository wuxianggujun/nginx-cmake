cmake_minimum_required(VERSION 3.20)

project(nginx C)

# 设置cmake模块路径
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# 包含自定义cmake模块
include(auto_group_files)

# 启用文件夹分组
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# 设置全局编译选项
if(MSVC)
    # 统一使用MT运行时库
    foreach(flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE)
        string(REGEX REPLACE "/MT" "/MD" ${flag_var} "${${flag_var}}")
    endforeach()
endif()

# 设置必要的目录结构
set(NGINX_ROOT_DIR ${CMAKE_SOURCE_DIR})
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${NGINX_ROOT_DIR}/cmake")

# 设置nginx工作目录
set(NGINX_INSTALL_DIR ${CMAKE_BINARY_DIR}/dist)
set(NGINX_CONF_DIR ${NGINX_INSTALL_DIR}/conf)
set(NGINX_LOGS_DIR ${NGINX_INSTALL_DIR}/logs)
set(NGINX_TEMP_DIR ${NGINX_INSTALL_DIR}/temp)

# 设置第三方库路径
set(NGINX_THIRD_PARTY_DIR ${NGINX_ROOT_DIR}/third_party)
set(NGINX_PCRE_DIR ${NGINX_THIRD_PARTY_DIR}/pcre)
set(NGINX_ZLIB_DIR ${NGINX_THIRD_PARTY_DIR}/zlib)
set(NGINX_OPENSSL_DIR ${NGINX_THIRD_PARTY_DIR}/openssl-3.4)

# 包含目录设置
include_directories(
    ${NGINX_ROOT_DIR}
    ${NGINX_ROOT_DIR}/src/core
    ${NGINX_ROOT_DIR}/src/event
    ${NGINX_ROOT_DIR}/src/event/modules
    ${NGINX_ROOT_DIR}/src/event/quic
    ${NGINX_ROOT_DIR}/src/os/win32  # 只包含Windows相关目录

    # 修改 zlib 包含路径，添加构建目录
    ${NGINX_ZLIB_DIR}
    ${CMAKE_BINARY_DIR}/third_party/zlib
    ${NGINX_PCRE_DIR}/include 
        ${CMAKE_BINARY_DIR}/third_party/pcre
    ${NGINX_OPENSSL_DIR}/include
    
    ${NGINX_ROOT_DIR}/src/http
    ${NGINX_ROOT_DIR}/src/http/modules
    ${NGINX_ROOT_DIR}/src/http/v2
    ${NGINX_ROOT_DIR}/src/mail
    ${NGINX_ROOT_DIR}/src/stream
)

# 创建cmake目录
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
endif()

# 添加自动分组文件的cmake配置
add_subdirectory(cmake)

add_subdirectory(third_party)

# 确保构建PCRE库
if(EXISTS "${NGINX_PCRE_DIR}/CMakeLists.txt")
    set(PCRE_BUILD_TESTS OFF CACHE BOOL "")
    set(PCRE_BUILD_PCREGREP OFF CACHE BOOL "")
    set(PCRE_BUILD_PCRECPP OFF CACHE BOOL "")
    set(PCRE_STATIC ON CACHE BOOL "")
    set(PCRE_SUPPORT_UTF ON CACHE BOOL "")
    set(PCRE_SUPPORT_UNICODE_PROPERTIES ON CACHE BOOL "")
    add_subdirectory(${NGINX_PCRE_DIR} ${CMAKE_BINARY_DIR}/third_party/pcre)
endif()

# 确保构建ZLIB库
if(EXISTS "${NGINX_ZLIB_DIR}/CMakeLists.txt")
    set(ZLIB_BUILD_SHARED_LIBS OFF CACHE BOOL "")
    set(SKIP_INSTALL_ALL ON CACHE BOOL "")
    add_subdirectory(${NGINX_ZLIB_DIR} ${CMAKE_BINARY_DIR}/third_party/zlib)
    endif()
    
# 添加全局定义
add_compile_definitions(
    NGX_HAVE_PCRE=1
    NGX_HAVE_ZLIB=1
    NGX_HAVE_OPENSSL=1
)

# 添加src作为子目录进行构建
add_subdirectory(src)

file(GLOB_RECURSE nginx_HEADERS "${NGINX_ROOT_DIR}/src/*.h")

# 创建运行时目录
foreach(DIR ${NGINX_CONF_DIR} ${NGINX_LOGS_DIR} ${NGINX_TEMP_DIR})
    if(NOT EXISTS ${DIR})
        file(MAKE_DIRECTORY ${DIR})
    endif()
endforeach()

# 欢迎页生成
set(WELCOME_PAGE ${NGINX_CONF_DIR}/html/index.html)
if(NOT EXISTS ${WELCOME_PAGE})
    file(MAKE_DIRECTORY ${NGINX_CONF_DIR}/html)
    file(WRITE ${WELCOME_PAGE} "<html><body><h1>Welcome to nginx!</h1></body></html>")
endif()

# 编译定义
target_compile_definitions(${PROJECT_NAME}
    PRIVATE
    $<$<BOOL:${NGX_HAVE_PCRE}>:NGX_HAVE_PCRE=1>
    $<$<BOOL:${NGX_HAVE_ZLIB}>:NGX_HAVE_ZLIB=1>
    $<$<BOOL:${NGX_HAVE_OPENSSL}>:NGX_HAVE_OPENSSL=1>
)

# 添加编译选项
if(MSVC)
    # 直接设置/MD或/MDd
    target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Debug>:/MDd>
        $<$<NOT:$<CONFIG:Debug>>:/MD>
    )
endif()
