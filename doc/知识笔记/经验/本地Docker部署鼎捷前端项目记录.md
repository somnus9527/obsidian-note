---
tags:
  - 知识笔记/经验
---
#### 文件夹结构
```text
local_test_athena 根目录
├── athena 各个项目打包之后的静态资源目录
│   ├── main 主应用资源
│   ├── scheduler 共享包资源
│   └── subApps 子应用资源
├── docker-compose.yaml docker-compose配置
└── dockerfile 脚本，dockerfiler配置，nginx配置，shell脚本，密匙文件
    ├── Dockerfile 
    ├── default.conf nginx server配置
    ├── dockerEnv.sh 替换各文件中占位符为环境变量的shell脚本
    ├── frontendShell.sh Docker启动入口
    ├── nginx.conf nginx配置，内部引用了default.conf
    ├── somnuszyy9527.local-key.pem 密匙文件
    └── somnuszyy9527.local.pem
```

#### 核心文件记录
> [!Note] docker-compose.yaml
> ```yaml
> version: '3.8'
> services:
>   web:
>     image: athena-web  #镜像名称
>     build:
>       context: .  #执行目录（当前目录）
>       dockerfile: ./dockerfile/Dockerfile #指定dockerfile
>     ports:
>       - "443:443" #容器端口443映射到宿主机端口443
>     environment:  #环境变量配置，只保留了nginx相关配置，这些还有些参考价值
>       - NGINX_CSP_DEFAULT_SRC=https://somnuszyy9527.local https://*.digiwincloud.com.cn https://*.digiwincloud.com wss://*.digiwincloud.com wss://*.digiwincloud.com.cn wss://*.digiwincloud.com.cn:8084 https://*.alicdn.com https://uipaas-assets.com https://cdn.jsdelivr.net
>       - NGINX_CSP_FRAME_SRC=https://somnuszyy9527.local https://*.digiwincloud.com https://*.digiwincloud.com.cn https://*.digiwincloud.com.cn:*
>       - NGINX_CSP_FRAME_ANCESTORS=https://somnuszyy9527.local https://*.digiwincloud.com.cn https://*.digiwincloud.com
>       - NGINX_REFERRER_POLICY=strict-origin-when-cross-origin
>       - NGINX_PERMISSIONS_POLICY=microphone=(self)
>       - NGINX_XFRAME_OPTIONS_CONDITION=~* (^https://(.+)\\.digiwincloud\\.com|^https://(.+)\\.local)
>       - NGINX_ACCESS_CONTROL_ALLOW_ORIGIN_CONDITION=~* (^https://(.+)\\.digiwincloud\\.com|^http://localhost|^https://(.+)\\.alicdn\\.com|^https://(.+)\\.local)
> ```

> [!Note] dockerfile/Dockerfile
> ```Dockerfile
> FROM nginx:1.12.1
> COPY ./athena /usr/share/nginx/html/athena/  #复制当前目录下的athena文件夹到容器指定文件夹
> RUN cd /usr/share/nginx/html/athena/main  && mv -f * ../../  #将 /usr/share/nginx/html/athena/main 目录下的所有文件和文件夹移动到上两级目录，即 /usr/share/nginx/html/ 下，并强制覆盖同名文件。
> RUN cd /usr/share/nginx/html/athena/subApps  && mv -f * ../../
> ADD ./dockerfile/nginx.conf /etc/nginx/ #把./dockerfile/nginx.conf复制到容器内的/etc/nginx/文件夹下
> ADD ./dockerfile/default.conf /etc/nginx/conf.d/
> ADD ./dockerfile/frontendShell.sh /usr/share/nginx/html/
> ADD ./dockerfile/somnuszyy9527.local.pem /etc/nginx/certs/
> ADD ./dockerfile/somnuszyy9527.local-key.pem /etc/nginx/certs/
> RUN chmod +x /usr/share/nginx/html/frontendShell.sh #给frontendShell.sh添加执行权限
> ADD ./dockerfile/dockerEnv.sh /usr/share/nginx/html/
> RUN chmod +x /usr/share/nginx/html/dockerEnv.sh
> WORKDIR /usr/share/nginx/html #设置工作目录
> EXPOSE 80 #设置容器监听的端口
> ENTRYPOINT ["/usr/share/nginx/html/frontendShell.sh"] #设置容器启动时默认执行的脚本
> ```

> [!Note] dockerfile/nginx.conf
> ```nginx
> user  nginx;
> worker_processes  1;
> 
> error_log  /var/log/nginx/error.log warn;
> pid        /var/run/nginx.pid;
> 
> 
> events {
>     worker_connections  1024;
> }
> 
> 
> http {
>     include       /etc/nginx/mime.types;
>     default_type  application/octet-stream;
> 
>     log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>                       '$status $body_bytes_sent "$http_referer" '
>                       '"$http_user_agent" "$http_x_forwarded_for"';
> 
>     access_log  /var/log/nginx/access.log  main;
> 
>     sendfile        on;
>     #tcp_nopush     on;
> 
>     keepalive_timeout  65;
> 
>     gzip_static on;
>     gzip_http_version 1.1;
>     gzip_vary on;
>     gzip_types application/javascript text/css;
>         
>     include /etc/nginx/conf.d/*.conf;
> 
>     server {
>         listen      80;
>         server_name somnuszyy9527.local; # 设置域名
>         # 将 HTTP 请求重定向到 HTTPS
>         return 301 https://$host$request_uri;
>     }
> }
> 
> ```

> [!Note] default.conf
> ```nginx
> server {
>     listen 0.0.0.0:443 ssl; #监听本地的443
>     server_name somnuszyy9527.local; #设置服务名
> 
>     ssl_certificate /etc/nginx/certs/somnuszyy9527.local.pem; #配置密匙
>     ssl_certificate_key /etc/nginx/certs/somnuszyy9527.local-key.pem;
>     # Security Headers
>     # 參數範例 "myapp1.digiwincloud.com myapp2.digiwincloud.com" 或 "*.digiwincloud.com.cn"
>     set $csp_uri "@NGINX_CSP_DEFAULT_SRC@";
>     # 設定嵌套 iframe 的來源，要使用哪些網址當 iframe 內容
>     set $frameSrc "@NGINX_CSP_FRAME_SRC@";
>     # 允許可以用 iframe 嵌入頁面的父源，是允許被哪些網址把本站嵌入 iframe
>     set $frameAncestors "@NGINX_CSP_FRAME_ANCESTORS@";
>     # 參數範例 "strict-origin-when-cross-origin"
>     set $referrerPolicy "@NGINX_REFERRER_POLICY@";
>     # Permissions-Policy 參數範例 "microphone=(self)"
>     set $permissionsPolicy "@NGINX_PERMISSIONS_POLICY@";
> 
>     # SAMEORIGIN 或 ALLOW-FROM 一個網址
>     set $xFrameOptions "SAMEORIGIN";
>     if ($http_referer @NGINX_XFRAME_OPTIONS_CONDITION@) {
>       set $xFrameOptions "ALLOW-FROM $http_referer";
>     }
> 
>     set $cors_origin "";
>     # 條件參數範例 "~* (^https://(.*)\\.digiwincloud\\.com\\.cn)"
>     if ($http_origin @NGINX_ACCESS_CONTROL_ALLOW_ORIGIN_CONDITION@) {
>        set $cors_origin $http_origin;
>     }
> 
>     set $CspDefaultSrc "default-src 'self' $csp_uri;";
>     set $CspScriptSrc "script-src 'self' $csp_uri 'unsafe-inline' 'unsafe-eval';";
>     set $CspImgSrc "img-src 'self' $csp_uri data:;";
>     set $CspStyleSrc "style-src 'self' $csp_uri 'unsafe-inline';";
>     set $CspFontSrc "font-src 'self' $csp_uri data:;";
>     set $CspFrameSrc "frame-src 'self' $frameSrc;";
>     set $CspFrameAncestors "frame-ancestors 'self' $frameAncestors;";
>     set $CspConnectSrc "connect-src 'self' $csp_uri;";
>     set $CspWorkerSrc "worker-src 'self' $csp_uri blob:;";
> 
>     location ~ ^/athena-designer-core/ {
>         add_header Content-Security-Policy "$CspDefaultSrc $CspWorkerSrc $CspConnectSrc $CspScriptSrc $CspImgSrc $CspStyleSrc $CspFontSrc $CspFrameSrc $CspFrameAncestors" always;
>         add_header X-Frame-Options "$xFrameOptions";
>         add_header Referrer-Policy "$referrerPolicy";
>         add_header X-Content-Type-Options "nosniff" always;
>         add_header Permissions-Policy "$permissionsPolicy";
>         add_header Access-Control-Allow-Origin "$cors_origin" always;
>         add_header Cache-Control "no-store";
>         root /usr/share/nginx/html;
>         try_files $uri $uri/ =404;
>     }
> 
>     location / {
>         expires -1;
>         add_header Content-Security-Policy "$CspDefaultSrc $CspWorkerSrc $CspConnectSrc $CspScriptSrc $CspImgSrc $CspStyleSrc $CspFontSrc $CspFrameSrc $CspFrameAncestors" always;
>         add_header X-Frame-Options "$xFrameOptions";
>         add_header Referrer-Policy "$referrerPolicy";
>         add_header X-Content-Type-Options "nosniff" always;
>         add_header Permissions-Policy "$permissionsPolicy";
>         add_header Access-Control-Allow-Origin "$cors_origin" always;
> 	    root  /usr/share/nginx/html;
>         index  index.html index.htm;
>         if ($request_filename ~ .*\.(html|json)$){
>             add_header Cache-Control no-store;
>         }
>         try_files $uri $uri/ @rewrites;
>     }
> 
>     location @rewrites {
>         rewrite ^(?!.*\.js$).*$  /index.html last;
>     }
> 
>     location = /index.html {
>         add_header Content-Security-Policy "$CspDefaultSrc $CspWorkerSrc $CspConnectSrc $CspScriptSrc $CspImgSrc $CspStyleSrc $CspFontSrc $CspFrameSrc $CspFrameAncestors" always;
>         add_header X-Frame-Options "$xFrameOptions";
>         add_header Referrer-Policy "$referrerPolicy";
>         add_header X-Content-Type-Options "nosniff" always;
>         add_header Permissions-Policy "$permissionsPolicy";
>         add_header Access-Control-Allow-Origin "$cors_origin" always;
>         #入口文件设置不缓存
>         add_header Cache-Control no-store;
>         root /usr/share/nginx/html;
>         index index.html index.htm;
>     }
> 
>     location ~ \.json$ {
>         add_header Content-Security-Policy "$CspDefaultSrc $CspWorkerSrc $CspConnectSrc $CspScriptSrc $CspImgSrc $CspStyleSrc $CspFontSrc $CspFrameSrc $CspFrameAncestors" always;
>         add_header X-Frame-Options "$xFrameOptions";
>         add_header Referrer-Policy "$referrerPolicy";
>         add_header X-Content-Type-Options "nosniff" always;
>         add_header Permissions-Policy "$permissionsPolicy";
>         add_header Access-Control-Allow-Origin "$cors_origin" always;
> 
>         add_header Cache-Control "no-store";
>         root /usr/share/nginx/html;  # 指向 JSON 文件存放目录
>         try_files $uri =404;
>     }
> 
>     #error_page  404              /404.html;
> 
>     # redirect server error pages to the static page /50x.html
>     #
>     error_page   500 502 503 504  /50x.html;
>     location = /50x.html {
>         root   /usr/share/nginx/html;
>     }
> 
>     # proxy the PHP scripts to Apache listening on 127.0.0.1:80
>     #
>     #location ~ \.php$ {
>     #    proxy_pass   http://127.0.0.1;
>     #}
> 
>     # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
>     #
>     #location ~ \.php$ {
>     #    root           html;
>     #    fastcgi_pass   127.0.0.1:9000;
>     #    fastcgi_index  index.php;
>     #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
>     #    include        fastcgi_params;
>     #}
> 
>     # deny access to .htaccess files, if Apache's document root
>     # concurs with nginx's one
>     #
>     #location ~ /\.ht {
>     #    deny  all;
>     #}
> }
> 
> 
> ```

> [!Note] frontendShell.sh
> ```sh
> #!/bin/sh
> #执行相应文件的环境变量替换操作
> bash dockerEnv.sh /usr/share/nginx/html/assets json api.dev.json
> bash dockerEnv.sh /etc/nginx/conf.d conf
> bash dockerEnv.sh /usr/share/nginx/html/athena-designer-editor-components/build/lowcode json
> nginx -g "daemon off;"
> ```

> [!Note] dockerEnv.sh
> ```sh
> #!/bin/sh
> 
> function log_info()
> {
>   local date=`date`
>   local cmd="$1"
>   echo "$date $cmd"
>   eval "$cmd"
>   echo "log info:$date $cmd" &>> "$SYS_LOG"
> }
> SYS_LOG=dockerEnv.log
> 
> #程式路徑
> Path=$1
> #檔案類型
> TypeName=$2
> #排除檔案
> removeName=$3
> 
> #修改方法一 maxdepth限制只找一层文件夹
> find $Path/ -maxdepth 1 -name "*.$TypeName" -a ! -name "$removeName"  | xargs  grep -rH '@' > envSpace.txt
> 
> sed 's/[[:space:]]//g' envSpace.txt > env.txt
> envDate=$(cat env.txt)
> 
> #迴圈解析@
> for date in ${envDate}; do
>  field=2
>  env=test
>  filePath=$(cut -d':' -f1 <<< "$date")
>  while [[ "$env" != ""  ]]; do
>   env=$(cut -d'@' -f$field <<< "$date")
>   let "field+=2"
>   if [ "$env" != "" ]; then
>    # 需要取代的環境變數名稱
>    envReplace=${env//./_}
>    # 自動加解析時所需的Escape Character，避免部署參數JSON設定過多'\'
>    envReplaceValue=$(eval echo "\$$envReplace" | sed 's/\\/\\\\/g; s/#/\\#/g;')
>    log_info "sed -i 's#@${env}@#${envReplaceValue}#g' \"$filePath\""
>   fi
>  done
> done
> 
> rm -f envSpace.txt
> rm -f env.txt
> ```

