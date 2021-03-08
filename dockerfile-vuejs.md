FROM alpine:3.8
MAINTAINER 'heavyflood@gsitm.com'
RUN apk update
RUN apk add --no-cache nginx
RUN adduser -D -g 'www' www
RUN mkdir /www
RUN chown -R www:www /var/lib/nginx
RUN chown -R www:www /www

COPY nginx.conf /etc/nginx/nginx.conf
COPY . /www/

ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
