user www;
worker_processes 2;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 4096;
}

http {
    gzip on;
    gzip_disable "msie6";
    gzip_comp_level 6;
    gzip_min_length 1000000;
    gzip_buffers 16 8k;
    gzip_proxied any;
    gzip_types application/javascript;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    sendfile on;
    send_timeout 10m;
    access_log /var/log/nginx/access.log;
    keepalive_timeout 5;
    client_body_timeout 120;

    server {
       listen 10080;
       root /www;
       index index.html index.htm;
       server_name localhost;
       client_max_body_size 600M;
       error_page 500 502 503 504 /50x.html;
       location / {
        if ($request_method = 'OPTIONS') {
          add_header 'Access-Control-Allow-Origin' '*';
          add_header 'Access-Control-Allow-Headers' '*';
          add_header 'Access-Control-Allow-Methods' 'POST, OPTIONS';
          add_header 'Access-Control-Max-Age' 1728000;
          add_header 'Content-Type' 'text/plain; charset=utf-8';
          return 204;
        }
        if ($request_method = 'POST') {
          add_header 'Access-Control-Allow-Origin' '*';
          add_header 'Access-Control-Allow-Methods' 'POST';
        }
         rewrite ^([^.]*[^/])$ $1/ last;
         try_files $uri $uri/ /index.html;
       }
       location = /50x.html {
             root /var/lib/nginx/html;
       }
       location /common_domain/common {
         rewrite /common_domain/(.*) /$1 break;
         proxy_pass http://oc-common-service.ns-ccb.svc.cluster.local:28080/common;
         proxy_redirect off;
         proxy_set_header Host $host;
         proxy_pass_request_headers on;
         client_max_body_size 600M;
         proxy_buffer_size 128k;
         proxy_buffers 4 256k;
         proxy_busy_buffers_size 256k;
       }
       location /common_domain/cls {
         rewrite /common_domain/(.*) /$1 break;
         proxy_pass http://oc-cls-service.ns-ccb.svc.cluster.local:38080/cls;
         proxy_redirect off;
         proxy_set_header Host $host;
         proxy_pass_request_headers on;
         client_max_body_size 600M;
         proxy_buffer_size 128k;
         proxy_buffers 4 256k;
         proxy_busy_buffers_size 256k;
       }
       location /common_domain/lecture {
         rewrite /common_domain/(.*) /$1 break;
         proxy_pass http://oc-lecture-service.ns-ccb.svc.cluster.local:48080/lecture;
         proxy_redirect off;
         proxy_set_header Host $host;
         proxy_pass_request_headers on;
         client_max_body_size 600M;
         proxy_buffer_size 128k;
         proxy_buffers 4 256k;
         proxy_busy_buffers_size 256k;
       }
       location /common_domain/ext {
         rewrite /common_domain/(.*) /$1 break;
         proxy_pass http://oc-ext-service.ns-ccb.svc.cluster.local:48081/ext;
         proxy_redirect off;
         proxy_set_header Host $host;
         proxy_pass_request_headers on;
         client_max_body_size 600M;
         proxy_buffer_size 128k;
         proxy_buffers 4 256k;
         proxy_busy_buffers_size 256k;
       }
       location /auth_domain {
         rewrite /auth_domain/(.*) /$1 break;
         proxy_pass http://oc-auth-service.ns-auth.svc.cluster.local:18080;
         proxy_redirect off;
         proxy_set_header Host $host;
       }
    }
}
