# dockerfile 常用镜像编写

## 前后端分离的dockerfile

### dockerfile文件
````bash
FROM g127/ps:20210520-buster

#
COPY target/topflames-oa-1.0-SNAPSHOT/  /DevProjectFiles/ws-root/webapps/lkyoa-service/
#
COPY www.app.com/  /DevProjectFiles/ws-root/www/www.app.com/

COPY frontend.ng.template /etc/nginx/conf.d/default.conf

RUN echo "init" \
    && DEBIAN_FRONTEND=noninteractive apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        curl \
        inetutils-ping \
        netcat \
        vim \
    && DEBIAN_FRONTEND=noninteractive apt-get autoremove -y \
    && rm -rf /var/lib/apt/lists/* 


# app args
ENV DIST_JAR=''  \
    SWITCH_RUN=ps \
    NGX_CONFIGURE_PATH=/opt/nginx/conf   \
    \
    CATALINA_REDIS_HOSTS=redis3:6379 \
    CATALINA_APPBASE=/DevProjectFiles/ws-root/webapps \
    \
    DEVOPS_DB_HOST='mysql:3306' \
    DEVOPS_DB_USERNAME=root \
    DEVOPS_DB_PASSWORD=root \
    \
    DEVOPS_REDIS_HOST=redis3 \
    DEVOPS_REDIS_PORT=6379 \
    DEVOPS_REDIS_PWD=  \
    \
    DEVOPS_SERVER_PORT=8080 \
    DEVOPS_CONTEXT_PATH=/ \
    \
    APP_ARGS='--spring.profiles.active=prod' \
    JAVA_OPTS=' ' \
    JVM_PERFORMANCE_OPTS=-server 
````

### nginx配置文件
````bash

server {
    listen 80;
    server_name www.app.com;

    # defined vars
    set    $www_domain       www.app.com;
    set    $www_root          /DevProjectFiles/ws-root/www/${www_domain};

    # defined vhost-common
    include vhosts/vhost-common.conf;

    # defined /lkyoa-service/
    location /lkyoa-service/ {
        include  vhosts/proxy.set.header.common.conf;
        proxy_pass               http://127.0.0.1:8080/lkyoa-service/;
        proxy_cookie_path        /lkyoa-service   /;
        proxy_cookie_path        /lkyoa-service/  /;
        index                    index.html index.htm;
        # 偶尔用到重定向
        # rewrite                "^/prod-api/(.*)$" /$1 break;
    }

    # Legacy
    # defined service resource caches
    location ~* ^/lkyoa-service/(^storage/rest)/(.*)\.(cab|css|js|ico|gif|bmp|jpg|jpeg|png|swf|mp3)$ {
        # default_type           "text/plain";
        # echo The current request uri is $request_uri;
        #
        proxy_cache_valid        200 304 301 302 10d;
        proxy_cache_valid        any 1m;
        proxy_cache_key          $host$uri$is_args$args;
        proxy_cache              cache_one;
        proxy_http_version       1.1;
        proxy_set_header Connection "";
        proxy_pass              http://127.0.0.1:8080/$request_uri;
    }

    # default news path
    location /lkyoa-service/noticeHtml/ {
        sub_filter               /topflames-oa/   /lkyoa-service/;
        sub_filter_once          off;
        sub_filter_types         *;
        alias /DevProjectFiles/ws-root/fileResources/notice/;
    }

    # default news path
    location /notice/ {
        sub_filter               /topflames-oa/   /lkyoa-service/;
        sub_filter_once          off;
        sub_filter_types         *;
        alias /DevProjectFiles/ws-root/fileResources/notice/;
    }

    location /cmsHtml/ {
        sub_filter               /topflames-oa/   /lkyoa-service/;
        sub_filter_once          off;
        # sub_filter_types       text/html;
        alias /DevProjectFiles/ws-root/fileResources/cms/www.app.com/;
    }
}
````