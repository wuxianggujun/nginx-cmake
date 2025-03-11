cmake_minimum_required(VERSION 3.17)

project(nginx LANGUAGES C)

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
set(NGINX_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${NGINX_SOURCE_DIR}/cmake")

# 设置nginx工作目录
set(NGINX_WORK_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# 设置第三方库路径
set(THIRD_PARTY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third_party)
set(PCRE_DIR ${THIRD_PARTY_DIR}/pcre)
set(ZLIB_DIR ${THIRD_PARTY_DIR}/zlib)
set(OPENSSL_DIR ${THIRD_PARTY_DIR}/openssl-3.4)

# 包含目录设置
include_directories(
    ${NGINX_SOURCE_DIR}
    ${NGINX_SOURCE_DIR}/src/core
    ${NGINX_SOURCE_DIR}/src/event
    ${NGINX_SOURCE_DIR}/src/event/modules
    ${NGINX_SOURCE_DIR}/src/event/quic
    ${NGINX_SOURCE_DIR}/src/os/win32  # 只包含Windows相关目录

    # 修改 zlib 包含路径，添加构建目录
    ${ZLIB_DIR}
    ${ZLIB_BINARY_DIR}    # 添加这一行，因为 zconf.h 通常在构建目录中生成
    ${PCRE_DIR}/include 
        ${PCRE_BINARY_DIR}
    ${OPENSSL_INCLUDE_DIRS}
    ${OPENSSL_DIR}/include
    
    ${NGINX_SOURCE_DIR}/src/http
    ${NGINX_SOURCE_DIR}/src/http/modules
    ${NGINX_SOURCE_DIR}/src/http/v2
    ${NGINX_SOURCE_DIR}/src/mail
    ${NGINX_SOURCE_DIR}/src/stream
)

# 创建cmake目录
if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
    file(MAKE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
endif()

# 添加自动分组文件的cmake配置
add_subdirectory(cmake)

add_subdirectory(third_party)

# 添加src作为子目录进行构建
add_subdirectory(src)

file(GLOB_RECURSE nginx_HEADERS "${NGINX_SOURCE_DIR}/src/*.h")

# 链接库
target_link_libraries(nginx
    PRIVATE
    # OpenSSL库放在最前面
    "${CMAKE_SOURCE_DIR}/third_party/openssl-3.4/lib/libssl.lib"
    "${CMAKE_SOURCE_DIR}/third_party/openssl-3.4/lib/libcrypto.lib"
    
    # 系统库
    kernel32.lib
    user32.lib
    advapi32.lib
    ws2_32.lib
    gdi32.lib
    crypt32.lib
    
    # 其他第三方库
    pcre
    zlibstatic
)

# 链接器选项
target_link_options(nginx
        PRIVATE
        /NODEFAULTLIB:libcmt.lib     # 排除静态运行时
        /NODEFAULTLIB:libcmtd.lib    # 排除静态运行时调试版
        /ignore:4217 /ignore:4286    # 忽略符号警告
        /VERBOSE:LIB                   # 显示详细链接信息
)

# 配置PCRE、ZLIB和OpenSSL支持 - 使用third_party中的库
# PCRE配置
if(NGX_HAVE_PCRE)
    message(STATUS "PCRE support enabled")
    message(STATUS "PCRE library: ${PCRE_LIBRARIES}")
    target_link_libraries(nginx PRIVATE ${PCRE_LIBRARIES})
    target_compile_definitions(nginx PRIVATE NGX_HAVE_PCRE=1)
endif()

# ZLIB配置
if(NGX_HAVE_ZLIB)
    target_link_libraries(nginx PRIVATE zlibstatic)
    target_compile_definitions(nginx PRIVATE NGX_HAVE_ZLIB=1)
endif()


# OpenSSL配置
if(NGX_HAVE_OPENSSL)
    target_link_libraries(nginx PRIVATE ${OPENSSL_LIBRARIES})
    target_compile_definitions(nginx 
        PRIVATE 
        NGX_HAVE_OPENSSL=1
    )
endif()


# 调用自动化分组函数
AUTO_GROUP_FILES(
        TARGET nginx
        BASE_DIR ${CMAKE_SOURCE_DIR}/src
        FILES ${nginx_HEADERS}
)

# 设置生成位置
set_target_properties(nginx PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${NGINX_WORK_DIR}
        RUNTIME_OUTPUT_DIRECTORY_DEBUG ${NGINX_WORK_DIR}
        RUNTIME_OUTPUT_DIRECTORY_RELEASE ${NGINX_WORK_DIR}
        RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${NGINX_WORK_DIR}
        VS_DEBUGGER_WORKING_DIRECTORY ${NGINX_WORK_DIR}
)

# 创建必要的目录结构
set(NGINX_DIRECTORIES
        ${NGINX_WORK_DIR}/logs
        ${NGINX_WORK_DIR}/conf
        ${NGINX_WORK_DIR}/html
        ${NGINX_WORK_DIR}/temp
        ${NGINX_WORK_DIR}/temp/client_body_temp
        ${NGINX_WORK_DIR}/temp/proxy_temp
        ${NGINX_WORK_DIR}/temp/fastcgi_temp
        ${NGINX_WORK_DIR}/temp/uwsgi_temp
        ${NGINX_WORK_DIR}/temp/scgi_temp
)

# 创建所有必要的目录
foreach(dir ${NGINX_DIRECTORIES})
    file(MAKE_DIRECTORY ${dir})
endforeach()

# 创建完整的nginx欢迎页面
if(NOT EXISTS "${NGINX_WORK_DIR}/html/index.html")
    file(WRITE "${NGINX_WORK_DIR}/html/index.html"
"<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href=\"http://nginx.org/\">nginx.org</a>.<br/>
Commercial support is available at
<a href=\"http://nginx.com/\">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>")
endif()

if(NOT EXISTS "${NGINX_WORK_DIR}/html/50x.html")
    file(WRITE "${NGINX_WORK_DIR}/html/50x.html"
"<!DOCTYPE html>
<html>
<head>
<title>Error</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>An error occurred.</h1>
<p>Sorry, the page you are looking for is currently unavailable.<br/>
Please try again later.</p>
<p>If you are the system administrator of this resource then you should check
the error log for details.</p>
<p><em>Faithfully yours, nginx.</em></p>
</body>
</html>")
endif()

# 编译定义
target_compile_definitions(nginx
    PRIVATE
    $<$<BOOL:${NGX_HAVE_PCRE}>:NGX_HAVE_PCRE=1>
    $<$<BOOL:${NGX_HAVE_ZLIB}>:NGX_HAVE_ZLIB=1>
    $<$<BOOL:${NGX_HAVE_OPENSSL}>:NGX_HAVE_OPENSSL=1>
)

# 添加编译选项
if(MSVC)
    # 直接设置/MD或/MDd
    target_compile_options(nginx PRIVATE
        $<$<CONFIG:Debug>:/MDd>
        $<$<NOT:$<CONFIG:Debug>>:/MD>
    )
endif()
