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
set(NGINX_WORK_DIR ${NGINX_SOURCE_DIR})

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
#add_subdirectory(src)

file(GLOB_RECURSE nginx_HEADERS "${NGINX_SOURCE_DIR}/src/*.h")


set(nginx_SOURCES
        ${NGINX_SOURCE_DIR}/src/core/nginx.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_log.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_palloc.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_array.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_list.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_hash.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_buf.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_queue.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_output_chain.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_string.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_parse.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_parse_time.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_inet.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_file.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_crc32.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_murmurhash.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_md5.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_sha1.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_rbtree.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_radix_tree.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_slab.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_times.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_shmtx.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_connection.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_cycle.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_spinlock.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_rwlock.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_cpuinfo.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_conf_file.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_module.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_resolver.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_open_file_cache.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_crypt.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_proxy_protocol.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_syslog.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_timer.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_posted.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_accept.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_udp.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_connect.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_openssl.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_pipe.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_errno.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_alloc.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_files.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_shmem.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_time.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_process.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_thread.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_socket.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_wsarecv.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_wsarecv_chain.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_udp_wsarecv.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_wsasend.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_wsasend_chain.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_win32_init.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_user.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_dlopen.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_event_log.c
        ${NGINX_SOURCE_DIR}/src/os/win32/ngx_process_cycle.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_acceptex.c
        ${NGINX_SOURCE_DIR}/src/event/modules/ngx_iocp_module.c
        ${NGINX_SOURCE_DIR}/src/event/modules/ngx_win32_select_module.c
        ${NGINX_SOURCE_DIR}/src/event/modules/ngx_win32_poll_module.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_openssl.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_openssl_cache.c
        ${NGINX_SOURCE_DIR}/src/event/ngx_event_openssl_stapling.c
        ${NGINX_SOURCE_DIR}/src/core/ngx_regex.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_core_module.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_special_response.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_request.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_parse.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_log_module.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_request_body.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_variables.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_script.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_upstream.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_upstream_round_robin.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_file_cache.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_huff_decode.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_huff_encode.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_write_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_header_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_chunked_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/v2/ngx_http_v2_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_range_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_gzip_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_postpone_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_ssi_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_charset_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_sub_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_addition_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_gunzip_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_userid_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_headers_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/ngx_http_copy_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_not_modified_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_slice_filter_module.c
        ${NGINX_SOURCE_DIR}/src/http/v2/ngx_http_v2.c
        ${NGINX_SOURCE_DIR}/src/http/v2/ngx_http_v2_table.c
        ${NGINX_SOURCE_DIR}/src/http/v2/ngx_http_v2_encode.c
        ${NGINX_SOURCE_DIR}/src/http/v2/ngx_http_v2_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_static_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_gzip_static_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_dav_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_autoindex_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_index_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_random_index_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_mirror_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_try_files_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_auth_request_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_auth_basic_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_access_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_limit_conn_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_limit_req_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_realip_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_geo_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_map_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_split_clients_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_referer_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_rewrite_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_ssl_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_proxy_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_fastcgi_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_uwsgi_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_scgi_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_grpc_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_memcached_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_empty_gif_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_browser_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_secure_link_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_flv_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_mp4_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_hash_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_ip_hash_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_least_conn_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_random_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_keepalive_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_upstream_zone_module.c
        ${NGINX_SOURCE_DIR}/src/http/modules/ngx_http_stub_status_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_core_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_handler.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_parse.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_ssl_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_pop3_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_pop3_handler.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_imap_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_imap_handler.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_smtp_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_smtp_handler.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_auth_http_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_proxy_module.c
        ${NGINX_SOURCE_DIR}/src/mail/ngx_mail_realip_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_variables.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_script.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_handler.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_core_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_log_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_proxy_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream_round_robin.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_write_filter_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_ssl_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_limit_conn_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_access_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_geo_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_map_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_split_clients_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_return_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_pass_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_set_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream_hash_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream_least_conn_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream_random_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_upstream_zone_module.c
        ${NGINX_SOURCE_DIR}/src/stream/ngx_stream_ssl_preread_module.c
        ngx_modules.c
)

# 添加可执行文件 
add_executable(nginx
        ${nginx_HEADERS}
        ${nginx_SOURCES}
        ${NGINX_SOURCE_DIR}/conf/nginx.conf
)

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
        FILES ${nginx_HEADERS} ${nginx_SOURCES}
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

# 创建基本的HTML文件
if(NOT EXISTS "${NGINX_WORK_DIR}/html/index.html")
    file(WRITE "${NGINX_WORK_DIR}/html/index.html"
        "<html><body><h1>Welcome to nginx!</h1></body></html>")
endif()

if(NOT EXISTS "${NGINX_WORK_DIR}/html/50x.html")
    file(WRITE "${NGINX_WORK_DIR}/html/50x.html"
        "<html><body><h1>An error occurred.</h1></body></html>")
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
        /D_CRT_SECURE_NO_WARNINGS
    )
endif()
