# 在文件开头添加必要的包含
include(CheckIncludeFile)
include(CheckSymbolExists)
include(CheckTypeSize)

# 添加包含路径
include_directories(
    ${CMAKE_SOURCE_DIR}  # 用于包含根目录下的 ngx_auto_headers.h 和 ngx_auto_config.h
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/core
    ${CMAKE_CURRENT_SOURCE_DIR}/event
    ${CMAKE_CURRENT_SOURCE_DIR}/event/modules
    ${CMAKE_CURRENT_SOURCE_DIR}/http
    ${CMAKE_CURRENT_SOURCE_DIR}/http/modules
    ${CMAKE_CURRENT_SOURCE_DIR}/http/v2
    ${CMAKE_CURRENT_SOURCE_DIR}/mail
    ${CMAKE_CURRENT_SOURCE_DIR}/stream
    ${PLATFORM_INCLUDE_DIRS}
)

# 收集头文件 - 只包含Windows相关头文件
if(WIN32)
    file(GLOB_RECURSE nginx_HEADERS 
        "core/*.h"
        "event/*.h"
        "http/*.h"
        "mail/*.h"
        "misc/*.h"
        "os/win32/*.h"  # 只包含Windows相关头文件
        "stream/*.h"
    )
else()
    file(GLOB_RECURSE nginx_HEADERS 
        "core/*.h"
        "event/*.h"
        "http/*.h"
        "mail/*.h"
        "misc/*.h"
        "os/unix/*.h"
        "stream/*.h"
    )
endif()

# 定义源文件列表 - Windows平台
if(WIN32)
    set(nginx_SOURCES
        # 添加已有的模块定义文件
        ${CMAKE_SOURCE_DIR}/ngx_modules.c

        # core模块
        core/nginx.c 
        core/ngx_log.c 
        core/ngx_palloc.c 
        core/ngx_array.c 
        core/ngx_list.c 
        core/ngx_hash.c 
        core/ngx_buf.c 
        core/ngx_queue.c 
        core/ngx_output_chain.c 
        core/ngx_string.c 
        core/ngx_parse.c 
        core/ngx_parse_time.c 
        core/ngx_inet.c 
        core/ngx_file.c 
        core/ngx_crc32.c 
        core/ngx_murmurhash.c 
        core/ngx_md5.c 
        core/ngx_sha1.c 
        core/ngx_rbtree.c 
        core/ngx_radix_tree.c 
        core/ngx_slab.c 
        core/ngx_times.c 
        core/ngx_shmtx.c 
        core/ngx_connection.c 
        core/ngx_cycle.c 
        core/ngx_spinlock.c 
        core/ngx_rwlock.c 
        core/ngx_cpuinfo.c 
        core/ngx_conf_file.c 
        core/ngx_module.c 
        core/ngx_resolver.c 
        core/ngx_open_file_cache.c 
        core/ngx_crypt.c 
        core/ngx_proxy_protocol.c 
        core/ngx_syslog.c
        core/ngx_thread_pool.c

        # event模块
        event/ngx_event.c 
        event/ngx_event_timer.c 
        event/ngx_event_posted.c 
        event/ngx_event_accept.c 
        event/ngx_event_udp.c 
        event/ngx_event_connect.c 
        event/ngx_event_pipe.c

        # Windows 平台特定源文件
        os/win32/ngx_errno.c 
        os/win32/ngx_alloc.c 
        os/win32/ngx_files.c 
        os/win32/ngx_shmem.c 
        os/win32/ngx_time.c 
        os/win32/ngx_process.c 
        os/win32/ngx_thread.c 
        os/win32/ngx_socket.c 
        os/win32/ngx_wsarecv.c 
        os/win32/ngx_wsarecv_chain.c 
        os/win32/ngx_udp_wsarecv.c 
        os/win32/ngx_wsasend.c 
        os/win32/ngx_wsasend_chain.c 
        os/win32/ngx_win32_init.c 
        os/win32/ngx_user.c 
        os/win32/ngx_dlopen.c 
        os/win32/ngx_event_log.c 
        os/win32/ngx_process_cycle.c
        event/ngx_event_acceptex.c 
        event/modules/ngx_iocp_module.c 
        event/modules/ngx_win32_select_module.c 
        event/modules/ngx_win32_poll_module.c

        # 通用模块
        event/ngx_event_openssl.c 
        event/ngx_event_openssl_stapling.c 
        core/ngx_regex.c 

        # HTTP模块
        http/ngx_http.c 
        http/ngx_http_core_module.c 
        http/ngx_http_special_response.c 
        http/ngx_http_request.c 
        http/ngx_http_parse.c 
        http/modules/ngx_http_log_module.c 
        http/ngx_http_request_body.c 
        http/ngx_http_variables.c 
        http/ngx_http_script.c 
        http/ngx_http_upstream.c 
        http/ngx_http_upstream_round_robin.c 
        http/ngx_http_file_cache.c 
        http/ngx_http_write_filter_module.c 
        http/ngx_http_header_filter_module.c 
        http/modules/ngx_http_chunked_filter_module.c 
        http/modules/ngx_http_range_filter_module.c 
        http/modules/ngx_http_gzip_filter_module.c 
        http/ngx_http_postpone_filter_module.c 
        http/modules/ngx_http_ssi_filter_module.c 
        http/modules/ngx_http_charset_filter_module.c 
        http/modules/ngx_http_sub_filter_module.c 
        http/modules/ngx_http_addition_filter_module.c 
        http/modules/ngx_http_gunzip_filter_module.c 
        http/modules/ngx_http_userid_filter_module.c 
        http/modules/ngx_http_headers_filter_module.c 
        http/ngx_http_copy_filter_module.c 
        http/modules/ngx_http_not_modified_filter_module.c 
        http/modules/ngx_http_slice_filter_module.c 
        http/v2/ngx_http_v2.c 
        http/v2/ngx_http_v2_table.c 
        http/v2/ngx_http_v2_encode.c 
        http/v2/ngx_http_v2_filter_module.c 
        http/v2/ngx_http_v2_module.c 
        http/modules/ngx_http_static_module.c 
        http/modules/ngx_http_gzip_static_module.c 
        http/modules/ngx_http_dav_module.c 
        http/modules/ngx_http_autoindex_module.c 
        http/modules/ngx_http_index_module.c 
        http/modules/ngx_http_random_index_module.c 
        http/modules/ngx_http_mirror_module.c 
        http/modules/ngx_http_try_files_module.c 
        http/modules/ngx_http_auth_request_module.c 
        http/modules/ngx_http_auth_basic_module.c 
        http/modules/ngx_http_access_module.c 
        http/modules/ngx_http_limit_conn_module.c 
        http/modules/ngx_http_limit_req_module.c 
        http/modules/ngx_http_realip_module.c 
        http/modules/ngx_http_geo_module.c 
        http/modules/ngx_http_map_module.c 
        http/modules/ngx_http_split_clients_module.c 
        http/modules/ngx_http_referer_module.c 
        http/modules/ngx_http_rewrite_module.c 
        http/modules/ngx_http_ssl_module.c 
        http/modules/ngx_http_proxy_module.c 
        http/modules/ngx_http_fastcgi_module.c 
        http/modules/ngx_http_uwsgi_module.c 
        http/modules/ngx_http_scgi_module.c 
        http/modules/ngx_http_grpc_module.c 
        http/modules/ngx_http_memcached_module.c 
        http/modules/ngx_http_empty_gif_module.c 
        http/modules/ngx_http_browser_module.c 
        http/modules/ngx_http_secure_link_module.c 
        http/modules/ngx_http_flv_module.c 
        http/modules/ngx_http_mp4_module.c 
        http/modules/ngx_http_upstream_hash_module.c 
        http/modules/ngx_http_upstream_ip_hash_module.c 
        http/modules/ngx_http_upstream_least_conn_module.c 
        http/modules/ngx_http_upstream_random_module.c 
        http/modules/ngx_http_upstream_keepalive_module.c 
        http/modules/ngx_http_upstream_zone_module.c 
        http/modules/ngx_http_stub_status_module.c 

        # MAIL模块
        mail/ngx_mail.c 
        mail/ngx_mail_core_module.c 
        mail/ngx_mail_handler.c 
        mail/ngx_mail_parse.c 
        mail/ngx_mail_ssl_module.c 
        mail/ngx_mail_pop3_module.c 
        mail/ngx_mail_pop3_handler.c 
        mail/ngx_mail_imap_module.c 
        mail/ngx_mail_imap_handler.c 
        mail/ngx_mail_smtp_module.c 
        mail/ngx_mail_smtp_handler.c 
        mail/ngx_mail_auth_http_module.c 
        mail/ngx_mail_proxy_module.c 
        mail/ngx_mail_realip_module.c 

        # STREAM模块
        stream/ngx_stream.c 
        stream/ngx_stream_variables.c 
        stream/ngx_stream_script.c 
        stream/ngx_stream_handler.c 
        stream/ngx_stream_core_module.c 
        stream/ngx_stream_log_module.c 
        stream/ngx_stream_proxy_module.c 
        stream/ngx_stream_upstream.c 
        stream/ngx_stream_upstream_round_robin.c 
        stream/ngx_stream_write_filter_module.c 
        stream/ngx_stream_ssl_module.c 
        stream/ngx_stream_limit_conn_module.c 
        stream/ngx_stream_access_module.c 
        stream/ngx_stream_geo_module.c 
        stream/ngx_stream_map_module.c 
        stream/ngx_stream_split_clients_module.c 
        stream/ngx_stream_return_module.c
        stream/ngx_stream_upstream_hash_module.c 
        stream/ngx_stream_upstream_least_conn_module.c 
        stream/ngx_stream_upstream_random_module.c 
        stream/ngx_stream_upstream_zone_module.c 
        stream/ngx_stream_ssl_preread_module.c
        stream/ngx_stream_set_module.c
        stream/ngx_stream_pass_module.c
    )
else()
    # Unix平台源文件列表...
endif()

# 添加可执行文件
add_executable(nginx 
    ${nginx_HEADERS} 
    ${nginx_SOURCES}
)

# 链接所需库
target_link_libraries(nginx
    PRIVATE
    $<$<PLATFORM_ID:Windows>:
        kernel32
        user32
        advapi32
        ws2_32
        gdi32
        crypt32
    >
    $<$<NOT:$<PLATFORM_ID:Windows>>:
        pthread
        dl
    >
)

# 配置PCRE、ZLIB和OpenSSL支持 - 使用third_party中的库
# PCRE配置
if(NGX_HAVE_PCRE)
    target_include_directories(nginx PRIVATE ${PCRE_INCLUDE_DIRS})
    target_link_libraries(nginx PRIVATE ${PCRE_LIBRARIES})
    target_compile_definitions(nginx PRIVATE NGX_HAVE_PCRE=1)
endif()

# ZLIB配置
if(NGX_HAVE_ZLIB)
    target_include_directories(nginx PRIVATE ${ZLIB_INCLUDE_DIRS})
    target_link_libraries(nginx PRIVATE ${ZLIB_LIBRARIES})
    target_compile_definitions(nginx PRIVATE NGX_HAVE_ZLIB=1)
endif()

# OpenSSL配置
if(NGX_HAVE_OPENSSL)
    target_include_directories(nginx PRIVATE ${OPENSSL_INCLUDE_DIRS})
    target_link_libraries(nginx PRIVATE ${OPENSSL_LIBRARIES})
    target_compile_definitions(nginx PRIVATE NGX_HAVE_OPENSSL=1)
endif()

# 自动分组源文件到VS项目中
AUTO_GROUP_FILES(
    TARGET nginx 
    BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    FILES ${nginx_HEADERS} ${nginx_SOURCES}
)

# 设置生成位置
set_target_properties(nginx PROPERTIES 
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
    RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_SOURCE_DIR}
    RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_SOURCE_DIR}
    RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${CMAKE_SOURCE_DIR}
)

# 平台特定设置
if(WIN32)
    # Windows特定编译定义
    target_compile_definitions(nginx PRIVATE 
        WIN32
        _WIN32
        NGINX_BINARY
        NDEBUG
        NGX_HAVE_MAP_DEVZERO=0
    )
else()
    # Unix/Linux特定编译定义
    target_compile_definitions(nginx PRIVATE
        _GNU_SOURCE
        _FILE_OFFSET_BITS=64
        NGINX_BINARY
        NDEBUG
    )
endif() 